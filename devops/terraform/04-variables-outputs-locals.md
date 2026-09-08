# Module 4 — Variables, Outputs & Locals

## Where Notely is right now

```text
  Notely so far:

    S3 bucket  (attachments)          <- Module 1
    VPC 10.0.0.0/16                   <- Module 2
      +- public subnet 10.0.1.0/24
      +- internet gateway
      +- route table
    CloudWatch log group              <- Module 3
    State in S3 with locking          <- Module 3

  Problem: every value is hardcoded.
```

By the end of this module, none of them are. Same infrastructure, but now you
can build a second copy for staging by changing one file.

Full picture: `notely-architecture.md`.

---

## Why this module exists

Look at Notely's code as it stands:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_a" {
  cidr_block        = "10.0.1.0/24"
  availability_zone = "ap-southeast-2a"
}

resource "aws_s3_bucket" "attachments" {
  bucket = "notely-attachments-a1b2c3d4"
}
```

Now your boss says: *"Set up a staging environment."*

With this code your options are all bad:

1. **Copy the whole directory and edit the values.** Now you have two copies of
   the same code that will drift apart within a month.
2. **Edit the values, apply, edit them back.** You have just destroyed
   production to make staging.
3. **Add a second set of resources with different names.** The file doubles in
   size and doubles again for the third environment.

Variables fix this. One copy of the code, different values fed in.

---

## The core idea (one sentence)

Variables turn your Terraform configuration from a fixed description of one
system into a reusable template that can build many.

> Hardcoded values are fine until you need a second copy. Then they are the only problem you have.

---

## Mental model (from Node.js)

You already know this idea. It is just functions.

```javascript
// Hardcoded. Only ever greets one person.
function greet() {
  return "Hello, Rahman";
}

// Parameterised. Greets anyone.
function greet(name) {
  return `Hello, ${name}`;
}
```

Terraform maps onto it directly:

| Node.js | Terraform |
|---|---|
| Function parameter | `variable` block |
| Default parameter value `function f(x = 5)` | `default` in a variable |
| `module.exports` / `return` | `output` block |
| A `const` inside the function | `locals` block |
| Calling `greet("Rahman")` | A `.tfvars` file |
| TypeScript type on a parameter | `type` in a variable |
| Throwing on a bad argument | `validation` block |

And the whole configuration directory is the function:

```javascript
// notely(environment, region, instanceCount) -> infrastructure
```

That is genuinely all this module is.

---

## Part 1 — `variable` blocks

### The simplest possible variable

```hcl
variable "environment" {}
```

That works, and you should never write it. Here is the full form:

```hcl
variable "environment" {
  description = "Which environment this is: dev, staging or prod"
  type        = string
  default     = "dev"
}
```

Four things you can set:

| Field | What it does |
|---|---|
| `description` | Documentation. Shows up in error messages and generated docs. |
| `type` | What kind of value is allowed |
| `default` | Used when nobody supplies a value. **Omit it to make the variable required.** |
| `sensitive` | Hides the value from CLI output |
| `nullable` | Whether `null` is an acceptable value (default `true`) |

### Using one

```hcl
resource "aws_s3_bucket" "attachments" {
  bucket = "notely-attachments-${var.environment}"
}
```

The `var.` prefix is how you read a variable. Always.

### Required vs optional

**No `default` = required.** Terraform will refuse to run without it:

```hcl
variable "db_password" {
  description = "The database password"
  type        = string
  sensitive   = true
  # no default - you MUST supply this
}
```

```bash
terraform plan
```

```text
var.db_password
  The database password

  Enter a value:
```

**With a `default` = optional.**

```hcl
variable "instance_type" {
  description = "EC2 instance size for the app servers"
  type        = string
  default     = "t3.micro"
}
```

> Give a default to anything with a sensible default. Leave it out for anything where guessing wrong would be bad — passwords, account IDs, environment names.

### Three examples of the same idea

**Example 1 — a simple string.**

```hcl
variable "region" {
  description = "AWS region to deploy into"
  type        = string
  default     = "ap-southeast-2"
}

provider "aws" {
  region = var.region
}
```

**Example 2 — a number.**

```hcl
variable "log_retention_days" {
  description = "How long to keep application logs"
  type        = number
  default     = 30
}

resource "aws_cloudwatch_log_group" "app" {
  name              = "/notely/app"
  retention_in_days = var.log_retention_days
}
```

**Example 3 — a boolean acting as a switch.**

```hcl
variable "enable_versioning" {
  description = "Keep old versions of uploaded attachments"
  type        = bool
  default     = true
}

resource "aws_s3_bucket_versioning" "attachments" {
  bucket = aws_s3_bucket.attachments.id

  versioning_configuration {
    status = var.enable_versioning ? "Enabled" : "Suspended"
  }
}
```

That `? :` is a ternary, exactly like JavaScript. Module 5 covers expressions
properly.

---

## Part 2 — Types

This is where your TypeScript knowledge pays off
(`../../backend/nodejs/typescript.md`).

### The primitives

```hcl
variable "name"    { type = string }
variable "count"   { type = number }
variable "enabled" { type = bool }
```

### Collections

```hcl
# A list - ordered, all the same type
variable "availability_zones" {
  type    = list(string)
  default = ["ap-southeast-2a", "ap-southeast-2b"]
}

# A set - unordered, no duplicates
variable "allowed_ports" {
  type    = set(number)
  default = [80, 443]
}

# A map - string keys, values all the same type
variable "tags" {
  type = map(string)
  default = {
    Project = "notely"
    Owner   = "rahman"
  }
}
```

The TypeScript equivalents:

| Terraform | TypeScript |
|---|---|
| `string` | `string` |
| `number` | `number` |
| `bool` | `boolean` |
| `list(string)` | `string[]` |
| `set(string)` | `Set<string>` |
| `map(string)` | `Record<string, string>` |
| `object({...})` | `interface { ... }` |
| `tuple([string, number])` | `[string, number]` |
| `any` | `any` |

### Objects — the useful one

An `object` lets each field have its own type. It is a TypeScript interface.

```hcl
variable "database" {
  description = "Settings for the Postgres database"

  type = object({
    instance_class    = string
    allocated_storage = number
    multi_az          = bool
    engine_version    = string
  })

  default = {
    instance_class    = "db.t3.micro"
    allocated_storage = 20
    multi_az          = false
    engine_version    = "16.3"
  }
}
```

Used like:

```hcl
resource "aws_db_instance" "notely" {
  instance_class    = var.database.instance_class
  allocated_storage = var.database.allocated_storage
  multi_az          = var.database.multi_az
}
```

The TypeScript this maps to:

```typescript
interface Database {
  instanceClass: string;
  allocatedStorage: number;
  multiAz: boolean;
  engineVersion: string;
}
```

### `optional()` — optional properties with defaults

Same as `?` in TypeScript, but better, because you can supply a default.

```hcl
variable "database" {
  type = object({
    instance_class    = string
    allocated_storage = optional(number, 20)
    multi_az          = optional(bool, false)
    backup_days       = optional(number, 7)
  })
}
```

Now a caller only has to provide `instance_class`:

```hcl
database = {
  instance_class = "db.t3.micro"
}
```

and the other three fill themselves in. This is extremely useful when writing
modules (Module 8).

### A list of objects

Combine them and you can describe quite complex things:

```hcl
variable "subnets" {
  description = "Notely's subnets"

  type = list(object({
    name              = string
    cidr_block        = string
    availability_zone = string
    public            = bool
  }))

  default = [
    { name = "public-a",  cidr_block = "10.0.1.0/24",  availability_zone = "ap-southeast-2a", public = true },
    { name = "public-b",  cidr_block = "10.0.2.0/24",  availability_zone = "ap-southeast-2b", public = true },
    { name = "app-a",     cidr_block = "10.0.11.0/24", availability_zone = "ap-southeast-2a", public = false },
    { name = "app-b",     cidr_block = "10.0.12.0/24", availability_zone = "ap-southeast-2b", public = false },
  ]
}
```

That single variable describes Notely's entire subnet layout. In Module 6 you
will loop over it and create all four with about six lines of code.

### `any` — use sparingly

```hcl
variable "whatever" {
  type = any
}
```

Terraform accepts anything. Like `any` in TypeScript, it turns off the checking
that was helping you. Use a real type unless you genuinely cannot.

---

## Part 3 — Validation

A `validation` block rejects bad values before Terraform touches AWS.

### The shape

```hcl
variable "environment" {
  description = "Which environment this is"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, staging, prod."
  }
}
```

- `condition` must evaluate to `true` for the value to be accepted
- `error_message` is what the user sees when it is not

```bash
terraform plan -var environment=production
```

```text
Error: Invalid value for variable

  on variables.tf line 1:
   1: variable "environment" {

Environment must be one of: dev, staging, prod.
```

Caught in half a second, before a single API call.

### More examples

**Example 1 — a naming rule.**

```hcl
variable "project_name" {
  type = string

  validation {
    condition     = can(regex("^[a-z][a-z0-9-]{2,20}$", var.project_name))
    error_message = "Project name must be lowercase letters, numbers and hyphens, 3-21 characters, starting with a letter."
  }
}
```

**Example 2 — a numeric range.**

```hcl
variable "app_server_count" {
  type    = number
  default = 2

  validation {
    condition     = var.app_server_count >= 2 && var.app_server_count <= 10
    error_message = "Notely needs at least 2 servers for high availability, and no more than 10."
  }
}
```

That error message teaches something. Good validation messages explain *why*,
not just *what*.

**Example 3 — validating a CIDR block.**

```hcl
variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"

  validation {
    condition     = can(cidrnetmask(var.vpc_cidr))
    error_message = "vpc_cidr must be a valid CIDR block, for example 10.0.0.0/16."
  }

  validation {
    condition     = tonumber(split("/", var.vpc_cidr)[1]) <= 16
    error_message = "vpc_cidr must be /16 or larger. Notely needs room for six subnets."
  }
}
```

Two `validation` blocks on one variable is allowed and encouraged — each checks
one thing and gives its own message.

### `can()` and `try()`

`can()` runs an expression and returns `true` or `false` instead of erroring.
That is what makes it useful inside `condition`:

```hcl
can(regex("^10\\.", var.vpc_cidr))     # true if it starts with 10.
can(cidrnetmask(var.vpc_cidr))          # true if it is valid CIDR
```

Think of it as a `try/catch` that returns a boolean.

---

## Part 4 — Sensitive variables

```hcl
variable "db_password" {
  description = "Password for the Notely database"
  type        = string
  sensitive   = true
}
```

What this does:

```text
  + resource "aws_db_instance" "notely" {
      + password = (sensitive value)
    }
```

What it does **not** do:

- It does not encrypt anything
- It does not keep the value out of the state file
- It does not stop `terraform output -raw` printing it

Sensitivity is also contagious: any expression built from a sensitive value
becomes sensitive too, which is helpful, and occasionally annoying when it hides
output you wanted to see.

> `sensitive = true` hides values from your terminal. That is all it does. The state file still holds the password in plaintext. Module 10 explains what to do about that.

---

## Part 5 — The six ways to set a variable

This trips people up constantly, so learn the order.

From **lowest** priority to **highest**:

| # | Method | Example |
|---|---|---|
| 1 | The `default` in the variable block | `default = "dev"` |
| 2 | Environment variable | `export TF_VAR_environment=staging` |
| 3 | `terraform.tfvars` | `environment = "staging"` |
| 4 | `terraform.tfvars.json` | `{"environment": "staging"}` |
| 5 | `*.auto.tfvars` (alphabetical order) | `prod.auto.tfvars` |
| 6 | `-var` and `-var-file` on the command line | `-var environment=prod` |

**Later beats earlier.** A `-var` flag beats everything.

### Which files load automatically

| File | Loaded automatically? |
|---|---|
| `terraform.tfvars` | **Yes** |
| `terraform.tfvars.json` | **Yes** |
| `anything.auto.tfvars` | **Yes** |
| `dev.tfvars` | **No** — you must pass `-var-file=dev.tfvars` |
| `prod.tfvars` | **No** — same |

This is the source of a lot of confusion. `dev.tfvars` looks like it should load.
It does not. Only the exact name `terraform.tfvars`, or anything ending
`.auto.tfvars`.

> The naming convention `dev.tfvars` / `prod.tfvars` **requires** `-var-file`. That is a feature: you cannot accidentally apply prod values by being in the wrong directory.

### Worked example

`variables.tf`:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}
```

`terraform.tfvars`:

```hcl
environment = "staging"
```

Then:

```bash
terraform plan
# -> "staging"   (tfvars beats default)

TF_VAR_environment=qa terraform plan
# -> "staging"   (tfvars still beats the env var)

terraform plan -var environment=prod
# -> "prod"      (-var beats everything)
```

### The `TF_VAR_` prefix

Any environment variable named `TF_VAR_<name>` sets the variable `<name>`:

```bash
export TF_VAR_db_password="s3cr3t"
export TF_VAR_environment="prod"
terraform apply
```

This is how CI pipelines inject secrets. The pipeline sets `TF_VAR_db_password`
from a masked variable, and it never appears in a file.

---

## Part 6 — Outputs

An `output` returns a value from your configuration. It is `module.exports`.

```hcl
output "attachments_bucket_name" {
  description = "Name of Notely's attachments bucket"
  value       = aws_s3_bucket.attachments.bucket
}
```

Outputs do three jobs:

1. **Show you something after apply** — the load balancer's DNS name, the
   database endpoint
2. **Feed other configurations** — via `terraform_remote_state` (Module 7)
3. **Form a module's public interface** — what the module gives back (Module 8)

### Reading outputs

```bash
terraform output                              # all of them
terraform output attachments_bucket_name      # one, with quotes
terraform output -raw attachments_bucket_name # one, no quotes - use in scripts
terraform output -json                        # all, as JSON
```

The `-raw` version matters:

```bash
# Wrong - includes the quotes
aws s3 ls s3://$(terraform output attachments_bucket_name)

# Right
aws s3 ls s3://$(terraform output -raw attachments_bucket_name)
```

### Sensitive outputs

```hcl
output "db_connection_string" {
  value     = "postgres://notely:${random_password.db.result}@${aws_db_instance.notely.endpoint}/notely"
  sensitive = true
}
```

```bash
terraform output db_connection_string
```

```text
db_connection_string = <sensitive>
```

If an output's value comes from something sensitive, Terraform **forces** you to
mark the output sensitive too. Forgetting produces an error, which is the tool
protecting you.

### Preconditions on outputs

An output can assert something is true before it is returned:

```hcl
output "app_url" {
  description = "The public URL for Notely"
  value       = "https://${aws_lb.main.dns_name}"

  precondition {
    condition     = length(aws_lb.main.subnets) >= 2
    error_message = "The load balancer must span at least two subnets for high availability."
  }
}
```

If the condition fails, the apply fails with your message. It is a test that runs
on every apply.

---

## Part 7 — Locals

A `local` is a named value computed inside your configuration.

```hcl
locals {
  name_prefix = "notely-${var.environment}"

  common_tags = {
    Project     = "notely"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

Read with `local.` (singular, even though the block is `locals`):

```hcl
resource "aws_s3_bucket" "attachments" {
  bucket = "${local.name_prefix}-attachments"
  tags   = local.common_tags
}
```

### Variables vs locals

The distinction is simple once you see it:

| | `variable` | `local` |
|---|---|---|
| **Set by** | Whoever runs Terraform | Your code |
| **Can be changed at run time** | Yes | No |
| **Can reference other values** | No | **Yes** |
| **Node.js equivalent** | Function parameter | `const` inside the function |

The killer difference is that last-but-one row. A variable's `default` **cannot**
reference anything:

```hcl
variable "bucket_name" {
  default = "notely-${var.environment}"   # ERROR - not allowed
}
```

A local can:

```hcl
locals {
  bucket_name = "notely-${var.environment}-attachments"   # fine
}
```

> If the value is an input, use a variable. If it is computed from inputs, use a local.

### Three examples

**Example 1 — a name prefix, used everywhere.**

```hcl
locals {
  name_prefix = "notely-${var.environment}"
}

# notely-dev-vpc, notely-dev-alb, notely-dev-db ...
```

Change `var.environment` and every name in the whole project updates.

**Example 2 — conditional sizing.**

```hcl
locals {
  is_production = var.environment == "prod"

  instance_type    = local.is_production ? "t3.medium" : "t3.micro"
  server_count     = local.is_production ? 4 : 1
  multi_az         = local.is_production
  log_retention    = local.is_production ? 90 : 7
  deletion_protect = local.is_production
}
```

One boolean drives five decisions. This is a very common and very readable
pattern.

**Example 3 — merged tags.**

```hcl
locals {
  common_tags = {
    Project     = "notely"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

resource "aws_db_instance" "notely" {
  tags = merge(local.common_tags, {
    Name       = "${local.name_prefix}-db"
    BackupPlan = "daily"
  })
}
```

`merge()` combines maps; later values win. Every resource gets the common tags
plus its own.

---

## Building it into Notely

Time to convert Notely from hardcoded to parameterised.

### Before

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "notely-vpc" }
}

resource "aws_subnet" "public_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "ap-southeast-2a"
  tags = { Name = "notely-public-a" }
}

resource "aws_s3_bucket" "attachments" {
  bucket = "notely-attachments-a1b2c3d4"
}

resource "aws_cloudwatch_log_group" "app" {
  name              = "/notely/app-a1b2c3d4"
  retention_in_days = 1
}
```

### After

Create `variables.tf`:

```hcl
variable "project_name" {
  description = "Short name for this project, used as a prefix on every resource"
  type        = string
  default     = "notely"

  validation {
    condition     = can(regex("^[a-z][a-z0-9-]{2,20}$", var.project_name))
    error_message = "project_name must be lowercase letters, numbers and hyphens, 3-21 characters."
  }
}

variable "environment" {
  description = "Which environment this is"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment must be one of: dev, staging, prod."
  }
}

variable "region" {
  description = "AWS region to deploy into"
  type        = string
  default     = "ap-southeast-2"
}

variable "vpc_cidr" {
  description = "Address range for the whole VPC"
  type        = string
  default     = "10.0.0.0/16"

  validation {
    condition     = can(cidrnetmask(var.vpc_cidr))
    error_message = "vpc_cidr must be a valid CIDR block, for example 10.0.0.0/16."
  }
}

variable "public_subnet_cidr" {
  description = "Address range for the first public subnet"
  type        = string
  default     = "10.0.1.0/24"
}

variable "availability_zone" {
  description = "Which AZ the first subnet lives in"
  type        = string
  default     = "ap-southeast-2a"
}

variable "log_retention_days" {
  description = "How many days to keep application logs"
  type        = number
  default     = 7

  validation {
    condition     = contains([1, 3, 5, 7, 14, 30, 60, 90, 365], var.log_retention_days)
    error_message = "log_retention_days must be a value CloudWatch accepts: 1, 3, 5, 7, 14, 30, 60, 90 or 365."
  }
}

variable "enable_versioning" {
  description = "Keep old versions of uploaded attachments"
  type        = bool
  default     = true
}
```

Create `locals.tf`:

```hcl
locals {
  # Everything in Notely is named "notely-dev-something".
  # Change var.environment and all of it updates at once.
  name_prefix = "${var.project_name}-${var.environment}"

  # One switch that drives several decisions.
  is_production = var.environment == "prod"

  # Tags applied to every resource that supports them.
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Ephemeral   = local.is_production ? "false" : "true"
  }
}
```

Now `network.tf` becomes:

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidr
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-a"
    Tier = "public"
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public.id
}
```

And `main.tf`:

```hcl
resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "attachments" {
  bucket = "${local.name_prefix}-attachments-${random_id.suffix.hex}"

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-attachments"
  })
}

resource "aws_s3_bucket_versioning" "attachments" {
  bucket = aws_s3_bucket.attachments.id

  versioning_configuration {
    status = var.enable_versioning ? "Enabled" : "Suspended"
  }
}

resource "aws_cloudwatch_log_group" "app" {
  name              = "/${var.project_name}/${var.environment}/app"
  retention_in_days = var.log_retention_days

  tags = local.common_tags
}
```

Update the provider to use the variable:

```hcl
provider "aws" {
  region = var.region

  default_tags {
    tags = local.common_tags
  }
}
```

With `default_tags` set, you can actually drop the `tags = merge(...)` from most
resources — the provider applies them everywhere. Keep the merge only where you
need a resource-specific `Name`.

### The environment files

`dev.tfvars`:

```hcl
environment        = "dev"
vpc_cidr           = "10.0.0.0/16"
public_subnet_cidr = "10.0.1.0/24"
log_retention_days = 7
enable_versioning  = false
```

`prod.tfvars`:

```hcl
environment        = "prod"
vpc_cidr           = "10.1.0.0/16"
public_subnet_cidr = "10.1.1.0/24"
log_retention_days = 90
enable_versioning  = true
```

Note prod uses `10.1.0.0/16`. Different environments should not overlap — if you
ever peer the VPCs together, overlapping ranges make it impossible.

### Run it

```bash
terraform plan -var-file=dev.tfvars
```

Everything renames itself to `notely-dev-*`. Because names changed, some
resources will be replaced — that is expected on this one-off migration.

```bash
terraform apply -var-file=dev.tfvars
```

Now try the other environment **without applying**:

```bash
terraform plan -var-file=prod.tfvars
```

Terraform proposes to rebuild everything with prod values in a `10.1.0.0/16`
VPC. Do not apply it — you would be replacing dev, because both environments
still share one state file. Fixing that is Module 9.

But look at what you just proved: **the same code can build a completely
different environment.** That is the whole point of this module.

### Don't commit your tfvars

```bash
echo "*.tfvars" >> .gitignore
echo "!*.tfvars.example" >> .gitignore
cp dev.tfvars dev.tfvars.example
```

`.tfvars` files often end up holding secrets. Commit an `.example` instead so
colleagues know what to fill in.

---

## Real-World Example

A team runs the same Terraform code for four environments: `dev`, `staging`,
`prod`, and a per-developer `sandbox`.

Their entire environment difference is one file each:

```text
envs/
├── dev.tfvars        20 lines
├── staging.tfvars    20 lines
├── prod.tfvars       22 lines
└── sandbox.tfvars    18 lines
```

`prod.tfvars`:

```hcl
environment          = "prod"
vpc_cidr             = "10.2.0.0/16"
app_instance_type    = "t3.large"
app_min_size         = 4
app_max_size         = 20
db_instance_class    = "db.r6g.xlarge"
db_multi_az          = true
db_backup_retention  = 30
log_retention_days   = 365
enable_deletion_protection = true
```

`sandbox.tfvars`:

```hcl
environment          = "sandbox"
vpc_cidr             = "10.9.0.0/16"
app_instance_type    = "t3.micro"
app_min_size         = 1
app_max_size         = 1
db_instance_class    = "db.t4g.micro"
db_multi_az          = false
db_backup_retention  = 0
log_retention_days   = 1
enable_deletion_protection = false
```

Same code. Prod costs about $900/month, sandbox about $25.

**Why this matters more than it looks.** When a developer tests something in
sandbox, they are testing the *actual production code*, just with smaller
numbers. A change that works in sandbox works in prod, because there is only one
codebase. The environments cannot drift apart, because there is nothing to
drift.

Compare that with four copied directories, where somebody fixes a bug in prod
and forgets staging, and six weeks later a deploy fails for reasons nobody can
explain.

---

## Common Mistakes Beginners Make

**1. Trying to reference a variable inside another variable's default.**

```hcl
variable "bucket_name" {
  default = "notely-${var.environment}"   # ERROR
}
```

Variable defaults must be literal. Use a `local` instead.

**2. Expecting `dev.tfvars` to load automatically.**

It does not. Only `terraform.tfvars` and `*.auto.tfvars` auto-load. Pass
`-var-file=dev.tfvars`.

**3. Thinking `sensitive = true` protects the state file.**

It hides terminal output. Nothing more.

**4. Committing `.tfvars` files.**

They frequently hold secrets. Gitignore them, commit `.tfvars.example`.

**5. Variables with no `description`.**

Six months later you will not remember what `var.mode` does. Neither will your
colleague, who is trying to use your module.

**6. Using `any` because the type is annoying to write.**

You lose the error that would have caught the mistake at plan time.

**7. Making everything a variable.**

Not every value needs to be configurable. If nobody will ever change it, hardcode
it or use a local. A module with 60 variables is unusable.

**8. Forgetting `-raw` when using an output in a script.**

```bash
BUCKET=$(terraform output bucket_name)   # BUCKET is "my-bucket" with quotes
BUCKET=$(terraform output -raw bucket_name)   # correct
```

**9. Not validating anything.**

A `validation` block takes 30 seconds to write and catches mistakes before they
reach AWS. The best value-for-effort in all of Terraform.

---

## Hands-On Lab — Parameterise Notely

**Cost: free.** Everything here is a VPC, subnets, S3 and CloudWatch.

### Goal

Convert Notely from hardcoded to fully parameterised, then prove the same code
can describe two different environments.

### Step 1: Set up

```bash
cd ~/terraform-labs/notely
```

You should already have `main.tf`, `network.tf`, `outputs.tf` from Modules 1–3.

### Step 2: Create the variables

Create `variables.tf` and `locals.tf` exactly as shown in "Building it into
Notely" above. Type them out.

### Step 3: Convert the resources

Update `main.tf` and `network.tf` to use `var.` and `local.` everywhere. Every
literal string that describes a choice should become a variable.

```bash
terraform fmt
terraform validate
```

### Step 4: Prove the validation works

```bash
terraform plan -var environment=production
```

```text
Error: Invalid value for variable
environment must be one of: dev, staging, prod.
```

Now a bad CIDR:

```bash
terraform plan -var environment=dev -var vpc_cidr=not-a-cidr
```

```text
Error: Invalid value for variable
vpc_cidr must be a valid CIDR block, for example 10.0.0.0/16.
```

Neither of those made a single AWS API call. That is the value of validation.

### Step 5: Prove the precedence order

Add a default:

```hcl
variable "environment" {
  type    = string
  default = "dev"
  # ... validation as before
}
```

Create `terraform.tfvars`:

```hcl
environment = "staging"
```

Now run three plans and watch which value wins. Use `terraform console` to see it
cleanly:

```bash
echo 'var.environment' | terraform console
# staging   <- tfvars beat the default

TF_VAR_environment=qa terraform console <<< 'var.environment'
# staging   <- tfvars still win; but "qa" would fail validation anyway
```

```bash
terraform console -var environment=prod <<< 'var.environment'
# prod      <- -var beats everything
```

Delete `terraform.tfvars` afterwards — for Notely we want explicit
`-var-file=dev.tfvars`.

### Step 6: Create the environment files

Create `dev.tfvars` and `prod.tfvars` as shown above.

```bash
terraform plan -var-file=dev.tfvars
```

### Step 7: Apply dev

```bash
terraform apply -var-file=dev.tfvars
```

Everything is renamed `notely-dev-*`. Check:

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=notely-dev-vpc" \
  --query 'Vpcs[0].Tags' --output table
```

You should see all your `common_tags` there, applied automatically by
`default_tags`.

### Step 8: See what prod would look like — without applying

```bash
terraform plan -var-file=prod.tfvars
```

Read the plan. Everything is being replaced with prod values in `10.1.0.0/16`.

**Do not apply this.** Both environments share one state file, so applying would
destroy dev to build prod. Module 9 fixes that.

### Step 9: Play with locals

```bash
terraform console -var-file=prod.tfvars
```

```text
> local.name_prefix
"notely-prod"

> local.is_production
true

> local.common_tags
{
  "Environment" = "prod"
  "Ephemeral" = "false"
  "ManagedBy" = "Terraform"
  "Project" = "notely"
}

> merge(local.common_tags, { Name = "test" })
{
  "Environment" = "prod"
  "Ephemeral" = "false"
  "ManagedBy" = "Terraform"
  "Name" = "test"
  "Project" = "notely"
}
```

Notice `local.is_production` is `true` here and would be `false` with
`dev.tfvars`. One variable, cascading through your whole configuration.

### Step 10: Tear down (or leave it)

Notely's current resources are all free, so you can leave them running for
Module 5. If you would rather be clean:

```bash
terraform destroy -var-file=dev.tfvars
```

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

### What you should have at the end

- Notely with zero hardcoded values
- Two validation rules that reject bad input before touching AWS
- Two environment files describing different systems from one codebase
- Understanding of why `dev.tfvars` needs `-var-file`

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **`variable`** | An input. Like a function parameter. |
| **`var.name`** | How you read one |
| **No `default`** | The variable is required |
| **`type`** | `string`, `number`, `bool`, `list()`, `set()`, `map()`, `object()` |
| **`object({...})`** | A TypeScript interface |
| **`optional(type, default)`** | An optional field with a fallback |
| **`validation`** | Rejects bad values before any API call |
| **`can()`** | Runs an expression, returns true/false instead of erroring |
| **`sensitive = true`** | Hides from terminal output. **Not** from state. |
| **`output`** | A return value. `module.exports`. |
| **`terraform output -raw`** | No quotes — the one you want in scripts |
| **`locals`** | Computed values. `const` inside a function. |
| **`local.name`** | How you read one (singular) |
| **Variable vs local** | Variable = input from outside. Local = computed inside. |
| **`merge()`** | Combines maps; later values win |
| **`terraform.tfvars`** | Loads automatically |
| **`*.auto.tfvars`** | Loads automatically |
| **`dev.tfvars`** | Does **not** load automatically. Use `-var-file`. |
| **`TF_VAR_name`** | Environment variable form. How CI injects secrets. |
| **Precedence** | default < `TF_VAR_` < `terraform.tfvars` < `*.auto.tfvars` < `-var` |

---

## Checkpoint (answer briefly)

1. Why can a `local` reference `var.environment`, but a variable's `default` cannot?
2. You have `dev.tfvars` in your directory and run `terraform plan`. Does it get used?
3. What is the difference between `type = object({...})` and `type = map(string)`?
4. A variable is marked `sensitive = true`. Name one thing that protects and one thing it does not.
5. You set a default of `"dev"`, put `environment = "staging"` in `terraform.tfvars`, and run `terraform apply -var environment=prod`. Which wins?
6. When should something be a variable, and when should it be a local?
7. Why does `optional(number, 20)` make a module nicer to use?

---

## Checkpoint — model answers

### 1. Locals can reference variables; defaults cannot

Because of **when** each one is resolved.

A variable's `default` is part of the variable's *declaration*. Terraform reads
all variable declarations first, before it knows any values — including the
values of other variables. So a default has nothing to reference yet. It must be
a literal.

```hcl
variable "bucket_name" {
  default = "notely-${var.environment}"   # ERROR: Variables not allowed
}
```

Locals are evaluated *after* all variables have their final values. By then
`var.environment` is a known string, so a local can build on it freely.

```hcl
locals {
  bucket_name = "notely-${var.environment}-attachments"   # fine
}
```

This is the practical reason most projects have both: variables for the raw
inputs, locals for everything derived from them.

### 2. Does `dev.tfvars` load automatically?

**No.**

Terraform automatically loads exactly two things:

- The file named exactly `terraform.tfvars` (or `terraform.tfvars.json`)
- Any file ending in `.auto.tfvars` (or `.auto.tfvars.json`)

`dev.tfvars` matches neither. Running `terraform plan` would use the defaults, or
prompt you for any required variable, and silently ignore the file sitting right
there.

You have to pass it explicitly:

```bash
terraform plan -var-file=dev.tfvars
```

This is deliberate and good. If `dev.tfvars` and `prod.tfvars` both auto-loaded,
they would conflict. Requiring the flag means you always state which environment
you mean, which makes it much harder to apply prod values by accident.

### 3. `object({...})` vs `map(string)`

**`map(string)`** — any number of keys, all values the same type.

```hcl
type = map(string)

# valid
{ Project = "notely", Owner = "rahman", Anything = "at all" }
```

You do not know the keys in advance, and Terraform will not check them. Good for
tags.

**`object({...})`** — a fixed set of named attributes, each with its own type.

```hcl
type = object({
  instance_class    = string
  allocated_storage = number
  multi_az          = bool
})
```

The keys are part of the type. Misspell one, or omit one, and Terraform errors at
plan time. Values can be different types.

In TypeScript terms: `map(string)` is `Record<string, string>`, `object({...})`
is an `interface`.

Rule of thumb: keys unknown and homogeneous → map. Keys known and heterogeneous →
object.

### 4. What `sensitive = true` does and does not do

**It protects:** terminal output. The value shows as `(sensitive value)` in plan
output, apply output, and `terraform output`. It also propagates — anything
computed from it is treated as sensitive too, so you cannot leak it by
interpolating it into a string.

**It does not protect:** the **state file**. The value is stored there in
plaintext, exactly as if you had never marked it. `grep` finds it in seconds.

It also does not stop `terraform output -raw` from printing it, and it does not
encrypt anything anywhere.

So the real protections for a secret are: encrypt state at rest, lock down who
can read the state bucket, never commit state, and preferably do not put the
secret in Terraform at all — generate it and store it in Secrets Manager, which
is Module 10.

### 5. Which value wins?

**`prod`.**

The precedence order, lowest to highest:

1. `default = "dev"` — lowest
2. `TF_VAR_environment`
3. `terraform.tfvars` → `"staging"`
4. `*.auto.tfvars`
5. `-var` / `-var-file` on the command line → `"prod"` — highest

`-var` beats everything, so `prod` wins.

The way to remember it: **the more explicitly and the more recently you said it,
the more it counts.** A default is you guessing in advance; a `-var` flag is you
typing it right now.

### 6. Variable or local?

**Variable** when the value is an **input** — something the person running
Terraform should be able to choose. Environment name, region, instance size,
whether to enable multi-AZ.

**Local** when the value is **computed** from other values, or is a constant that
nobody outside should change. A name prefix built from project and environment, a
merged tag map, an `is_production` boolean derived from the environment name.

Two practical tests:

- *Would somebody sensibly want to pass a different value?* → variable
- *Is it just the same expression repeated in five places?* → local

The second one is the everyday case. If you catch yourself writing
`"notely-${var.environment}"` in eight resources, that is a local.

Over-using variables is a real mistake: a module with sixty variables is harder
to use than one with eight, because every extra variable is a decision the caller
now has to make.

### 7. Why `optional()` makes a module nicer

Because it lets a caller supply only what they care about.

Without it, every attribute of an object is mandatory:

```hcl
type = object({
  instance_class    = string
  allocated_storage = number
  multi_az          = bool
  backup_days       = number
})
```

Even a caller who is happy with all the defaults must write all four:

```hcl
database = {
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  multi_az          = false
  backup_days       = 7
}
```

With `optional()`:

```hcl
type = object({
  instance_class    = string
  allocated_storage = optional(number, 20)
  multi_az          = optional(bool, false)
  backup_days       = optional(number, 7)
})
```

the caller writes:

```hcl
database = {
  instance_class = "db.t3.micro"
}
```

and the rest fill themselves in.

This is the difference between a module that is pleasant to adopt and one where
every use site is thirty lines of boilerplate. It is the same reason default
parameter values exist in JavaScript.

---

## Next lesson

**Module 5 — Expressions & Functions** (`05-expressions-and-functions.md`)

You now have variables feeding into Notely. Module 5 is about *transforming*
them: `for` expressions (which are `.map()` and `.filter()`), the built-in
function library, `templatefile()` for the script that installs Node.js on the
servers, and `dynamic` blocks for generating repeated configuration.

That is the last piece of the language before Module 6 lets you create many
resources at once — and finally gives Notely its second availability zone, its
load balancer, and its servers.
