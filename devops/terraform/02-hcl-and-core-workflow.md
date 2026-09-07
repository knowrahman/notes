# Module 2 — HCL & the Core Workflow

## Why this module exists

In Module 1 you ran `init`, `plan`, `apply` and `destroy`, and you copied some
HCL without it being explained. That is fine for a first resource. It stops being
fine the moment a plan says something you did not expect.

This module covers the language itself — blocks, types, expressions, references —
and then goes back through the four commands properly. In particular: **how
Terraform decides what order to create things in**, which is the single most
useful thing to understand about the tool after state.

---

## The core idea (one sentence)

HCL is a language for describing a graph of resources, and Terraform builds that
graph automatically from the references between them.

> You never tell Terraform what order to do things in. You reference one resource from another, and the order falls out of that.

---

## Mental model (mapped to what you already know)

HCL looks like JSON that grew up. If you have written a `.gitlab-ci.yml`
(`../CI_CD/Gitlab-CICD.md`) or a `serverless.yml`, the shape is familiar — but
there is one crucial difference.

In YAML pipelines, you declare stage order explicitly:

```yaml
stages:
  - build
  - test
  - deploy
```

You are telling the tool the sequence. In HCL you never do that. Instead:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id      # <-- this reference IS the ordering
}
```

Because the subnet needs `aws_vpc.main.id`, and that value does not exist until
the VPC is created, Terraform *derives* that the VPC must come first. Order is a
consequence of data flow, not a declaration.

This is closer to how a build system like Make or MSBuild works than to a CI
pipeline: you declare what depends on what, and the tool schedules it.

---

## HCL syntax fundamentals

### Blocks

Everything in HCL is a block. A block has a **type**, zero or more **labels**,
and a body in braces.

```hcl
block_type "label_one" "label_two" {
  argument = value
}
```

The block types you will meet:

| Block | Labels | Purpose |
|---|---|---|
| `terraform` | none | Settings for Terraform itself |
| `provider` | provider name | Configure a provider |
| `resource` | type, name | Create and manage something |
| `data` | type, name | Read something Terraform does not own |
| `variable` | name | Declare an input |
| `output` | name | Declare an output |
| `locals` | none | Named intermediate values |
| `module` | name | Call a child module |
| `moved` / `import` / `removed` | none | Refactoring and import operations |

Note that `locals` is plural and takes no label, while `variable` is singular and
takes one. That inconsistency is real and catches everyone once.

### Arguments and expressions

Inside a block body, `name = expression`:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"   # literal string
  instance_type = var.instance_type          # variable reference
  subnet_id     = aws_subnet.public.id       # resource reference
  count         = 3                          # number
  monitoring    = true                       # bool

  tags = {                                   # map
    Name = "web-server"
  }
}
```

### Nested blocks

Some arguments are blocks rather than values. The difference is the missing `=`:

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"

  ingress {                    # nested block — no equals sign
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {                    # you can repeat them
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

> If it has an `=`, it is an argument. If it does not, it is a block. Getting this wrong is the most common HCL syntax error.

### Comments

```hcl
# Hash comment — the conventional style

// Double-slash also works

/*
  Block comment.
  Rarely used.
*/
```

Use `#`. `terraform fmt` leaves both alone, but every codebase you will read uses
`#`.

### File organisation

Terraform loads **every `.tf` file in the directory** and concatenates them.
There is no import statement and no ordering between files — a resource in
`z.tf` can reference one in `a.tf` freely.

The conventional layout:

| File | Contents |
|---|---|
| `main.tf` | Primary resources |
| `variables.tf` | All `variable` blocks |
| `outputs.tf` | All `output` blocks |
| `providers.tf` | `terraform` and `provider` blocks |
| `locals.tf` | `locals` blocks |
| `versions.tf` | Sometimes split out from `providers.tf` |

This is purely convention for humans. Terraform does not care. But follow it —
every Terraform developer expects to find variables in `variables.tf`.

---

## Types

HCL has a real type system, and understanding it prevents a lot of confusing
errors.

### Primitives

```hcl
string = "hello"
number = 42
number = 3.14
bool   = true
```

Terraform converts between primitives automatically where unambiguous:
`"42"` becomes `42` where a number is expected, and `1` becomes `true` where a
bool is expected. Do not rely on this — be explicit.

### Collections

**List** — ordered, indexed by number, all elements the same type:

```hcl
availability_zones = ["ap-southeast-2a", "ap-southeast-2b", "ap-southeast-2c"]

# access
availability_zones[0]     # "ap-southeast-2a"
```

**Set** — unordered, no duplicates, no indexing:

```hcl
# You cannot do my_set[0] — sets have no order.
# Convert with tolist() first.
```

**Map** — key/value, keys always strings:

```hcl
tags = {
  Environment = "prod"
  Owner       = "platform"
}

# access
tags["Environment"]       # "prod"
tags.Environment          # same thing, only for valid identifiers
```

### Structural types

**Object** — like a map, but each attribute can be a different type:

```hcl
server = {
  name       = "web-01"
  cpu        = 2
  monitoring = true
}
```

**Tuple** — like a list, but each element can be a different type:

```hcl
mixed = ["hello", 42, true]
```

### The distinction that matters

| | **List** | **Set** | **Map** | **Object** |
|---|---|---|---|---|
| **Ordered** | Yes | No | No | No |
| **Duplicates** | Allowed | Not allowed | Keys unique | Keys unique |
| **Indexed by** | Number | Nothing | String key | Attribute name |
| **Element types** | All same | All same | All same | Can differ |

The one to watch is **list vs set**. Many AWS provider arguments are sets, not
lists, and the error message when you index into one is not obvious:

```text
Error: Invalid index
This value does not have any indices.
```

That means you have a set. Use `tolist()` or, better, restructure to use
`for_each` (Module 6).

### Strings and interpolation

```hcl
name = "web-${var.environment}-server"
```

`${...}` interpolates an expression into a string. Anything valid as an
expression works inside it:

```hcl
bucket = "logs-${var.env}-${data.aws_caller_identity.current.account_id}"
```

**Do not interpolate when you do not need to.** This is a very common beginner
habit:

```hcl
instance_type = "${var.instance_type}"    # unnecessary
instance_type = var.instance_type          # correct
```

`terraform fmt` will not fix it, but linters flag it, and older Terraform
versions emitted a deprecation warning.

### Heredocs

For multi-line strings — IAM policies, user data scripts:

```hcl
user_data = <<-EOF
  #!/bin/bash
  yum update -y
  yum install -y nginx
  systemctl start nginx
EOF
```

The `-` in `<<-EOF` strips leading indentation, which keeps your HCL readable.
Always use `<<-` rather than `<<`.

For JSON, prefer `jsonencode()` over a heredoc — it validates the structure and
handles escaping:

```hcl
# Fragile
policy = <<-EOF
  { "Version": "2012-10-17", "Statement": [...] }
EOF

# Better
policy = jsonencode({
  Version = "2012-10-17"
  Statement = [
    {
      Effect   = "Allow"
      Action   = ["s3:GetObject"]
      Resource = "${aws_s3_bucket.data.arn}/*"
    }
  ]
})
```

---

## The `resource` block

### Anatomy

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
}
```

| Part | Value | What it is |
|---|---|---|
| Block type | `resource` | Tells Terraform to create and manage this |
| First label | `aws_instance` | **Resource type** — defined by the provider |
| Second label | `web` | **Local name** — yours, used for references |
| Address | `aws_instance.web` | How you refer to it elsewhere |

Three names get confused constantly, so be precise:

- **Resource type** (`aws_instance`) — fixed by the provider. The prefix before
  the first underscore tells you which provider owns it.
- **Local name** (`web`) — your label. Only exists inside Terraform. Changing it
  makes Terraform think you deleted one resource and created another.
- **The real name in AWS** — whatever the `name`/`tags.Name`/`bucket` argument
  says. Completely separate from the local name.

```hcl
resource "aws_s3_bucket" "logs" {       # local name: logs
  bucket = "acme-prod-logs-2024"        # real name in AWS
}
```

> Renaming the local name is a destroy-and-recreate unless you use a `moved` block. Module 13.

### Arguments vs attributes

**Arguments** are what you set. **Attributes** are what you read.

```hcl
resource "aws_s3_bucket" "data" {
  bucket = "my-bucket"       # argument — you set it
}

output "arn" {
  value = aws_s3_bucket.data.arn    # attribute — AWS computed it
}
```

Most arguments are also readable as attributes. Some attributes are computed
only — `id`, `arn`, `bucket_domain_name` — and cannot be set. The provider
documentation lists them separately under "Attribute Reference".

Finding what is available: the AWS provider docs at
`registry.terraform.io/providers/hashicorp/aws/latest/docs`. Every resource page
has Argument Reference (what you can set) and Attribute Reference (what you can
read). You will have this page open constantly.

---

## The dependency graph

This is the part worth slowing down for.

### Implicit dependencies

When one resource references another, Terraform records an edge in a directed
graph:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id       # subnet depends on vpc
  cidr_block = "10.0.1.0/24"
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id           # igw depends on vpc
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id           # depends on vpc

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id   # and on igw
  }
}
```

The graph:

```text
                aws_vpc.main
               /      |      \
              /       |       \
   aws_subnet.public  |    aws_internet_gateway.main
                      |         /
                      |        /
                 aws_route_table.public
```

Terraform:

1. Creates `aws_vpc.main` first — nothing else can proceed without its ID
2. Creates `aws_subnet.public` and `aws_internet_gateway.main` **in parallel** —
   neither depends on the other
3. Creates `aws_route_table.public` last

You wrote no ordering instructions. The order came entirely from which resource
needed which value.

### Seeing the graph

```bash
terraform graph
```

Outputs DOT format. Render it if you have Graphviz:

```bash
terraform graph | dot -Tpng > graph.png
```

Rarely necessary in practice, but worth doing once on a stack with ten resources
to make the model concrete.

### Parallelism

Terraform walks the graph and processes independent nodes concurrently — up to 10
at a time by default.

```bash
terraform apply -parallelism=1     # serialise, for debugging
terraform apply -parallelism=20    # more aggressive
```

You will occasionally need `-parallelism=1` to get a readable error out of a
failing apply. Otherwise leave it alone.

### Destroy order

On `destroy`, Terraform walks the graph **backwards**. The route table goes
first, then the IGW and subnet, then the VPC last. This is correct — AWS refuses
to delete a VPC that still contains a subnet.

### When implicit dependencies are not enough

Sometimes a real dependency exists that Terraform cannot see, because no value
flows between the resources. The classic case: an EC2 instance that reads from
S3 needs its IAM policy attached before it boots, but nothing in the instance
config references the policy.

```hcl
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = "t3.micro"

  depends_on = [aws_iam_role_policy_attachment.app_s3]
}
```

`depends_on` adds an explicit graph edge. Module 6 covers it properly. The rule
for now:

> Use `depends_on` only when there is genuinely no value to reference. If you can restructure to create a real reference, do that instead.

---

## The core workflow in depth

### `terraform init`, properly

```bash
terraform init
```

Four jobs:

1. **Backend initialisation** — works out where state lives. On first run with a
   remote backend it offers to migrate existing local state (Module 3).
2. **Provider installation** — reads `required_providers`, resolves versions
   against the lock file, downloads binaries into `.terraform/providers/`.
3. **Lock file management** — creates or verifies `.terraform.lock.hcl`.
4. **Module installation** — downloads any `module` blocks' sources into
   `.terraform/modules/`.

Useful flags:

| Flag | Purpose |
|---|---|
| `-upgrade` | Ignore the lock file, resolve to newest allowed versions |
| `-reconfigure` | Reinitialise the backend, discarding saved config |
| `-migrate-state` | Move state when changing backends |
| `-backend=false` | Skip backend init (useful for `validate` in CI) |
| `-backend-config=FILE` | Supply backend settings from a file (Module 9) |

When to re-run `init`:

- After adding or changing a provider
- After adding or changing a `module` block's source
- After changing the backend
- On a fresh clone of the repo
- In every CI job

It is safe to run any time and never touches infrastructure.

### Reading a plan properly

This is a skill worth deliberately practising. The five change symbols:

| Symbol | Terraform's word | Meaning | Danger |
|---|---|---|---|
| `+` | create | New resource | Low |
| `~` | update in-place | Modified without replacement | Low |
| `-` | destroy | Deleted | **High** |
| `-/+` | replace | Destroyed **then** recreated | **Highest** |
| `+/-` | replace (create first) | Created then old destroyed | Medium |
| `<=` | read | Data source lookup | None |

A `~` update:

```text
  # aws_instance.web will be updated in-place
  ~ resource "aws_instance" "web" {
        id            = "i-0abc123"
      ~ instance_type = "t3.micro" -> "t3.small"
        tags          = {
            "Name" = "web"
        }
    }
```

Only `instance_type` has a `~`. Unchanged attributes are shown for context
without a symbol. The instance keeps its ID — it is stopped, resized, started.

A `-/+` replacement — this is the one to read carefully:

```text
  # aws_instance.web must be replaced
-/+ resource "aws_instance" "web" {
      ~ ami           = "ami-0abc" -> "ami-0def" # forces replacement
      ~ id            = "i-0abc123" -> (known after apply)
      ~ private_ip    = "10.0.1.42" -> (known after apply)
        instance_type = "t3.micro"
    }

Plan: 1 to add, 0 to change, 1 to destroy.
```

Two things to notice:

- **`# forces replacement`** — this comment tells you exactly which attribute
  triggered it. Always find this comment before approving a replacement.
- The bottom line says `1 to add, 1 to destroy` — not `1 to change`. Replacements
  count in both columns.

Some attributes cannot be changed in place because the underlying API has no
update operation for them. Changing an EC2 AMI, an RDS engine, or an S3 bucket
name means a new resource. On stateful resources, that means **data loss**.

> The bottom line of a plan is `X to add, Y to change, Z to destroy`. If Z is not zero and you did not intend to delete something, stop.

The `-/+` vs `+/-` distinction: by default Terraform destroys then creates, which
causes downtime. `create_before_destroy` in a `lifecycle` block flips it to
`+/-` (Module 6).

### Saved plans

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

Applying a saved plan file:

- Requires no confirmation prompt (the plan was already reviewed)
- Applies **exactly** the reviewed changes — if reality has drifted since, the
  apply fails rather than doing something unexpected

This is the pattern every CI pipeline uses: plan on the merge request, save the
artifact, apply that exact artifact after approval. Module 12.

The saved plan is a binary file. Read it with:

```bash
terraform show tfplan            # human readable
terraform show -json tfplan      # machine readable, for policy checks
```

> A saved plan file contains variable values, including secrets. Never commit it or publish it as an unprotected CI artifact.

### Targeting

```bash
terraform apply -target=aws_instance.web
```

Applies only that resource and its dependencies.

This exists for emergencies — recovering from a partially failed apply, or
working around a provider bug. It is not a workflow. Routine use means your state
and config quietly diverge, because you keep applying subsets and never the whole
thing.

Terraform prints a warning when you use it, and the warning is right.

### Other useful commands

**`terraform fmt`** — canonical formatting. Aligns `=` signs, fixes indentation.

```bash
terraform fmt              # format current directory
terraform fmt -recursive   # include subdirectories
terraform fmt -check       # exit non-zero if unformatted — for CI
terraform fmt -diff        # show what would change
```

Run it before every commit. There is one correct format; arguing about it is
wasted time.

**`terraform validate`** — checks syntax and internal consistency.

```bash
terraform validate
```

It catches: syntax errors, references to resources that do not exist, type
mismatches, missing required arguments.

It does **not** catch: anything requiring the cloud API. An invalid AMI ID or a
bucket name someone else owns passes validation and fails at apply. Validation is
offline and requires `init` first, but no credentials.

**`terraform console`** — an interactive REPL. Genuinely the fastest way to learn
expressions:

```bash
terraform console
```

```text
> 1 + 2
3

> upper("hello")
"HELLO"

> [for n in ["a", "b", "c"] : upper(n)]
[
  "A",
  "B",
  "C",
]

> cidrsubnet("10.0.0.0/16", 8, 1)
"10.0.1.0/24"

> aws_s3_bucket.lab.arn
"arn:aws:s3:::tf-lab-01-a1b2c3d4"
```

It loads your state, so you can inspect real resource attributes. Use it whenever
a function's behaviour is unclear rather than guessing and running an apply.

**`terraform show`** — the current state, human-readable:

```bash
terraform show
terraform show -json | jq '.values.root_module.resources[].address'
```

**`terraform output`**:

```bash
terraform output                    # all outputs
terraform output bucket_name        # one, quoted
terraform output -raw bucket_name   # one, unquoted — for shell scripts
terraform output -json              # all, as JSON
```

`-raw` is the one you want in scripts. Without it you get shell-hostile quotes.

---

## Real-World Example

A team's `.tf` files for a small service:

```text
infra/
├── versions.tf      # terraform block, required_providers
├── providers.tf     # provider "aws" with default_tags
├── variables.tf     # environment, region, instance_type
├── network.tf       # VPC, subnets, route tables, IGW
├── compute.tf       # ASG, launch template, ALB
├── database.tf      # RDS instance, subnet group, parameter group
├── security.tf      # security groups, IAM roles
├── outputs.tf       # ALB DNS name, RDS endpoint
└── terraform.tfvars # non-secret values for this environment
```

Nine files, all loaded together as one root module. The split is entirely for
human navigation — Terraform sees one flat namespace.

The daily loop:

```bash
git checkout -b add-redis-cache
# edit compute.tf
terraform fmt
terraform validate
terraform plan
# read the plan, paste it into the PR description
git commit -am "Add ElastiCache Redis for session storage"
git push
```

The reviewer reads the plan output in the PR before reading the HCL, because the
plan is the ground truth about what will actually happen. Twelve lines of HCL can
produce a plan that replaces a database; the diff alone will not tell you that,
but the plan will.

---

## Common Mistakes Beginners Make

**1. Confusing arguments and blocks.**

```hcl
ingress = {           # wrong — ingress is a block
  from_port = 443
}

ingress {             # correct — no equals sign
  from_port = 443
}
```

**2. Unnecessary interpolation.**

```hcl
instance_type = "${var.instance_type}"   # noise
instance_type = var.instance_type         # correct
```

**3. Using the local name expecting it to appear in AWS.**

```hcl
resource "aws_instance" "web-server" { ... }
```

Nothing in AWS is called `web-server`. You need a `Name` tag:

```hcl
resource "aws_instance" "web_server" {
  tags = {
    Name = "web-server"
  }
}
```

(Also: use underscores in local names, not hyphens. Both work, but underscores
are the universal convention.)

**4. Renaming a local name and not understanding the plan.**

Rename `aws_instance.web` to `aws_instance.app` and the plan says
`1 to add, 1 to destroy`. Terraform tracks resources by address; a new address is
a new resource. Use a `moved` block (Module 13).

**5. Reaching for `depends_on` too early.**

If you find yourself adding `depends_on`, first check whether you can reference
an attribute instead. A real reference is better in every way: it is
self-documenting and it cannot go stale.

**6. Approving a plan without checking the destroy count.**

Read the last line. Every time.

**7. Indexing into a set.**

```text
Error: Invalid index
```

Means the value is a set, not a list. `tolist()` converts, but usually the better
answer is `for_each` (Module 6).

**8. Expecting file order to matter.**

`a.tf` does not run before `b.tf`. There is no file ordering. Only the dependency
graph orders anything.

---

## Hands-On Lab — HCL Without a Cloud Account

**Cost: free. No AWS account required.** This lab uses only the `local`,
`random` and `null` providers, so everything runs on your machine.

The point is to concentrate on the language and the graph without cloud latency
or billing risk.

### Goal

Build a small stack of interdependent resources, observe the dependency graph,
practise reading plans, and use `terraform console`.

### Step 1: Set up

```bash
mkdir -p ~/terraform-labs/02-hcl
cd ~/terraform-labs/02-hcl
```

### Step 2: Providers

Create `versions.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.4"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}
```

No `provider` block needed — these providers have no required configuration.

### Step 3: A chain of dependencies

Create `main.tf`:

```hcl
# 1. A random pet name. Nothing depends on anything yet.
resource "random_pet" "project" {
  length    = 2
  separator = "-"
}

# 2. A random password. Independent of the pet.
resource "random_password" "db" {
  length           = 24
  special          = true
  override_special = "!#$%&*()-_=+[]{}<>:?"
}

# 3. Depends on BOTH of the above.
resource "local_file" "config" {
  filename = "${path.module}/generated/config.json"

  content = jsonencode({
    project     = random_pet.project.id
    environment = "lab"
    database = {
      host     = "localhost"
      port     = 5432
      password = random_password.db.result
    }
  })
}

# 4. Depends only on the pet name.
resource "local_file" "readme" {
  filename = "${path.module}/generated/README.md"

  content = <<-EOT
    # ${random_pet.project.id}

    Generated by Terraform.
    Config lives at ${local_file.config.filename}
  EOT
}
```

Look at the last one carefully: it references `local_file.config.filename`, so
`readme` depends on `config`, which depends on both random resources.

Create `outputs.tf`:

```hcl
output "project_name" {
  description = "The generated project name"
  value       = random_pet.project.id
}

output "config_path" {
  description = "Where the config file was written"
  value       = local_file.config.filename
}

output "db_password" {
  description = "The generated database password"
  value       = random_password.db.result
  sensitive   = true
}
```

### Step 4: Init and plan

```bash
terraform init
terraform plan
```

Read the plan. Expect `Plan: 4 to add, 0 to change, 0 to destroy.`

Find `(known after apply)` on `local_file.config.content` — Terraform cannot
compute the file content until the random values exist.

### Step 5: Apply and inspect

```bash
terraform apply
```

```bash
cat generated/config.json
cat generated/README.md
```

Try to read the password:

```bash
terraform output db_password
```

```text
db_password = <sensitive>
```

`sensitive = true` hides it from output. To actually get it:

```bash
terraform output -raw db_password
```

> `sensitive = true` hides values from CLI output. It does **not** encrypt them — the password is plaintext in the state file. Module 10.

### Step 6: See the graph

```bash
terraform graph
```

Trace the edges in the DOT output. You should find `local_file.config` depending
on both `random_pet.project` and `random_password.db`, and `local_file.readme`
depending on `local_file.config`.

If you have Graphviz:

```bash
terraform graph | dot -Tpng > graph.png && open graph.png
```

### Step 7: Practise reading plans — an in-place update

Change the README content in `main.tf`:

```hcl
content = <<-EOT
  # ${random_pet.project.id}

  Generated by Terraform.
  Config lives at ${local_file.config.filename}

  Updated in Module 2.
EOT
```

```bash
terraform plan
```

`Plan: 0 to add, 1 to change, 0 to destroy.` — a `~` in-place update on one
resource. The random values are untouched.

Apply it.

### Step 8: Practise reading plans — a forced replacement

Change the pet name length:

```hcl
resource "random_pet" "project" {
  length    = 3        # was 2
  separator = "-"
}
```

```bash
terraform plan
```

Now you get something much bigger:

```text
Plan: 4 to add, 0 to change, 4 to destroy.
```

Read why. `length` on `random_pet` forces replacement — look for the
`# forces replacement` comment. And because both `local_file` resources reference
the pet's ID, the change cascades: all four resources are replaced.

**This is the single most important thing in this lab.** A one-line change caused
four replacements, because of the dependency graph. On real infrastructure, that
same cascade is how a small edit takes down a database.

Apply it and confirm the generated files changed.

### Step 9: `terraform console`

```bash
terraform console
```

```text
> random_pet.project.id
"charming-corgi"

> upper(random_pet.project.id)
"CHARMING-CORGI"

> length(random_password.db.result)
24

> split("-", random_pet.project.id)
[
  "charming",
  "corgi",
]

> jsonencode({ a = 1, b = "two" })
"{\"a\":1,\"b\":\"two\"}"

> cidrsubnet("10.0.0.0/16", 8, 3)
"10.0.3.0/24"

> [for i in range(3) : "subnet-${i}"]
[
  "subnet-0",
  "subnet-1",
  "subnet-2",
]
```

`exit` to leave. Spend a few minutes here — it is the fastest feedback loop
Terraform has.

### Step 10: `fmt` and `validate`

Deliberately break the formatting:

```hcl
resource "random_pet" "project" {
length=3
    separator="-"
}
```

```bash
terraform fmt -diff
```

It shows and fixes the change. Then:

```bash
terraform validate
```

```text
Success! The configuration is valid.
```

Now break it properly — reference something that does not exist:

```hcl
output "broken" {
  value = random_pet.nonexistent.id
}
```

```bash
terraform validate
```

```text
Error: Reference to undeclared resource
```

Remove it again.

### Step 11: Tear down

```bash
terraform destroy
```

```bash
ls generated/
```

The files are gone — Terraform deleted them because it owned them.

### What you should have at the end

- A four-resource dependency chain built and destroyed
- Direct experience of a replacement cascading through a graph
- Comfort with `terraform console`
- Understanding of what `sensitive = true` does and does not do

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **Block** | `type "label" { ... }` — everything in HCL is one |
| **Argument** | `name = value` — has an equals sign |
| **Nested block** | `ingress { ... }` — no equals sign |
| **Resource address** | `aws_instance.web` — type plus local name |
| **Local name** | Your label. Not visible in AWS. Renaming = recreate. |
| **Argument** | What you set |
| **Attribute** | What you read (some are computed-only) |
| **Implicit dependency** | Created by referencing another resource |
| **`depends_on`** | Explicit edge, for when no value flows |
| **Dependency graph** | Built from references; determines all ordering |
| **Parallelism** | 10 concurrent operations by default |
| **`+`** | Create |
| **`~`** | Update in place |
| **`-`** | Destroy |
| **`-/+`** | Replace — destroy then create. Read carefully. |
| **`# forces replacement`** | The comment naming the attribute that caused it |
| **`(known after apply)`** | Value not computable until apply. Normal. |
| **`-out=tfplan`** | Save a plan; apply exactly that |
| **`terraform fmt`** | Canonical formatting. Run before every commit. |
| **`terraform validate`** | Offline syntax and reference check |
| **`terraform console`** | REPL. The fastest way to learn expressions. |
| **`-target`** | Emergency escape hatch, not a workflow |

### File conventions

| File | Contents |
|---|---|
| `versions.tf` | `terraform` block, `required_providers` |
| `providers.tf` | `provider` blocks |
| `variables.tf` | `variable` blocks |
| `main.tf` | Resources |
| `outputs.tf` | `output` blocks |

All `.tf` files in a directory load together. File order never matters.

---

## Checkpoint (answer briefly)

1. You never write ordering instructions in Terraform. So how does it know to create a VPC before a subnet?
2. What is the difference between `ingress { ... }` and `ingress = { ... }`?
3. You rename `resource "aws_instance" "web"` to `resource "aws_instance" "api"`. What does the plan show, and why?
4. In a plan, what is the difference between `~` and `-/+`? Which one should make you stop?
5. What does `(known after apply)` mean, and is it a problem?
6. Why does CI use `terraform plan -out=tfplan` followed by `terraform apply tfplan`, rather than just `terraform apply -auto-approve`?
7. A resource attribute cannot be indexed — `Error: Invalid index`. What is the likely cause?

---

## Checkpoint — model answers

### 1. How does Terraform know the VPC comes first?

From the **reference**. The subnet's config contains `vpc_id = aws_vpc.main.id`.
That value does not exist until the VPC has been created, so Terraform records a
dependency edge from the subnet to the VPC and creates the VPC first.

The general rule: every time you reference one resource's attribute from another,
you create a graph edge. Terraform builds a directed acyclic graph from all such
references, then walks it — creating independent resources in parallel, and
walking it backwards on destroy so the VPC is deleted last.

You never declare order. Order is a consequence of data flow.

### 2. `ingress { }` vs `ingress = { }`

The first is a **nested block**, the second is an **argument assigned a map**.
They are different syntax for different things, and the provider schema decides
which one a given name expects.

`ingress` on `aws_security_group` is a block, so it takes no `=`. Writing
`ingress = { ... }` produces a syntax error.

The rule: if there is an `=`, it is an argument taking a value. If there is not,
it is a nested block. When in doubt, check the provider docs — blocks are
documented with their own sub-argument lists.

### 3. Renaming the local name

The plan shows `1 to add, 1 to destroy`.

Terraform identifies resources by their **address** — `aws_instance.web`. State
maps that address to a real EC2 instance ID. When you rename to
`aws_instance.api`, Terraform sees an address in state (`web`) that no longer
appears in the config, and an address in config (`api`) that is not in state. It
concludes: destroy one, create the other.

The instance itself has not changed at all. Terraform simply has no way to know
the two addresses refer to the same thing.

The fix is a `moved` block, which tells Terraform the address changed without the
resource changing (Module 13). Before `moved` blocks existed, you used
`terraform state mv` (Module 3).

### 4. `~` vs `-/+`

`~` is **update in place**. The resource keeps its identity — same ID, same ARN,
same data. An EC2 instance is stopped, resized, restarted. Disruptive perhaps,
but nothing is lost.

`-/+` is **replace**: the existing resource is destroyed and a brand-new one
created. New ID, new ARN, and for anything stateful, **the data is gone**.

`-/+` is the one to stop on. It happens when you change an attribute the
underlying API cannot update — an EC2 AMI, an RDS engine, an S3 bucket name.
Terraform annotates the responsible attribute with `# forces replacement`; find
that comment before approving.

Also check the bottom line: replacements show as both an add and a destroy, so
`Plan: 1 to add, 0 to change, 1 to destroy` on what you thought was a small edit
is a warning sign.

### 5. `(known after apply)`

It means the value is computed by the provider or the cloud, and does not exist
until the resource is actually created. ARNs, instance IDs, private IPs — AWS
assigns these, so at plan time Terraform genuinely does not know them.

It is not a problem. It is expected on every new resource.

It does have one practical consequence: when an unknown value feeds into another
resource's argument, that downstream resource's plan is also partly unknown. On a
large stack this is why plans sometimes show less detail than you would like — the
uncertainty propagates through the graph.

### 6. Why saved plans in CI?

Because `apply -auto-approve` runs a **fresh plan** and applies it without anyone
seeing it. Between the plan a reviewer approved on the merge request and the
apply that runs after merge, reality can change — someone else applied, a
colleague clicked something in the console, a data source now returns a different
AMI. The applied changes would not be the reviewed changes.

`plan -out=tfplan` freezes the exact set of changes into a file. `apply tfplan`
applies precisely that, and **fails** if state has moved since the plan was
generated, rather than silently doing something different.

So the guarantee is: what a human approved is exactly what runs, or nothing runs.
That is the whole point of the pattern (Module 12).

Caveat: the plan file contains variable values including secrets, so it must be a
protected artifact, never a public one.

### 7. `Error: Invalid index`

The value is a **set**, not a list. Sets are unordered and have no indices, so
`my_set[0]` is meaningless — there is no "first" element.

Many AWS provider attributes are sets rather than lists, precisely because the
API makes no ordering guarantee.

Two fixes:

- `tolist(my_set)[0]` — converts to a list. Works, but the resulting order is
  arbitrary, so relying on the index is fragile.
- Restructure to iterate the set with `for_each` or a `for` expression instead of
  indexing. Almost always the better answer (Modules 5 and 6).

---

## Next lesson

**Module 3 — Terraform State** (`03-terraform-state.md`)

You have seen `terraform.tfstate` twice now and both times been told "Module 3
explains this". This is that module, and it is the most important one in the
curriculum.

State is what makes Terraform work and it is what makes Terraform break. Almost
every confusing Terraform error a beginner hits — resources recreated for no
reason, "resource already exists", two people overwriting each other — traces
back to a state misunderstanding.

Module 3's lab migrates a project from local state to a proper S3 backend with
DynamoDB locking, which is the setup every real team uses.
