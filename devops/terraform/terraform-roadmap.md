# Terraform & Infrastructure as Code Mastery Roadmap

This is the study plan for learning Terraform from zero to production-capable.

Every module has its own file in this folder. Work through them in order — the
numbering is the study order. Tick the boxes as you go.

> Terraform is not a new cloud. It is a new way to *describe* the cloud.

---

## One example, built up module by module

The whole course builds a single system: **Notely**, a Node.js note-taking API.

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

Each module adds one more piece. Nothing is thrown away — Notely only grows.

| Module | What happens to Notely |
|---|---|
| **0** | Draw the network on paper. No Terraform yet. |
| **1** | The S3 attachments bucket |
| **2** | VPC, a subnet, internet gateway, route table |
| **3** | Its state moves into S3 with locking |
| **4** | Every hardcoded value becomes a variable |
| **5** | Security groups, computed CIDRs, the server startup script |
| **6** | Six subnets across two AZs, the load balancer, two servers |
| **7** | Stop hardcoding AMI IDs and AZ names |
| **8** | Reorganised into reusable modules |
| **9** | Three environments: dev, staging, prod |
| **10** | The database, with its password in Secrets Manager |
| **11** | Tests and security scanners over all of it |
| **12** | A pipeline that plans on review and applies on approval |
| **13** | Adopt a bucket somebody created by hand |
| **14** | Route 53, alarms, SNS — and the whole thing runs |

Full details, the network layout, and what each piece costs:
**`notely-architecture.md`**

---

## Who this roadmap assumes you are

You are a backend developer who:

- Writes JavaScript and Node.js — see `../../backend/nodejs/node-js.md`
- Has used Docker — see `../docker/docker-notes.md`
- Has built CI/CD pipelines in GitLab CI and GitHub Actions — see
  `../CI_CD/Gitlab-CICD.md` and `../CI_CD/CI_CD.md`
- Knows some AWS — see `../../cloud/AWS/README.md`
- Has **never** used Terraform
- Is **not** confident about VPCs, subnets and how they wire together

That last point is why there is a **Module 0**. It explains AWS networking from
zero — IP addresses, CIDR, subnets, route tables, security groups — before any
Terraform appears. Everything after it assumes you have read it.

Terraform will also feel familiar in places, because it borrows patterns you
already use:

| Terraform | Node.js |
|---|---|
| `terraform init` | `npm install` |
| `.terraform/` | `node_modules/` |
| `.terraform.lock.hcl` | `package-lock.json` |
| `required_providers` | `dependencies` in `package.json` |
| `~> 5.0` | `^5.0.0` |
| The state file | A migrations history table |
| A module | An npm package |
| `variable` | A function parameter |
| `output` | `module.exports` |
| `for` expressions | `.map()` and `.filter()` |
| `object({...})` | A TypeScript interface |

## Prerequisites before Module 1

- [ ] An AWS account you are willing to create and destroy resources in
- [ ] AWS CLI installed and a working named profile (`aws sts get-caller-identity` returns your account)
- [ ] A code editor with a Terraform/HCL extension (VS Code: HashiCorp Terraform)
- [ ] Git, and a scratch directory outside this notes repo for lab code

---

## How to use these notes

Each module file follows the same shape:

| Section | What it gives you |
|---|---|
| **Why this module exists** | The problem the module solves. Read this even if you skim the rest. |
| **The core idea (one sentence)** | The one thing to memorise. |
| **Mental model** | The concept mapped onto something you already know. |
| **Teaching sections** | The actual content, with tables and `hcl` examples. |
| **Real-World Example** | How this shows up on a real team. |
| **Common Mistakes Beginners Make** | The traps. Read before the lab, not after. |
| **Hands-On Lab** | Type it out yourself. Do not copy-paste. |
| **Summary Table** | The one-page recap. |
| **Checkpoint** | Questions to answer out loud before moving on. |
| **Checkpoint — model answers** | Check yourself. Do not read early. |

All code in these notes lives inline in fenced `hcl` blocks. Retype it into a
scratch directory to run it — the point of the lab is the typing.

---

## Lab safety rules (read once, apply always)

Terraform creates **real, billable** infrastructure. These rules are not optional.

- [ ] Every lab runs in a scratch directory, never in this notes repo
- [ ] Early modules use `local`, `null` and `random` providers — **no AWS account needed, no cost**
- [ ] AWS labs stay inside free tier: `t3.micro`, S3, IAM, VPC, DynamoDB on-demand
- [ ] Any lab that costs money says so at the top, in bold
- [ ] Never commit `.tfstate`, `.tfvars`, or `.terraform/` to git
- [ ] Set a billing alarm on your AWS account before Module 1's lab

> Always run `terraform destroy` when you finish a lab. Every single time.

### The cost denylist

These are the resources that generate surprise AWS bills. None of them appear in
a lab without a bold cost warning first.

| Resource | Rough cost | Note |
|---|---|---|
| **NAT Gateway** | ~$32/month + data | **The number one accidental Terraform bill.** Use an S3 gateway VPC endpoint instead — those are free. |
| **EKS control plane** | ~$73/month | Never in a lab |
| **ALB / NLB** | ~$16/month | Only if a module explicitly says so |
| **Interface VPC endpoints** | ~$7/month each | Never in a lab |
| **RDS instance** | varies | Free tier only, and only for 12 months |
| **Unattached Elastic IP** | ~$3.60/month | The classic zombie cost after a failed destroy |
| **Route 53 hosted zone** | $0.50/month | Flagged where used |
| **Transit Gateway, GuardDuty, Config** | varies | Never in a lab |

Free or free-tier, and used freely in these labs: S3, IAM, VPC, subnets, route
tables, internet gateways, security groups, S3 gateway endpoints, CloudWatch log
groups, SNS, SQS, DynamoDB on-demand, SSM Parameter Store, Lambda, ECR.

### The sweep command

Every lab in these notes sets `default_tags` with `Project = "tf-learning"`. That
gives you one command to find anything a failed `destroy` left behind:

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=tf-learning \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

Empty output means you are clean. Run it after every session.

### Set a budget alarm before Module 1's lab

```bash
aws budgets create-budget \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --budget '{
    "BudgetName": "tf-learning-guardrail",
    "BudgetLimit": {"Amount": "5", "Unit": "USD"},
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }'
```

Add an email notification to it in the console. Five dollars is a generous
ceiling for everything in this curriculum except the final capstone.

---

**SECTION 0: AWS Networking, From Zero**

Module file: `00-aws-networking-primer.md`

Not Terraform. The foundation everything else assumes. Read this first.

1.  IP addresses and CIDR, explained from nothing

    - [ ] What an IP address actually is
    - [ ] Private address ranges, and why VPCs use `10.0.0.0/16`
    - [ ] What `/16`, `/24` and `/32` mean
    - [ ] `0.0.0.0/0` — and why it is fine in a route table but dangerous on port 22

2.  Regions and Availability Zones

    - [ ] Region = a city, AZ = a building in it
    - [ ] Why one AZ is a demo and two is a system

3.  The VPC and its subnets

    - [ ] What a VPC is, and why it is just a private network
    - [ ] What a subnet is, and why it lives in exactly one AZ
    - [ ] Public vs private — and why the only difference is one route table row

4.  Getting traffic in and out

    - [ ] Internet gateways
    - [ ] Route tables, and reading one properly
    - [ ] NAT gateways, and why they cost $32/month
    - [ ] VPC gateway endpoints, the free alternative

5.  Firewalls

    - [ ] Security groups: stateful, allow-only, attached to resources
    - [ ] Referencing a security group instead of an IP range
    - [ ] NACLs, and why you can mostly ignore them

6.  Putting it together

    - [ ] Trace one request through all sixteen hops
    - [ ] Draw Notely's complete network

---

**SECTION 1: IaC & Terraform Basics**

Module file: `01-iac-and-terraform-basics.md`

1.  What Infrastructure as Code actually solves (and what it does not)

    - [ ] The problems with clicking in the console
    - [ ] Idempotency, reproducibility, reviewability
    - [ ] Declarative vs imperative

2.  Where Terraform sits

    - [ ] Terraform vs CloudFormation (you know CFN already — this is the key comparison)
    - [ ] Terraform vs Serverless Framework vs CDK vs Pulumi vs Ansible
    - [ ] When *not* to use Terraform

3.  Installation and setup

    - [ ] Installing Terraform (macOS, Linux, Windows)
    - [ ] `tfenv` for version management
    - [ ] AWS credential configuration and the provider auth chain

4.  Providers and the registry

    - [ ] What a provider actually is
    - [ ] `required_providers` and version constraints
    - [ ] The `.terraform.lock.hcl` lock file

5.  Your first `apply`

    - [ ] The four commands: `init`, `plan`, `apply`, `destroy`
    - [ ] Reading plan output
    - [ ] **Lab:** create and destroy a single S3 bucket

---

**SECTION 2: HCL & the Core Workflow**

Module file: `02-hcl-and-core-workflow.md`

1.  HCL syntax fundamentals

    - [ ] Blocks, arguments, identifiers, expressions
    - [ ] Types: string, number, bool, list, set, map, object, tuple
    - [ ] Comments, heredocs, string interpolation

2.  The `resource` block

    - [ ] Resource type vs local name vs the real cloud name
    - [ ] Arguments vs attributes
    - [ ] Resource addresses (`aws_instance.web`)

3.  The dependency graph

    - [ ] Implicit dependencies via references
    - [ ] Why the graph exists and how to see it
    - [ ] Parallelism and what it means for ordering

4.  The core workflow in depth

    - [ ] What `init` really does
    - [ ] Reading a plan properly: `+`, `-`, `~`, `-/+`, `<=`
    - [ ] Saved plans (`-out`) and why CI uses them
    - [ ] `terraform destroy` and targeted operations

5.  Formatting and validation

    - [ ] `terraform fmt`
    - [ ] `terraform validate`
    - [ ] `terraform console` for experimenting

6.  **Lab:** a multi-resource local-only stack (no AWS account, no cost)

---

**SECTION 3: Terraform State**

Module file: `03-terraform-state.md`

1.  What state is and why it must exist

    - [ ] The mapping problem Terraform has to solve
    - [ ] What is actually inside a `.tfstate` file
    - [ ] Why state contains secrets in plaintext

2.  The refresh cycle

    - [ ] Config vs state vs reality — the three-way comparison
    - [ ] What drift is and how Terraform detects it

3.  Local state and its limits

    - [ ] Why local state breaks the moment a second person appears
    - [ ] `terraform.tfstate.backup`

4.  Remote backends

    - [ ] The S3 backend
    - [ ] State locking with DynamoDB (and S3 native locking)
    - [ ] Encryption, versioning, and recovering a corrupted state
    - [ ] Bootstrapping the backend (the chicken-and-egg problem)

5.  State manipulation commands

    - [ ] `state list`, `state show`, `state mv`, `state rm`, `state pull/push`
    - [ ] `terraform refresh` and why it is deprecated in favour of `-refresh-only`
    - [ ] When it is safe to hand-edit state (almost never)

6.  **Lab:** migrate a project from local state to an S3 + DynamoDB backend

---

**SECTION 4: Variables, Outputs & Locals**

Module file: `04-variables-outputs-locals.md`

- [ ] `variable` blocks: `type`, `default`, `description`, `nullable`
- [ ] Complex types: `object()`, `map(string)`, `list(object())`
- [ ] `validation` blocks with custom error messages
- [ ] `sensitive = true` and where it does and does not protect you
- [ ] The six ways to set a variable, and their precedence order
- [ ] `.tfvars`, `.auto.tfvars`, `TF_VAR_` environment variables
- [ ] `output` blocks, and outputs as a module's public API
- [ ] `locals` — when to use them instead of variables
- [ ] **Lab:** parameterise the Module 3 stack for dev/prod

---

**SECTION 5: Expressions & Functions**

Module file: `05-expressions-and-functions.md`

- [ ] Operators, conditionals, and the ternary
- [ ] `for` expressions over lists and maps
- [ ] Splat expressions
- [ ] String functions, collection functions, encoding functions
- [ ] `try`, `can`, `coalesce`, `lookup`, `merge`
- [ ] `templatefile()` and `file()`
- [ ] `dynamic` blocks — and when they hurt readability
- [ ] Type conversion rules and the errors they cause
- [ ] **Lab:** build a security group with dynamic ingress rules

---

**SECTION 6: Meta-Arguments**

Module file: `06-meta-arguments.md`

- [ ] `count` and the index-shifting problem
- [ ] `for_each` and why it is almost always the right answer
- [ ] Converting `count` to `for_each` safely
- [ ] `depends_on` — when implicit dependencies are not enough
- [ ] `lifecycle`: `create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`
- [ ] `provider` and provider aliases for multi-region/multi-account
- [ ] **Lab:** deploy the same stack to two regions

---

**SECTION 7: Data Sources**

Module file: `07-data-sources.md`

- [ ] `data` blocks vs `resource` blocks
- [ ] Finding AMIs, VPCs, subnets, and AZs dynamically
- [ ] `aws_caller_identity`, `aws_region`, `aws_partition`
- [ ] `terraform_remote_state` for cross-stack references
- [ ] Reading secrets from SSM and Secrets Manager
- [ ] When a data source resolves — plan time vs apply time
- [ ] **Lab:** stop hardcoding AMI IDs and account numbers

---

**SECTION 8: Modules**

Module file: `08-modules.md`

- [ ] What a module is (every directory is already one)
- [ ] Root module vs child module
- [ ] `module` blocks, `source`, and passing inputs
- [ ] Module outputs and composition
- [ ] Sources: local paths, git, the public registry
- [ ] Versioning and `version` constraints
- [ ] Writing a reusable module: interface design, defaults, docs
- [ ] When *not* to write a module (premature abstraction)
- [ ] **Lab:** extract a reusable VPC module and consume it twice

---

**SECTION 9: Environments & Project Layout**

Module file: `09-environments-and-layout.md`

- [ ] Workspaces: what they are and their real limitations
- [ ] Directory-per-environment — the layout most teams actually use
- [ ] Backend configuration per environment (`-backend-config`)
- [ ] Splitting state: blast radius and plan time
- [ ] Terragrunt — what problem it solves, and whether you need it
- [ ] **Lab:** restructure into `envs/dev` and `envs/prod`

---

**SECTION 10: Secrets & Provider Authentication**

Module file: `10-secrets-and-auth.md`

- [ ] The AWS provider credential chain, in order
- [ ] Assume-role and cross-account access
- [ ] OIDC federation from CI (no long-lived keys)
- [ ] Why secrets end up in state, and what to do about it
- [ ] Secrets Manager and SSM Parameter Store patterns
- [ ] `.gitignore` rules that are non-negotiable
- [ ] **Lab:** wire a database password without committing it

---

**SECTION 11: Testing & Validation**

Module file: `11-testing-and-validation.md`

- [ ] `terraform fmt -check` and `terraform validate` in CI
- [ ] `tflint` for provider-specific linting
- [ ] `checkov` / `tfsec` for security scanning
- [ ] `terraform test` (native, `.tftest.hcl`)
- [ ] Terratest — when a Go test suite is worth it
- [ ] `precondition` and `postcondition` blocks
- [ ] Policy as code: OPA and Sentinel, briefly
- [ ] **Lab:** add a full validation pipeline locally

---

**SECTION 12: Terraform in CI/CD**

Module file: `12-terraform-in-cicd.md`

- [ ] The plan-on-MR, apply-on-merge pattern
- [ ] Saved plan artifacts and why you must apply the plan you reviewed
- [ ] Manual approval gates
- [ ] State locking under concurrent pipelines
- [ ] **GitLab CI implementation** — builds directly on `../CI_CD/Gitlab-CICD.md`
- [ ] **GitHub Actions implementation** — builds directly on `../CI_CD/CI_CD.md`
- [ ] Credentials in CI: OIDC over static keys
- [ ] **Lab:** a complete `.gitlab-ci.yml` for a Terraform repo

---

**SECTION 13: Importing Existing Infrastructure & Drift**

Module file: `13-import-and-drift.md`

- [ ] `terraform import` (the command) vs `import` blocks (the config)
- [ ] Generating configuration with `-generate-config-out`
- [ ] The realistic import workflow for a legacy account
- [ ] `moved` blocks for safe refactoring
- [ ] `removed` blocks for safe deletion from state
- [ ] Detecting drift with `-refresh-only` and `-detailed-exitcode`
- [ ] Handling out-of-band changes as a team policy
- [ ] **Lab:** import a console-created bucket into Terraform

---

**SECTION 14: Real-World Project Structure & Best Practices**

Module file: `14-real-world-project.md`

- [ ] Repository layout for a real team
- [ ] Naming and tagging conventions (and enforcing them with `default_tags`)
- [ ] Cost awareness: what to never put in a lab, `infracost`
- [ ] Blast radius: sizing your state files
- [ ] Code review checklist for Terraform PRs
- [ ] Upgrading Terraform and provider versions safely
- [ ] Common production pitfalls
- [ ] **Capstone lab:** a complete 3-tier AWS application, built from modules

---

## Suggested pace

| Week | Modules | Focus |
|---|---|---|
| 1 | 0–2 | Networking foundations, then HCL and reading plans |
| 2 | 3 | State — do not rush this one, it explains most later confusion |
| 3 | 4–5 | The language: variables and expressions |
| 4 | 6–7 | Meta-arguments and data sources |
| 5 | 8–9 | Modules and environment layout |
| 6 | 10–11 | Secrets and validation |
| 7 | 12–13 | CI/CD and import |
| 8 | 14 | Capstone |

Slower is fine. Module 3 is the one that pays for itself.

---

## Glossary

| Term | Meaning |
|---|---|
| **HCL** | HashiCorp Configuration Language — the syntax Terraform files are written in |
| **Provider** | A plugin that teaches Terraform how to talk to an API (AWS, GitHub, Datadog) |
| **Resource** | A single piece of infrastructure Terraform creates and owns |
| **Data source** | A read-only lookup of something Terraform does *not* own |
| **State** | Terraform's record of which real resource corresponds to which config block |
| **Backend** | Where state is stored (local disk, S3, HCP Terraform) |
| **Module** | A reusable directory of Terraform configuration |
| **Root module** | The directory you run `terraform apply` in |
| **Plan** | The proposed set of changes, computed before anything is applied |
| **Drift** | Real infrastructure no longer matching what state says it should be |
| **Idempotent** | Applying twice produces the same result as applying once |

---

## Progress

- [ ] Module 0 — AWS Networking, From Zero
- [ ] Module 1 — IaC & Terraform Basics
- [ ] Module 2 — HCL & the Core Workflow
- [ ] Module 3 — Terraform State
- [ ] Module 4 — Variables, Outputs & Locals
- [ ] Module 5 — Expressions & Functions
- [ ] Module 6 — Meta-Arguments
- [ ] Module 7 — Data Sources
- [ ] Module 8 — Modules
- [ ] Module 9 — Environments & Project Layout
- [ ] Module 10 — Secrets & Provider Authentication
- [ ] Module 11 — Testing & Validation
- [ ] Module 12 — Terraform in CI/CD
- [ ] Module 13 — Importing Existing Infrastructure & Drift
- [ ] Module 14 — The Complete Build

---

## Files in this folder

| File | What it is |
|---|---|
| `terraform-roadmap.md` | This file. The plan and progress tracker. |
| `notely-architecture.md` | The running example: diagrams, components, costs |
| `00-aws-networking-primer.md` | VPC, subnets and routing from zero |
| `01-iac-and-terraform-basics.md` | Why IaC, providers, the four commands |
| `02-hcl-and-core-workflow.md` | HCL syntax, the dependency graph, reading plans |
| `03-terraform-state.md` | State, backends, locking, drift |
| `04-variables-outputs-locals.md` | Inputs, outputs, computed values |
| `05-expressions-and-functions.md` | `for` expressions, functions, templates |
| `06-meta-arguments.md` | `count`, `for_each`, `lifecycle`, providers |
| `07-data-sources.md` | Looking things up instead of hardcoding them |
| `08-modules.md` | Writing and using reusable modules |
| `09-environments-and-layout.md` | dev / staging / prod, project structure |
| `10-secrets-and-auth.md` | Credentials, Secrets Manager, the database |
| `11-testing-and-validation.md` | fmt, validate, tflint, checkov, `terraform test` |
| `12-terraform-in-cicd.md` | Pipelines for GitLab CI and GitHub Actions |
| `13-import-and-drift.md` | Adopting existing infrastructure, `moved`, `removed` |
| `14-real-world-project.md` | The capstone, and running it in production |
