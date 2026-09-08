# Module 6 — Meta-Arguments

## Where Notely is right now

```text
  Notely so far:

    S3 bucket (attachments) + versioning
    CloudWatch log group
    VPC + ONE public subnet + internet gateway + route table
    Three security groups (alb -> app -> db)
    Computed CIDRs, user-data template ready
```

By the end of this module:

```text
    VPC
      +- public subnet   AZ a  +  public subnet   AZ b    <- new
      +- app subnet      AZ a  +  app subnet      AZ b    <- new
      +- data subnet     AZ a  +  data subnet     AZ b    <- new
    Application Load Balancer across both public subnets   <- new
    Two EC2 servers running the Node.js app                <- new
```

This is the biggest single jump in the course. Notely goes from a network
diagram to a working website.

Full picture: `notely-architecture.md`.

---

## Why this module exists

Notely needs six subnets. You know exactly what they look like — Module 5
computed all their CIDR blocks. But right now you would have to write six
near-identical `resource` blocks:

```hcl
resource "aws_subnet" "public_a"  { cidr_block = "10.0.1.0/24"  ... }
resource "aws_subnet" "public_b"  { cidr_block = "10.0.2.0/24"  ... }
resource "aws_subnet" "app_a"     { cidr_block = "10.0.11.0/24" ... }
resource "aws_subnet" "app_b"     { cidr_block = "10.0.12.0/24" ... }
resource "aws_subnet" "data_a"    { cidr_block = "10.0.21.0/24" ... }
resource "aws_subnet" "data_b"    { cidr_block = "10.0.22.0/24" ... }
```

Six blocks that differ in two values. Add a third AZ and it becomes nine.

Meta-arguments fix this. And one of them — `count` — will quietly ruin your day
if you pick it here, which is most of what this module is about.

---

## The core idea (one sentence)

Meta-arguments are settings you can put on *any* resource, and the two most
important ones create many copies of that resource from a collection.

> Choosing between `count` and `for_each` looks like a style decision. It is not. Get it wrong and removing one item destroys everything after it.

---

## Mental model (from JavaScript)

`count` and `for_each` are two ways to loop, and they behave like two different
JavaScript data structures.

```javascript
// count -> an array. Things are identified by POSITION.
const servers = ["web-0", "web-1", "web-2"];
servers.splice(1, 1);   // remove the middle one
// ["web-0", "web-2"]  -> what WAS index 2 is now index 1. Everything shifted.

// for_each -> an object. Things are identified by NAME.
const servers = { a: "web-a", b: "web-b", c: "web-c" };
delete servers.b;
// { a: "web-a", c: "web-c" }  -> a and c are untouched. Nothing shifted.
```

That is the entire difference, and it is why `for_each` is almost always right.

| | `count` | `for_each` |
|---|---|---|
| Takes | a number | a map or a set |
| Identified by | position: `[0]`, `[1]` | key: `["a"]`, `["b"]` |
| JavaScript analogue | array | object |
| Removing a middle item | **shifts everything after it** | affects only that item |
| Address looks like | `aws_subnet.this[0]` | `aws_subnet.this["public-a"]` |

---

## What a meta-argument is

Normal arguments come from the provider. `cidr_block` exists because the AWS
provider says `aws_subnet` has one.

**Meta-arguments come from Terraform itself**, and work on every resource, from
every provider.

There are six:

| Meta-argument | What it does |
|---|---|
| `count` | Create N copies |
| `for_each` | Create one copy per item in a map or set |
| `depends_on` | Force an ordering Terraform cannot infer |
| `lifecycle` | Change how create/update/destroy behaves |
| `provider` | Use a non-default provider configuration |
| `provisioner` | Run a script (avoid — see the end of this module) |

---

## Part 1 — `count`

### The basics

```hcl
resource "aws_instance" "app" {
  count = 3

  ami           = "ami-0abc123"
  instance_type = "t3.micro"

  tags = {
    Name = "notely-app-${count.index}"
  }
}
```

Three instances, named `notely-app-0`, `notely-app-1`, `notely-app-2`.

Inside the block, `count.index` is 0, 1, 2.

The addresses become a list:

```text
aws_instance.app[0]
aws_instance.app[1]
aws_instance.app[2]
```

And referencing them:

```hcl
aws_instance.app[0].id       # one of them
aws_instance.app[*].id       # all of them, as a list
```

### `count` as an on/off switch

This is `count`'s genuinely best use:

```hcl
resource "aws_nat_gateway" "main" {
  count = var.enable_nat_gateway ? 1 : 0

  subnet_id     = aws_subnet.public["public-a"].id
  allocation_id = aws_eip.nat[0].id
}
```

`count = 0` creates nothing. `count = 1` creates one. It is the standard way to
make a resource conditional.

Referencing a conditional resource needs care:

```hcl
# If count might be 0, this errors when it is
aws_nat_gateway.main[0].id

# Safer
one(aws_nat_gateway.main[*].id)      # returns null if empty
try(aws_nat_gateway.main[0].id, null)
```

### The index-shifting problem

Here is the thing you must see for yourself.

```hcl
variable "subnet_cidrs" {
  default = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

resource "aws_subnet" "this" {
  count      = length(var.subnet_cidrs)
  cidr_block = var.subnet_cidrs[count.index]
  vpc_id     = aws_vpc.main.id
}
```

Apply that. State now holds:

```text
aws_subnet.this[0]  ->  10.0.1.0/24
aws_subnet.this[1]  ->  10.0.2.0/24
aws_subnet.this[2]  ->  10.0.3.0/24
```

Now remove the **middle** one:

```hcl
default = ["10.0.1.0/24", "10.0.3.0/24"]
```

What you want: subnet 2 is deleted, the others are untouched.

What you get:

```text
  ~ aws_subnet.this[1] must be replaced
      ~ cidr_block = "10.0.2.0/24" -> "10.0.3.0/24"  # forces replacement

  - aws_subnet.this[2] will be destroyed

Plan: 1 to add, 0 to change, 2 to destroy.
```

Terraform matched by position:

| Address | Was | Now | Result |
|---|---|---|---|
| `[0]` | `10.0.1.0/24` | `10.0.1.0/24` | unchanged |
| `[1]` | `10.0.2.0/24` | `10.0.3.0/24` | **destroyed and recreated** |
| `[2]` | `10.0.3.0/24` | *(gone)* | **destroyed** |

You wanted to delete one subnet. You deleted two and rebuilt one.

On subnets that is annoying. On EC2 instances it is downtime. On an RDS instance
it is **data loss**.

> `count` matches resources by position. Removing anything but the last item shifts every index after it, and Terraform reads a shifted index as "this resource changed".

### Now the same thing with `for_each`

```hcl
variable "subnets" {
  default = {
    "a" = "10.0.1.0/24"
    "b" = "10.0.2.0/24"
    "c" = "10.0.3.0/24"
  }
}

resource "aws_subnet" "this" {
  for_each   = var.subnets
  cidr_block = each.value
  vpc_id     = aws_vpc.main.id
}
```

State holds:

```text
aws_subnet.this["a"]  ->  10.0.1.0/24
aws_subnet.this["b"]  ->  10.0.2.0/24
aws_subnet.this["c"]  ->  10.0.3.0/24
```

Remove `"b"`:

```text
  - aws_subnet.this["b"] will be destroyed

Plan: 0 to add, 0 to change, 1 to destroy.
```

Exactly what you asked for. `"a"` and `"c"` were never touched, because their
keys did not change.

**This is the whole argument for `for_each`.**

---

## Part 2 — `for_each`

### The basics

`for_each` takes a **map** or a **set of strings**. Never a list.

```hcl
# With a map
resource "aws_subnet" "this" {
  for_each = {
    public-a = "10.0.1.0/24"
    public-b = "10.0.2.0/24"
  }

  cidr_block = each.value
  vpc_id     = aws_vpc.main.id

  tags = { Name = each.key }
}

# With a set
resource "aws_iam_user" "team" {
  for_each = toset(["rahman", "alice", "bob"])
  name     = each.value
}
```

Inside the block you get two values:

| | With a map | With a set |
|---|---|---|
| `each.key` | the map key | the item itself |
| `each.value` | the map value | the item itself |

### Maps of objects — the pattern you will actually use

```hcl
locals {
  subnets = {
    "public-a" = { cidr = "10.0.1.0/24",  az = "ap-southeast-2a", public = true  }
    "public-b" = { cidr = "10.0.2.0/24",  az = "ap-southeast-2b", public = true  }
    "app-a"    = { cidr = "10.0.11.0/24", az = "ap-southeast-2a", public = false }
    "app-b"    = { cidr = "10.0.12.0/24", az = "ap-southeast-2b", public = false }
  }
}

resource "aws_subnet" "this" {
  for_each = local.subnets

  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr
  availability_zone       = each.value.az
  map_public_ip_on_launch = each.value.public

  tags = {
    Name = "${local.name_prefix}-${each.key}"
  }
}
```

Four subnets, one resource block. Add a fifth by adding one line to the map.

### The "known at plan time" rule

This one confuses everyone once:

```text
Error: Invalid for_each argument

The "for_each" map includes keys derived from resource attributes that cannot
be determined until apply.
```

**`for_each` keys must be known during `plan`.** Terraform needs to know the
resource addresses before it creates anything.

```hcl
# BROKEN - the bucket IDs don't exist until apply
resource "aws_s3_bucket_versioning" "this" {
  for_each = toset(aws_s3_bucket.data[*].id)
  ...
}

# FIXED - use the same keys the buckets were created from
resource "aws_s3_bucket_versioning" "this" {
  for_each = aws_s3_bucket.data
  bucket   = each.value.id
  ...
}
```

The rule of thumb: **keys must come from variables, locals, or data sources —
not from attributes of resources that do not exist yet.**

Values are fine to be unknown. Only the keys must be known.

### Referencing `for_each` resources

```hcl
aws_subnet.this["public-a"].id           # one specific one
values(aws_subnet.this)[*].id            # all IDs as a list
[for k, v in aws_subnet.this : v.id]     # same thing, explicit
keys(aws_subnet.this)                    # all the keys

# Filter to just the public ones
[for k, v in aws_subnet.this : v.id if local.subnets[k].public]
```

Note `aws_subnet.this[*].id` does **not** work with `for_each` — splat is for
lists. Use `values()` first.

### Converting `count` to `for_each` safely

If you already have `count` resources in state, switching to `for_each` naively
destroys and recreates everything. Use `moved` blocks:

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

Run `terraform plan` and you should see `Plan: 0 to add, 0 to change, 0 to
destroy` — Terraform just relabels them in state. Module 13 covers `moved`
properly.

### The decision rule

> Use `for_each`. Use `count` only when the answer is genuinely "how many", and that number is 0 or 1.

| Situation | Use |
|---|---|
| A set of named things (subnets, users, buckets) | `for_each` |
| Turning one resource on or off | `count = x ? 1 : 0` |
| N identical, interchangeable things where order truly does not matter | `count` (rare) |
| Anything you might remove one item from later | `for_each` |

---

## Part 3 — `depends_on`

### When you need it

Terraform works out ordering from references (Module 2). Sometimes a real
dependency exists with no value flowing between the resources.

The classic case:

```hcl
resource "aws_iam_role_policy_attachment" "app_s3" {
  role       = aws_iam_role.app.name
  policy_arn = aws_iam_policy.s3_access.arn
}

resource "aws_instance" "app" {
  ami                  = var.ami_id
  iam_instance_profile = aws_iam_instance_profile.app.name

  # The instance references the PROFILE, but not the POLICY ATTACHMENT.
  # Without this, the server can boot before it has S3 permissions,
  # and the app fails on its first upload.
  depends_on = [aws_iam_role_policy_attachment.app_s3]
}
```

### When you do NOT need it

```hcl
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id

  depends_on = [aws_vpc.main]     # pointless - the reference already did this
}
```

The reference to `aws_vpc.main.id` already created the dependency.

> If you can restructure to reference an attribute instead, do that. A real reference is self-documenting and cannot go stale.

`depends_on` also has a cost: it is coarse. It makes Terraform treat the whole
resource as dependent, which can reduce parallelism and force unnecessary
replacements downstream.

---

## Part 4 — `lifecycle`

Five settings that change how Terraform handles a resource.

### `create_before_destroy`

By default, replacing a resource means **destroy then create** — which is
downtime.

```hcl
resource "aws_instance" "app" {
  # ...

  lifecycle {
    create_before_destroy = true
  }
}
```

Now it creates the new one first, then destroys the old. The plan symbol changes
from `-/+` to `+/-`.

Essential for anything serving traffic. Note that names must be unique, so
resources with a fixed `name` may need `name_prefix` instead.

### `prevent_destroy`

```hcl
resource "aws_db_instance" "notely" {
  # ...

  lifecycle {
    prevent_destroy = true
  }
}
```

Any plan that would destroy this resource **fails**:

```text
Error: Instance cannot be destroyed

Resource aws_db_instance.notely has lifecycle.prevent_destroy set, but the plan
calls for this resource to be destroyed.
```

Put this on production databases and state buckets. It has saved a lot of jobs.

To actually destroy it, you must remove the setting first — which is a code
change, which is a pull request, which is a review. That is the point.

### `ignore_changes`

```hcl
resource "aws_instance" "app" {
  ami = var.ami_id

  lifecycle {
    ignore_changes = [
      ami,               # a deploy pipeline updates this
      tags["LastDeploy"] # something else manages this tag
    ]
  }
}
```

Terraform stops caring about drift in those attributes.

Legitimate uses: an attribute genuinely managed by another system (a deploy
pipeline, an autoscaling policy, AWS itself).

Illegitimate use: silencing a diff you do not understand. That hides real drift
forever.

```hcl
lifecycle {
  ignore_changes = all      # almost always wrong
}
```

### `replace_triggered_by`

Force a replacement when something else changes:

```hcl
resource "aws_instance" "app" {
  # ...

  lifecycle {
    replace_triggered_by = [
      aws_launch_template.app.latest_version
    ]
  }
}
```

Useful when a dependency changes in a way that does not automatically propagate.

### `precondition` and `postcondition`

Assertions that run during plan and apply:

```hcl
resource "aws_lb" "main" {
  subnets = values(aws_subnet.public)[*].id

  lifecycle {
    precondition {
      condition     = length(values(aws_subnet.public)) >= 2
      error_message = "The load balancer needs at least two subnets in different AZs."
    }
  }
}
```

Tests that live in your configuration. Module 11 goes further.

---

## Part 5 — `provider` and aliases

By default a resource uses the default provider configuration. Aliases let you
have several.

```hcl
provider "aws" {
  region = "ap-southeast-2"       # the default
}

provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"
}
```

Then:

```hcl
# Uses the default (Sydney)
resource "aws_s3_bucket" "attachments" {
  bucket = "notely-attachments"
}

# Uses the alias (Virginia)
resource "aws_acm_certificate" "cdn" {
  provider          = aws.us_east_1
  domain_name       = "notely.example.com"
  validation_method = "DNS"
}
```

The classic real reason: **CloudFront certificates must live in `us-east-1`**,
regardless of where the rest of your infrastructure is.

Aliases also handle multi-account setups:

```hcl
provider "aws" {
  alias  = "prod"
  region = "ap-southeast-2"

  assume_role {
    role_arn = "arn:aws:iam::111122223333:role/TerraformDeploy"
  }
}
```

---

## Part 6 — `provisioner` (and why to avoid it)

Provisioners run scripts as part of creating a resource.

```hcl
resource "aws_instance" "app" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -",
      "sudo yum install -y nodejs",
    ]
  }
}
```

HashiCorp's own documentation calls these a last resort. Here is why:

| Problem | What it means |
|---|---|
| **Not in state** | Terraform has no record of what the script did |
| **Not in the plan** | You cannot see what will happen before it happens |
| **Not idempotent** | Running twice may not be safe |
| **No drift detection** | If someone undoes it, Terraform never notices |
| **Fails badly** | A failed provisioner marks the whole resource tainted |
| **Needs connectivity** | SSH or WinRM access from wherever Terraform runs |

### What to use instead

| Instead of | Use |
|---|---|
| `remote-exec` installing software | **user-data** (what Notely does) or a pre-baked AMI |
| `local-exec` calling an API | A proper provider, or a `terraform_data` resource |
| `file` copying files | S3 plus user-data, or bake it into the image |
| Waiting for something | A `data` source, or the resource's own `timeouts` |

Notely installs Node.js through user-data, generated with `templatefile()` in
Module 5. That is the right pattern: the script is part of the instance
definition, so changing it replaces the instance in a controlled way.

### `terraform_data`

If you genuinely need to run something local, `terraform_data` replaced
`null_resource` in Terraform 1.4:

```hcl
resource "terraform_data" "db_migration" {
  triggers_replace = [aws_db_instance.notely.id]

  provisioner "local-exec" {
    command = "npx prisma migrate deploy"
  }
}
```

Even here, a CI pipeline step is usually better.

> Provisioners are an escape hatch. If you are using one, ask what you would do if Terraform did not have them — that is usually the better answer.

---

## Building it into Notely

Now the big one.

### 1. All six subnets with `for_each`

Replace the single `aws_subnet.public_a` in `network.tf`:

```hcl
locals {
  # Build the full subnet map from the computed CIDRs.
  # This produces six entries: public-a, public-b, app-a, app-b, data-a, data-b
  subnets = merge(
    { for i, az in local.azs : "public-${substr(az, -1, 1)}" => {
      cidr_block        = local.public_subnet_cidrs[i]
      availability_zone = az
      tier              = "public"
      public            = true
    } },
    { for i, az in local.azs : "app-${substr(az, -1, 1)}" => {
      cidr_block        = local.app_subnet_cidrs[i]
      availability_zone = az
      tier              = "app"
      public            = false
    } },
    { for i, az in local.azs : "data-${substr(az, -1, 1)}" => {
      cidr_block        = local.data_subnet_cidrs[i]
      availability_zone = az
      tier              = "data"
      public            = false
    } },
  )

  # Handy filtered views
  public_subnet_ids = [for k, v in aws_subnet.this : v.id if local.subnets[k].public]
  app_subnet_ids    = [for k, v in aws_subnet.this : v.id if local.subnets[k].tier == "app"]
  data_subnet_ids   = [for k, v in aws_subnet.this : v.id if local.subnets[k].tier == "data"]
}

resource "aws_subnet" "this" {
  for_each = local.subnets

  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr_block
  availability_zone       = each.value.availability_zone
  map_public_ip_on_launch = each.value.public

  tags = {
    Name = "${local.name_prefix}-${each.key}"
    Tier = each.value.tier
  }
}
```

Check the map before applying:

```bash
terraform console -var-file=dev.tfvars <<< 'keys(local.subnets)'
```

```text
[
  "app-a",
  "app-b",
  "data-a",
  "data-b",
  "public-a",
  "public-b",
]
```

### 2. Route tables

Public subnets get a route to the internet gateway. Private ones do not.

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = { Name = "${local.name_prefix}-public-rt" }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  # No 0.0.0.0/0 route. This is what makes these subnets private.
  # The automatic "local" route for the VPC CIDR is still there.

  tags = { Name = "${local.name_prefix}-private-rt" }
}

# Associate each subnet with the right table, using for_each again.
resource "aws_route_table_association" "this" {
  for_each = local.subnets

  subnet_id = aws_subnet.this[each.key].id
  route_table_id = each.value.public ? aws_route_table.public.id : aws_route_table.private.id
}
```

One `for_each` handles all six associations, with a ternary picking the table.

### 3. The S3 gateway endpoint (free, replaces a NAT gateway)

Notely's app servers are in private subnets and need to reach S3. A NAT gateway
would cost $32/month. A gateway endpoint costs nothing.

```hcl
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"

  route_table_ids = [aws_route_table.private.id]

  tags = { Name = "${local.name_prefix}-s3-endpoint" }
}
```

That adds a route to the private route table sending S3 traffic down a private
path. The servers can now read and write attachments without any internet
access at all.

### 4. IAM role for the servers

```hcl
resource "aws_iam_role" "app" {
  name = "${local.name_prefix}-app"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Action    = "sts:AssumeRole"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy" "app_s3" {
  name = "${local.name_prefix}-s3-access"
  role = aws_iam_role.app.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"]
        Resource = "${aws_s3_bucket.attachments.arn}/*"
      },
      {
        Effect   = "Allow"
        Action   = ["s3:ListBucket"]
        Resource = aws_s3_bucket.attachments.arn
      },
      {
        Effect   = "Allow"
        Action   = ["logs:CreateLogStream", "logs:PutLogEvents"]
        Resource = "${aws_cloudwatch_log_group.app.arn}:*"
      },
    ]
  })
}

resource "aws_iam_instance_profile" "app" {
  name = "${local.name_prefix}-app"
  role = aws_iam_role.app.name
}
```

IAM is free.

### 5. The EC2 servers

```hcl
variable "app_instance_type" {
  description = "EC2 instance size for the app servers"
  type        = string
  default     = "t3.micro"
}

variable "app_ami_id" {
  description = "AMI for the app servers. Module 7 looks this up automatically."
  type        = string
  # Amazon Linux 2023 in ap-southeast-2. Yours may differ - see Module 7.
  default = "ami-0892a9c01908fafd1"
}

resource "aws_instance" "app" {
  for_each = { for k, v in local.subnets : k => v if v.tier == "app" }

  ami                    = var.app_ami_id
  instance_type          = var.app_instance_type
  subnet_id              = aws_subnet.this[each.key].id
  vpc_security_group_ids = [aws_security_group.app.id]
  iam_instance_profile   = aws_iam_instance_profile.app.name

  user_data                   = local.user_data
  user_data_replace_on_change = true

  tags = {
    Name = "${local.name_prefix}-app-${substr(each.key, -1, 1)}"
  }

  lifecycle {
    create_before_destroy = true
  }

  depends_on = [aws_iam_role_policy.app_s3]
}
```

Two instances, one per app subnet, from one block.

`user_data_replace_on_change = true` means editing the startup script replaces
the instances — otherwise the change would sit in state doing nothing.

The `depends_on` is a genuine one: the instance references the *profile*, not the
*policy*, so without it the server can boot before it has S3 permissions.

### 6. The load balancer

**Cost warning: an ALB costs about $0.022/hour — roughly $16/month, or about
$0.53 if you leave it up for a day.** This is the first thing in Notely that
costs real money.

```hcl
resource "aws_lb" "main" {
  name               = "${local.name_prefix}-alb"
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = local.public_subnet_ids

  tags = { Name = "${local.name_prefix}-alb" }

  lifecycle {
    precondition {
      condition     = length(local.public_subnet_ids) >= 2
      error_message = "An ALB needs at least two subnets in different availability zones."
    }
  }
}

resource "aws_lb_target_group" "app" {
  name     = "${local.name_prefix}-app"
  port     = var.app_port
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/health"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 3
    matcher             = "200"
  }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_lb_target_group_attachment" "app" {
  for_each = aws_instance.app

  target_group_arn = aws_lb_target_group.app.arn
  target_id        = each.value.id
  port             = var.app_port
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

Note `aws_lb_target_group_attachment` uses `for_each = aws_instance.app` — it
iterates the instances directly, reusing their keys. That is the "keys must be
known at plan time" rule satisfied properly.

Add an output:

```hcl
output "notely_url" {
  description = "Public URL for Notely"
  value       = "http://${aws_lb.main.dns_name}"
}
```

---

## Real-World Example

A team ran 12 microservices, each with an ECS service, a target group, a log
group and an alarm. Originally 12 copies of four resources — about 600 lines.

They rewrote it as one map:

```hcl
locals {
  services = {
    "api"       = { port = 3000, cpu = 512,  memory = 1024, min = 2, max = 10 }
    "worker"    = { port = 3001, cpu = 1024, memory = 2048, min = 1, max = 5  }
    "scheduler" = { port = 3002, cpu = 256,  memory = 512,  min = 1, max = 1  }
    # ... nine more
  }
}

resource "aws_ecs_service" "this" {
  for_each = local.services
  # ...
}

resource "aws_lb_target_group" "this" {
  for_each = local.services
  # ...
}

resource "aws_cloudwatch_log_group" "this" {
  for_each = local.services
  # ...
}
```

600 lines became about 120, and adding a service became a five-line map entry
reviewed in a two-minute PR.

**The bit that mattered most.** Six months in, they decommissioned the
`scheduler` service. They deleted its map entry and the plan said:

```text
Plan: 0 to add, 0 to change, 4 to destroy.
```

Four resources, all belonging to `scheduler`. Nothing else moved.

Had they used `count` with a list, `scheduler` sitting in the middle would have
shifted every service after it, and the plan would have proposed replacing eight
production ECS services. Someone would have caught it — probably — but that is
not a thing you want to depend on.

---

## Common Mistakes Beginners Make

**1. Using `count` for a list of named things.**

The index-shifting problem. Use `for_each`.

**2. Passing a list to `for_each`.**

```hcl
for_each = ["a", "b"]     # ERROR
for_each = toset(["a", "b"])   # correct
```

`for_each` needs a map or a set.

**3. `for_each` keys that are not known at plan time.**

Keys cannot come from attributes of resources that do not exist yet. Values can.

**4. Using splat on a `for_each` resource.**

```hcl
aws_subnet.this[*].id          # doesn't work - it's a map
values(aws_subnet.this)[*].id  # correct
```

**5. `depends_on` when a reference would do.**

If you can reference an attribute, do that instead.

**6. `ignore_changes = all` to silence a confusing diff.**

You have now hidden all future drift on that resource.

**7. Forgetting `create_before_destroy` on things serving traffic.**

Default behaviour is destroy-then-create. That is downtime.

**8. Forgetting `user_data_replace_on_change`.**

Editing the startup script does nothing to running instances without it.

**9. Reaching for a provisioner.**

Use user-data or a baked image.

**10. Not putting `prevent_destroy` on the database.**

It costs one line and prevents the worst possible afternoon.

---

## Hands-On Lab — Notely Grows Up

### Part A: See the index-shifting problem (free, no AWS)

Before touching Notely, prove the `count` problem to yourself.

```bash
mkdir -p ~/terraform-labs/count-vs-foreach
cd ~/terraform-labs/count-vs-foreach
```

`main.tf`:

```hcl
terraform {
  required_providers {
    local = { source = "hashicorp/local", version = "~> 2.4" }
  }
}

variable "names" {
  type    = list(string)
  default = ["alpha", "bravo", "charlie"]
}

resource "local_file" "with_count" {
  count    = length(var.names)
  filename = "${path.module}/out/count-${var.names[count.index]}.txt"
  content  = var.names[count.index]
}

resource "local_file" "with_foreach" {
  for_each = toset(var.names)
  filename = "${path.module}/out/foreach-${each.value}.txt"
  content  = each.value
}
```

```bash
terraform init
terraform apply
terraform state list
```

```text
local_file.with_count[0]
local_file.with_count[1]
local_file.with_count[2]
local_file.with_foreach["alpha"]
local_file.with_foreach["bravo"]
local_file.with_foreach["charlie"]
```

Now remove the **middle** name:

```hcl
default = ["alpha", "charlie"]
```

```bash
terraform plan
```

Read it carefully:

```text
  # local_file.with_count[1] must be replaced
  # local_file.with_count[2] will be destroyed
  # local_file.with_foreach["bravo"] will be destroyed

Plan: 1 to add, 0 to change, 3 to destroy.
```

`count` destroyed two and rebuilt one. `for_each` destroyed exactly one.

**That is the lesson of this module.** Everything else is detail.

```bash
terraform destroy
```

### Part B: Six subnets in Notely (free)

```bash
cd ~/terraform-labs/notely
```

Update `locals.tf` and `network.tf` as shown in "Building it into Notely" steps
1–3.

Because you are replacing `aws_subnet.public_a` with `aws_subnet.this`, add a
`moved` block so the existing subnet is kept:

```hcl
moved {
  from = aws_subnet.public_a
  to   = aws_subnet.this["public-a"]
}

moved {
  from = aws_route_table_association.public_a
  to   = aws_route_table_association.this["public-a"]
}
```

```bash
terraform plan -var-file=dev.tfvars
```

You should see the existing subnet being *moved*, not destroyed, and five new
subnets being added.

```bash
terraform apply -var-file=dev.tfvars
```

Check them:

```bash
aws ec2 describe-subnets \
  --filters "Name=tag:Project,Values=notely" \
  --query 'Subnets[].{Name:Tags[?Key==`Name`]|[0].Value,Cidr:CidrBlock,AZ:AvailabilityZone}' \
  --output table
```

Six subnets, two AZs, three tiers. All free.

Delete the `moved` blocks once the apply succeeds — they have done their job.

### Part C: IAM, endpoint, and the servers

**Cost: EC2 `t3.micro` is free tier for 12 months. Outside free tier, about
$0.01/hour each — roughly $0.50 for a day with two running.**

Add the S3 endpoint, IAM role and `aws_instance.app` blocks from steps 3–5.

```bash
terraform plan -var-file=dev.tfvars
```

**If you would rather not spend anything, stop here.** Reading this plan teaches
you most of what the apply would. You can see exactly what would be created.

To continue:

```bash
terraform apply -var-file=dev.tfvars
```

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=notely" "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].{Name:Tags[?Key==`Name`]|[0].Value,AZ:Placement.AvailabilityZone,IP:PrivateIpAddress}' \
  --output table
```

Two instances, one per AZ, both with private IPs and no public IP — because they
are in private subnets.

### Part D: The load balancer

**Cost: about $0.022/hour. Roughly $0.53 for a full day. Destroy it when you are
done.**

Add the ALB, target group, attachments and listener from step 6.

```bash
terraform apply -var-file=dev.tfvars
terraform output notely_url
```

Wait two or three minutes for the health checks to pass, then:

```bash
curl $(terraform output -raw notely_url)
```

```json
{"app":"notely","env":"dev","host":"ip-10-0-11-42.ap-southeast-2.compute.internal"}
```

Run it several times. The `host` changes as the load balancer alternates between
your two servers.

**That is Notely working.** A request came from the internet, hit the load
balancer in a public subnet, was forwarded to a Node.js process on a server in a
private subnet, and came back. Every hop is something you built.

Check the target health:

```bash
aws elbv2 describe-target-health \
  --target-group-arn $(terraform output -raw target_group_arn 2>/dev/null || \
    aws elbv2 describe-target-groups --names notely-dev-app \
      --query 'TargetGroups[0].TargetGroupArn' --output text) \
  --query 'TargetHealthDescriptions[].{Target:Target.Id,Health:TargetHealth.State}' \
  --output table
```

Both should say `healthy`.

### Part E: Prove `for_each` behaves

Remove one AZ:

```bash
terraform plan -var-file=dev.tfvars -var 'region=ap-southeast-2' \
  -var-file=dev.tfvars
```

Instead, edit `local.azs` in `locals.tf` to one AZ and plan:

```text
Plan: 0 to add, 0 to change, 4 to destroy.
```

Exactly the `-b` resources. Nothing in AZ a is touched. Revert the change.

### Part F: Tear down

**Do this.** The ALB is billing by the hour.

```bash
terraform destroy -var-file=dev.tfvars
```

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

Empty output means you are clean.

### What you should have at the end

- Direct experience of `count` destroying more than you asked it to
- Notely with six subnets across two AZs from one resource block
- Two servers running Node.js, reachable only through the load balancer
- A working URL that returned JSON from a server you never logged into
- Everything destroyed

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **Meta-argument** | A setting Terraform provides on every resource |
| **`count`** | Create N copies. Identified by position. |
| **`count.index`** | 0, 1, 2 ... inside the block |
| **`count = x ? 1 : 0`** | The on/off switch. `count`'s best use. |
| **Index shifting** | Removing a middle item shifts everything after it |
| **`for_each`** | Create one per item. Identified by key. |
| **`each.key` / `each.value`** | The current item |
| **`for_each` needs** | A map or a set. Never a list. |
| **Keys known at plan time** | `for_each` keys cannot come from unknown attributes |
| **Splat on `for_each`** | Doesn't work — use `values(x)[*].id` |
| **The rule** | Use `for_each`. Use `count` only for 0-or-1. |
| **`moved`** | Convert `count` to `for_each` without destroying anything |
| **`depends_on`** | Explicit ordering when no value flows |
| **`create_before_destroy`** | Build the new one first. No downtime. |
| **`prevent_destroy`** | Any plan that would delete this fails |
| **`ignore_changes`** | Stop tracking drift on an attribute |
| **`replace_triggered_by`** | Replace when something else changes |
| **`precondition`** | An assertion checked at plan time |
| **`provider = aws.alias`** | Use a different region or account |
| **`provisioner`** | Last resort. Use user-data or a baked image. |
| **`terraform_data`** | Replaced `null_resource` in Terraform 1.4 |

---

## Checkpoint (answer briefly)

1. You have five subnets created with `count` from a list. You delete the second one. What happens, and why?
2. Why does `for_each` require a map or a set rather than a list?
3. What does "for_each keys must be known at plan time" mean, and what breaks if they are not?
4. When is `count` genuinely the right choice?
5. What does `create_before_destroy` change, and why does Notely's EC2 block use it?
6. Notely's `aws_instance.app` has `depends_on = [aws_iam_role_policy.app_s3]`. Why can't Terraform work that out on its own?
7. Why does Notely use an S3 gateway endpoint instead of a NAT gateway?

---

## Checkpoint — model answers

### 1. Deleting the second of five `count` subnets

Terraform destroys **four** resources and creates **three**.

The reason is that `count` identifies resources by position in state:

| Address | Before | After | Result |
|---|---|---|---|
| `[0]` | subnet 1 | subnet 1 | unchanged |
| `[1]` | subnet 2 | subnet 3 | **replaced** |
| `[2]` | subnet 3 | subnet 4 | **replaced** |
| `[3]` | subnet 4 | subnet 5 | **replaced** |
| `[4]` | subnet 5 | *(gone)* | **destroyed** |

Removing the item at index 1 shifted every later item down by one. Terraform
compares `aws_subnet.this[1]` in state (CIDR `10.0.2.0/24`) with
`aws_subnet.this[1]` in config (now CIDR `10.0.3.0/24`), sees a changed
`cidr_block`, and since CIDR cannot be changed in place, replaces it. Same for
`[2]` and `[3]`. `[4]` no longer exists in config, so it is destroyed.

You asked to delete one thing. Four were destroyed.

With `for_each` keyed by name, deleting one key destroys exactly one resource,
because the other keys are unchanged.

### 2. Why `for_each` needs a map or set

Because it identifies resources by a **stable key**, and lists do not have stable
keys — they have positions, which move.

The whole point of `for_each` over `count` is that removing an item does not
disturb the others. That only works if each item has an identity independent of
its position. A map key is such an identity. A set element is its own identity
(that is why sets work — the string itself becomes the key). A list index is not:
it changes when the list changes.

If Terraform accepted lists, it would silently reintroduce the exact problem
`for_each` exists to solve.

Converting is easy:

```hcl
for_each = toset(var.names)                     # a list of strings
for_each = { for s in var.subnets : s.name => s }  # a list of objects
```

The second form is the common one, and it also forces you to decide what the
stable identity of each item is — which is a useful thing to be forced to think
about.

### 3. Keys known at plan time

It means the **set of keys** must be computable during `terraform plan`, before
anything is created.

Terraform has to know the full list of resource addresses in advance, because the
plan is a list of addresses and what will happen to each. It cannot plan to
create `aws_subnet.this[???]`.

**Values** may be unknown — `(known after apply)` is fine for an argument.
**Keys** may not.

What breaks:

```hcl
resource "aws_s3_bucket_versioning" "this" {
  for_each = toset(aws_s3_bucket.data[*].id)   # bucket IDs don't exist yet
}
```

```text
Error: Invalid for_each argument
The "for_each" map includes keys derived from resource attributes that cannot
be determined until apply.
```

The fix is to key off something already known — usually the same map that created
the upstream resources:

```hcl
resource "aws_s3_bucket_versioning" "this" {
  for_each = aws_s3_bucket.data      # same keys as the buckets
  bucket   = each.value.id           # the value can be unknown; that's fine
}
```

Notely does exactly this with `aws_lb_target_group_attachment` iterating over
`aws_instance.app`.

### 4. When `count` is right

**When the question is genuinely "how many", not "which ones" — and in practice
that means 0 or 1.**

The conditional resource:

```hcl
resource "aws_nat_gateway" "main" {
  count = var.enable_nat_gateway ? 1 : 0
}
```

This is idiomatic and correct. There is no meaningful key to use, the resource is
either there or not, and there is no list to remove a middle element from. Using
`for_each` here would be contrived.

The other defensible case is N genuinely interchangeable, unnamed things where
order carries no meaning and you only ever add or remove from the end. It is
rarer than people think — as soon as you can name them, `for_each` is better.

Everything else is `for_each`.

### 5. `create_before_destroy`

By default, when Terraform must replace a resource it **destroys the old one
first, then creates the new one**. Between those two steps, the resource does not
exist. If it was serving traffic, that is an outage.

`create_before_destroy = true` reverses it: create the replacement, then destroy
the original. The plan symbol changes from `-/+` to `+/-`.

Notely's EC2 instances use it because they serve requests. Changing the AMI or the
user-data script forces a replacement, and without this setting both servers
could go down before their replacements existed.

Two things to watch:

**Names must not collide.** Both old and new exist simultaneously, so any
resource with a fixed unique `name` will fail. Use `name_prefix` where the
provider offers it — which is why Notely's target group also sets
`create_before_destroy`.

**It cascades.** Anything depending on the resource may also need the setting, or
Terraform cannot order the operations correctly.

### 6. Why `depends_on` is needed for the IAM policy

Because there is **no value flowing** between the instance and the policy.

Terraform builds its dependency graph from references. Trace what the instance
actually references:

```hcl
iam_instance_profile = aws_iam_instance_profile.app.name
```

That creates an edge: instance depends on **instance profile**. The profile in
turn references the **role**. So the graph is:

```text
role  ->  instance profile  ->  instance
```

The `aws_iam_role_policy.app_s3` resource attaches permissions *to* the role, but
nothing in the instance's configuration mentions it. So it sits off to one side
of the graph with no ordering relationship to the instance at all — Terraform is
free to create them in either order, or in parallel.

The real-world consequence: the instance boots, user-data runs, the app starts
and tries to write to S3 — and gets `AccessDenied`, because the policy has not
been attached yet. It usually works, which is worse than never working, because
the failure is intermittent and only shows up under load or in a fresh
environment.

`depends_on` adds the missing edge explicitly.

This is the legitimate use case for `depends_on`: a genuine dependency that
carries no data. It cannot be fixed by referencing an attribute, because there is
no attribute to reference.

### 7. S3 gateway endpoint instead of a NAT gateway

**Cost, mostly. Also security.**

Notely's app servers live in private subnets, so they have no route to the
internet. They need to reach S3 to store and fetch attachments.

A **NAT gateway** would give them general outbound internet access, and S3 access
along with it. It costs about **$0.045/hour — roughly $32/month** — plus data
processing charges, and doing it properly means one per AZ, so about $64/month
for Notely's two zones. That is a lot of money for a learning project, and it is
the single most common source of surprise AWS bills.

An **S3 gateway endpoint** adds a route to the private route table sending S3
traffic down a private AWS path. It costs **nothing**, no hourly charge and no
data charge.

It is also more secure: traffic to S3 never traverses the internet at all, and
you can attach an endpoint policy restricting which buckets are reachable
through it.

The trade-off is scope. A gateway endpoint only reaches S3 (or DynamoDB — those
are the only two gateway endpoints AWS offers). It will not help with
`npm install`, calling a third-party API, or OS updates. Notely handles that by
baking dependencies into user-data at boot from a public AMI, which is why the
servers can still install Node.js.

A real production Notely would likely have both: a NAT gateway for general
outbound, and the S3 endpoint anyway, because free and private beats paid and
public for the traffic it covers.

---

## Next lesson

**Module 7 — Data Sources** (`07-data-sources.md`)

Notely still has a hardcoded AMI ID that will be out of date within a month, and
availability zone names built with string concatenation that will break in a
region with different naming.

Module 7 fixes both: `data` blocks let Terraform *look things up* rather than be
told them. It is a short module, and it removes the last of Notely's magic
values.
