# Module 1 — IaC & Terraform Basics

## What we are building across this whole course

Every module from here on builds one more piece of the same system: **Notely**, a
note-taking API written in Node.js.

By Module 14 it looks like this:

```text
                    Internet
                        |
                    Route 53          notely.example.com
                        |
            Application Load Balancer          public subnets
                  /            \
           EC2 server      EC2 server          private subnets
           Node.js API     Node.js API
                  \            /
              RDS PostgreSQL database          private subnets

     S3 (attachments) · CloudWatch Logs · SNS alerts · Secrets Manager
```

Right now that probably looks like a lot. It is not, and you will build it one
small piece at a time:

| Module | What gets added to Notely |
|---|---|
| **1** (this one) | The S3 bucket for file attachments |
| 2 | The VPC, a subnet, an internet gateway |
| 4 | The same thing, but driven by variables |
| 6 | Two availability zones, the load balancer, the servers |
| 8 | All of it reorganised into reusable modules |
| 9 | Three copies: dev, staging, prod |
| 14 | The finished system |

The full picture, the reasoning behind every choice, and what each piece costs
are in `notely-architecture.md`. Keep it open in a tab.

> If any of the networking words above are unfamiliar — VPC, subnet, load balancer — read `00-aws-networking-primer.md` first. It explains all of them from zero.

---

## Why this module exists

You already build infrastructure on AWS. You have clicked through the VPC wizard,
you have written CloudFormation, you have deployed Lambdas with the Serverless
Framework. So the question is not "what is infrastructure" — it is:

**Why does a separate tool called Terraform exist, and what does it do better?**

This module answers that, gets Terraform installed, and gets one real resource
created and destroyed. By the end you will have run all four commands that make
up 95% of daily Terraform use.

---

## The core idea (one sentence)

Terraform is a tool that reads a description of the infrastructure you want,
compares it against what already exists, and makes the minimum set of API calls
needed to close the gap.

> Terraform does not "run" your infrastructure. It reconciles reality with your description.

That word — **reconcile** — is the whole tool. Everything else is detail.

---

## Mental model (mapped to what you already know)

You already know npm (`../../backend/nodejs/node-js.md`). Terraform works almost
exactly the same way, with different words.

| npm | Terraform |
|---|---|
| `package.json` lists what you want | `.tf` files list the infrastructure you want |
| `npm install` fetches it | `terraform init` downloads the providers |
| `node_modules/` holds the downloaded code | `.terraform/` holds the downloaded providers |
| `package-lock.json` pins exact versions | `.terraform.lock.hcl` pins exact versions |
| `^5.0.0` means "5.x, not 6" | `~> 5.0` means "5.x, not 6" |
| Running `npm install` twice does nothing new | Running `terraform apply` twice does nothing new |

That is the tooling half. The other half is database migrations. If you have used
Prisma, Knex, Sequelize or TypeORM:

| Database migrations | Terraform |
|---|---|
| You describe the schema you want | You describe the infrastructure you want |
| The tool compares it against the real database | Terraform compares it against the real cloud |
| It works out the difference | It works out the difference — the **plan** |
| You run the migration | You run `terraform apply` |
| A migrations table records what's applied | The **state file** records what exists |
| Running it twice does nothing | Running it twice does nothing |

If that lands, you already understand Terraform's architecture. The state file is
the migrations table. The plan is the generated migration. `terraform apply` is
`prisma migrate deploy`.

The one place the analogy breaks: EF owns the whole database schema, but
Terraform only owns the resources *you told it about*. Anything created by hand
in the console is invisible to Terraform until you explicitly import it (Module 13).

---

## What Infrastructure as Code actually solves

### The problem with the console

Creating a VPC by hand in the AWS console takes about fifteen minutes. Creating
the same VPC in a second region, six months later, so that it matches exactly?
That is where it falls apart.

Concretely, console-built infrastructure fails at:

| Problem | What it looks like in practice |
|---|---|
| **Reproducibility** | Prod works, staging does not. Nobody remembers which checkbox differs. |
| **Auditability** | "Who opened port 22 to 0.0.0.0/0?" CloudTrail says *who*, never *why*. |
| **Review** | There is no pull request for a mouse click. |
| **Disaster recovery** | The region is down. How long to rebuild by hand? |
| **Knowledge** | The one person who built it has left. |
| **Drift** | Someone "temporarily" changed an instance type. It is now permanent and undocumented. |

Infrastructure as Code fixes all six by making the infrastructure a **text file in
git**. Once it is text in git, it inherits everything git already gives your
application code: diffs, review, history, blame, rollback, branching.

> If your infrastructure is not in version control, you do not have infrastructure. You have a rumour.

### Declarative vs imperative

This is the distinction that trips people up most, so be precise about it.

**Imperative** — you write the *steps*:

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-abc123 --cidr-block 10.0.1.0/24
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-abc123 --internet-gateway-id igw-xyz
```

You are giving instructions. The script says *how*. Run it twice and you get two
VPCs — or an error. To change something you must write a new script that knows
the current state, and you must write it correctly.

**Declarative** — you write the *desired end state*:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}
```

You are describing an outcome. Terraform figures out *how*. Run it twice and the
second run does nothing, because reality already matches. Change `10.0.1.0/24` to
`10.0.2.0/24` and Terraform works out that it needs to replace exactly one subnet.

> Imperative code answers "what do I do?". Declarative code answers "what should be true?".

### Idempotency

An operation is **idempotent** if doing it repeatedly has the same effect as
doing it once.

`terraform apply` is idempotent. This matters more than it sounds:

- You can run it on a schedule to correct drift
- A failed apply can be safely retried
- CI can apply on every merge without special-casing "already deployed"
- You never need to ask "has this been run yet?"

The `aws ec2 create-vpc` command is **not** idempotent — running it twice gives
you two VPCs. That single property is most of the reason IaC tools exist.

---

## Where Terraform sits

### Terraform vs CloudFormation

This is the comparison that matters most to you, because you already know
CloudFormation from your DVA-C02 study (`../../cloud/AWS/README.md`).

| | **CloudFormation** | **Terraform** |
|---|---|---|
| **Vendor** | AWS only | Multi-cloud, and much more than cloud |
| **Language** | JSON / YAML | HCL (purpose-built) |
| **State** | Managed by AWS, invisible to you | A file **you** own and must store |
| **Where state lives** | AWS service | Local disk or a backend you configure |
| **Preview changes** | Change Sets | `terraform plan` |
| **Rollback on failure** | Automatic | **None** — it stops and leaves partial state |
| **Modularity** | Nested stacks, StackSets | Modules (far nicer) |
| **New AWS service support** | Usually day one | Days to weeks behind |
| **Drift detection** | Built-in drift detection | `terraform plan` shows it naturally |
| **Cost** | Free | Free (CLI); HCP Terraform is paid |
| **Ecosystem** | AWS-only | 4,000+ providers: GitHub, Datadog, Cloudflare, Kubernetes, PostgreSQL |

The two genuinely important differences:

**1. You own the state.** CloudFormation hides state inside AWS; that is one
fewer thing to manage, but also one fewer thing you can inspect or fix. Terraform
hands you the state file and the responsibility that comes with it. This is why
Module 3 exists and why it is the longest module.

**2. No automatic rollback.** If a CloudFormation stack fails halfway, AWS rolls
it back. If a Terraform apply fails halfway, the resources it already created
*stay created*, and state records them. You fix the config and apply again.
This feels worse and is usually better — automatic rollback of a half-built
stack can itself fail, leaving `UPDATE_ROLLBACK_FAILED`, which anyone who has
run CloudFormation in anger has met.

### CloudFormation → Terraform, term by term

You have roughly 1,650 lines of CloudFormation notes already
(`../../cloud/AWS/README.md`, section 1.2.7 onwards). Almost all of it converts
directly. This table is the conversion.

| CloudFormation (you know this) | Terraform equivalent | The difference that bites |
|---|---|---|
| **Stack** | Root module + its state file | State is yours to store, not AWS's |
| **Template** (`.yaml` / `.json`) | Configuration — *all* `.tf` files in a directory | Terraform merges the whole directory; there is no single template file |
| **Logical resource** (`MyBucket`) | Resource address (`aws_s3_bucket.data`) | Same idea, different syntax |
| **Physical resource** (`acme-logs-2024`) | The `id` attribute recorded in state | Same idea |
| **`Parameters`** | `variable` blocks | Terraform variables have real types and `validation` blocks |
| **`Outputs`** | `output` blocks | Nearly identical |
| **`Mappings`** | `locals` holding a map | Locals are more flexible |
| **`!Ref` / `!GetAtt`** | Direct reference: `aws_vpc.main.id` | No function needed — just reference it |
| **`!Sub`** | `"${...}"` interpolation | Same idea, less ceremony |
| **`Conditions`** | Ternary `? :`, and `count = x ? 1 : 0` | Terraform has no separate conditions section |
| **`DependsOn`** | `depends_on` meta-argument | Rarely needed — Terraform infers dependencies from references |
| **Nested stacks** | Modules | Terraform modules are considerably nicer to work with |
| **`Export` / `!ImportValue`** | `terraform_remote_state` data source | Module 7 |
| **StackSets** | Provider aliases + `for_each` | Module 6 |
| **`DeletionPolicy: Retain`** | `lifecycle { prevent_destroy = true }` | Module 6 |
| **Change Set** | `terraform plan` (and `plan -out=tfplan`) | Terraform's is faster and reads better |
| **Drift detection** | `terraform plan -refresh-only` | Terraform shows drift on every ordinary plan too |
| **`cfn-init` / `cfn-signal`** | Provisioners — **and you should avoid them** | Use user-data or a baked AMI instead |
| **Stack policy** | No direct equivalent | Use IAM on the state bucket plus `prevent_destroy` |
| **CFN Registry** | Terraform Registry | Terraform's is far larger |

The single row with no CloudFormation equivalent at all is the first one:
**state**. CloudFormation keeps the logical-to-physical mapping inside AWS, so
you have never had to think about where it lives, who can read it, or what
happens when two people write it at once. In Terraform all three are your
problem. That is why Module 3 is the longest module in this curriculum.

---

The reason most teams pick Terraform even on AWS-only workloads is the last row:
one tool and one language for AWS *and* GitHub repos *and* Datadog monitors *and*
Cloudflare DNS.

### Terraform vs the Serverless Framework

You have used the Serverless Framework (`../../cloud/aws serverless/serverless-framework.md`).
They overlap but solve different problems.

| | **Serverless Framework** | **Terraform** |
|---|---|---|
| **Scope** | Lambda-centric applications | All infrastructure |
| **Abstraction level** | High — one `function` block becomes 6 resources | Low — one block, one resource |
| **Under the hood** | Generates CloudFormation | Calls AWS APIs directly |
| **Packaging code** | Yes — zips and uploads your handler | No — that is your build pipeline's job |
| **Good at** | Shipping a serverless app fast | Owning an entire estate |

They are frequently used together: Terraform builds the VPC, RDS, S3 buckets and
IAM roles; the Serverless Framework deploys the functions into it.

> Serverless Framework is an application deployment tool. Terraform is an infrastructure ownership tool.

### The wider landscape

| Tool | What it is | Use it when |
|---|---|---|
| **Terraform** | Declarative, HCL, multi-cloud | Default choice for infrastructure |
| **OpenTofu** | Open-source fork of Terraform (after the 2023 licence change) | You need a fully OSS licence; near drop-in compatible |
| **Pulumi** | IaC in TypeScript/JavaScript/Python/Go | Your team strongly prefers a real programming language |
| **AWS CDK** | Generates CloudFormation from TypeScript/Python | AWS-only, and you want loops and classes |
| **Ansible** | Imperative-ish configuration management | Configuring software *inside* servers |
| **Kubernetes manifests** | Declarative container orchestration | Workloads on a cluster (Terraform builds the cluster) |

The distinction worth holding on to: **Terraform provisions infrastructure;
Ansible configures it.** Terraform creates the EC2 instance. Ansible installs
nginx on it. Using Terraform for the second job is possible via provisioners and
is almost always a mistake (Module 6 covers why).

Since you write JavaScript, CDK and Pulumi will look tempting — you could write
your infrastructure in TypeScript and never learn a new language. Resist for now.

Here is why. In TypeScript you *can* write a `for` loop that creates 300 servers
based on the result of an API call. That flexibility sounds good until you are
reading someone else's infrastructure code at 2am trying to work out how many
servers it will actually make. HCL cannot do that, and the restriction is
deliberate — you can always read a `.tf` file and know what it builds. Learn
Terraform first. If you later move to CDK or Pulumi, everything transfers.

### When *not* to use Terraform

Being honest about limits:

- **Application deployment.** Rolling out a new container image every twenty
  minutes is a job for your CD pipeline, not `terraform apply`.
- **Software configuration inside a VM.** Use Ansible, or better, bake an image.
- **One-off throwaway experiments.** Clicking in the console for ten minutes to
  test an idea is fine. Codify it once you know you want to keep it.
- **Anything with no API.** Terraform can only manage what a provider can call.

---

## Installation and setup

### Installing Terraform

**macOS (Homebrew):**

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

**Linux (Debian/Ubuntu):**

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | \
  sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform
```

**Windows (Chocolatey):**

```bash
choco install terraform
```

Verify:

```bash
terraform version
```

```text
Terraform v1.9.5
on darwin_arm64
```

Anything on 1.x is fine for these notes.

### Managing multiple versions with `tfenv`

Real projects pin Terraform versions, and different projects pin different ones.
`tfenv` switches between them the way `nvm` does for Node.

```bash
brew install tfenv

tfenv list-remote          # what's available
tfenv install 1.9.5        # install a specific version
tfenv use 1.9.5            # switch to it
tfenv install latest       # or just the newest
```

Drop a `.terraform-version` file in a project directory and `tfenv` picks it up
automatically:

```bash
echo "1.9.5" > .terraform-version
```

Worth setting up now — you will need it the first time you touch an older project.

### Shell autocomplete

Small quality-of-life win, takes one command:

```bash
terraform -install-autocomplete
```

Restart your shell afterwards.

### AWS credentials

Terraform does not have its own credential system for AWS. It uses exactly the
same chain the AWS CLI and SDKs use, in this order:

1. Provider block arguments (**never do this** — see below)
2. Environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`
3. Shared credentials file: `~/.aws/credentials`
4. Shared config file: `~/.aws/config` (including SSO and assume-role profiles)
5. ECS container credentials
6. EC2 instance profile (IMDS)

Set up a profile:

```bash
aws configure --profile terraform-lab
```

Confirm it works before you touch Terraform:

```bash
aws sts get-caller-identity --profile terraform-lab
```

```json
{
    "UserId": "AIDXXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/rahman"
}
```

Then tell your shell which profile to use:

```bash
export AWS_PROFILE=terraform-lab
export AWS_REGION=ap-southeast-2
```

> Never put `access_key` or `secret_key` inside a `.tf` file. They get committed, and committed AWS keys are scraped from GitHub within minutes.

---

## Providers and the registry

### What a provider actually is

Terraform's core knows nothing about AWS. It knows about resources, state, graphs
and plans — pure abstractions. Everything cloud-specific lives in a **provider**:
a separately-versioned binary that Terraform downloads.

A provider does three things:

1. Declares which resource types exist (`aws_s3_bucket`, `aws_instance`, …)
2. Declares each one's schema — arguments, attributes, types, validation
3. Implements CRUD by calling the real API

```text
your .tf files
      ↓
 Terraform Core   ← state file
      ↓
  AWS Provider    ← the binary in .terraform/
      ↓
   AWS API
```

This is why Terraform is multi-cloud without knowing anything about clouds. The
Datadog provider is the same architecture pointed at a different API. The
registry has over 4,000 of them: `github`, `cloudflare`, `kubernetes`, `helm`,
`postgresql`, even `random` and `local`.

### Declaring providers

Every real project starts with a `terraform` block:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-southeast-2"
}
```

Reading that carefully:

| Line | Meaning |
|---|---|
| `required_version` | Which Terraform CLI versions may run this config |
| `required_providers` | Which providers to download, and from where |
| `source` | Registry address: `<namespace>/<name>` — here, `hashicorp/aws` |
| `version` | The version constraint (see below) |
| `provider "aws"` | Configuration *for* the provider — region, profile, default tags |

Note the two blocks are different things. `required_providers` says *which plugin
to fetch*. `provider "aws"` says *how to configure it*.

### Version constraints

```hcl
version = "5.31.0"      # exactly this version
version = ">= 5.0"      # this or newer — risky, allows 6.x
version = "~> 5.0"      # >= 5.0 and < 6.0  (allows minor + patch)
version = "~> 5.31"     # >= 5.31 and < 6.0
version = "~> 5.31.0"   # >= 5.31.0 and < 5.32.0 (patch only)
```

The `~>` operator is called the **pessimistic constraint**: allow updates to the
rightmost component only.

Recommended default is `~> 5.0`. Major provider versions contain breaking
changes, so pinning below the major boundary is the sensible floor.

### The lock file

After `terraform init`, you get `.terraform.lock.hcl`:

```hcl
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"
  constraints = "~> 5.0"
  hashes = [
    "h1:0Q8Cx0TSy5Xtk8BDbkFRPYWNfXFRO1kNIzOtHkjnLYw=",
    ...
  ]
}
```

This is `package-lock.json` for infrastructure. It records the exact version
resolved and cryptographic hashes of the binaries.

> **Commit `.terraform.lock.hcl` to git.** Never gitignore it.

Without it, you get `~> 5.0` and your colleague gets `~> 5.0` and you end up on
different provider versions producing different plans. To deliberately upgrade
within your constraint:

```bash
terraform init -upgrade
```

### Multiple providers

Nothing stops you using several at once:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
    github = {
      source  = "integrations/github"
      version = "~> 6.0"
    }
  }
}
```

Now one `terraform apply` can create an S3 bucket, generate a random suffix, and
configure a GitHub repository's branch protection. This is the multi-provider
capability that CloudFormation structurally cannot offer.

### `default_tags` — set this up now

The AWS provider can tag every taggable resource automatically:

```hcl
provider "aws" {
  region = "ap-southeast-2"

  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = "lab"
      Owner       = "rahman"
    }
  }
}
```

Every resource created by this provider now carries those tags without you
writing them anywhere else. `ManagedBy = "Terraform"` in particular is worth
having from day one: six months from now, in a console full of resources, it is
how you tell what you are allowed to click.

---

## The four commands

### `terraform init`

Run once per project, and again whenever providers or the backend change.

```bash
terraform init
```

It:

1. Downloads the providers into `.terraform/`
2. Writes or verifies `.terraform.lock.hcl`
3. Initialises the backend (where state lives — Module 3)
4. Downloads any modules referenced

```text
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

Safe to run repeatedly. It changes nothing in your cloud account.

### `terraform plan`

The command you will run most.

```bash
terraform plan
```

Terraform:

1. Reads your `.tf` files (desired state)
2. Reads the state file (what it believes exists)
3. Queries the AWS API (what actually exists)
4. Prints the diff

```text
Terraform will perform the following actions:

  # aws_s3_bucket.notes will be created
  + resource "aws_s3_bucket" "notes" {
      + bucket                      = "rahman-tf-lab-a1b2c3d4"
      + id                          = (known after apply)
      + arn                         = (known after apply)
      + force_destroy               = false
      + tags_all                    = {
          + "ManagedBy" = "Terraform"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

The symbols:

| Symbol | Meaning |
|---|---|
| `+` | Create |
| `-` | Destroy |
| `~` | Update in place |
| `-/+` | **Destroy and recreate** — read this one carefully |
| `<=` | Read (a data source) |

`(known after apply)` means the value does not exist yet — AWS assigns it. That
is normal and not a warning.

> `plan` changes nothing. Run it as often as you like. The only command that changes infrastructure is `apply`.

### `terraform apply`

```bash
terraform apply
```

Runs a plan, shows it, and asks for confirmation:

```text
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

Type `yes` — literally the word, not `y`. Then:

```text
aws_s3_bucket.notes: Creating...
aws_s3_bucket.notes: Creation complete after 2s [id=rahman-tf-lab-a1b2c3d4]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

You can skip the prompt with `-auto-approve`:

```bash
terraform apply -auto-approve
```

Use it in CI. Do not use it interactively — that prompt is the last thing between
you and deleting a production database.

### `terraform destroy`

```bash
terraform destroy
```

Destroys everything in the state file. It shows a plan of deletions first and
requires the same `yes`.

```text
Plan: 0 to add, 0 to change, 1 to destroy.

Destroy complete! Resources: 1 destroyed.
```

> Run `destroy` at the end of every lab. Forgotten lab infrastructure is the most common way beginners get a surprise AWS bill.

---

## Real-World Example

A four-person team runs a Node.js API on ECS Fargate behind an ALB, with RDS
Postgres and an S3 bucket for uploads.

**Without Terraform:**

Someone built it in the console in 2023. There is a Confluence page with
screenshots, last updated eleven months ago. Staging was built later and is
"basically the same" — except staging's RDS is `db.t3.micro` and prod's is
`db.t3.medium`, and staging's security group has an extra rule someone added
while debugging. Nobody is confident about what would happen if the region went
down.

**With Terraform:**

The whole stack is about 400 lines of HCL in a git repo. Staging and prod are the
same code with different `.tfvars` files, so drift between them is structurally
impossible. Adding a Redis cache means a pull request that a colleague reviews:
the plan output is pasted in the PR, so the reviewer sees exactly which 3
resources will be created before anyone approves. Rebuilding in another region is
a variable change and a twenty-minute apply.

The concrete daily difference: an engineer wants to open port 5432 to a new
security group. Instead of clicking in the console at 4pm on a Friday, they open
a PR. The reviewer sees `~ update in-place` on one security group rule and
nothing else. That is the entire value proposition.

---

## Common Mistakes Beginners Make

**1. Hardcoding credentials in `.tf` files.**

```hcl
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"   # never
  secret_key = "wJalrXUtnFEMI/..."      # never
}
```

Use environment variables or a named profile. Committed keys are scraped from
public GitHub within minutes, and bots spin up crypto miners in your account.

**2. Not committing `.terraform.lock.hcl`.**

It looks like a generated artifact, so people gitignore it. Then two developers
resolve different provider versions and get different plans from identical code.

**3. Committing state files.**

`terraform.tfstate` contains every attribute of every resource, including
database passwords, in plaintext. It must be gitignored (Module 3).

**4. Running `apply` without reading the plan.**

The plan is not a formality. `-/+ destroy and then create replacement` on an RDS
instance is how you lose a database. Read every line, particularly the count line
at the bottom.

**5. Using `>=` version constraints.**

```hcl
version = ">= 5.0"   # allows 6.0, 7.0 — breaking changes
version = "~> 5.0"   # correct
```

**6. Editing infrastructure in the console after Terraform created it.**

Terraform will detect the change and revert it on the next apply — or worse,
produce a confusing plan. Once a resource is managed by Terraform, the console is
read-only.

**7. Forgetting to `destroy` after a lab.**

A NAT Gateway costs about $32/month. An idle RDS instance costs more. Destroy
everything you create in a lab.

---

## Hands-On Lab — Notely's First Resource

**Cost: free.** One S3 bucket with no objects in it is free tier, and you destroy
it at the end.

### Goal

Create Notely's attachments bucket — the place uploaded files will live — then
observe the state file, make a change, watch drift happen, and destroy it.

This is a real piece of the final architecture. It survives to Module 14.

### Step 1: Set up

```bash
mkdir -p ~/terraform-labs/notely
cd ~/terraform-labs/notely
```

Not inside your notes repo. This directory is where Notely lives from now on —
every module adds to it.

### Step 2: Write the configuration

Create `main.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}

provider "aws" {
  region = "ap-southeast-2"

  default_tags {
    tags = {
      ManagedBy = "Terraform"
      Project   = "notely"
      Ephemeral = "true"
    }
  }
}

# S3 bucket names must be globally unique across all of AWS -
# not just unique in your account. Someone else may already own
# "notely-attachments", so we add a random suffix.
resource "random_id" "suffix" {
  byte_length = 4
}

# Where Notely stores files people attach to their notes.
resource "aws_s3_bucket" "attachments" {
  bucket = "notely-attachments-${random_id.suffix.hex}"
}

output "attachments_bucket_name" {
  description = "The name of Notely's attachments bucket"
  value       = aws_s3_bucket.attachments.bucket
}

output "attachments_bucket_arn" {
  description = "The ARN of Notely's attachments bucket"
  value       = aws_s3_bucket.attachments.arn
}
```

Type it out rather than pasting. You are building muscle memory for HCL's shape.

### Step 3: Initialise

```bash
terraform init
```

Then look at what appeared:

```bash
ls -la
```

You should see `.terraform/` (the downloaded providers — this directory is large)
and `.terraform.lock.hcl` (the lock file).

### Step 4: Plan

```bash
terraform plan
```

Read the output. You are looking for:

- `Plan: 2 to add, 0 to change, 0 to destroy.` — two resources, because
  `random_id` is a resource too
- `(known after apply)` on the bucket ARN — AWS has not assigned it yet
- The `tags_all` block showing your `default_tags` applied automatically

### Step 5: Apply

```bash
terraform apply
```

Type `yes`. Then:

```bash
terraform output
```

```text
attachments_bucket_arn  = "arn:aws:s3:::notely-attachments-a1b2c3d4"
attachments_bucket_name = "notely-attachments-a1b2c3d4"
```

Confirm it is real:

```bash
aws s3 ls | grep notely-attachments
```

### Step 6: Look at the state file

```bash
cat terraform.tfstate
```

It is JSON. Find the `resources` array and read one entry. Notice that it records
every attribute of the bucket, not just the ones you set. This file is how
Terraform knows `aws_s3_bucket.attachments` maps to `notely-attachments-a1b2c3d4`.

Module 3 is entirely about this file. For now, just register that it exists and
that it is important.

### Step 7: Prove idempotency

```bash
terraform apply
```

```text
No changes. Your infrastructure matches the configuration.
```

Nothing happened, because reality already matches the description. That is the
whole reconciliation model in one command.

### Step 8: Make a change

Add versioning to the bucket. Append to `main.tf`:

```hcl
# Versioning means an overwritten or deleted attachment can be recovered.
resource "aws_s3_bucket_versioning" "attachments" {
  bucket = aws_s3_bucket.attachments.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

```bash
terraform plan
```

`Plan: 1 to add, 0 to change, 0 to destroy.` — Terraform adds only the new
resource. It does not touch the bucket, because the bucket has not changed.

```bash
terraform apply
```

### Step 9: See drift

Change something by hand, the way a colleague might:

```bash
aws s3api put-bucket-versioning \
  --bucket $(terraform output -raw attachments_bucket_name) \
  --versioning-configuration Status=Suspended
```

Now:

```bash
terraform plan
```

Terraform detects that reality no longer matches your config and proposes to fix
it:

```text
  ~ resource "aws_s3_bucket_versioning" "attachments" {
      ~ versioning_configuration {
          ~ status = "Suspended" -> "Enabled"
        }
    }
```

This is **drift detection**, and you got it for free. Apply to correct it:

```bash
terraform apply
```

### Step 10: Tear it down

```bash
terraform destroy
```

Type `yes`. Then verify:

```bash
aws s3 ls | grep notely-attachments
```

No output means it is gone.

### What you should have at the end

- A bucket created, modified, drifted, corrected, and destroyed
- A read of a real state file
- All four commands run at least once
- An empty AWS account again

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **IaC** | Infrastructure described as text in version control |
| **Declarative** | You describe the end state; the tool works out the steps |
| **Idempotent** | Applying twice = applying once |
| **Terraform** | Reconciles your description against reality via provider APIs |
| **Provider** | Plugin that teaches Terraform one API (AWS, GitHub, Datadog) |
| **`required_providers`** | Which plugins to download |
| **`provider "aws"`** | How to configure the plugin (region, tags, profile) |
| **`~> 5.0`** | Pessimistic constraint: `>= 5.0, < 6.0` |
| **`.terraform.lock.hcl`** | Exact resolved versions — **commit this** |
| **`terraform init`** | Download providers, set up backend |
| **`terraform plan`** | Show the diff. Changes nothing. |
| **`terraform apply`** | Make reality match the config |
| **`terraform destroy`** | Delete everything in state |
| **Drift** | Reality diverging from config; `plan` reveals it |
| **State** | The map from config blocks to real resources (Module 3) |

### Terraform vs CloudFormation, in one line each

| | |
|---|---|
| **State** | CFN hides it; Terraform hands it to you |
| **Rollback** | CFN auto-rolls-back; Terraform stops and leaves partial state |
| **Scope** | CFN is AWS; Terraform is everything with an API |
| **Language** | CFN is YAML/JSON; Terraform is HCL |

---

## Checkpoint (answer briefly)

1. Terraform is declarative. What does that actually mean for what you write in a `.tf` file?
2. Why is `terraform apply` safe to run twice, when `aws ec2 create-vpc` is not?
3. What is the difference between the `required_providers` block and the `provider "aws"` block?
4. Why must `.terraform.lock.hcl` be committed to git?
5. What does `~> 5.0` allow, and what does it prevent?
6. Your colleague changes a security group in the AWS console. What happens on your next `terraform plan`?
7. Name one thing CloudFormation does that Terraform deliberately does not.

---

## Checkpoint — model answers

### 1. What does declarative mean here?

You write the **desired end state**, not the steps to reach it. Your `.tf` file
says "a VPC with CIDR 10.0.0.0/16 should exist" — it never says "call
CreateVpc". Terraform compares that description against reality and derives the
API calls itself.

The practical consequence: the same file describes creating the VPC, changing it,
and leaving it alone. Which of those happens depends on what already exists, not
on what you wrote.

### 2. Why is `apply` safe to run twice?

Because Terraform reconciles rather than executes. On the second run it reads
state, queries AWS, finds that reality already matches the config, and does
nothing — `No changes.`

`aws ec2 create-vpc` is an imperative instruction with no knowledge of what
exists. Run it twice and you have created two VPCs, because it was never asking
"does this already exist?" — it was told to create one.

That property is called **idempotency**, and it is why you can safely retry a
failed apply, run apply from CI on every merge, and even run it on a schedule to
correct drift.

### 3. `required_providers` vs `provider "aws"`

They answer different questions.

`required_providers` is **dependency declaration** — which plugin binary to
download, from which registry namespace, at which version. It lives inside the
`terraform` block and is closer to a `package.json` dependency entry.

`provider "aws"` is **configuration of that plugin once it is downloaded** —
which region to talk to, which credentials profile, what default tags to apply.

You can have one `required_providers` entry for AWS and several `provider "aws"`
blocks with different aliases, one per region. That is Module 6.

### 4. Why commit the lock file?

Because a version *constraint* is not a version. `~> 5.0` resolves to whatever
the newest matching version is at the moment you run `init`. You run `init` today
and get 5.31.0; your colleague runs it next month and gets 5.40.0.

Different provider versions can generate different plans from identical
configuration — new default values, changed attribute handling, fixed bugs that
changed behaviour. The lock file pins the exact resolved version and its hashes,
so everyone and every CI run uses the same binary. Same reason you commit
`package-lock.json`.

### 5. What does `~> 5.0` allow?

It allows `>= 5.0.0` and `< 6.0.0` — any minor or patch release in the 5.x line,
but never 6.0.

It prevents an automatic jump to the next major version. Provider majors contain
breaking changes: renamed arguments, removed resources, changed defaults. `~> 5.0`
means you get bug fixes and new resources for free, but upgrading to 6.x is a
deliberate act with a code change and a review.

Contrast `>= 5.0`, which happily installs 6.0 the day it ships and breaks your
plan with no warning.

### 6. Colleague changes a security group in the console

Your next `terraform plan` **detects the drift and proposes to undo it**.

The mechanism: plan queries the real AWS API for every resource in state. The
security group's real ingress rules no longer match your config, so Terraform
shows a `~` update returning it to the configured state. Apply, and the
colleague's change is reverted.

This is correct behaviour — Terraform's job is to make reality match the config —
but it is also why "once a resource is in Terraform, the console is read-only" has
to be a team rule. If the change was legitimate, it belongs in the `.tf` file and
a pull request.

### 7. One thing CloudFormation does that Terraform does not

**Automatic rollback on failure.** If a CloudFormation stack update fails halfway,
AWS automatically reverts to the previous state. Terraform does not: it stops at
the failure, and every resource it already created stays created and is recorded
in state. You fix the configuration and apply again.

Terraform's behaviour sounds worse but is usually preferable. CloudFormation's
rollback can itself fail, and `UPDATE_ROLLBACK_FAILED` is a genuinely unpleasant
state to be in. Terraform leaving you with a partially-applied stack and accurate
state is at least always recoverable by fixing the config.

(Also valid: CloudFormation manages state for you, and CFN typically supports
brand-new AWS services on launch day while the Terraform provider lags by days
or weeks.)

---

## Next lesson

**Module 2 — HCL & the Core Workflow** (`02-hcl-and-core-workflow.md`)

You have run the four commands. Next you learn the language properly: HCL's block
syntax and type system, how resource references build a dependency graph without
you declaring one, and how to read plan output line by line rather than skipping
to the summary.

Then Notely gets its network: a VPC, a subnet, an internet gateway and a route
table — exactly the pieces you drew in Module 0, now in code. All four are
**free**, so this costs nothing.
