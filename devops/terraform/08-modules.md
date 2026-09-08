# Module 8 — Modules

## Where Notely is right now

```text
  ~/terraform-labs/notely/
  ├── versions.tf      providers
  ├── variables.tf     ~10 variables
  ├── locals.tf        subnet maths, tags, user_data
  ├── data.tf          AMI, AZs, caller identity
  ├── network.tf       VPC, 6 subnets, IGW, route tables, endpoint
  ├── security.tf      3 security groups
  ├── main.tf          S3, log group, IAM, EC2, ALB
  ├── outputs.tf
  └── templates/user-data.sh.tftpl

  About 350 lines in one flat directory.
```

By the end of this module:

```text
  notely/
  ├── main.tf            ~50 lines - reads like the architecture diagram
  ├── variables.tf
  ├── outputs.tf
  └── modules/
      ├── network/       VPC, subnets, routing
      ├── security/      the three security groups
      ├── web/           EC2 servers + load balancer
      └── storage/       S3 bucket + log group
```

Full picture: `notely-architecture.md`.

---

## Why this module exists

Notely's configuration works. It is also 350 lines in one namespace, and it has
three problems:

**1. You cannot reuse any of it.** Your team starts a second service. It needs
the same VPC shape. Your only option is copy and paste, and the two copies drift
apart from day one.

**2. It is hard to read.** Everything is at the same level of detail. Someone new
opening `main.tf` sees an IAM policy statement next to a load balancer listener
next to an S3 bucket, with no structure telling them what matters.

**3. Everything can touch everything.** Any resource can reference any other.
There are no boundaries, so there is nothing stopping the storage code
reaching into networking internals.

Modules fix all three. And they are the thing that makes Module 9 — three
environments — actually pleasant instead of a copy-paste nightmare.

---

## The core idea (one sentence)

A module is just a directory of `.tf` files that you can call from another
directory, passing in variables and getting back outputs.

> You have already written a module. Every Terraform directory is one. `modules/network` is the same idea as `notely/`, just called from somewhere else.

---

## Mental model (from Node.js)

Modules are npm packages. The mapping is almost exact.

```javascript
// modules/network/index.js
function createNetwork({ vpcCidr, azCount }) {
  // ... build things ...
  return { vpcId, publicSubnetIds, privateSubnetIds };
}
module.exports = { createNetwork };
```

```javascript
// main.js
const { createNetwork } = require('./modules/network');

const network = createNetwork({ vpcCidr: '10.0.0.0/16', azCount: 2 });
console.log(network.vpcId);
```

In Terraform:

```hcl
module "network" {
  source = "./modules/network"

  vpc_cidr = "10.0.0.0/16"
  az_count = 2
}

output "vpc_id" {
  value = module.network.vpc_id
}
```

| Node.js | Terraform |
|---|---|
| A package / a `require`d file | A module directory |
| Function parameters | `variable` blocks in the module |
| Default parameter values | `default` on a variable |
| `module.exports` | `output` blocks |
| `require('./modules/network')` | `source = "./modules/network"` |
| `npm install` | `terraform init` |
| `node_modules/` | `.terraform/modules/` |
| `package.json` dependency | `source` + `version` |
| `^1.2.0` | `~> 1.2` |
| npm registry | Terraform Registry |
| Not exporting something makes it private | Not declaring an output makes it inaccessible |

That last row is important and often missed: **you can only read a module's
outputs.** Everything else inside it is genuinely private.

---

## Part 1 — The vocabulary

### Root module vs child module

**Root module** — the directory you run `terraform apply` in. Notely's top level.

**Child module** — a directory called by a `module` block.

You have been writing root modules since Module 1.

### Calling a module

```hcl
module "network" {
  source = "./modules/network"

  # everything else here is an input variable of that module
  project_name = var.project_name
  environment  = var.environment
  vpc_cidr     = var.vpc_cidr
  az_count     = var.az_count
}
```

| Part | Meaning |
|---|---|
| `module` | Block type |
| `"network"` | Your local name for this instance |
| `source` | Where the module code lives |
| Other arguments | Values for the module's variables |

### Reading a module's outputs

```hcl
module.network.vpc_id
module.network.public_subnet_ids
```

```text
module.<local name>.<output name>
```

You cannot reach inside:

```hcl
module.network.aws_vpc.main.id       # NOT VALID
```

If you need it, the module must declare it as an output. That restriction is the
point — it is the module's public interface.

### Module addresses in state

```text
module.network.aws_vpc.main
module.network.aws_subnet.this["public-a"]
module.web.aws_instance.app["app-a"]
```

Everything inside a module gets prefixed. This matters for `terraform state`
commands and `moved` blocks.

---

## Part 2 — Standard module structure

A module directory should look like this:

```text
modules/network/
├── main.tf         the resources
├── variables.tf    the inputs
├── outputs.tf      the outputs
├── versions.tf     required_providers
└── README.md       what it does and how to use it
```

Two rules that matter:

**A module must not contain a `provider` block.**

```hcl
# modules/network/main.tf
provider "aws" {          # DON'T
  region = "ap-southeast-2"
}
```

Providers are configured by the **root** module and inherited. A module that
configures its own provider cannot be used with `for_each`, cannot be used with
aliases, and produces deprecation warnings. It is the Terraform equivalent of a
library calling `process.exit()`.

The module should declare which providers it *needs*, though:

```hcl
# modules/network/versions.tf
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**A module must not contain a `backend` block.** Same reasoning — the root
decides where state lives.

---

## Part 3 — Designing the interface

This is the part that separates a module people enjoy using from one they curse.

### Keep the inputs small

```hcl
# Bad - 40 variables, every one required
module "network" {
  source                    = "./modules/network"
  vpc_cidr                  = "10.0.0.0/16"
  public_subnet_cidr_a      = "10.0.1.0/24"
  public_subnet_cidr_b      = "10.0.2.0/24"
  app_subnet_cidr_a         = "10.0.11.0/24"
  # ... 36 more
}

# Good - 4 variables, the rest computed or defaulted
module "network" {
  source       = "./modules/network"
  project_name = "notely"
  environment  = "dev"
  vpc_cidr     = "10.0.0.0/16"
  az_count     = 2
}
```

The second one computes the subnet CIDRs internally with `cidrsubnet`. The caller
does not need to care.

> Every required variable is a decision you are forcing on the caller. Make sure it is a decision they actually want to make.

### Use `optional()` for the rest

```hcl
variable "config" {
  type = object({
    instance_type = string
    min_size      = optional(number, 2)
    max_size      = optional(number, 6)
    health_path   = optional(string, "/health")
  })
}
```

Callers supply one field, or all four.

### Validate at the boundary

A module's variables are its API. Validate them:

```hcl
variable "az_count" {
  type = number

  validation {
    condition     = var.az_count >= 2
    error_message = "This module builds highly available infrastructure and requires at least 2 availability zones."
  }
}
```

The error message teaches the caller something. That is what good API errors do.

### Output what people will need

```hcl
output "vpc_id" {
  description = "The VPC's ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets, for load balancers"
  value       = [for k, v in aws_subnet.this : v.id if local.subnets[k].public]
}

output "app_subnet_ids" {
  description = "IDs of the app tier subnets, for EC2 instances"
  value       = [for k, v in aws_subnet.this : v.id if local.subnets[k].tier == "app"]
}
```

Every output needs a `description`. It is the documentation people actually read.

---

## Part 4 — Module sources

```hcl
# Local path - starts with ./ or ../
source = "./modules/network"

# Terraform Registry - namespace/name/provider
source  = "terraform-aws-modules/vpc/aws"
version = "~> 5.0"

# GitHub, pinned to a tag
source = "git::https://github.com/acme/terraform-modules.git//network?ref=v1.4.0"

# Generic git over SSH
source = "git::ssh://git@github.com/acme/tf-modules.git//network?ref=v1.4.0"

# S3
source = "s3::https://s3.amazonaws.com/acme-modules/network.zip"
```

### The `version` trap

**`version` only works with registry sources.**

```hcl
# Works
source  = "terraform-aws-modules/vpc/aws"
version = "~> 5.0"

# ERROR - version is not valid for a git source
source  = "git::https://github.com/acme/modules.git//network"
version = "~> 1.0"
```

For git sources, pin with `?ref=`:

```hcl
source = "git::https://github.com/acme/modules.git//network?ref=v1.4.0"
```

The `//` before the subdirectory is required — it separates the repository from
the path inside it.

### Always pin a version

```hcl
# Dangerous - you get whatever is newest today
source = "terraform-aws-modules/vpc/aws"

# Correct
source  = "terraform-aws-modules/vpc/aws"
version = "~> 5.8"
```

Same reasoning as `package.json`. Without a version, two people running `init` on
different days get different modules.

### `terraform init` downloads modules

```bash
terraform init
```

```text
Initializing modules...
- network in modules/network
- web in modules/web
Downloading registry.terraform.io/terraform-aws-modules/vpc/aws 5.8.1...
```

They land in `.terraform/modules/`. **Re-run `init` whenever you add or change a
module `source`.** Forgetting is the most common module error:

```text
Error: Module not installed
```

---

## Part 5 — Using registry modules

The Terraform Registry has thousands of pre-built modules. The AWS VPC one is
used by an enormous number of teams:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.8"

  name = "notely-dev"
  cidr = "10.0.0.0/16"

  azs             = ["ap-southeast-2a", "ap-southeast-2b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.11.0/24", "10.0.12.0/24"]

  enable_nat_gateway = false      # $32/month if you turn this on
  single_nat_gateway = false

  tags = {
    Project = "notely"
  }
}
```

That is about 20 lines replacing 150.

### Should you use them?

| Use a registry module when | Write your own when |
|---|---|
| It is a well-known, complex pattern (VPC, EKS) | Your requirements are specific |
| It is actively maintained | The module does far more than you need |
| You have read the source | You want to actually understand it |
| Your team has no strong opinions | Your organisation has standards to enforce |

**Read the source before adopting one.** The AWS VPC module is about 3,000 lines
and supports dozens of options you will never use. That is fine — but you are
taking on a dependency, and when something goes wrong you will be reading that
code.

For Notely we write our own, because the whole point is learning what is
underneath.

---

## Part 6 — Meta-arguments on modules

Modules support `count`, `for_each`, `depends_on` and `providers`:

```hcl
# Conditional module
module "monitoring" {
  count  = var.enable_monitoring ? 1 : 0
  source = "./modules/monitoring"
}

# Many instances of a module
module "service" {
  for_each = local.services
  source   = "./modules/service"

  name = each.key
  port = each.value.port
}

# Passing an aliased provider in
module "us_east_certs" {
  source = "./modules/certificates"

  providers = {
    aws = aws.us_east_1
  }
}
```

Referencing a `count`ed or `for_each`ed module:

```hcl
module.monitoring[0].dashboard_url
module.service["api"].endpoint
```

---

## Part 7 — Anti-patterns

**1. The module that wraps one resource.**

```hcl
# modules/s3-bucket/main.tf
resource "aws_s3_bucket" "this" {
  bucket = var.name
}
```

That is not a module, it is a rename. It adds a layer of indirection and gives
you nothing. A module should encapsulate a *pattern* — several resources that
belong together.

**2. The kitchen-sink module.**

One module that builds the VPC, the servers, the database and the CI pipeline.
Nobody can use part of it, and every change risks everything.

**3. `provider` blocks inside modules.**

Covered above. Breaks `for_each`, aliases, and reusability.

**4. Deep nesting.**

```text
root -> platform -> networking -> vpc -> subnets
```

Three levels deep and you cannot follow a value from the root to where it is
used. Two levels is plenty.

**5. Abstracting too early.**

You need a thing twice before you know what varies. Write it inline the first
time, extract it the second.

**6. Hardcoded values inside a module.**

```hcl
# modules/web/main.tf
resource "aws_instance" "app" {
  instance_type = "t3.micro"      # caller cannot change this
}
```

If a caller might reasonably want it different, it is a variable.

---

## Building it into Notely

The big refactor. Four modules.

### `modules/network`

`modules/network/variables.tf`:

```hcl
variable "name_prefix" {
  description = "Prefix for all resource names, e.g. notely-dev"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC. Must be /16 or larger."
  type        = string

  validation {
    condition     = can(cidrnetmask(var.vpc_cidr))
    error_message = "vpc_cidr must be a valid CIDR block, for example 10.0.0.0/16."
  }

  validation {
    condition     = tonumber(split("/", var.vpc_cidr)[1]) <= 16
    error_message = "vpc_cidr must be /16 or larger to fit three tiers of subnets."
  }
}

variable "az_count" {
  description = "How many availability zones to spread across"
  type        = number
  default     = 2

  validation {
    condition     = var.az_count >= 2 && var.az_count <= 3
    error_message = "az_count must be 2 or 3. One zone is not highly available."
  }
}

variable "enable_nat_gateway" {
  description = "Create NAT gateways so private subnets can reach the internet. COSTS ABOUT $32/MONTH PER GATEWAY."
  type        = bool
  default     = false
}
```

`modules/network/main.tf`:

```hcl
data "aws_availability_zones" "available" {
  state = "available"

  filter {
    name   = "opt-in-status"
    values = ["opt-in-not-required"]
  }
}

locals {
  azs = slice(data.aws_availability_zones.available.names, 0, var.az_count)

  # Three tiers, one subnet per tier per AZ.
  # cidrsubnet carves them out of var.vpc_cidr automatically.
  subnets = merge(
    { for i, az in local.azs : "public-${substr(az, -1, 1)}" => {
      cidr_block        = cidrsubnet(var.vpc_cidr, 8, i + 1)
      availability_zone = az
      tier              = "public"
      public            = true
    } },
    { for i, az in local.azs : "app-${substr(az, -1, 1)}" => {
      cidr_block        = cidrsubnet(var.vpc_cidr, 8, i + 11)
      availability_zone = az
      tier              = "app"
      public            = false
    } },
    { for i, az in local.azs : "data-${substr(az, -1, 1)}" => {
      cidr_block        = cidrsubnet(var.vpc_cidr, 8, i + 21)
      availability_zone = az
      tier              = "data"
      public            = false
    } },
  )
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = { Name = "${var.name_prefix}-vpc" }
}

resource "aws_subnet" "this" {
  for_each = local.subnets

  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr_block
  availability_zone       = each.value.availability_zone
  map_public_ip_on_launch = each.value.public

  tags = {
    Name = "${var.name_prefix}-${each.key}"
    Tier = each.value.tier
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "${var.name_prefix}-igw" }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = { Name = "${var.name_prefix}-public-rt" }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "${var.name_prefix}-private-rt" }
}

resource "aws_route_table_association" "this" {
  for_each = local.subnets

  subnet_id      = aws_subnet.this[each.key].id
  route_table_id = each.value.public ? aws_route_table.public.id : aws_route_table.private.id
}

# Free private path to S3 - avoids needing a NAT gateway for attachments.
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${data.aws_region.current.name}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]

  tags = { Name = "${var.name_prefix}-s3-endpoint" }
}

data "aws_region" "current" {}
```

`modules/network/outputs.tf`:

```hcl
output "vpc_id" {
  description = "The VPC's ID"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "The VPC's CIDR block, useful for security group rules"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "Public subnet IDs, for load balancers"
  value       = [for k, v in aws_subnet.this : v.id if local.subnets[k].public]
}

output "app_subnet_ids" {
  description = "App tier subnet IDs, for EC2 instances"
  value       = [for k, v in aws_subnet.this : v.id if local.subnets[k].tier == "app"]
}

output "data_subnet_ids" {
  description = "Data tier subnet IDs, for RDS"
  value       = [for k, v in aws_subnet.this : v.id if local.subnets[k].tier == "data"]
}

output "availability_zones" {
  description = "The AZs actually used"
  value       = local.azs
}
```

### `modules/security`

`modules/security/variables.tf`:

```hcl
variable "name_prefix" {
  type = string
}

variable "vpc_id" {
  description = "VPC to create the security groups in"
  type        = string
}

variable "app_port" {
  description = "Port the application listens on"
  type        = number
  default     = 3000
}

variable "alb_ingress_ports" {
  description = "Ports the load balancer accepts public traffic on"
  type        = list(number)
  default     = [80, 443]
}

variable "db_port" {
  description = "Database port"
  type        = number
  default     = 5432
}
```

`modules/security/main.tf`:

```hcl
resource "aws_security_group" "alb" {
  name        = "${var.name_prefix}-alb"
  description = "Load balancer - the only thing reachable from the internet"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.alb_ingress_ports

    content {
      description = "Public web traffic on ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    description = "Forward to the app servers"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.name_prefix}-alb-sg" }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "app" {
  name        = "${var.name_prefix}-app"
  description = "Application servers - reachable only from the load balancer"
  vpc_id      = var.vpc_id

  ingress {
    description     = "App traffic from the load balancer only"
    from_port       = var.app_port
    to_port         = var.app_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    description = "Outbound to AWS APIs and package registries"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.name_prefix}-app-sg" }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "db" {
  name        = "${var.name_prefix}-db"
  description = "Database - reachable only from the app servers"
  vpc_id      = var.vpc_id

  ingress {
    description     = "Database traffic from the app servers only"
    from_port       = var.db_port
    to_port         = var.db_port
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  tags = { Name = "${var.name_prefix}-db-sg" }

  lifecycle {
    create_before_destroy = true
  }
}
```

`modules/security/outputs.tf`:

```hcl
output "alb_security_group_id" {
  description = "Attach this to the load balancer"
  value       = aws_security_group.alb.id
}

output "app_security_group_id" {
  description = "Attach this to the application servers"
  value       = aws_security_group.app.id
}

output "db_security_group_id" {
  description = "Attach this to the database"
  value       = aws_security_group.db.id
}
```

### `modules/storage`

`modules/storage/main.tf`:

```hcl
data "aws_caller_identity" "current" {}

resource "aws_s3_bucket" "attachments" {
  bucket = "${var.name_prefix}-attachments-${data.aws_caller_identity.current.account_id}"
}

resource "aws_s3_bucket_versioning" "attachments" {
  bucket = aws_s3_bucket.attachments.id

  versioning_configuration {
    status = var.enable_versioning ? "Enabled" : "Suspended"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "attachments" {
  bucket = aws_s3_bucket.attachments.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "attachments" {
  bucket = aws_s3_bucket.attachments.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_cloudwatch_log_group" "app" {
  name              = "/${var.name_prefix}/app"
  retention_in_days = var.log_retention_days
}
```

Note the module bundles four S3 resources that always belong together. That is
what makes it a module rather than a rename — a caller gets a *correctly
configured* bucket, not just a bucket.

`modules/storage/outputs.tf`:

```hcl
output "attachments_bucket_name" {
  description = "Name of the attachments bucket"
  value       = aws_s3_bucket.attachments.bucket
}

output "attachments_bucket_arn" {
  description = "ARN of the attachments bucket"
  value       = aws_s3_bucket.attachments.arn
}

output "log_group_name" {
  description = "CloudWatch log group for the application"
  value       = aws_cloudwatch_log_group.app.name
}

output "log_group_arn" {
  description = "ARN of the log group"
  value       = aws_cloudwatch_log_group.app.arn
}
```

### `modules/web`

This one takes the most inputs, because it wires everything together.

`modules/web/variables.tf`:

```hcl
variable "name_prefix" { type = string }
variable "vpc_id" { type = string }

variable "public_subnet_ids" {
  description = "Subnets for the load balancer. At least two, in different AZs."
  type        = list(string)

  validation {
    condition     = length(var.public_subnet_ids) >= 2
    error_message = "A load balancer requires at least two subnets in different availability zones."
  }
}

variable "app_subnet_ids" {
  description = "Subnets for the application servers"
  type        = list(string)
}

variable "alb_security_group_id" { type = string }
variable "app_security_group_id" { type = string }

variable "instance_type" {
  description = "EC2 instance size"
  type        = string
  default     = "t3.micro"
}

variable "app_port" {
  type    = number
  default = 3000
}

variable "health_check_path" {
  type    = string
  default = "/health"
}

variable "user_data" {
  description = "Startup script for the servers"
  type        = string
}

variable "instance_profile_name" {
  description = "IAM instance profile granting S3 and CloudWatch access"
  type        = string
}
```

`modules/web/main.tf`:

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
}

resource "aws_instance" "app" {
  for_each = toset(var.app_subnet_ids)

  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = each.value
  vpc_security_group_ids = [var.app_security_group_id]
  iam_instance_profile   = var.instance_profile_name

  user_data                   = var.user_data
  user_data_replace_on_change = true

  tags = { Name = "${var.name_prefix}-app" }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_lb" "main" {
  name               = "${var.name_prefix}-alb"
  load_balancer_type = "application"
  security_groups    = [var.alb_security_group_id]
  subnets            = var.public_subnet_ids

  tags = { Name = "${var.name_prefix}-alb" }
}

resource "aws_lb_target_group" "app" {
  name_prefix = "ntly-"
  port        = var.app_port
  protocol    = "HTTP"
  vpc_id      = var.vpc_id

  health_check {
    path                = var.health_check_path
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

Note `name_prefix = "ntly-"` on the target group. With
`create_before_destroy`, both old and new exist briefly, so a fixed `name` would
collide. `name_prefix` lets AWS generate a unique suffix. (It is limited to six
characters, hence the abbreviation.)

`modules/web/outputs.tf`:

```hcl
output "alb_dns_name" {
  description = "Public DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}

output "alb_zone_id" {
  description = "Route 53 zone ID of the ALB, for alias records"
  value       = aws_lb.main.zone_id
}

output "url" {
  description = "URL to reach the application"
  value       = "http://${aws_lb.main.dns_name}"
}

output "instance_ids" {
  description = "IDs of the application servers"
  value       = [for i in aws_instance.app : i.id]
}
```

### The new root `main.tf`

Here is the payoff:

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  user_data = templatefile("${path.module}/templates/user-data.sh.tftpl", {
    environment        = var.environment
    region             = var.region
    app_port           = var.app_port
    attachments_bucket = module.storage.attachments_bucket_name
    log_group          = module.storage.log_group_name
  })
}

module "network" {
  source = "./modules/network"

  name_prefix = local.name_prefix
  vpc_cidr    = var.vpc_cidr
  az_count    = var.az_count
}

module "security" {
  source = "./modules/security"

  name_prefix = local.name_prefix
  vpc_id      = module.network.vpc_id
  app_port    = var.app_port
}

module "storage" {
  source = "./modules/storage"

  name_prefix        = local.name_prefix
  enable_versioning  = var.enable_versioning
  log_retention_days = var.log_retention_days
}

module "web" {
  source = "./modules/web"

  name_prefix           = local.name_prefix
  vpc_id                = module.network.vpc_id
  public_subnet_ids     = module.network.public_subnet_ids
  app_subnet_ids        = module.network.app_subnet_ids
  alb_security_group_id = module.security.alb_security_group_id
  app_security_group_id = module.security.app_security_group_id
  instance_type         = var.app_instance_type
  app_port              = var.app_port
  user_data             = local.user_data
  instance_profile_name = aws_iam_instance_profile.app.name
}
```

**Read that.** It says: build a network, build security groups in it, build
storage, build the web tier wired to all three.

That is the architecture diagram, in code. Someone who has never seen Notely can
understand its shape in thirty seconds — and if they want the detail, it is in a
module they can open.

`outputs.tf`:

```hcl
output "notely_url" {
  description = "Public URL for Notely"
  value       = module.web.url
}

output "vpc_id" {
  value = module.network.vpc_id
}

output "attachments_bucket" {
  value = module.storage.attachments_bucket_name
}
```

### Migrating without destroying anything

Your existing resources are at addresses like `aws_vpc.main`. They now need to be
at `module.network.aws_vpc.main`.

Use `moved` blocks:

```hcl
moved {
  from = aws_vpc.main
  to   = module.network.aws_vpc.main
}

moved {
  from = aws_subnet.this
  to   = module.network.aws_subnet.this
}

moved {
  from = aws_internet_gateway.main
  to   = module.network.aws_internet_gateway.main
}

moved {
  from = aws_route_table.public
  to   = module.network.aws_route_table.public
}

moved {
  from = aws_route_table.private
  to   = module.network.aws_route_table.private
}

moved {
  from = aws_route_table_association.this
  to   = module.network.aws_route_table_association.this
}

moved {
  from = aws_vpc_endpoint.s3
  to   = module.network.aws_vpc_endpoint.s3
}

moved {
  from = aws_security_group.alb
  to   = module.security.aws_security_group.alb
}

moved {
  from = aws_security_group.app
  to   = module.security.aws_security_group.app
}

moved {
  from = aws_security_group.db
  to   = module.security.aws_security_group.db
}

moved {
  from = aws_s3_bucket.attachments
  to   = module.storage.aws_s3_bucket.attachments
}

moved {
  from = aws_cloudwatch_log_group.app
  to   = module.storage.aws_cloudwatch_log_group.app
}
```

Note you can move a whole `for_each` resource in one block — Terraform handles
all the keys.

```bash
terraform init      # required - new modules to install
terraform plan -var-file=dev.tfvars
```

The goal is:

```text
Plan: 0 to add, 0 to change, 0 to destroy.
```

with a list of moves above it. **An empty plan is your proof the refactor is
safe.** If anything shows as add/destroy, a `moved` block is missing or wrong.

Delete the `moved` blocks after a successful apply.

---

## Real-World Example

A platform team maintains six internal modules used by 40 services:

```text
terraform-modules/
├── vpc/
├── ecs-service/
├── rds-postgres/
├── s3-bucket/
├── alb/
└── monitoring/
```

Tagged with semver, consumed like:

```hcl
module "api" {
  source = "git::ssh://git@github.com/acme/terraform-modules.git//ecs-service?ref=v3.2.0"

  name          = "notely-api"
  cluster_arn   = data.aws_ecs_cluster.main.arn
  image         = "${var.ecr_repo}:${var.image_tag}"
  cpu           = 512
  memory        = 1024
  desired_count = 3
}
```

**The thing that made it work.** When a security review required every S3 bucket
to block public access and enable encryption, they changed `s3-bucket` once,
tagged `v2.0.0`, and opened 40 pull requests bumping the ref. Each one was a
one-line diff a reviewer could approve in seconds.

Without modules that would have been 40 separate hand-edits across 40
repositories, with 40 chances to get it wrong and no way to verify coverage.

**The thing that nearly broke it.** Their first `ecs-service` module had 63 input
variables, because every time someone needed something configurable they added
another. Nobody could use it without a 60-line block, and half the variables
were only ever set to their default.

The rewrite grouped them into three objects with `optional()` fields and cut the
required inputs to five. Adoption went up immediately.

> A module's job is to make the common case easy. If using it is as much work as writing the resources yourself, it has failed.

---

## Common Mistakes Beginners Make

**1. Forgetting `terraform init` after adding a module.**

```text
Error: Module not installed
```

Adding or changing a `source` always needs `init`.

**2. Putting a `provider` block in a module.**

Breaks `for_each`, aliases, and reuse. The root configures providers.

**3. Trying to reach inside a module.**

```hcl
module.network.aws_vpc.main.id     # not valid
module.network.vpc_id              # add an output
```

**4. Using `version` with a git source.**

Only registry sources support `version`. Git uses `?ref=`.

**5. Forgetting `//` before a subdirectory in a git source.**

```hcl
source = "git::https://github.com/acme/modules.git/network"    # wrong
source = "git::https://github.com/acme/modules.git//network"   # correct
```

**6. Not pinning a registry module version.**

Same problem as an unpinned npm dependency.

**7. Refactoring into modules without `moved` blocks.**

Every resource gets a new address, so Terraform destroys and recreates
everything. On a database that is catastrophic.

**8. Building a module before you need one.**

Write it inline once. Extract it the second time, when you know what varies.

**9. Outputs with no description.**

The description is the documentation. Modules without them are guessing games.

---

## Hands-On Lab — Refactor Notely into Modules

**Cost: free** for parts A–C. Part D optionally applies the billable web tier.

### Part A: Create the module directories

```bash
cd ~/terraform-labs/notely
mkdir -p modules/{network,security,storage,web}
```

Create the four modules exactly as shown above. Each gets `main.tf`,
`variables.tf`, `outputs.tf` and a `versions.tf`:

```hcl
# modules/*/versions.tf - the same in each
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Part B: Rewrite the root

Replace `network.tf`, `security.tf` and most of `main.tf` with the four `module`
blocks.

Keep in the root: `versions.tf`, `variables.tf`, `outputs.tf`, the IAM role and
instance profile, `locals.tf`, and `templates/`.

(IAM stays in the root for now because it references both the storage bucket and
the log group. In the capstone it moves into its own module.)

```bash
rm network.tf security.tf
terraform fmt -recursive
terraform validate
```

### Part C: Migrate state with `moved` blocks

Create `moved.tf` with every block listed above.

```bash
terraform init
terraform plan -var-file=dev.tfvars
```

**Read this plan very carefully.** You want:

```text
Plan: 0 to add, 0 to change, 0 to destroy.
```

If you see resources being destroyed, a `moved` block is missing. Find the
address in the destroy list and add one.

```bash
terraform apply -var-file=dev.tfvars
```

Then verify the new addresses:

```bash
terraform state list
```

```text
aws_iam_instance_profile.app
aws_iam_role.app
module.network.aws_internet_gateway.main
module.network.aws_subnet.this["app-a"]
module.network.aws_vpc.main
module.security.aws_security_group.alb
module.storage.aws_s3_bucket.attachments
...
```

Everything moved. Nothing was destroyed.

```bash
rm moved.tf
terraform plan -var-file=dev.tfvars
```

Still empty. The refactor is complete.

### Part D: Prove reusability

This is the moment the module pays off. Add a **second** network, without
touching the module:

```hcl
module "network_experiment" {
  source = "./modules/network"

  name_prefix = "notely-experiment"
  vpc_cidr    = "10.9.0.0/16"
  az_count    = 3
}
```

```bash
terraform plan -var-file=dev.tfvars
```

`Plan: 14 to add` — a complete second VPC with nine subnets, from four lines.

**Do not apply it.** The point was to see the plan. Delete the block.

### Part E: Try a registry module

For comparison, see what the community VPC module produces:

```bash
mkdir -p ~/terraform-labs/registry-comparison
cd ~/terraform-labs/registry-comparison
```

```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = "ap-southeast-2"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.8"

  name = "comparison"
  cidr = "10.50.0.0/16"

  azs             = ["ap-southeast-2a", "ap-southeast-2b"]
  public_subnets  = ["10.50.1.0/24", "10.50.2.0/24"]
  private_subnets = ["10.50.11.0/24", "10.50.12.0/24"]

  enable_nat_gateway = false     # leave this false - $32/month
}
```

```bash
terraform init
terraform plan
```

Read the plan and compare it with your own module's. Then read the source:

```bash
ls .terraform/modules/vpc/
wc -l .terraform/modules/vpc/main.tf
```

About 1,500 lines in one file. That is what you are taking on when you adopt it —
lots of capability, and lots of code you did not write.

```bash
rm -rf ~/terraform-labs/registry-comparison
```

### Part F: Tear down

```bash
cd ~/terraform-labs/notely
terraform destroy -var-file=dev.tfvars
```

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

### What you should have at the end

- Four modules with clean interfaces
- A root `main.tf` that reads like the architecture diagram
- A state migration proven safe by an empty plan
- Direct experience that a second VPC costs four lines

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **Module** | A directory of `.tf` files you can call from elsewhere |
| **Root module** | The directory you run `apply` in |
| **Child module** | A directory called by a `module` block |
| **`source`** | Where the module code is |
| **`module.name.output`** | The only way to read from a module |
| **Private by default** | You cannot reference a module's internal resources |
| **No `provider` in a module** | The root configures providers; modules inherit |
| **No `backend` in a module** | Same reason |
| **`version`** | Registry sources only |
| **`?ref=v1.2.0`** | How you pin a git source |
| **`//subdir`** | Points at a directory inside a git repo |
| **`terraform init`** | Required after adding or changing a `source` |
| **`.terraform/modules/`** | Where downloaded modules land (`node_modules/`) |
| **`optional(type, default)`** | Keeps the caller's block short |
| **`moved`** | Refactor into modules without destroying anything |
| **Empty plan** | Your proof a refactor was safe |
| **`count` / `for_each` on modules** | Supported, same semantics as resources |
| **Anti-pattern** | A module wrapping a single resource |
| **Anti-pattern** | A module with 60 required variables |

---

## Checkpoint (answer briefly)

1. Why can't you write `module.network.aws_vpc.main.id`?
2. Why must a module not contain a `provider` block?
3. You add `version = "~> 1.0"` alongside a git `source` and get an error. Why?
4. You refactor a flat configuration into modules and the plan says `0 to add, 0 to change, 0 to destroy`. What made that possible, and what would have happened without it?
5. When should you extract something into a module?
6. What is wrong with a module that wraps a single `aws_s3_bucket`?
7. Notely's `modules/web` target group uses `name_prefix` instead of `name`. Why?

---

## Checkpoint — model answers

### 1. Why you cannot reach into a module

Because **outputs are a module's entire public interface**. Everything else is
private.

This is deliberate, and it is the same idea as `module.exports` in Node.js: a
file can define twenty functions and export three, and callers can only use those
three.

The benefit is that a module's internals can change freely. If
`modules/network` renames `aws_vpc.main` to `aws_vpc.this`, or splits a subnet
resource in two, nothing outside breaks — as long as `vpc_id` still comes out the
other end. If callers could reach inside, every internal change would be a
breaking change.

So when you need a value, add an output:

```hcl
# modules/network/outputs.tf
output "vpc_id" {
  description = "The VPC's ID"
  value       = aws_vpc.main.id
}
```

Being forced to do that is useful pressure — it makes you decide, deliberately,
what the module promises to provide.

### 2. No `provider` block in a module

Because provider configuration belongs to the **root module**, and putting it in
a child breaks several things at once.

**`count` and `for_each` stop working.** Terraform cannot instantiate a module
multiple times if each instance would define its own provider — provider
configurations must be static and known before the graph is built.

```text
Error: Module is incompatible with count, for_each, and depends_on
```

**Provider aliases stop working.** The caller can no longer pass in a different
region or account with `providers = { aws = aws.us_east_1 }`, because the module
has already decided.

**It is not reusable.** A module that hardcodes `region = "ap-southeast-2"` is
useless to anyone deploying elsewhere.

The correct pattern is for the module to declare what it *needs* without
configuring it:

```hcl
# modules/network/versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

The root configures, the module inherits. The Node.js analogue: a library should
not read environment variables or call `process.exit()` — it takes what it needs
as arguments.

### 3. `version` with a git source

Because **`version` only applies to registry sources.**

The Terraform Registry is a version-aware system: it knows a module has releases
1.0.0, 1.1.0, 2.0.0, and can resolve `~> 1.0` against that list.

A git URL is just a URL. Terraform has no way to enumerate available versions
from it, so there is nothing for a constraint to resolve against.

```text
Error: Invalid combination of "version" and "source"
```

For git sources, pin with a `ref` query parameter, which git *does* understand:

```hcl
source = "git::https://github.com/acme/modules.git//network?ref=v1.4.0"
```

`ref` accepts a tag, a branch or a commit SHA. Use a **tag** for releases — a
branch moves under you, which reintroduces the problem you were trying to avoid.

### 4. The empty plan after refactoring

**`moved` blocks made it possible.**

The problem: Terraform tracks resources by address. Moving `aws_vpc.main` into a
module changes its address to `module.network.aws_vpc.main`. Terraform sees an
address in state that is no longer in the configuration, and a new address in the
configuration that is not in state, and concludes: destroy one, create the other.

`moved` blocks tell it these are the same resource:

```hcl
moved {
  from = aws_vpc.main
  to   = module.network.aws_vpc.main
}
```

Terraform updates the address in state and plans no infrastructure change at all.

**Without them**, refactoring Notely into modules would have destroyed and
recreated every resource — the VPC, all six subnets, the security groups, the
S3 bucket. New IDs, new addresses, and for the bucket, **all the attachments
gone**. On a real system with a database, it would be a data-loss incident caused
purely by moving code between files.

This is why the empty plan matters so much: `0 to add, 0 to change, 0 to destroy`
is a *proof* that the refactor changed only code, not infrastructure. Treat it as
the acceptance test for any restructuring.

### 5. When to extract a module

**The second time you need the same thing** — not the first.

The first time you build something you do not yet know which parts vary between
uses and which are fixed. Extract too early and you invent an interface based on
a guess, then spend months bending it to fit cases you did not anticipate.

Concrete signals it is time:

- You are about to copy and paste a group of resources
- Several resources always appear together and are meaningless apart (a bucket
  plus its versioning, encryption and public-access-block)
- A section of a file has a clear boundary — you could describe it in one phrase
  like "the network" or "the web tier"
- More than one team or environment needs the same pattern

Signals it is **not** time:

- The module would wrap a single resource
- You have only one caller and no concrete plan for a second
- You cannot name what the module does without using the word "and" twice

Notely is a good example of the right timing: the extraction happened in Module 8
after the resources existed and their relationships were understood, not in
Module 2 when the shape was still being worked out.

### 6. Wrapping a single resource

Because it adds indirection and provides nothing in return.

```hcl
# modules/s3-bucket/main.tf
resource "aws_s3_bucket" "this" {
  bucket = var.name
}
```

A caller now writes `module "bucket" { source = "..." name = "x" }` instead of
`resource "aws_s3_bucket" "x" { bucket = "x" }`. Same length, but now the reader
has to open another directory to find out what it does, and every attribute they
want must be plumbed through as a variable and an output.

A module should encapsulate a **pattern** — several resources that belong
together and are easy to get wrong individually. Notely's `modules/storage` is
worth it because it bundles four resources: the bucket, versioning, encryption,
and the public access block. A caller gets a *correctly configured* bucket, and
cannot forget the public-access-block, which is the one that matters.

The test: does the module encode a decision, or just a name? If a decision, keep
it. If just a name, delete it.

### 7. `name_prefix` on the target group

Because of `create_before_destroy`.

The target group has:

```hcl
lifecycle {
  create_before_destroy = true
}
```

which means when it must be replaced, Terraform creates the new one **before**
destroying the old. For a moment both exist.

AWS requires target group names to be unique within an account and region. If the
resource used a fixed `name = "notely-dev-app"`, the create step would fail:

```text
Error: A target group with the same name 'notely-dev-app' exists
```

`name_prefix` asks AWS to generate a unique name from that prefix, so the two can
coexist during the swap.

The awkward detail: for target groups, `name_prefix` is limited to **six
characters**, which is why Notely uses the abbreviated `"ntly-"` rather than
something readable.

The same pattern applies to any resource combining `create_before_destroy` with a
unique name — IAM roles, launch templates, security groups. Most of them offer
`name_prefix` for exactly this reason.

---

## Next lesson

**Module 9 — Environments & Project Layout** (`09-environments-and-layout.md`)

Notely is modules now, and there is a problem you have been ignoring since
Module 4: `dev.tfvars` and `prod.tfvars` both exist, but running the prod one
would destroy dev, because they share a single state file.

Module 9 fixes it. It covers workspaces, why most teams do not use them for
environments, and the directory-per-environment layout that is the real answer —
after which Notely genuinely has dev, staging and prod side by side.
