# Module 13 — Importing Existing Infrastructure & Drift

## Where Notely is right now

```text
  Complete system, three environments, seven modules, sixteen tests,
  and a pipeline that plans on every pull request.

  Everything Notely owns, Terraform created.
```

That is the easy case, and it is not the case you will walk into at work.

By the end of this module you can adopt infrastructure somebody built by hand,
and restructure your code without destroying anything.

Full picture: `notely-architecture.md`.

---

## Why this module exists

You join a company. There are three years of infrastructure in the AWS console.
Nobody knows exactly what is there. You are asked to "put it in Terraform".

You cannot start from an empty state file — `terraform apply` would try to create
everything and fail with "already exists" on the first resource.

You need **import**: telling Terraform "this thing already exists, and from now
on you manage it".

The second half of this module is the flip side. Your Module 12 drift job is now
running daily, and sooner or later it will go red. What then?

---

## The core idea (one sentence)

Import writes a resource into state without creating it, and your goal is always
the same: get to an empty plan.

> An empty plan means your code and reality agree. It is the acceptance test for every import, every refactor, and every drift fix.

---

## Mental model (from Node.js)

You have done this. It is adopting an untracked file into git:

```bash
# The file exists on disk. Git does not know about it.
git add existing-file.js

# Now git tracks it. The file did not change.
git status    # clean
```

`terraform import` is `git add` for infrastructure:

| git | Terraform |
|---|---|
| File exists, untracked | Resource exists, unmanaged |
| `git add file` | `terraform import addr id` |
| Now tracked; file unchanged | Now in state; resource unchanged |
| `git status` clean | `terraform plan` empty |
| `git mv` | `moved` block |
| `git rm --cached` | `removed` block |

And the goal in both cases is the same: a clean status.

---

## Part 1 — Drift, properly

Drift is real infrastructure no longer matching what Terraform recorded.

### Where it comes from

| Cause | Example |
|---|---|
| **Console edits** | Someone changed an instance type at 2am during an incident |
| **Other tools** | An autoscaling policy changed the desired count |
| **AWS itself** | A minor engine version upgrade during the maintenance window |
| **Another Terraform** | Two configurations both think they own something |
| **Manual "temporary" fixes** | A security group rule added to unblock someone |

### Detecting it

Every plan detects drift, because plan refreshes state against reality first.

```bash
terraform plan
```

```text
Note: Objects have changed outside of Terraform

  ~ aws_instance.app["app-a"]
      ~ instance_type = "t3.micro" -> "t3.small"

Plan: 0 to add, 1 to change, 0 to destroy.
```

Terraform is telling you two things: someone changed this outside Terraform, and
I intend to change it back.

For automation, `-detailed-exitcode` (Module 12):

```bash
terraform plan -detailed-exitcode
# 0 = no changes, 1 = error, 2 = changes
```

### The three responses

**Response A: revert it.** Your configuration is right, the change was not.

```bash
terraform apply
```

This is the default and usually correct. Terraform's job is to make reality match
the code.

**Response B: accept it.** The change was right, your configuration is stale.

```bash
terraform plan -refresh-only
terraform apply -refresh-only
```

This updates **state** to match reality without changing infrastructure. Then you
update the configuration to match, and the next plan is empty.

```bash
# 1. Accept reality into state
terraform apply -refresh-only

# 2. Update the code to match
#    instance_type = "t3.small"

# 3. Verify
terraform plan     # No changes.
```

**Response C: stop tracking it.** The attribute is genuinely managed elsewhere.

```hcl
resource "aws_db_instance" "this" {
  engine_version = "16.3"

  lifecycle {
    ignore_changes = [
      # AWS applies minor upgrades in the maintenance window.
      # Without this, every plan tries to revert them.
      engine_version,
    ]
  }
}
```

Legitimate when another system genuinely owns the value. Illegitimate as a way to
silence a diff you have not understood — that hides all future drift on that
attribute forever.

### `terraform refresh` is deprecated

The old command updated state without showing you anything first. Use
`-refresh-only`, which shows a plan you can read before agreeing.

| Command | Shows a plan first | Changes state | Changes infrastructure |
|---|---|---|---|
| `terraform refresh` | No | Yes | No |
| `terraform plan -refresh-only` | Yes | No | No |
| `terraform apply -refresh-only` | Yes | Yes | **No** |

---

## Part 2 — `import` blocks (the modern way)

Terraform 1.5 added declarative import. Use this rather than the old command.

```hcl
import {
  to = aws_s3_bucket.legacy_uploads
  id = "acme-legacy-uploads"
}

resource "aws_s3_bucket" "legacy_uploads" {
  bucket = "acme-legacy-uploads"
}
```

```bash
terraform plan
```

```text
  # aws_s3_bucket.legacy_uploads will be imported
    resource "aws_s3_bucket" "legacy_uploads" {
        arn    = "arn:aws:s3:::acme-legacy-uploads"
        bucket = "acme-legacy-uploads"
        id     = "acme-legacy-uploads"
    }

Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

```bash
terraform apply
```

### Why blocks beat the old command

| | `terraform import` (command) | `import` block |
|---|---|---|
| Shows a plan first | **No** — it just does it | Yes |
| Reviewable in a PR | No | **Yes** |
| Can import many at once | No, one at a time | Yes |
| Works with `for_each` | No | Yes |
| Can generate config | No | **Yes** |
| Undo | `state rm` | Delete the block |

The old command still works and you will see it in older documentation:

```bash
terraform import aws_s3_bucket.legacy_uploads acme-legacy-uploads
```

But it modifies state immediately with no preview, which is exactly what you do
not want when adopting production infrastructure.

### Importing many at once

```hcl
import {
  for_each = toset(["logs", "uploads", "backups"])

  to = aws_s3_bucket.legacy[each.key]
  id = "acme-legacy-${each.key}"
}

resource "aws_s3_bucket" "legacy" {
  for_each = toset(["logs", "uploads", "backups"])
  bucket   = "acme-legacy-${each.key}"
}
```

### Generating the configuration

The best feature. You do not have to hand-write the resource block:

```hcl
import {
  to = aws_security_group.legacy_web
  id = "sg-0abc123def456"
}
```

```bash
terraform plan -generate-config-out=generated.tf
```

Terraform writes `generated.tf`:

```hcl
resource "aws_security_group" "legacy_web" {
  description = "Web servers"
  name        = "acme-web-sg"
  vpc_id      = "vpc-0123456789abcdef0"

  ingress = [
    {
      cidr_blocks      = ["0.0.0.0/0"]
      description      = ""
      from_port        = 443
      to_port          = 443
      protocol         = "tcp"
      # ... every attribute, filled in
    },
  ]
}
```

**It always needs editing.** Generated configuration:

- Hardcodes every ID that should be a reference (`vpc_id = "vpc-0123..."` should
  be `aws_vpc.main.id`)
- Includes read-only attributes you must remove
- Has no variables, no locals, no structure
- Ignores your naming conventions

Treat it as a starting point that saves you thirty minutes of transcription, not
as finished code.

### Finding resource IDs

This is the fiddly part — every resource type uses a different ID format.

| Resource | ID format | Example |
|---|---|---|
| `aws_s3_bucket` | the bucket name | `acme-uploads` |
| `aws_instance` | instance ID | `i-0abc123def456` |
| `aws_vpc` | VPC ID | `vpc-0abc123` |
| `aws_subnet` | subnet ID | `subnet-0abc123` |
| `aws_security_group` | group ID | `sg-0abc123` |
| `aws_iam_role` | role **name** | `acme-app-role` |
| `aws_iam_policy` | policy **ARN** | `arn:aws:iam::123:policy/x` |
| `aws_db_instance` | identifier | `acme-prod-db` |
| `aws_cloudwatch_log_group` | log group name | `/acme/app` |
| `aws_route_table_association` | `subnet-id/rtb-id` | `subnet-abc/rtb-def` |
| `aws_lb` | ARN | `arn:aws:elasticloadbalancing:...` |
| `aws_secretsmanager_secret` | ARN or name | `acme/db-credentials` |

**Always check the provider documentation.** Every resource page has an "Import"
section at the bottom showing the exact format.

Some are genuinely surprising — `aws_route_table_association` needs two IDs
joined by a slash, and `aws_s3_bucket_versioning` needs the bucket name.

---

## Part 3 — The realistic import workflow

Six steps. Do them in order.

### Step 1: Inventory

Find what exists.

```bash
# Everything with a given tag
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=legacy-app \
  --query 'ResourceTagMappingList[].ResourceARN' --output text

# Or per service
aws s3api list-buckets --query 'Buckets[].Name' --output text
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{Id:InstanceId,Name:Tags[?Key==`Name`]|[0].Value}' \
  --output table
aws ec2 describe-security-groups \
  --query 'SecurityGroups[].{Id:GroupId,Name:GroupName}' --output table
```

Write the list down. You will lose track otherwise.

### Step 2: Import one resource, in isolation

Do not try to import forty things at once. Start with one, in a scratch
directory, and learn what the plan looks like.

Pick something low-risk — an S3 bucket, a log group, an SNS topic. Not the
production database.

### Step 3: Generate and edit

```hcl
import {
  to = aws_s3_bucket.uploads
  id = "acme-legacy-uploads"
}
```

```bash
terraform plan -generate-config-out=generated.tf
```

Then edit `generated.tf`:

- Move it into a sensible file
- Replace hardcoded IDs with references
- Extract values into variables
- Remove read-only attributes
- Rename to match your conventions

### Step 4: Iterate to an empty plan

```bash
terraform plan
```

The first plan after an import almost never comes back empty. You will see
differences between the generated config and reality — a missing tag, a default
you did not specify.

Adjust the configuration — **not** the infrastructure — until:

```text
Plan: 0 to add, 0 to change, 0 to destroy.
```

**This is the whole job.** An empty plan means your code accurately describes
what exists.

If the plan wants to *change* something, your code is wrong. If it wants to
*destroy and recreate* something, stop immediately — you have misconfigured an
attribute that forces replacement, and applying would delete production.

### Step 5: Apply and remove the import blocks

```bash
terraform apply
```

Then delete the `import` blocks. They are one-time instructions; leaving them in
is harmless but noisy.

### Step 6: Repeat

One resource, or one small group, at a time. Resist the urge to import
everything in one pull request — a 40-resource import PR is unreviewable.

> Never import into a state file that manages production until you have practised on something you can afford to break.

### Bulk import tools

`terraformer`, `aws2tf` and Former2 can generate configuration for an entire
account.

They are useful for the inventory step. The output is **not** production code —
no modules, no variables, no references, thousands of lines. Use them to
discover, then write the real configuration yourself.

---

## Part 4 — `moved` blocks

You have used these twice already (Modules 6 and 8). Here is the full picture.

A `moved` block tells Terraform that a resource's **address** changed, but the
resource did not.

### The four cases

**1. Renaming a resource:**

```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.app
}
```

**2. Moving into a module:**

```hcl
moved {
  from = aws_vpc.main
  to   = module.network.aws_vpc.main
}
```

**3. Moving a whole module:**

```hcl
moved {
  from = module.network
  to   = module.notely.module.network
}
```

Everything inside comes with it.

**4. Converting `count` to `for_each`:**

```hcl
moved {
  from = aws_subnet.this[0]
  to   = aws_subnet.this["public-a"]
}

moved {
  from = aws_subnet.this[1]
  to   = aws_subnet.this["public-b"]
}
```

This one needs a block per element, because you are mapping positions to keys.

### Why `moved` and not `state mv`

| | `terraform state mv` | `moved` block |
|---|---|---|
| When it runs | Immediately, by hand | During `apply` |
| Shows a plan first | No | **Yes** |
| In version control | No | **Yes** |
| Reviewable | No | Yes |
| Runs for colleagues | No — everyone must run it | **Yes, automatically** |
| Runs in CI | No | Yes |

That fifth row is the decisive one. With `state mv`, you fix your state and
everyone else's next apply proposes destroying and recreating everything. With a
`moved` block, the fix travels with the code.

### Keeping them around

`moved` blocks are safe to leave in for a while. Terraform ignores one whose
`from` address is not in state. Keep them for a release or two so colleagues and
long-lived branches pick them up, then delete.

---

## Part 5 — `removed` blocks

Terraform 1.7 added the declarative version of `state rm`: stop managing
something **without destroying it**.

```hcl
removed {
  from = aws_s3_bucket.legacy_uploads

  lifecycle {
    destroy = false      # forget it, do not delete it
  }
}
```

```bash
terraform plan
```

```text
  # aws_s3_bucket.legacy_uploads will no longer be managed by Terraform,
  # but will not be destroyed.

Plan: 0 to add, 0 to change, 0 to destroy.
```

The `lifecycle { destroy = false }` is essential. Without it, the block means
"remove this and delete it".

### When you need it

- Handing a resource to another team's configuration
- Splitting one state file into several
- Retiring a resource from management before decommissioning it separately
- Undoing an import that turned out to be wrong

### `removed` vs `state rm`

Same relationship as `moved` vs `state mv`: the block is planned, reviewable, in
version control, and runs for everyone.

---

## Part 6 — Refactoring safely: the empty-plan invariant

This is the single most useful idea in the module.

> Any change that only moves code should produce `Plan: 0 to add, 0 to change, 0 to destroy.`

That covers:

- Renaming resources
- Moving resources between files
- Extracting modules
- Restructuring directories
- Converting `count` to `for_each`
- Importing existing infrastructure

If a code-only change produces a non-empty plan, **something is wrong**, and you
should find out what before applying.

The workflow:

```bash
# 1. Before you touch anything
terraform plan     # confirm it is already empty

# 2. Make the change

# 3. Add moved/import blocks

# 4. Verify
terraform plan     # must be empty again

# 5. Apply
terraform apply
```

Step 1 matters. If the plan is not empty *before* your change, you cannot tell
whether a difference afterwards is yours or was already there.

---

## Building it into Notely

Two exercises: adopt something built by hand, and restructure safely.

### 1. Adopt a console-created bucket

Imagine someone on the team created an S3 bucket by hand for exports, and now it
needs to be managed.

Create it by hand to simulate this:

```bash
ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
aws s3api create-bucket \
  --bucket notely-legacy-exports-${ACCOUNT} \
  --region ap-southeast-2 \
  --create-bucket-configuration LocationConstraint=ap-southeast-2

aws s3api put-bucket-tagging \
  --bucket notely-legacy-exports-${ACCOUNT} \
  --tagging 'TagSet=[{Key=Project,Value=notely},{Key=Ephemeral,Value=true}]'
```

Now adopt it. In `envs/dev/`:

```hcl
import {
  to = module.notely.module.storage.aws_s3_bucket.exports
  id = "notely-legacy-exports-123456789012"    # your account ID
}
```

And in `modules/storage/main.tf`:

```hcl
resource "aws_s3_bucket" "exports" {
  bucket = "notely-legacy-exports-${data.aws_caller_identity.current.account_id}"
}
```

```bash
terraform plan
```

```text
  # module.notely.module.storage.aws_s3_bucket.exports will be imported

Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

**Read that carefully.** `1 to import`, and crucially `0 to destroy`. If it said
`1 to add` instead of `1 to import`, the bucket name does not match and Terraform
would try to create a second one.

```bash
terraform apply
terraform state list | grep exports
```

Then delete the `import` block. It has done its job.

### 2. Bring it up to Notely's standards

The imported bucket has none of the protections Notely's other buckets have. Add
them:

```hcl
resource "aws_s3_bucket_public_access_block" "exports" {
  bucket = aws_s3_bucket.exports.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "exports" {
  bucket = aws_s3_bucket.exports.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

```bash
terraform plan
```

```text
Plan: 2 to add, 0 to change, 0 to destroy.
```

Two new resources, the bucket untouched. **This is the real value of importing** —
once something is under management, your standards apply to it automatically, and
`checkov` starts checking it.

### 3. Practise a safe rename

Rename `aws_s3_bucket.exports` to `aws_s3_bucket.data_exports`.

First without a `moved` block:

```hcl
resource "aws_s3_bucket" "data_exports" {
  bucket = "notely-legacy-exports-${data.aws_caller_identity.current.account_id}"
}
```

```bash
terraform plan
```

```text
Plan: 1 to add, 0 to change, 1 to destroy.
```

**A destroy.** On a bucket with objects in it, that is data loss — from renaming
a variable.

Now add the block:

```hcl
moved {
  from = module.notely.module.storage.aws_s3_bucket.exports
  to   = module.notely.module.storage.aws_s3_bucket.data_exports
}
```

```bash
terraform plan
```

```text
Plan: 0 to add, 0 to change, 0 to destroy.
```

Empty. That is the invariant.

### 4. Hand it back with `removed`

Suppose the exports bucket belongs to the data team's configuration instead.

```hcl
removed {
  from = module.notely.module.storage.aws_s3_bucket.data_exports

  lifecycle {
    destroy = false
  }
}
```

Comment out the resource block and its two companions.

```bash
terraform plan
```

```text
  # ... will no longer be managed by Terraform, but will not be destroyed.

Plan: 0 to add, 0 to change, 0 to destroy.
```

```bash
terraform apply
aws s3 ls | grep notely-legacy-exports
```

The bucket is still there. Terraform has simply forgotten it.

**Clean up** — since Terraform no longer manages it, `destroy` will not remove it:

```bash
aws s3 rb s3://notely-legacy-exports-${ACCOUNT} --force
```

That is the trade-off of `removed`: you have taken responsibility for it back.

### 5. Handle drift end to end

```bash
cd ~/terraform-labs/notely/envs/dev
terraform plan     # confirm empty
```

Create drift, the way a colleague would during an incident:

```bash
aws logs put-retention-policy \
  --log-group-name /notely-dev/app \
  --retention-in-days 30
```

```bash
terraform plan
```

```text
Note: Objects have changed outside of Terraform

  ~ retention_in_days = 7 -> 30

Plan: 0 to add, 1 to change, 0 to destroy.
```

**Response A — revert:**

```bash
terraform apply     # back to 7
```

**Response B — accept.** Recreate the drift, then:

```bash
terraform plan -refresh-only
```

```text
  ~ retention_in_days = 7 -> 30

This is a refresh-only plan, so Terraform will not take any actions to
undo these. If you were expecting these changes then you can apply this
plan to record the updated values in the Terraform state.
```

```bash
terraform apply -refresh-only
```

State now says 30. But your **code** still says 7:

```bash
terraform plan
```

```text
Plan: 0 to add, 1 to change, 0 to destroy.
```

Terraform wants to change it back. So update the code:

```hcl
log_retention_days = 30
```

```bash
terraform plan     # No changes.
```

**Both responses end at an empty plan.** That is always the destination.

---

## Real-World Example

An engineer joined a company with four years of console-built AWS infrastructure
across two accounts. The brief: "get it into Terraform".

**What they did not do:** run `terraformer` over the whole account and commit
12,000 lines.

**What they did, over four months:**

```text
Month 1   Inventory only. A spreadsheet of every resource, its ID, its
          owner, and whether anyone knew what it did.
          Outcome: 23 resources nobody could account for. Three were
          deleted after asking around. That paid for the month.

Month 2   Import the boring, safe things: S3 buckets, log groups,
          SNS topics, IAM roles. One PR per service, five to ten
          resources each. Every PR ends with an empty plan.

Month 3   Networking. VPCs, subnets, route tables, security groups.
          Slower - route table associations have compound IDs and the
          generated config needed a lot of editing. Nothing was
          destroyed.

Month 4   Databases and load balancers, one at a time, each with
          prevent_destroy added in the same PR that imported it.
```

**The rules they worked to:**

1. Never more than ten resources in one pull request
2. Every PR must end with `0 to add, 0 to change, 0 to destroy`
3. `prevent_destroy` goes on in the same PR as the import for anything stateful
4. Practise every import in the dev account first
5. If the plan says "destroy", stop and escalate

**The near-miss.** Importing an RDS instance, the generated configuration had
`db_name = null` because the attribute reads differently than it writes. The plan
showed:

```text
-/+ resource "aws_db_instance" "prod" {
      ~ db_name = "appdb" -> null # forces replacement
    }
```

Rule 5 caught it. Applying would have destroyed the production database.

The fix was one line — set `db_name = "appdb"` explicitly — but only because
somebody read the plan.

> The generated configuration is a draft. The plan is the review. Never skip the review.

---

## Common Mistakes Beginners Make

**1. Applying an import plan that says "destroy".**

Stop. An import should never destroy anything. If it does, an attribute in your
configuration forces replacement.

**2. Trusting generated configuration.**

It hardcodes IDs, includes read-only attributes, and ignores your conventions.
Always edit it.

**3. Importing forty resources in one pull request.**

Nobody can review it. Ten at most.

**4. Using `terraform import` (the command) instead of an `import` block.**

The command modifies state with no preview.

**5. Renaming a resource without a `moved` block.**

Terraform destroys and recreates it. On stateful resources that is data loss.

**6. Using `state mv` instead of a `moved` block.**

Your state is fixed; everyone else's is broken.

**7. `removed` without `lifecycle { destroy = false }`.**

That deletes the resource. The opposite of what you wanted.

**8. `ignore_changes` to silence a diff you do not understand.**

You have hidden all future drift on that attribute.

**9. Refactoring without checking the plan was empty first.**

You cannot tell your changes from pre-existing drift.

**10. Practising imports on production.**

Use the dev account.

---

## Hands-On Lab — Adopt, Refactor, and Fix Drift

**Cost: free.** S3 buckets with no objects and CloudWatch log groups are free
tier.

### Part A: Create something by hand

```bash
ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
REGION=ap-southeast-2

aws s3api create-bucket \
  --bucket notely-manual-${ACCOUNT} \
  --region ${REGION} \
  --create-bucket-configuration LocationConstraint=${REGION}

aws sns create-topic --name notely-manual-alerts

aws logs create-log-group --log-group-name /notely/manual-test
```

Three resources Terraform knows nothing about.

### Part B: Import the bucket

```bash
mkdir -p ~/terraform-labs/import-practice
cd ~/terraform-labs/import-practice
```

`main.tf`:

```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = "ap-southeast-2"
}

data "aws_caller_identity" "current" {}

import {
  to = aws_s3_bucket.manual
  id = "notely-manual-REPLACE_WITH_YOUR_ACCOUNT_ID"
}
```

Note there is **no resource block yet**. Let Terraform write it:

```bash
terraform init
terraform plan -generate-config-out=generated.tf
```

```bash
cat generated.tf
```

Read it. Notice the hardcoded bucket name and the attributes you did not ask for.

### Part C: Reach an empty plan

```bash
terraform plan
```

```text
Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

`1 to import` and `0 to destroy` is what you want.

```bash
terraform apply
terraform state list
```

Improve the generated code — replace the hardcoded account ID:

```hcl
resource "aws_s3_bucket" "manual" {
  bucket = "notely-manual-${data.aws_caller_identity.current.account_id}"
}
```

```bash
terraform plan
```

Still empty. The bucket is unchanged; only your code got better.

Delete the `import` block.

### Part D: Import the other two

```hcl
import {
  to = aws_sns_topic.alerts
  id = "arn:aws:sns:ap-southeast-2:ACCOUNT_ID:notely-manual-alerts"
}

import {
  to = aws_cloudwatch_log_group.manual
  id = "/notely/manual-test"
}
```

Note the three different ID formats: a bucket **name**, an SNS **ARN**, and a log
group **path**. This is the fiddly part of importing, and why you check the
provider docs.

```bash
terraform plan -generate-config-out=generated2.tf
cat generated2.tf
terraform apply
```

### Part E: Apply your standards

Now that they are managed, add the protections:

```hcl
resource "aws_s3_bucket_public_access_block" "manual" {
  bucket = aws_s3_bucket.manual.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "manual" {
  bucket = aws_s3_bucket.manual.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

```bash
terraform plan
```

`Plan: 2 to add` — and nothing changed on the bucket itself.

```bash
terraform apply
```

### Part F: Rename, the wrong way and the right way

```hcl
resource "aws_s3_bucket" "manual_uploads" {     # renamed
  bucket = "notely-manual-${data.aws_caller_identity.current.account_id}"
}
```

(Also update the two references.)

```bash
terraform plan
```

```text
Plan: 3 to add, 0 to change, 3 to destroy.
```

**Three destroys from renaming a label.** Do not apply.

Add the `moved` blocks:

```hcl
moved {
  from = aws_s3_bucket.manual
  to   = aws_s3_bucket.manual_uploads
}

moved {
  from = aws_s3_bucket_public_access_block.manual
  to   = aws_s3_bucket_public_access_block.manual_uploads
}

moved {
  from = aws_s3_bucket_server_side_encryption_configuration.manual
  to   = aws_s3_bucket_server_side_encryption_configuration.manual_uploads
}
```

```bash
terraform plan
```

```text
Plan: 0 to add, 0 to change, 0 to destroy.
```

```bash
terraform apply
rm generated.tf generated2.tf 2>/dev/null; true
```

Delete the `moved` blocks afterwards.

### Part G: Drift, both responses

Change something outside Terraform:

```bash
aws logs put-retention-policy \
  --log-group-name /notely/manual-test \
  --retention-in-days 14
```

```bash
terraform plan
```

Terraform notices and wants to revert.

**Response A:**

```bash
terraform apply
```

**Response B:** recreate the drift, then:

```bash
aws logs put-retention-policy \
  --log-group-name /notely/manual-test \
  --retention-in-days 14

terraform plan -refresh-only
terraform apply -refresh-only
terraform plan
```

The last plan still shows a difference — state now says 14, your code says
otherwise. Update the code:

```hcl
resource "aws_cloudwatch_log_group" "manual" {
  name              = "/notely/manual-test"
  retention_in_days = 14
}
```

```bash
terraform plan     # No changes.
```

### Part H: Release with `removed`

```hcl
removed {
  from = aws_sns_topic.alerts

  lifecycle {
    destroy = false
  }
}
```

Comment out the `aws_sns_topic` resource.

```bash
terraform plan
terraform apply
aws sns list-topics --query 'Topics[?contains(TopicArn, `notely-manual-alerts`)]'
```

Still there. Terraform forgot it without deleting it.

### Part I: Tear down

```bash
terraform destroy
```

That handles the bucket and log group. The SNS topic is no longer managed, so:

```bash
aws sns delete-topic \
  --topic-arn $(aws sns list-topics \
    --query 'Topics[?contains(TopicArn, `notely-manual-alerts`)].TopicArn' \
    --output text)
```

```bash
cd ~ && rm -rf ~/terraform-labs/import-practice

aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

### What you should have at the end

- Three resources imported, with three different ID formats
- Configuration generated, then improved
- Security standards applied to adopted infrastructure
- A rename that would have destroyed three resources, made safe
- Drift resolved both ways
- A resource released from management without being deleted

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **Drift** | Reality no longer matching state |
| **Detecting it** | Every plan does. `-detailed-exitcode` returns 2. |
| **Response A** | `terraform apply` — revert it |
| **Response B** | `apply -refresh-only` — accept it, then fix the code |
| **Response C** | `ignore_changes` — stop tracking it |
| **`terraform refresh`** | Deprecated. Use `-refresh-only`. |
| **`import` block** | Declarative, planned, reviewable. Terraform 1.5+. |
| **`terraform import`** | The old command. No preview. |
| **`-generate-config-out`** | Writes the resource block for you |
| **Generated config** | Always needs editing. A draft, not code. |
| **`1 to import, 0 to destroy`** | What a correct import plan looks like |
| **Resource IDs** | Different per type. Check the provider docs. |
| **`moved`** | Address changed, resource did not |
| **`moved` vs `state mv`** | The block travels with the code |
| **`removed`** | Stop managing without destroying |
| **`lifecycle { destroy = false }`** | Essential in a `removed` block |
| **The empty-plan invariant** | Code-only changes produce an empty plan |
| **Check the plan is empty first** | Or you cannot tell your change from existing drift |
| **Import in small batches** | Ten resources per PR at most |
| **Bulk tools** | Good for inventory, not for production code |

---

## Checkpoint (answer briefly)

1. Why use an `import` block rather than `terraform import`?
2. Your import plan says `1 to import, 0 to add, 1 to destroy`. What do you do?
3. What is the difference between `terraform apply` and `terraform apply -refresh-only` when handling drift?
4. Why is a `moved` block better than `terraform state mv`?
5. What does `removed` do without `lifecycle { destroy = false }`?
6. What is the empty-plan invariant, and why check the plan before you start?
7. Why is generated configuration a starting point rather than an answer?

---

## Checkpoint — model answers

### 1. `import` block vs `terraform import`

**Because the block shows you a plan first, and the command does not.**

`terraform import aws_s3_bucket.x my-bucket` modifies state immediately. There is
no preview and no confirmation. If you got the address wrong or the ID wrong, you
find out afterwards, and undoing it means `terraform state rm`.

An `import` block is part of your configuration, so it goes through the normal
plan-and-review cycle:

```text
Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

You read that before anything happens.

Three further advantages:

**It is reviewable.** The import appears in a pull request diff alongside the
resource block it creates. A colleague can check both.

**It can import many at once**, including with `for_each`. The command does one
at a time.

**It can generate the configuration** with `-generate-config-out`, which the
command cannot do at all.

The command still works and appears throughout older documentation, but for
adopting production infrastructure the preview is not optional.

### 2. An import plan showing a destroy

**Stop. Do not apply.**

A correct import plan destroys nothing:

```text
Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

A destroy means Terraform has matched the resource to your configuration and
concluded that an attribute cannot be changed in place — so it intends to delete
the real resource and build a replacement. On a database or a bucket with
objects, that is data loss.

**What to do:**

1. Find the `# forces replacement` comment in the plan. It names the attribute.
2. Compare that attribute's value in your configuration with reality:
   ```bash
   terraform state show <address>
   aws <service> describe-... --output json
   ```
3. Correct the configuration to match what actually exists.
4. Re-plan until it is `0 to destroy`.

The most common cause is exactly the near-miss in this module's real-world
example: an attribute that reads back differently than it writes, so the
generated configuration says `null` and Terraform reads that as a change. Setting
it explicitly fixes it.

The general rule: **when importing, the configuration must be bent to match
reality, never the other way round.**

### 3. `apply` vs `apply -refresh-only` for drift

They resolve the disagreement in **opposite directions**.

**`terraform apply`** makes **reality match your code**. Someone changed an
instance type in the console; your configuration says `t3.micro`; apply changes
it back. This is the default and usually correct — Terraform's whole job is to
make the world match the code.

**`terraform apply -refresh-only`** makes **state match reality**, and changes no
infrastructure at all. It records what is actually there.

Use `-refresh-only` when the out-of-band change was *right* and your code is
stale. Perhaps someone scaled up during an incident and the larger size should
stay.

The important detail: `-refresh-only` alone does not finish the job. After it,
state says `t3.small` and your code still says `t3.micro`, so the next plan wants
to change it back. You must also update the code:

```bash
terraform apply -refresh-only   # 1. accept reality into state
# 2. edit the code to match
terraform plan                  # 3. No changes.
```

Both paths end at an empty plan. That is how you know the disagreement is
resolved.

### 4. `moved` block vs `state mv`

**Because the block travels with the code, and the command does not.**

`terraform state mv` fixes *your* state file, on *your* machine, right now. Your
plan is clean. Then you push the rename.

Your colleague pulls, runs `terraform plan`, and sees:

```text
Plan: 1 to add, 0 to change, 1 to destroy.
```

Their state still has the old address. So does CI's. Everyone must run the same
`state mv` by hand, in the right order, or they will destroy and recreate the
resource.

A `moved` block is part of the configuration:

```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.app
}
```

It is committed, it is reviewed in the pull request, and it is applied
automatically by anyone who runs `terraform apply` — including the pipeline. The
fix is distributed with the change that needed it.

Three smaller advantages: it shows a plan first, it is visible in the diff so a
reviewer knows a rename happened, and it is idempotent — Terraform ignores a
`moved` block whose `from` address is not in state, so it is safe to leave in for
a release or two.

### 5. `removed` without `destroy = false`

**It destroys the resource.**

```hcl
removed {
  from = aws_s3_bucket.legacy
}
```

That means "remove this from my configuration and delete it" — which is the
normal meaning of deleting a resource block.

```hcl
removed {
  from = aws_s3_bucket.legacy

  lifecycle {
    destroy = false
  }
}
```

This means "remove this from state but leave the real thing alone".

The difference in the plan is unmissable if you read it:

```text
# without destroy = false
Plan: 0 to add, 0 to change, 1 to destroy.

# with destroy = false
# ... will no longer be managed by Terraform, but will not be destroyed.
Plan: 0 to add, 0 to change, 0 to destroy.
```

Since the whole point of `removed` is usually to *keep* the resource — handing it
to another team, splitting a state file, undoing an import — omitting the
lifecycle block does the opposite of what you intended.

It is the same distinction as `git rm` versus `git rm --cached`.

### 6. The empty-plan invariant

**Any change that only moves code should produce
`Plan: 0 to add, 0 to change, 0 to destroy.`**

Renaming a resource, moving it between files, extracting a module, restructuring
directories, converting `count` to `for_each`, importing something that already
exists — none of these change infrastructure. They change how Terraform *refers*
to it.

So an empty plan is your proof the refactor was safe. A non-empty plan is a bug
report.

**Why you check before starting:** you need a baseline.

If you run a plan after your changes and see `1 to change`, you cannot tell
whether you broke something or whether that difference was already there —
somebody edited the console last week and nobody noticed.

```bash
terraform plan     # must be empty BEFORE you touch anything
# ... make the change ...
terraform plan     # must be empty AFTER
```

With a clean baseline, any difference is unambiguously yours. Without one, you
are debugging two problems at once.

This one habit prevents most refactoring accidents.

### 7. Why generated configuration is a draft

`terraform plan -generate-config-out=generated.tf` writes a resource block that
matches reality exactly. That saves real transcription effort. But it is not code
you would have written, for four reasons.

**It hardcodes everything.** Every ID appears as a literal:

```hcl
vpc_id    = "vpc-0123456789abcdef0"
subnet_id = "subnet-0abc123"
```

Those should be `aws_vpc.main.id` and `aws_subnet.this["app-a"].id`. Without real
references, Terraform has no dependency graph, so it may destroy things in the
wrong order and your configuration is not portable to another environment.

**It includes attributes you should not set.** Read-only and computed attributes
appear in the output and cause errors or spurious diffs. They have to be removed.

**It has no structure.** No variables, no locals, no modules, no tags from your
standard set. Everything is a literal in one flat file.

**It ignores your conventions.** Resource names come from AWS, not from your
naming scheme.

There is also the failure mode from the real-world example: attributes that read
back differently than they write. `db_name` came out as `null`, which read as a
change that would have replaced a production database.

So the workflow is: generate it to save typing, then edit it into real code, then
verify with an empty plan. The generation is the draft; **the plan is the
review.**

---

## Next lesson

**Module 14 — The Complete Build** (`14-real-world-project.md`)

This is the capstone. Everything from Modules 0 through 13, assembled: the full
Notely architecture with Route 53, CloudWatch alarms and SNS notifications, built
from modules, deployed across three environments, tested, scanned, and shipped
through a pipeline.

It also covers the things that only matter once a system is real — repository
layout for a team, tagging standards, code review checklists for Terraform, cost
awareness, upgrade strategy, and the runbooks for when something goes wrong at
3am.
