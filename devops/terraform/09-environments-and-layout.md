# Module 9 — Environments & Project Layout

## Where Notely is right now

```text
  notely/
  ├── main.tf          four module blocks
  ├── variables.tf
  ├── outputs.tf
  ├── dev.tfvars       environment = "dev",  vpc_cidr = 10.0.0.0/16
  ├── prod.tfvars      environment = "prod", vpc_cidr = 10.1.0.0/16
  └── modules/network, security, storage, web

  ONE state file:  s3://notely-tfstate-xxx/notely/dev/terraform.tfstate
```

The problem you have been ignoring since Module 4: **both tfvars files point at
the same state file.** Running `terraform apply -var-file=prod.tfvars` would not
create prod — it would *convert* dev into prod, destroying everything on the way.

By the end of this module Notely has three genuinely independent environments.

Full picture: `notely-architecture.md`.

---

## Why this module exists

Try it and see what happens:

```bash
terraform apply -var-file=dev.tfvars     # builds dev
terraform plan -var-file=prod.tfvars     # what does this say?
```

```text
Plan: 23 to add, 0 to change, 23 to destroy.
```

Terraform reads the state file, sees dev's resources, compares them with prod's
configuration, and concludes: destroy all of dev, build prod in its place.

The configuration was never the problem. **The state file is.** One state file
can only describe one deployment.

So the question this module answers is: how do you get several state files from
one codebase?

---

## The core idea (one sentence)

An environment is a separate state file, and how you get separate state files is
the only real decision here.

> Environments are not a Terraform feature. They are a consequence of how you split state.

---

## Mental model (from Node.js)

You already solve this problem in application code, and there are two familiar
shapes.

**Shape A — one deployment, config switched at runtime:**

```javascript
const config = require(`./config/${process.env.NODE_ENV}`);
```

One codebase, one running process, behaviour selected by an environment
variable. This is roughly what **workspaces** do.

**Shape B — separate deployments:**

```text
apps/
├── api-dev/       .env, own database, own domain
├── api-staging/   .env, own database, own domain
└── api-prod/      .env, own database, own domain
```

Separate deployments that happen to run the same code. This is
**directory-per-environment**, and it is what most teams use.

The difference matters because of blast radius. In shape A, a mistake in the
switching logic affects everything. In shape B, prod is a different deployment
that you have to deliberately go and touch.

---

## Part 1 — Workspaces

Terraform has a built-in feature called workspaces. It is the first thing people
find, and usually the wrong answer for environments — but you need to know it,
because you will inherit projects that use it.

### How they work

```bash
terraform workspace list        # show them
terraform workspace new staging # create one
terraform workspace select dev  # switch
terraform workspace show        # which am I in?
terraform workspace delete qa   # remove one
```

Every configuration starts in a workspace called `default`.

### What a workspace actually changes

**Only the state file path.** That is it.

With an S3 backend and `key = "notely/terraform.tfstate"`:

| Workspace | State file location |
|---|---|
| `default` | `notely/terraform.tfstate` |
| `dev` | `env:/dev/notely/terraform.tfstate` |
| `prod` | `env:/prod/notely/terraform.tfstate` |

Same bucket. Same credentials. Same configuration. Different key.

### Using `terraform.workspace`

The workspace name is available as an expression:

```hcl
locals {
  environment = terraform.workspace

  config = {
    dev = {
      instance_type = "t3.micro"
      az_count      = 2
      multi_az      = false
    }
    staging = {
      instance_type = "t3.small"
      az_count      = 2
      multi_az      = false
    }
    prod = {
      instance_type = "t3.large"
      az_count      = 3
      multi_az      = true
    }
  }

  settings = local.config[terraform.workspace]
}

module "web" {
  source        = "./modules/web"
  instance_type = local.settings.instance_type
}
```

This is the common workspace pattern. It works.

### Why most teams do not use them for environments

Six reasons, and the first is the one that ends the argument.

**1. One command stands between you and production.**

```bash
terraform workspace select prod
# ... get distracted, lunch, a meeting ...
terraform apply     # you meant dev
```

There is no visual difference. Same directory, same files, same prompt. The only
indicator is `terraform workspace show`, which you have to remember to run.

With directories, you are in `envs/prod/`. Your shell prompt says so. `ls` says
so. It is much harder to be in the wrong place without noticing.

**2. One backend, so one set of credentials.**

All workspaces share the same backend block. That means the same bucket and the
same IAM permissions. You cannot say "only the CI role may write prod state" —
anyone who can write dev state can write prod state.

**3. One configuration, so environments cannot differ structurally.**

Prod needs a read replica and a WAF that dev does not. With workspaces you get:

```hcl
count = terraform.workspace == "prod" ? 1 : 0
```

sprinkled through your code. Ten of those and the configuration is hard to read
and hard to reason about.

**4. No per-environment provider settings.**

Different AWS accounts per environment — a very common and very sensible
setup — is awkward, because the provider block is shared.

**5. `terraform.workspace` scatters environment logic.**

The name can be referenced anywhere. Finding everywhere an environment is
special-cased means grepping the whole codebase.

**6. Blast radius.**

A destructive change to the shared configuration can hit every environment at
once. With separate directories, prod has its own copy that you must edit
deliberately.

### When workspaces ARE the right tool

They are genuinely good for **short-lived, identical copies**:

- A per-pull-request preview environment
- A per-developer sandbox
- Testing a change in an isolated copy before merging

```bash
terraform workspace new pr-4821
terraform apply
# ... run tests ...
terraform destroy
terraform workspace delete pr-4821
```

That is exactly what they were designed for. The environments are identical, they
share credentials legitimately, and they are gone in an hour.

> Workspaces are for many copies of the *same* thing. Environments are usually not the same thing.

### A warning about the name

**Terraform Cloud / HCP Terraform workspaces are a completely different concept**
that happens to share the word. A TFC workspace is a full project with its own
variables, credentials, run history and permissions — much closer to a
directory-per-environment than to a CLI workspace.

---

## Part 2 — Directory per environment

The layout most teams settle on:

```text
notely/
├── modules/
│   ├── network/
│   ├── security/
│   ├── storage/
│   └── web/
└── envs/
    ├── dev/
    │   ├── main.tf          calls the modules
    │   ├── backend.tf       its own state key
    │   ├── variables.tf
    │   ├── terraform.tfvars dev's values
    │   └── outputs.tf
    ├── staging/
    │   └── ... same files, different values
    └── prod/
        └── ... same files, different values
```

Each `envs/*` directory is its own root module with its own state.

```bash
cd envs/dev
terraform init
terraform apply

cd ../prod
terraform init      # completely separate state
terraform apply
```

### What this gives you

| Property | Why it matters |
|---|---|
| **Separate state** | dev cannot affect prod. Structurally impossible. |
| **Separate backends** | Different buckets, different IAM, different accounts |
| **Separate providers** | Different regions, different roles per environment |
| **Visible context** | Your working directory tells you where you are |
| **Structural differences** | prod can have resources dev does not, with no `count` tricks |
| **Independent applies** | Deploying dev does not touch prod's state at all |
| **Reviewable** | A PR touching `envs/prod/` is obviously a production change |

That last one is underrated. In code review, a diff under `envs/prod/` gets a
different level of attention than one under `envs/dev/`. With workspaces, both
changes look identical.

### The duplication objection

"But now I have the same `main.tf` three times."

Two answers.

**First, it is not much.** Because everything real lives in modules, each
environment's `main.tf` is a thin wrapper:

```hcl
# envs/prod/main.tf
module "notely" {
  source = "../../modules/notely"

  project_name       = "notely"
  environment        = "prod"
  vpc_cidr           = "10.2.0.0/16"
  az_count           = 3
  app_instance_type  = "t3.large"
  log_retention_days = 365
  enable_versioning  = true
}
```

About fifteen lines. Duplicating fifteen lines three times is not a maintenance
problem.

**Second, the duplication is the feature.** Environments *should* be able to
differ. Forcing them to be identical is how you end up with `count =
terraform.workspace == "prod" ? 1 : 0` everywhere.

### Reducing it further with a wrapper module

You can go one step further and make a single `modules/notely` that composes the
four sub-modules, so each environment calls exactly one module. That is what the
example above does, and it is what the capstone in Module 14 uses.

```text
modules/
├── network/
├── security/
├── storage/
├── web/
└── notely/        <- composes the four above
```

Each `envs/*/main.tf` then has one `module` block.

---

## Part 3 — Backend configuration per environment

Each environment needs its own state key.

### Option A: a `backend.tf` in each directory

```hcl
# envs/dev/backend.tf
terraform {
  backend "s3" {
    bucket       = "notely-tfstate-a1b2c3d4"
    key          = "notely/dev/terraform.tfstate"
    region       = "ap-southeast-2"
    encrypt      = true
    use_lockfile = true
  }
}
```

```hcl
# envs/prod/backend.tf
terraform {
  backend "s3" {
    bucket       = "notely-tfstate-a1b2c3d4"
    key          = "notely/prod/terraform.tfstate"
    region       = "ap-southeast-2"
    encrypt      = true
    use_lockfile = true
  }
}
```

Simple, explicit, and each key is visible in the file. This is what Notely uses.

### Option B: partial configuration

Backend blocks cannot use variables — they are read before variables exist. But
you can leave values out and supply them at `init` time:

```hcl
# backend.tf - shared, incomplete
terraform {
  backend "s3" {
    region       = "ap-southeast-2"
    encrypt      = true
    use_lockfile = true
    # bucket and key supplied at init
  }
}
```

```hcl
# dev.backend.hcl
bucket = "notely-tfstate-a1b2c3d4"
key    = "notely/dev/terraform.tfstate"
```

```bash
terraform init -backend-config=dev.backend.hcl
```

Useful when the bucket name differs per environment or per AWS account, and
essential in CI where the values come from pipeline variables:

```bash
terraform init \
  -backend-config="bucket=$TF_STATE_BUCKET" \
  -backend-config="key=notely/$ENVIRONMENT/terraform.tfstate"
```

> Backend blocks cannot contain variables or expressions. Partial configuration is the workaround, and it is the standard one.

---

## Part 4 — Splitting by layer as well

Environments are one axis. **Layers** are the other, and larger organisations use
both.

```text
envs/prod/
├── 10-network/     VPC, subnets, routing        changes twice a year
├── 20-data/        RDS, S3, backups             changes monthly
├── 30-platform/    IAM, shared services         changes monthly
└── 40-app/         EC2, ALB, target groups      changes daily
```

Four state files per environment.

### Why split

**Blast radius.** A mistake in `40-app` cannot produce a plan that deletes the
VPC, because the VPC is not in that state file. Terraform physically cannot touch
it.

**Plan speed.** Refreshing 200 resources takes minutes. Refreshing the 12 in the
app layer takes seconds. When you deploy several times a day, that adds up.

**Permissions.** The CI role that deploys the app does not need permission to
delete VPCs. Separate state means you can grant separate IAM.

**Change frequency.** Things that change together belong together. The network
and the app change on completely different rhythms.

### How the layers connect

Two options, covered in Module 7.

**Remote state:**

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "notely-tfstate-a1b2c3d4"
    key    = "notely/prod/network/terraform.tfstate"
    region = "ap-southeast-2"
  }
}

module "web" {
  source            = "../../../modules/web"
  vpc_id            = data.terraform_remote_state.network.outputs.vpc_id
  public_subnet_ids = data.terraform_remote_state.network.outputs.public_subnet_ids
}
```

**SSM Parameter Store** — looser coupling, no state access needed:

```hcl
# The network layer publishes
resource "aws_ssm_parameter" "vpc_id" {
  name  = "/notely/${var.environment}/network/vpc-id"
  type  = "String"
  value = module.network.vpc_id
}

# The app layer reads
data "aws_ssm_parameter" "vpc_id" {
  name = "/notely/${var.environment}/network/vpc-id"
}
```

### When to split

Not on day one. Notely at its current size — about 25 resources — belongs in one
state file per environment.

Split when:

- Plans take more than a couple of minutes
- Different teams own different parts
- You want different IAM permissions for different changes
- One state file has grown past roughly 100 resources

> Splitting state early is premature optimisation. Splitting it late is a painful migration. Watch for the signals.

---

## Part 5 — Terragrunt, briefly

You will hear about Terragrunt. It is a third-party wrapper that reduces the
remaining duplication:

```hcl
# envs/prod/app/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

terraform {
  source = "../../../modules//notely"
}

dependency "network" {
  config_path = "../network"
}

inputs = {
  environment = "prod"
  vpc_id      = dependency.network.outputs.vpc_id
}
```

It generates the backend configuration, handles dependencies between layers, and
can run commands across many directories at once.

**Should you use it?** Not while learning, and not on a project Notely's size.
It adds a tool, a syntax, and a layer of indirection between you and Terraform.

Consider it when you have dozens of state files across several accounts and the
duplication is genuinely painful. Many teams that size use it happily.

---

## Building it into Notely

Restructure into three environments.

### 1. The new layout

```bash
cd ~/terraform-labs/notely
mkdir -p envs/{dev,staging,prod}
```

Target:

```text
notely/
├── modules/
│   ├── network/
│   ├── security/
│   ├── storage/
│   ├── web/
│   └── notely/          <- new: composes the other four
└── envs/
    ├── dev/
    ├── staging/
    └── prod/
```

### 2. The composition module

`modules/notely/variables.tf`:

```hcl
variable "project_name" {
  type    = string
  default = "notely"
}

variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment must be dev, staging or prod."
  }
}

variable "region" {
  type    = string
  default = "ap-southeast-2"
}

variable "vpc_cidr" {
  type = string
}

variable "az_count" {
  type    = number
  default = 2
}

variable "app_instance_type" {
  type    = string
  default = "t3.micro"
}

variable "app_port" {
  type    = number
  default = 3000
}

variable "log_retention_days" {
  type    = number
  default = 7
}

variable "enable_versioning" {
  type    = bool
  default = true
}
```

`modules/notely/main.tf`:

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
  source = "../network"

  name_prefix = local.name_prefix
  vpc_cidr    = var.vpc_cidr
  az_count    = var.az_count
}

module "security" {
  source = "../security"

  name_prefix = local.name_prefix
  vpc_id      = module.network.vpc_id
  app_port    = var.app_port
}

module "storage" {
  source = "../storage"

  name_prefix        = local.name_prefix
  enable_versioning  = var.enable_versioning
  log_retention_days = var.log_retention_days
}

module "iam" {
  source = "../iam"

  name_prefix            = local.name_prefix
  attachments_bucket_arn = module.storage.attachments_bucket_arn
  log_group_arn          = module.storage.log_group_arn
}

module "web" {
  source = "../web"

  name_prefix           = local.name_prefix
  vpc_id                = module.network.vpc_id
  public_subnet_ids     = module.network.public_subnet_ids
  app_subnet_ids        = module.network.app_subnet_ids
  alb_security_group_id = module.security.alb_security_group_id
  app_security_group_id = module.security.app_security_group_id
  instance_type         = var.app_instance_type
  app_port              = var.app_port
  user_data             = local.user_data
  instance_profile_name = module.iam.instance_profile_name
}
```

(Move the IAM role and instance profile from the old root into `modules/iam` —
it takes the bucket ARN and log group ARN and returns the instance profile
name.)

`modules/notely/outputs.tf`:

```hcl
output "url" {
  description = "Public URL for this Notely environment"
  value       = module.web.url
}

output "vpc_id" {
  value = module.network.vpc_id
}

output "attachments_bucket" {
  value = module.storage.attachments_bucket_name
}

output "alb_dns_name" {
  value = module.web.alb_dns_name
}

output "alb_zone_id" {
  value = module.web.alb_zone_id
}
```

### 3. The dev environment

`envs/dev/backend.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket       = "notely-tfstate-a1b2c3d4"    # yours from Module 3
    key          = "notely/dev/terraform.tfstate"
    region       = "ap-southeast-2"
    encrypt      = true
    use_lockfile = true
  }
}

provider "aws" {
  region = "ap-southeast-2"

  default_tags {
    tags = {
      Project     = "notely"
      Environment = "dev"
      ManagedBy   = "Terraform"
      Ephemeral   = "true"
    }
  }
}
```

`envs/dev/main.tf`:

```hcl
module "notely" {
  source = "../../modules/notely"

  environment        = "dev"
  vpc_cidr           = "10.0.0.0/16"
  az_count           = 2
  app_instance_type  = "t3.micro"
  log_retention_days = 7
  enable_versioning  = false
}
```

`envs/dev/outputs.tf`:

```hcl
output "url" {
  value = module.notely.url
}

output "vpc_id" {
  value = module.notely.vpc_id
}

output "attachments_bucket" {
  value = module.notely.attachments_bucket
}
```

Twelve lines of `main.tf`. That is the whole dev environment.

### 4. Staging and prod

`envs/staging/main.tf`:

```hcl
module "notely" {
  source = "../../modules/notely"

  environment        = "staging"
  vpc_cidr           = "10.1.0.0/16"
  az_count           = 2
  app_instance_type  = "t3.small"
  log_retention_days = 30
  enable_versioning  = true
}
```

`envs/prod/main.tf`:

```hcl
module "notely" {
  source = "../../modules/notely"

  environment        = "prod"
  vpc_cidr           = "10.2.0.0/16"
  az_count           = 3
  app_instance_type  = "t3.large"
  log_retention_days = 365
  enable_versioning  = true
}
```

Copy `backend.tf` and `outputs.tf` into each, changing the `key` and the
`Environment` tag.

**Note the CIDR blocks never overlap.** `10.0`, `10.1`, `10.2`. If you ever peer
these VPCs together, overlapping ranges would make it impossible.

### 5. The three environments side by side

| | dev | staging | prod |
|---|---|---|---|
| VPC CIDR | `10.0.0.0/16` | `10.1.0.0/16` | `10.2.0.0/16` |
| Availability zones | 2 | 2 | 3 |
| Subnets | 6 | 6 | 9 |
| Instance type | `t3.micro` | `t3.small` | `t3.large` |
| App servers | 2 | 2 | 3 |
| Log retention | 7 days | 30 days | 365 days |
| S3 versioning | off | on | on |
| State key | `notely/dev/...` | `notely/staging/...` | `notely/prod/...` |

Same modules. Different numbers.

### 6. Migrating dev's state

Your existing state lives at `notely/dev/terraform.tfstate` and its addresses are
`module.network.aws_vpc.main`. Moving into `envs/dev` adds another level:
`module.notely.module.network.aws_vpc.main`.

`envs/dev/moved.tf`:

```hcl
moved {
  from = module.network
  to   = module.notely.module.network
}

moved {
  from = module.security
  to   = module.notely.module.security
}

moved {
  from = module.storage
  to   = module.notely.module.storage
}

moved {
  from = module.web
  to   = module.notely.module.web
}
```

You can move a whole module in one block. Everything inside comes with it.

```bash
cd envs/dev
terraform init
terraform plan
```

Target: `0 to add, 0 to change, 0 to destroy`.

---

## Real-World Example

A team of eight runs a SaaS product across three AWS accounts:

```text
infrastructure/
├── modules/
│   ├── network/
│   ├── ecs-service/
│   ├── rds/
│   └── observability/
└── envs/
    ├── dev/          AWS account 111111111111
    │   ├── network/
    │   └── app/
    ├── staging/      AWS account 222222222222
    │   ├── network/
    │   └── app/
    └── prod/         AWS account 333333333333
        ├── network/
        ├── data/
        └── app/
```

Seven state files. Note prod has an extra `data/` layer, because prod's database
is significant enough to deserve its own blast radius — and that structural
difference is trivial to express with directories and impossible to express
cleanly with workspaces.

**The IAM setup that makes it work.** Each environment's state bucket lives in
that environment's account. The CI pipeline assumes a different role per
environment. The prod role can only be assumed by pipelines running on the
default branch, after a manual approval.

So even if someone ran the prod configuration from their laptop, they would not
have credentials.

**The incident that shaped it.** They used to run everything from one account
with workspaces. An engineer ran `terraform apply` believing they were in the
`dev` workspace. They were in `prod`. The plan showed 40 changes; they assumed it
was dev catching up on a week of changes and typed `yes`.

Nothing was destroyed — they got lucky — but the postmortem action was
"environments must be structurally impossible to confuse". Separate directories,
separate accounts, separate credentials. Three layers of protection instead of
one command you have to remember.

---

## Common Mistakes Beginners Make

**1. Using workspaces for dev/staging/prod.**

It is the first thing you find, and it has all six problems listed above.

**2. Two tfvars files, one state file.**

Exactly where Notely was at the start of this module. Applying the second one
destroys the first.

**3. Overlapping VPC CIDRs across environments.**

`10.0.0.0/16` in both dev and prod works fine until you try to peer them, and
then it is unfixable without rebuilding.

**4. Trying to use variables in a `backend` block.**

```hcl
backend "s3" {
  key = "notely/${var.environment}/terraform.tfstate"   # not allowed
}
```

Backends are read before variables exist. Use partial configuration.

**5. Copying whole configurations instead of calling shared modules.**

Then dev and prod drift apart, and a fix applied to one is forgotten in the
other.

**6. Splitting into layers too early.**

Twenty-five resources belong in one state file. Splitting adds remote-state
plumbing you do not need yet.

**7. Never splitting at all.**

A 400-resource state file where every plan takes five minutes and every change
risks the whole estate.

**8. Confusing CLI workspaces with Terraform Cloud workspaces.**

Same word, entirely different things.

**9. Giving every environment the same credentials.**

If the same role can write dev and prod state, a mistake in dev tooling can reach
prod.

---

## Hands-On Lab — Three Environments

**Cost: free** for parts A–D. Part E is optional and billable.

### Part A: See the problem

```bash
cd ~/terraform-labs/notely
terraform plan -var-file=prod.tfvars
```

```text
Plan: 23 to add, 0 to change, 23 to destroy.
```

Read a few lines of it. Terraform is proposing to destroy your dev VPC and build
a prod one. **Do not apply.** This is the problem the module solves.

### Part B: Try workspaces, to feel why they are not the answer

```bash
terraform workspace list
```

```text
* default
```

```bash
terraform workspace new staging
terraform workspace list
```

```text
  default
* staging
```

```bash
terraform plan -var-file=dev.tfvars
```

```text
Plan: 23 to add, 0 to change, 0 to destroy.
```

Interesting — it wants to create everything, because this workspace has empty
state. That is workspaces working correctly.

Now the important bit:

```bash
terraform workspace show
```

```text
staging
```

```bash
pwd
```

```text
/home/rahman/terraform-labs/notely
```

**Your working directory looks identical to when you were in `default`.** Nothing
about your shell, your files, or your prompt tells you which workspace you are
in. That is the whole argument.

Clean up:

```bash
terraform workspace select default
terraform workspace delete staging
```

### Part C: Restructure

```bash
mkdir -p envs/{dev,staging,prod}
mkdir -p modules/{notely,iam}
```

- Move the IAM role and instance profile into `modules/iam`
- Move `templates/` into `modules/notely/templates/`
- Create `modules/notely/{main,variables,outputs}.tf` as shown
- Create `envs/dev/{backend,main,outputs}.tf`
- Copy those to `staging` and `prod`, changing the key, environment and sizing

```bash
cd envs/dev
terraform fmt -recursive ../..
```

### Part D: Migrate dev's state

Create `envs/dev/moved.tf` with the four module moves.

```bash
cd ~/terraform-labs/notely/envs/dev
terraform init
terraform plan
```

You want:

```text
Plan: 0 to add, 0 to change, 0 to destroy.
```

If not, read the destroy list and add the missing `moved` block.

```bash
terraform apply
terraform state list | head -20
```

```text
module.notely.module.iam.aws_iam_instance_profile.app
module.notely.module.network.aws_subnet.this["app-a"]
module.notely.module.network.aws_vpc.main
module.notely.module.security.aws_security_group.alb
module.notely.module.storage.aws_s3_bucket.attachments
...
```

Two levels of module nesting, everything moved, nothing destroyed.

```bash
rm moved.tf
```

### Part E: Plan staging without touching dev

```bash
cd ../staging
terraform init
terraform plan
```

```text
Plan: 23 to add, 0 to change, 0 to destroy.
```

**Read that carefully.** `0 to destroy`. Staging has its own empty state file, so
planning it does not propose touching dev at all.

Compare that with Part A, where the same intent produced `23 to destroy`. That
difference is the entire point of this module.

```bash
cd ../dev
terraform plan
```

```text
No changes.
```

Dev is untouched. The two environments genuinely do not know about each other.

**Optional and billable:** `terraform apply` in staging creates a second complete
Notely. That is two ALBs at ~$0.022/hour each. If you do it, destroy both after.

### Part F: Prove prod is different, not just renamed

```bash
cd ../prod
terraform init
terraform plan
```

Count the subnets in the plan. Prod uses `az_count = 3`, so there should be
**nine** subnets, not six — and `t3.large` instances.

Same modules, structurally different environment, no `count` tricks anywhere.

### Part G: Tear down

```bash
cd ~/terraform-labs/notely/envs/dev
terraform destroy
```

Repeat in any other environment you applied.

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

### What you should have at the end

- Direct experience of one state file trying to convert dev into prod
- A workspace created and deleted, and a feel for why the ambiguity is dangerous
- Three environment directories with independent state
- Dev migrated with an empty plan
- Proof that planning staging does not threaten dev

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **An environment** | A separate state file |
| **The core problem** | One state file describes one deployment |
| **Workspace** | Changes only the state file path |
| **`terraform workspace new/select/list/show`** | Manage them |
| **`terraform.workspace`** | The current name, usable in expressions |
| **`env:/<name>/<key>`** | Where workspace state lands |
| **Workspaces are good for** | PR previews, per-developer sandboxes |
| **Workspaces are bad for** | dev/staging/prod |
| **Why** | One command from prod, shared backend, shared credentials |
| **Directory per environment** | The standard answer |
| **What it gives you** | Separate state, backends, providers, permissions, and a visible working directory |
| **The duplication** | Small, and it is the feature |
| **Backend blocks** | Cannot contain variables |
| **Partial configuration** | `-backend-config=dev.hcl`, the workaround |
| **Layer splitting** | network / data / app as separate states |
| **Why split layers** | Blast radius, plan speed, permissions, change frequency |
| **When to split** | Slow plans, many teams, >100 resources |
| **Connecting layers** | `terraform_remote_state` or SSM parameters |
| **Terragrunt** | A wrapper. Not needed at Notely's size. |
| **TFC workspaces** | A different concept with the same name |
| **Non-overlapping CIDRs** | `10.0`, `10.1`, `10.2` — so you can peer later |

---

## Checkpoint (answer briefly)

1. Why does `terraform plan -var-file=prod.tfvars` propose destroying dev?
2. What does a workspace actually change?
3. Give two reasons workspaces are a poor fit for dev/staging/prod.
4. When are workspaces genuinely the right tool?
5. Why can't you write `key = "notely/${var.environment}/terraform.tfstate"` in a backend block, and what do you do instead?
6. What is the argument for accepting some duplication across `envs/*` directories?
7. Why do Notely's three environments use `10.0.0.0/16`, `10.1.0.0/16` and `10.2.0.0/16` rather than all using `10.0.0.0/16`?

---

## Checkpoint — model answers

### 1. Why prod's plan destroys dev

Because **both configurations share one state file**.

Terraform's plan is always a three-way comparison: configuration, state, reality.
The state file currently describes dev's resources — a VPC at `10.0.0.0/16`, six
subnets, `t3.micro` instances.

When you point the same state file at prod's variables, Terraform compares dev's
recorded resources against a configuration describing `10.1.0.0/16` and
`t3.large`, and concludes the only way to reconcile them is to destroy what
exists and build the new thing.

The `-var-file` flag changes the *desired* state. It does nothing to the
*recorded* state. Terraform has no concept of "these are two different
deployments" unless they are in two different state files.

This is why the answer to "how do I do environments" is always, ultimately,
"separate state".

### 2. What a workspace changes

**Only the state file path. Nothing else.**

With an S3 backend and `key = "notely/terraform.tfstate"`:

| Workspace | State path |
|---|---|
| `default` | `notely/terraform.tfstate` |
| `dev` | `env:/dev/notely/terraform.tfstate` |
| `prod` | `env:/prod/notely/terraform.tfstate` |

Everything else is shared: the same configuration files, the same backend bucket,
the same provider block, the same credentials, the same directory.

The only other thing you get is the `terraform.workspace` expression, which
returns the current name so you can branch on it yourself.

That is genuinely the entire feature. It is much smaller than people expect from
the name, and understanding how small it is explains most of its limitations.

### 3. Two reasons workspaces are bad for environments

**One command stands between you and production.** `terraform workspace select
prod` and `terraform workspace select dev` leave you in an identical-looking
shell in an identical directory with identical files. The only way to know which
one you are in is to run `terraform workspace show` and remember to read it. With
directories, `pwd` and your shell prompt tell you, and you cannot apply prod
without deliberately being in `envs/prod/`.

**One backend means one set of credentials.** All workspaces share the same
backend block, so the same bucket and the same IAM permissions. You cannot say
"only the CI role may write prod state", because prod state is in the same bucket
under a different prefix, reachable by anyone who can write dev.

(Also valid: shared configuration means structural differences need
`count = terraform.workspace == "prod" ? 1 : 0` sprinkled everywhere; no
per-environment provider settings, which makes multi-account awkward; and a
destructive change to the shared configuration can affect every environment at
once.)

### 4. When workspaces are right

**When you need many short-lived, genuinely identical copies of the same thing.**

The clearest case is a per-pull-request preview environment:

```bash
terraform workspace new pr-4821
terraform apply
# run integration tests against it
terraform destroy
terraform workspace delete pr-4821
```

Everything that makes workspaces bad for environments is fine here:

- Shared credentials are correct — it is the same CI role every time
- Shared configuration is correct — the whole point is that the preview is
  identical to the template
- The ambiguity does not matter, because none of these is production
- They exist for an hour

Per-developer sandboxes are the same shape. So is spinning up an isolated copy to
test a risky change before merging.

The distinction in one line: workspaces are for **many copies of one thing**;
environments are usually **different things that share code**.

### 5. Variables in a backend block

**You cannot, because the backend is initialised before variables exist.**

`terraform init` has to know where state lives before it can do anything else —
including reading `.tfvars` files, evaluating locals, or resolving variable
defaults. There is a bootstrapping order, and the backend comes first. So the
backend block accepts only literal values.

```hcl
backend "s3" {
  key = "notely/${var.environment}/terraform.tfstate"   # error
}
```

**The workaround is partial configuration**: leave values out of the block and
supply them at init time.

```hcl
terraform {
  backend "s3" {
    region  = "ap-southeast-2"
    encrypt = true
    # bucket and key omitted
  }
}
```

```bash
terraform init -backend-config=dev.backend.hcl
```

or inline:

```bash
terraform init \
  -backend-config="bucket=$TF_STATE_BUCKET" \
  -backend-config="key=notely/$ENVIRONMENT/terraform.tfstate"
```

The second form is what CI pipelines use, since the values come from pipeline
variables.

For Notely we take the simpler route: a literal `backend.tf` per environment
directory, so each key is written down and visible.

### 6. The argument for duplication across `envs/*`

Two parts.

**It is small.** Because all the real logic lives in modules, each environment
directory is a thin wrapper — a `backend.tf` with a different key and a `main.tf`
with one module block and a dozen arguments. Duplicating fifteen lines three
times is not a maintenance burden. The thing you would be tempted to
de-duplicate is exactly the part that should differ.

**It is the feature, not the cost.** Environments genuinely need to differ. Prod
has three availability zones, larger instances, longer log retention, deletion
protection, and possibly extra resources like a read replica or a WAF that dev
does not have at all.

If you force all three to share one configuration, every one of those differences
becomes a conditional:

```hcl
count = terraform.workspace == "prod" ? 1 : 0
```

Ten of those and nobody can read the configuration, and nobody can tell what prod
actually looks like without mentally executing the file.

With directories, `envs/prod/main.tf` *is* the description of production. You
read it and you know.

There is also a review benefit: a pull request touching `envs/prod/` is visibly a
production change and gets the attention it deserves.

### 7. Non-overlapping CIDRs

**So the VPCs can be connected later.**

Each environment's VPC is isolated today, so `10.0.0.0/16` in all three would
work fine — for now.

The moment you want two of them to talk to each other, it breaks. VPC peering,
Transit Gateway, and site-to-site VPN all require **non-overlapping** address
ranges, because routing is done by IP address. If dev and prod both contain
`10.0.11.42`, there is no way to write a route table entry that distinguishes
them. AWS simply refuses to create the peering connection.

Realistic reasons this comes up:

- A shared-services VPC that all environments need to reach
- Connecting to an office network over VPN, where the office also uses `10.x`
- Migrating data from prod to staging for testing
- A monitoring or CI VPC that needs to reach all environments

The fix costs nothing if you do it now: give each environment its own /16 out of
the `10.x` space. It is close to unfixable later, because changing a VPC's CIDR
means rebuilding it and everything in it.

Keep a simple allocation table somewhere — dev `10.0`, staging `10.1`, prod
`10.2`, shared services `10.10` — and you never have to think about it again.

---

## Next lesson

**Module 10 — Secrets & Provider Authentication** (`10-secrets-and-auth.md`)

Notely has three environments and no database. It has had a `random_password`
sitting in state since Module 3, and Module 3 showed you it is in there in
plaintext.

Module 10 adds the RDS Postgres database, and does it properly: the password is
generated, stored in Secrets Manager, read by the application at runtime, and
never written into a file you could commit. It also covers the AWS credential
chain, assume-role, and OIDC federation from CI — which Module 12 needs.
