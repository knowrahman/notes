# Module 7 — Data Sources

## Where Notely is right now

```text
  Notely so far:

    VPC + 6 subnets across 2 AZs
    Internet gateway, route tables, S3 gateway endpoint
    3 security groups (alb -> app -> db)
    2 EC2 servers running the Node.js app
    Application Load Balancer
    S3 attachments bucket, CloudWatch log group
    IAM role and instance profile

  Still hardcoded:
    var.app_ami_id = "ami-0892a9c01908fafd1"   <- out of date within weeks
    local.azs = ["${var.region}a", "${var.region}b"]   <- guessing AZ names
```

By the end of this module both are looked up automatically.

Full picture: `notely-architecture.md`.

---

## Why this module exists

Notely has two lies in it.

**Lie one: the AMI ID.**

```hcl
default = "ami-0892a9c01908fafd1"
```

An AMI ID is region-specific and version-specific. That value:

- Is wrong in every region except `ap-southeast-2`
- Becomes outdated the moment Amazon publishes a new Amazon Linux image, which
  is roughly monthly
- Means your servers are launched from an image missing weeks of security
  patches
- Will eventually be deregistered, and your apply will fail

**Lie two: the availability zone names.**

```hcl
azs = ["${var.region}a", "${var.region}b"]
```

This assumes AZ names are always the region plus a letter. Usually true —
`ap-southeast-2a`. But AWS does not guarantee every account has access to every
AZ, and some accounts genuinely cannot use `us-east-1a`. Guessing is fragile.

Data sources fix both. They let Terraform **ask AWS** rather than be told.

---

## The core idea (one sentence)

A `data` block reads something that already exists, so you can reference it
without Terraform managing it.

> `resource` says "make this exist". `data` says "tell me about this".

---

## Mental model (from Node.js)

```javascript
// resource - you create it and you own it
const server = await ec2.createInstance({ ... });

// data - you look it up, someone else owns it
const image = await ec2.describeImages({ owners: ['amazon'] });
```

The distinction that matters:

| | `resource` | `data` |
|---|---|---|
| Terraform creates it | Yes | No |
| Terraform can change it | Yes | No |
| Terraform can destroy it | Yes | **No** |
| Appears in state | Yes, as `managed` | Yes, as `data` (cached only) |
| Plan symbol | `+`, `~`, `-` | `<=` |
| `terraform destroy` touches it | Yes | No |

A data source is read-only. You cannot break anything with one. That makes them
very safe to experiment with.

---

## Part 1 — The syntax

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

Reading it:

| Part | Meaning |
|---|---|
| `data` | This is a lookup, not a creation |
| `aws_ami` | The data source type |
| `amazon_linux` | Your local name |
| the body | Arguments narrowing the search |

And referencing it — note the `data.` prefix:

```hcl
resource "aws_instance" "app" {
  ami = data.aws_ami.amazon_linux.id
}
```

```text
data.<type>.<name>.<attribute>
^^^^
this prefix is required, and is the thing people forget
```

---

## Part 2 — The data sources you will use constantly

### `aws_caller_identity` — who am I?

```hcl
data "aws_caller_identity" "current" {}
```

No arguments. Gives you:

```hcl
data.aws_caller_identity.current.account_id   # "123456789012"
data.aws_caller_identity.current.arn          # your role/user ARN
data.aws_caller_identity.current.user_id
```

Used everywhere in IAM policies and resource naming:

```hcl
resource "aws_s3_bucket" "attachments" {
  # Globally unique without a random suffix, because account IDs are unique
  bucket = "notely-${var.environment}-attachments-${data.aws_caller_identity.current.account_id}"
}
```

### `aws_region` — where am I?

```hcl
data "aws_region" "current" {}
```

```hcl
data.aws_region.current.name          # "ap-southeast-2"
data.aws_region.current.description   # "Asia Pacific (Sydney)"
```

Useful when you want the region without threading a variable everywhere.

### `aws_availability_zones` — which AZs can I actually use?

This is the one that fixes Notely's second lie.

```hcl
data "aws_availability_zones" "available" {
  state = "available"

  # Exclude Local Zones and Wavelength Zones - they behave differently
  # and you almost never want them by accident.
  filter {
    name   = "opt-in-status"
    values = ["opt-in-not-required"]
  }
}
```

```hcl
data.aws_availability_zones.available.names
# ["ap-southeast-2a", "ap-southeast-2b", "ap-southeast-2c"]
```

Then take as many as you need:

```hcl
locals {
  azs = slice(data.aws_availability_zones.available.names, 0, var.az_count)
}
```

This now works in **any** region, with whatever AZs that account actually has.

### `aws_ami` — the latest image

This fixes Notely's first lie.

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }
}
```

```hcl
data.aws_ami.amazon_linux.id            # "ami-0892a9c01908fafd1"
data.aws_ami.amazon_linux.name
data.aws_ami.amazon_linux.creation_date
```

Three things matter here:

**`owners` is required and important.** Without it you might match an AMI
published by a stranger. `["amazon"]` means AWS's own images. For Ubuntu it is
`["099720109477"]` (Canonical's account). Never omit it.

**`most_recent = true`** picks the newest match. Without it, multiple matches is
an error.

**The `name` filter with a wildcard** is how you pin to a family without pinning
to a version.

Some useful patterns:

```hcl
# Amazon Linux 2023, x86
values = ["al2023-ami-*-x86_64"]

# Amazon Linux 2023, ARM (Graviton - cheaper)
values = ["al2023-ami-*-arm64"]

# Ubuntu 22.04, owner 099720109477
values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
```

### A warning about `most_recent`

`most_recent = true` means your infrastructure changes when Amazon publishes a
new image — even if you changed nothing.

```text
  ~ aws_instance.app["app-a"] must be replaced
      ~ ami = "ami-0892a9c" -> "ami-0f4e1d2" # forces replacement
```

You ran `terraform plan` on Monday having touched nothing, and it wants to
replace both your servers.

That is usually **good** — you get patched images. But it must be deliberate.
Three ways to handle it:

```hcl
# 1. Accept it. Fine for dev, and for anything behind a load balancer
#    with create_before_destroy.

# 2. Pin the AMI in production via a variable, and update it deliberately.
ami = var.environment == "prod" ? var.pinned_ami_id : data.aws_ami.amazon_linux.id

# 3. Ignore AMI changes and let a separate process roll instances.
lifecycle {
  ignore_changes = [ami]
}
```

Notely uses option 1 with `create_before_destroy`, which is the right trade for a
learning project and for most dev environments.

### `aws_vpc` and `aws_subnets` — finding infrastructure you did not create

Useful when another team owns the network:

```hcl
data "aws_vpc" "shared" {
  tags = {
    Name = "shared-services-vpc"
  }
}

data "aws_subnets" "private" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.shared.id]
  }

  tags = {
    Tier = "private"
  }
}
```

```hcl
data.aws_vpc.shared.id
data.aws_vpc.shared.cidr_block
data.aws_subnets.private.ids     # a list of subnet IDs
```

### `aws_iam_policy_document` — the better way to write IAM

You have been writing IAM policies with `jsonencode`. That works. This is nicer:

```hcl
data "aws_iam_policy_document" "app_s3" {
  statement {
    sid    = "ReadWriteAttachments"
    effect = "Allow"

    actions = [
      "s3:GetObject",
      "s3:PutObject",
      "s3:DeleteObject",
    ]

    resources = ["${aws_s3_bucket.attachments.arn}/*"]
  }

  statement {
    sid       = "ListAttachmentsBucket"
    effect    = "Allow"
    actions   = ["s3:ListBucket"]
    resources = [aws_s3_bucket.attachments.arn]
  }

  statement {
    sid    = "WriteLogs"
    effect = "Allow"

    actions = [
      "logs:CreateLogStream",
      "logs:PutLogEvents",
    ]

    resources = ["${aws_cloudwatch_log_group.app.arn}:*"]
  }
}

resource "aws_iam_role_policy" "app" {
  name   = "${local.name_prefix}-app"
  role   = aws_iam_role.app.id
  policy = data.aws_iam_policy_document.app_s3.json
}
```

Why it is better than `jsonencode`:

| | `jsonencode` | `aws_iam_policy_document` |
|---|---|---|
| Validates policy structure | No | **Yes** |
| Catches typos in `Effect` | No | Yes |
| Supports `dynamic` blocks | Awkward | Naturally |
| Can merge documents | Manually | `source_policy_documents` |
| Readability | Fine | Better for long policies |

Both are acceptable. Use the data source for anything non-trivial.

### `aws_ssm_parameter` and `aws_secretsmanager_secret_version`

Reading configuration and secrets that live outside Terraform:

```hcl
data "aws_ssm_parameter" "db_host" {
  name = "/notely/${var.environment}/db/host"
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "notely/${var.environment}/db-password"
}
```

```hcl
data.aws_ssm_parameter.db_host.value
jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
```

> **Reading a secret with a data source puts it in your state file in plaintext.** That is not always wrong, but know that you did it. Module 10 covers the alternatives.

### `terraform_remote_state` — reading another configuration's outputs

When your infrastructure is split across several state files:

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"

  config = {
    bucket = "notely-tfstate-a1b2c3d4"
    key    = "notely/${var.environment}/network/terraform.tfstate"
    region = "ap-southeast-2"
  }
}
```

```hcl
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.app_subnet_ids[0]
}
```

Two important caveats:

**It only reads `output` blocks.** If the network configuration does not export
something as an output, you cannot see it.

**It requires read access to the other state file** — which, as Module 3
explained, means read access to every secret in that state. That is a real
security consideration.

The alternative many teams prefer: publish values to SSM Parameter Store and read
those instead. Looser coupling, and no state access needed.

```hcl
# network config publishes
resource "aws_ssm_parameter" "vpc_id" {
  name  = "/notely/${var.environment}/network/vpc-id"
  type  = "String"
  value = aws_vpc.main.id
}

# app config reads
data "aws_ssm_parameter" "vpc_id" {
  name = "/notely/${var.environment}/network/vpc-id"
}
```

---

## Part 3 — When data sources resolve

This causes real confusion, so be precise about it.

A data source is read **during plan**, if it can be.

```hcl
data "aws_ami" "amazon_linux" { ... }    # read at plan time
```

Terraform calls the AWS API while planning, gets the AMI ID, and the plan shows
the real value.

But if a data source depends on something that does not exist yet, it **cannot**
be read at plan time:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

data "aws_subnets" "in_new_vpc" {
  filter {
    name   = "vpc-id"
    values = [aws_vpc.main.id]      # doesn't exist yet on first apply
  }
}
```

The plan shows:

```text
  <= data "aws_subnets" "in_new_vpc" {
      + ids = (known after apply)
    }
```

It is deferred to apply time. That is fine, but the uncertainty spreads: anything
using those IDs also becomes unknown, so your plan shows less detail.

> Prefer referencing resources directly over looking them up with a data source. `aws_subnet.this["app-a"].id` is always better than a data source searching for the subnet you just made.

### Data sources are re-read every time

There is no caching between runs. Every `plan` and `apply` re-queries. That is
what makes `most_recent = true` on an AMI able to surprise you.

### `depends_on` on a data source

Occasionally you need to force a data source to wait:

```hcl
data "aws_instances" "app" {
  instance_tags = {
    Project = "notely"
  }

  depends_on = [aws_instance.app]
}
```

Without it, the lookup might run before the instances exist and return nothing.

---

## Building it into Notely

Three fixes.

### 1. Look up the AMI

Create `data.tf`:

```hcl
# Who is running this?
data "aws_caller_identity" "current" {}

# Which region are we in?
data "aws_region" "current" {}

# Which availability zones can this account actually use?
data "aws_availability_zones" "available" {
  state = "available"

  filter {
    name   = "opt-in-status"
    values = ["opt-in-not-required"]
  }
}

# The most recent Amazon Linux 2023 image, in whatever region we are in.
# owners = ["amazon"] matters - without it you could match a stranger's AMI.
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }
}
```

Delete `variable "app_ami_id"` entirely, and update the instance:

```hcl
resource "aws_instance" "app" {
  for_each = { for k, v in local.subnets : k => v if v.tier == "app" }

  ami           = data.aws_ami.amazon_linux.id     # was var.app_ami_id
  instance_type = var.app_instance_type
  # ... rest unchanged
}
```

### 2. Look up the availability zones

Add a variable for how many AZs to use:

```hcl
variable "az_count" {
  description = "How many availability zones to spread across"
  type        = number
  default     = 2

  validation {
    condition     = var.az_count >= 2 && var.az_count <= 3
    error_message = "az_count must be 2 or 3. One AZ is not highly available; more than three is unusual."
  }
}
```

Then in `locals.tf`, replace the guessed names:

```hcl
locals {
  # Was: azs = ["${var.region}a", "${var.region}b"]
  # Now: ask AWS which zones this account can actually use.
  azs = slice(data.aws_availability_zones.available.names, 0, var.az_count)
}
```

Check it:

```bash
terraform console -var-file=dev.tfvars
```

```text
> data.aws_availability_zones.available.names
[
  "ap-southeast-2a",
  "ap-southeast-2b",
  "ap-southeast-2c",
]

> local.azs
[
  "ap-southeast-2a",
  "ap-southeast-2b",
]
```

Now try three:

```bash
terraform console -var-file=dev.tfvars -var az_count=3 <<< 'keys(local.subnets)'
```

```text
[
  "app-a", "app-b", "app-c",
  "data-a", "data-b", "data-c",
  "public-a", "public-b", "public-c",
]
```

Nine subnets from changing one number. That is `for_each` and data sources
working together.

### 3. Rewrite the IAM policy as a document

Replace the `jsonencode` policy in `main.tf`:

```hcl
data "aws_iam_policy_document" "app_assume_role" {
  statement {
    effect  = "Allow"
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

data "aws_iam_policy_document" "app_permissions" {
  statement {
    sid    = "ReadWriteAttachments"
    effect = "Allow"

    actions = [
      "s3:GetObject",
      "s3:PutObject",
      "s3:DeleteObject",
    ]

    resources = ["${aws_s3_bucket.attachments.arn}/*"]
  }

  statement {
    sid       = "ListAttachmentsBucket"
    effect    = "Allow"
    actions   = ["s3:ListBucket"]
    resources = [aws_s3_bucket.attachments.arn]
  }

  statement {
    sid    = "WriteApplicationLogs"
    effect = "Allow"

    actions = [
      "logs:CreateLogStream",
      "logs:PutLogEvents",
    ]

    resources = ["${aws_cloudwatch_log_group.app.arn}:*"]
  }
}

resource "aws_iam_role" "app" {
  name               = "${local.name_prefix}-app"
  assume_role_policy = data.aws_iam_policy_document.app_assume_role.json
}

resource "aws_iam_role_policy" "app" {
  name   = "${local.name_prefix}-app"
  role   = aws_iam_role.app.id
  policy = data.aws_iam_policy_document.app_permissions.json
}
```

### 4. Use the account ID in the bucket name

You can now drop `random_id`:

```hcl
resource "aws_s3_bucket" "attachments" {
  # Account IDs are globally unique, so this name is too.
  bucket = "${local.name_prefix}-attachments-${data.aws_caller_identity.current.account_id}"
}
```

Deterministic, readable, and no random suffix to look up.

(If you already applied with the random suffix, changing the name replaces the
bucket. For a lab that is fine. In production you would leave it alone.)

---

## Real-World Example

A company runs infrastructure in three regions. Their configuration is identical
in all three, because it looks everything up:

```hcl
data "aws_region" "current" {}
data "aws_caller_identity" "current" {}

data "aws_availability_zones" "available" {
  state = "available"
  filter {
    name   = "opt-in-status"
    values = ["opt-in-not-required"]
  }
}

data "aws_ami" "app" {
  most_recent = true
  owners      = [data.aws_caller_identity.current.account_id]

  filter {
    name   = "name"
    values = ["company-app-base-*"]
  }
}

locals {
  azs = slice(data.aws_availability_zones.available.names, 0, 3)
}
```

Note `owners = [account_id]` — they bake their own AMIs with Packer, so they look
up their *own* latest image, not Amazon's.

Deploying to a fourth region means changing one provider region. Every AZ name,
every AMI ID, every account reference resolves itself.

**The failure that taught them this.** Before data sources, their `us-east-1`
configuration hardcoded `["us-east-1a", "us-east-1b"]`. A new AWS account they
opened simply did not have capacity in `us-east-1a` for the instance type they
wanted, and every apply failed with an opaque capacity error. With
`aws_availability_zones` the configuration would have picked zones the account
could actually use.

---

## Common Mistakes Beginners Make

**1. Forgetting the `data.` prefix.**

```hcl
ami = aws_ami.amazon_linux.id        # wrong - looks like a resource
ami = data.aws_ami.amazon_linux.id   # correct
```

**2. Omitting `owners` on `aws_ami`.**

You could match an image published by anyone. Always specify.

**3. Using a data source to find something Terraform just created.**

```hcl
# Don't
data "aws_subnet" "app" {
  filter { name = "tag:Name", values = ["notely-app-a"] }
}

# Do
aws_subnet.this["app-a"].id
```

The direct reference is faster, always correct, and creates a proper dependency
edge.

**4. Being surprised when `most_recent = true` replaces your servers.**

It is working as designed. Decide deliberately whether you want it.

**5. Reading a secret with a data source and forgetting it lands in state.**

Plaintext. Every time.

**6. Assuming `terraform_remote_state` can read anything.**

It can only read `output` blocks from the other configuration.

**7. Expecting a data source to be created or destroyed.**

It is read-only. `terraform destroy` will not touch what it points at.

---

## Hands-On Lab — Remove Notely's Magic Values

**Cost: free** for parts A–C. Part D optionally re-applies the servers.

### Part A: Explore data sources in the console

```bash
cd ~/terraform-labs/notely
```

Create `data.tf` with all four data sources shown above.

```bash
terraform init
terraform console -var-file=dev.tfvars
```

```text
> data.aws_caller_identity.current.account_id
"123456789012"

> data.aws_region.current.name
"ap-southeast-2"

> data.aws_availability_zones.available.names
[
  "ap-southeast-2a",
  "ap-southeast-2b",
  "ap-southeast-2c",
]

> data.aws_ami.amazon_linux.id
"ami-0892a9c01908fafd1"

> data.aws_ami.amazon_linux.name
"al2023-ami-2024.x.xxxxxxxx.0-kernel-6.1-x86_64"

> data.aws_ami.amazon_linux.creation_date
"2026-08-14T05:12:33.000Z"
```

That last one is the point — check how recent the image is. Then compare with
the hardcoded value you were using.

### Part B: Replace the AMI and AZ lookups

- Delete `variable "app_ami_id"`
- Add `variable "az_count"`
- Change `local.azs` to use `slice(data.aws_availability_zones.available.names, 0, var.az_count)`
- Change `aws_instance.app` to use `data.aws_ami.amazon_linux.id`

```bash
terraform fmt
terraform validate
terraform plan -var-file=dev.tfvars
```

If your hardcoded AMI was stale, the plan will show the instances being replaced
with a newer image. Read the `# forces replacement` line.

### Part C: Prove az_count works

```bash
terraform console -var-file=dev.tfvars -var az_count=3 <<< 'length(local.subnets)'
```

```text
9
```

```bash
terraform plan -var-file=dev.tfvars -var az_count=3
```

`Plan: 3 to add` — three new subnets, one per tier, in AZ c. Nothing existing is
touched, because `for_each` keys are stable.

**Do not apply this** unless you want nine subnets. Revert to the default.

### Part D: Rewrite the IAM policy

Replace the `jsonencode` policies with `aws_iam_policy_document` data sources.

```bash
terraform plan -var-file=dev.tfvars
```

The plan should show **no change to the policy** — the JSON produced is
equivalent. If it shows a diff, compare the two carefully; the data source may
have ordered statements differently, which is harmless.

Look at the generated JSON:

```bash
terraform console -var-file=dev.tfvars <<< 'data.aws_iam_policy_document.app_permissions.json'
```

Now break it deliberately:

```hcl
statement {
  effect = "Alow"      # typo
  ...
}
```

```bash
terraform validate
```

```text
Error: expected effect to be one of ["Allow" "Deny"], got Alow
```

`jsonencode` would have accepted that happily and failed at apply time with an
AWS error. That is the value of the data source.

Fix the typo.

### Part E: See a deferred data source

Add this temporarily:

```hcl
data "aws_instances" "notely_app" {
  instance_tags = {
    Project = "notely"
  }

  depends_on = [aws_instance.app]
}

output "discovered_instance_ids" {
  value = data.aws_instances.notely_app.ids
}
```

```bash
terraform plan -var-file=dev.tfvars
```

```text
  <= data "aws_instances" "notely_app" {
      + ids = (known after apply)
    }
```

The `<=` symbol is a data source read. `(known after apply)` because it depends
on instances that may not exist yet.

Remove it afterwards — it is a demonstration, not something Notely needs.

### Part F: Tear down

If you applied anything billable:

```bash
terraform destroy -var-file=dev.tfvars
```

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

### What you should have at the end

- Zero hardcoded AMI IDs in Notely
- AZ names looked up, not guessed
- `az_count` able to take Notely from 2 zones to 3 with one number
- IAM policies written as documents, with validation
- Understanding of the `<=` plan symbol

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **`data` block** | Read something that already exists |
| **`data.` prefix** | Required when referencing. Easy to forget. |
| **`<=` in a plan** | A data source read |
| **Read-only** | `destroy` never touches what a data source points at |
| **`aws_caller_identity`** | Your account ID |
| **`aws_region`** | The current region |
| **`aws_availability_zones`** | Which AZs this account can use |
| **`aws_ami`** | Find an image. **Always set `owners`.** |
| **`most_recent = true`** | Newest match. Can replace servers unexpectedly. |
| **`aws_iam_policy_document`** | Validated IAM policies. Better than `jsonencode`. |
| **`aws_ssm_parameter`** | Read config from Parameter Store |
| **`terraform_remote_state`** | Read another config's **outputs only** |
| **Plan time vs apply time** | Read at plan, unless it depends on something unbuilt |
| **Prefer direct references** | Don't look up what Terraform just created |
| **Secrets in data sources** | Land in state, in plaintext |

---

## Checkpoint (answer briefly)

1. What is the difference between a `resource` block and a `data` block?
2. Why is `owners = ["amazon"]` important on `aws_ami`?
3. You run `terraform plan` on Monday having changed nothing, and it wants to replace both app servers. What is the likely cause?
4. Why is `aws_subnet.this["app-a"].id` better than a data source that searches for that subnet?
5. What does `<=` mean in plan output?
6. What are the two downsides of `terraform_remote_state`?
7. Notely now uses `slice(data.aws_availability_zones.available.names, 0, var.az_count)`. What does that fix?

---

## Checkpoint — model answers

### 1. `resource` vs `data`

**`resource`** declares something Terraform **creates, owns and manages**. It
appears in state as `managed`, it can be updated when your configuration changes,
and `terraform destroy` deletes it.

**`data`** declares something Terraform **only reads**. Terraform did not create
it and will never modify or delete it. It appears in state only as a cached
result, and `destroy` ignores it entirely.

The practical consequence: data sources are completely safe. There is no way to
break infrastructure with one. You can add a data source to a production
configuration and the worst that happens is an error if the lookup finds nothing.

The plan symbols reflect this: resources get `+`, `~`, `-`, `-/+`; data sources
only ever get `<=`.

### 2. Why `owners` matters on `aws_ami`

Because AMI names are **not unique or protected**. Anyone with an AWS account can
publish a public AMI and call it whatever they like.

If your filter is:

```hcl
filter {
  name   = "name"
  values = ["al2023-ami-*-x86_64"]
}
```

with no `owners`, you are asking for "any public image whose name matches this
pattern". Someone could publish a matching image containing anything they want,
and `most_recent = true` would happily select it because it is newer than
Amazon's.

You would then be booting production servers from a stranger's disk image.

`owners = ["amazon"]` restricts the search to images published by AWS. Other
values you will see:

- `["099720109477"]` — Canonical, for official Ubuntu
- `["self"]` — your own account's images
- `[data.aws_caller_identity.current.account_id]` — the same, written portably

This is a genuine supply-chain concern, not a theoretical one. Always set it.

### 3. Servers being replaced with no config change

**`most_recent = true` on the AMI data source, and Amazon published a new image.**

Data sources are re-read on every plan. There is no caching between runs. So on
Monday the `aws_ami` lookup returned a different AMI ID than it did on Friday,
because AWS released a new Amazon Linux build over the weekend.

`ami` is an attribute that cannot be changed in place on a running instance, so
Terraform plans a replacement:

```text
~ ami = "ami-0892a9c" -> "ami-0f4e1d2" # forces replacement
```

This is working as designed, and it is often what you want — new images carry
security patches. But it must be a decision, not a surprise.

Options: accept it and rely on `create_before_destroy` (what Notely does); pin
the AMI to a variable in production and bump it deliberately; or
`ignore_changes = [ami]` and let a separate process roll the instances.

The one thing you should not do is discover it during an incident.

### 4. Direct reference vs data source lookup

Three reasons, in order of importance.

**It creates a real dependency edge.** `aws_subnet.this["app-a"].id` tells
Terraform the instance depends on that subnet, so it is created first and
destroyed last. A data source searching by tag creates no such relationship —
Terraform may run the lookup before the subnet exists.

**It cannot be wrong.** The reference is resolved from state and always points at
the exact resource. A tag-based lookup could match a different subnet, match
several and error, or match none and fail — especially if someone changes a tag.

**It is faster.** No API call. The value is already known.

Data sources are for things **outside** your configuration: infrastructure
another team owns, images AWS publishes, values in Parameter Store. Anything your
own configuration creates should be referenced directly.

### 5. `<=` in plan output

It means **a data source will be read**.

The five plan symbols:

| Symbol | Meaning |
|---|---|
| `+` | create |
| `~` | update in place |
| `-` | destroy |
| `-/+` | replace |
| `<=` | **read** (a data source) |

It appears in a plan when the data source could not be resolved during planning —
typically because it depends on a resource that does not exist yet, so the read
is deferred to apply time. Its attributes show as `(known after apply)`.

Data sources that Terraform *could* resolve at plan time do not appear as a
change at all; their values are simply used.

Seeing `<=` is not a problem. It does tell you that anything downstream of that
data source will also be unknown in the plan, which is why a plan containing
deferred data sources often shows less detail than you would like.

### 6. Downsides of `terraform_remote_state`

**It can only read `output` blocks.** If the other configuration does not
explicitly export a value, it is invisible. That means the two configurations are
coupled through an interface someone has to maintain — add a resource and you
must remember to add an output too.

**It requires read access to the other state file.** As Module 3 established,
state contains every attribute of every resource in plaintext, including
passwords. Granting a configuration read access to another state file grants it
access to all of that team's secrets. That is a much bigger permission than
"let me see the VPC ID".

There is a third, softer problem: it creates a hard dependency between state
files. Moving or renaming the other configuration's state key breaks yours.

The common alternative is publishing to SSM Parameter Store: the network
configuration writes `/notely/dev/network/vpc-id`, the app configuration reads
it. Looser coupling, IAM permissions scoped to exactly those parameters, and no
state access at all.

### 7. What the AZ lookup fixes

It removes an assumption that is usually true and occasionally wrong.

The old code was:

```hcl
azs = ["${var.region}a", "${var.region}b"]
```

This assumes AZ names are always the region name plus a letter, and that every
account can use the first two.

Two things break that:

**Not every account has access to every AZ.** AWS maps zone *names* to physical
zones differently per account, and some accounts genuinely cannot launch certain
instance types in certain zones — or cannot use a given zone at all. Hardcoding
`us-east-1a` in a new account can produce capacity errors that look like nothing
to do with configuration.

**Not every region follows the pattern.** Local Zones and Wavelength Zones have
names like `us-west-2-lax-1a`, and blindly appending a letter to a region does
not produce a valid zone.

The data source asks AWS which zones this specific account can actually use right
now, and `slice(..., 0, var.az_count)` takes as many as you asked for. The
configuration is now correct in any region, in any account, without editing.

The `opt-in-status` filter is part of that fix too — it excludes Local and
Wavelength Zones, which you almost never want selected by accident.

---

## Next lesson

**Module 8 — Modules** (`08-modules.md`)

Notely's configuration now works well but is one large flat directory: about 300
lines of network, compute, security and storage all mixed together, and no way to
reuse any of it.

Module 8 is the big reorganisation. You extract `modules/network`,
`modules/web` and `modules/database`, and Notely's root configuration shrinks to
about forty lines that read like a summary of the architecture.

It is also what makes Module 9 possible — once Notely is modules, spinning up
staging and prod is straightforward.
