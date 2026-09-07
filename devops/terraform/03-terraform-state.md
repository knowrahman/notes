# Module 3 — Terraform State

## Why this module exists

This is the most important module in the curriculum, and it is the one your
CloudFormation experience actively works against you on.

Your AWS notes say, of CloudFormation:

> It is cloud formations job to keep the logical and physical resources in sync.

That is exactly right, and it is exactly what Terraform's state file does. The
difference is that CloudFormation does it **inside AWS, where you never see it**.
Terraform hands you the file and the responsibility.

Almost every genuinely confusing Terraform error a beginner hits is a state
problem wearing a disguise:

- "Why is Terraform recreating a resource I did not change?"
- "Error: resource already exists"
- "My colleague ran apply and now my plan wants to delete everything"
- "The state file is locked and I cannot do anything"

All state. Understand this module and those stop being mysteries.

---

## The core idea (one sentence)

State is Terraform's record of which real-world object corresponds to which
resource block in your configuration.

> Config says what you want. The cloud says what exists. State is the only thing that knows *which* existing thing is *which* config block.

---

## Mental model (mapped to what you already know)

### From CloudFormation

CloudFormation has exactly this concept. You already know it under different
names:

| CloudFormation | Terraform |
|---|---|
| Logical resource (`MyBucket`) | Resource address (`aws_s3_bucket.data`) |
| Physical resource (`acme-prod-logs-2024`) | The `id` recorded in state |
| The stack's internal mapping between them | **The state file** |
| Stored by AWS, invisible | Stored by you, in a file you can open |

When your notes say CloudFormation keeps logical and physical resources in sync,
they are describing a state file. AWS just does not let you see it.

### From EF Core

Module 1 used this analogy and it holds here too. State is the
`__EFMigrationsHistory` table: a record of what has already been applied, so the
tool knows what to do next. Delete that table and EF thinks the database is
empty and tries to create everything from scratch. Delete your state file and
Terraform does exactly the same thing.

### Why the cloud API alone is not enough

The obvious question: why does Terraform not just ask AWS what exists?

Because "what exists" does not answer "which config block does this belong to".
Suppose you have three S3 buckets in your account, and this config:

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "acme-logs"
}
```

Terraform lists the buckets and finds `acme-logs`, `acme-uploads`, `acme-backups`.
Which one does `aws_s3_bucket.logs` refer to? It cannot tell. Worse: is
`acme-uploads` a resource Terraform used to manage and should now delete, or is
it someone else's bucket it should never touch?

State answers both. It says: `aws_s3_bucket.logs` → `acme-logs`, and nothing
else in this account is mine.

Three things state provides that the API cannot:

1. **Identity mapping** — address to real ID
2. **Ownership** — which resources Terraform is responsible for
3. **Metadata** — dependency edges and provider info recorded at create time,
   needed to compute a correct destroy order even after you delete the config

---

## What is actually in a state file

Open one. It is JSON:

```json
{
  "version": 4,
  "terraform_version": "1.9.5",
  "serial": 7,
  "lineage": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "outputs": {
    "bucket_name": {
      "value": "tf-lab-01-a1b2c3d4",
      "type": "string"
    }
  },
  "resources": [
    {
      "mode": "managed",
      "type": "aws_s3_bucket",
      "name": "lab",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "tf-lab-01-a1b2c3d4",
            "arn": "arn:aws:s3:::tf-lab-01-a1b2c3d4",
            "bucket": "tf-lab-01-a1b2c3d4",
            "region": "ap-southeast-2",
            "tags_all": { "ManagedBy": "Terraform" }
          },
          "dependencies": [
            "random_id.suffix"
          ]
        }
      ]
    }
  ]
}
```

The fields that matter:

| Field | Meaning |
|---|---|
| `version` | State file format version. Terraform's, not yours. |
| `terraform_version` | Which CLI wrote it. Older CLIs refuse newer state. |
| `serial` | Increments on every write. Used to detect stale writes. |
| `lineage` | A UUID identifying this state's ancestry. Prevents pushing unrelated state. |
| `outputs` | Cached output values — this is how `terraform output` is instant |
| `resources[].mode` | `managed` (a `resource`) or `data` (a `data` source) |
| `resources[].type` + `.name` | Together these form the address |
| `instances[].attributes` | **Every attribute of the real resource** |
| `instances[].dependencies` | The graph edges, recorded at create time |

### State contains secrets in plaintext

This is not a footnote. It is the single most important operational fact about
Terraform.

`attributes` holds *every* attribute, including sensitive ones:

```json
"attributes": {
  "identifier": "prod-db",
  "username": "admin",
  "password": "S3cr3t-P4ssw0rd-Here",
  "endpoint": "prod-db.abc123.ap-southeast-2.rds.amazonaws.com"
}
```

That is an RDS password, in plaintext, in a JSON file.

`sensitive = true` does **not** help. It only hides values from CLI output. The
state file is unaffected.

Consequences you must internalise now:

- **Never commit state to git.** Especially in a public repo.
- **Encrypt state at rest.** The S3 backend's `encrypt = true` is not optional.
- **Restrict who can read the state bucket.** Read access to state is read
  access to every secret in your infrastructure.
- **Treat a leaked state file as a credential leak.** Rotate, do not just delete.

> Anyone who can read your state file can read your database passwords. Secure it like a password vault, because it is one.

---

## The three-way comparison

Every `terraform plan` compares three things, not two:

```text
   CONFIG              STATE               REALITY
 (.tf files)      (terraform.tfstate)     (AWS API)
      |                   |                   |
      |                   |                   |
      +--------- terraform plan --------------+
                          |
                     the diff
```

| Comparison | What it detects | Terraform's response |
|---|---|---|
| Config vs State | You edited your `.tf` files | Plan the change |
| State vs Reality | Someone changed AWS outside Terraform | **Drift** — plan to revert it |
| Config in, State out | You added a new resource block | Create it |
| State in, Config out | You deleted a resource block | Destroy it |

The **refresh** step is where state is compared against reality. By default,
`plan` and `apply` refresh first: for every resource in state, Terraform calls
the API and updates its in-memory copy of the attributes.

```bash
terraform plan                  # refreshes by default
terraform plan -refresh=false   # skip it — faster, but you may miss drift
```

`-refresh=false` is worth knowing about for very large states where refresh takes
minutes. Use it knowing you are trading accuracy for speed.

### Drift, and the right way to handle it

Drift is real infrastructure no longer matching what state records.

Someone resizes an instance in the console. Next plan:

```text
  ~ resource "aws_instance" "web" {
      ~ instance_type = "t3.small" -> "t3.micro"
    }
```

Terraform proposes to change it *back*, because your config says `t3.micro`.
That is correct behaviour — Terraform's job is to make reality match config.

But sometimes the console change was right and the config is stale. For that,
there is refresh-only mode:

```bash
terraform plan -refresh-only
terraform apply -refresh-only
```

This updates **state** to match reality without changing any infrastructure. You
are telling Terraform "accept what is actually there". You then update your
config to match, and the next plan is empty.

| Command | Changes infrastructure | Changes state |
|---|---|---|
| `terraform plan` | No | No |
| `terraform apply` | Yes | Yes |
| `terraform plan -refresh-only` | No | No (preview only) |
| `terraform apply -refresh-only` | **No** | **Yes** |

> `apply -refresh-only` is the safe way to say "reality is right, my state is wrong".

The old `terraform refresh` command did this without showing you a plan first.
It is deprecated for exactly that reason — use `-refresh-only` and read the diff.

---

## Local state and why it breaks

By default state is a file called `terraform.tfstate` in your working directory,
with `terraform.tfstate.backup` holding the previous version.

Local state works fine for exactly one situation: one person, one machine, a
project nobody else touches.

It fails the moment anything else is true:

| Problem | What happens |
|---|---|
| **Second person** | They have no state. Their plan wants to create everything you already created. Apply, and you get duplicates or "already exists" errors. |
| **No locking** | Two applies at once interleave writes. State is corrupted. |
| **Laptop dies** | State is gone. Terraform no longer knows about any of your infrastructure. |
| **CI** | Every pipeline run starts with empty state. |
| **Secrets** | A plaintext secrets file sitting in a working directory that also contains a git repo. |
| **No history** | Overwrite it wrongly and there is no previous version beyond the single `.backup`. |

The one that bites hardest is the first. Two engineers with separate local states
managing the same infrastructure is genuinely difficult to untangle.

> Local state is for labs. Any project with a second person, or a pipeline, needs a remote backend.

### The `.gitignore` rules

Non-negotiable, particularly in a public repository:

```text
# Terraform
**/.terraform/*
*.tfstate
*.tfstate.*
crash.log
crash.*.log
*.tfvars
*.tfvars.json
override.tf
override.tf.json
*_override.tf
*_override.tf.json
.terraformrc
terraform.rc
.terraform.tfstate.lock.info
tfplan
*.tfplan
```

Two deliberate exceptions:

- **`.terraform.lock.hcl` is NOT ignored.** Commit it (Module 1).
- **`*.tfvars` is ignored**, so share example values as `terraform.tfvars.example`
  and have people copy it.

---

## Remote backends

A **backend** determines where state is stored and whether operations are locked.

```hcl
terraform {
  backend "s3" {
    bucket       = "acme-terraform-state"
    key          = "prod/network/terraform.tfstate"
    region       = "ap-southeast-2"
    encrypt      = true
    use_lockfile = true
  }
}
```

What each argument does:

| Argument | Purpose |
|---|---|
| `bucket` | The S3 bucket holding state |
| `key` | Path within the bucket. **The most important one** — it is what separates one state from another |
| `region` | Bucket's region |
| `encrypt` | Server-side encryption. Always `true`. |
| `kms_key_id` | Use a customer-managed KMS key rather than SSE-S3 |
| `use_lockfile` | S3-native locking (Terraform 1.10+) |
| `dynamodb_table` | The older locking mechanism (see below) |

The `key` deserves attention. It is how you get separate states for separate
things:

```text
acme-terraform-state/
├── dev/network/terraform.tfstate
├── dev/app/terraform.tfstate
├── prod/network/terraform.tfstate
└── prod/app/terraform.tfstate
```

Four independent states in one bucket. A mistake in `dev/app` cannot touch
`prod/network`, because Terraform never loads it. That is **blast radius
control**, and Module 9 builds on it.

### What the S3 bucket needs

You are storing plaintext secrets, so:

| Setting | Why |
|---|---|
| **Versioning enabled** | Your undo button. A bad state write is recoverable. |
| **Encryption enabled** | Secrets at rest. |
| **Public access blocked** | Obviously. |
| **Restrictive bucket policy** | Read access = read every secret you have. |

Versioning is the one people skip and later wish they had not. If a `state rm`
goes wrong, the previous state is one S3 version away.

### State locking

Two engineers run `apply` simultaneously against the same state. Both read
serial 5. Both compute a plan. Both write serial 6. One write wins, the other is
lost — and now state does not match reality, with resources created that nothing
knows about.

Locking prevents this. Before any write, Terraform acquires a lock; anyone else
gets:

```text
Error: Error acquiring the state lock

Lock Info:
  ID:        7c4b9e2a-1234-5678-90ab-cdef12345678
  Path:      acme-terraform-state/prod/terraform.tfstate
  Operation: OperationTypeApply
  Who:       rahman@laptop
  Created:   2026-09-07 10:23:45 UTC
```

Two mechanisms exist:

**S3 native locking (Terraform 1.10+)** — the modern answer:

```hcl
terraform {
  backend "s3" {
    bucket       = "acme-terraform-state"
    key          = "prod/terraform.tfstate"
    region       = "ap-southeast-2"
    encrypt      = true
    use_lockfile = true
  }
}
```

One argument, no extra infrastructure. It uses S3 conditional writes.

**DynamoDB locking** — the classic approach, still everywhere:

```hcl
terraform {
  backend "s3" {
    bucket         = "acme-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "ap-southeast-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

The table needs exactly one thing: a partition key named `LockID` of type
string. You already know DynamoDB partition keys
(`../../cloud/aws serverless/dynamo-db.md`) — that is all this is. Terraform
writes an item keyed by the state path, and the conditional-write semantics of
DynamoDB make it atomic.

Use `use_lockfile` on new projects. Know `dynamodb_table` because you will
inherit projects that use it.

### When a lock gets stuck

A pipeline job is cancelled mid-apply. The lock is never released. Everything
now fails with "Error acquiring the state lock".

```bash
terraform force-unlock 7c4b9e2a-1234-5678-90ab-cdef12345678
```

The ID comes from the error message.

> Before force-unlocking, confirm nobody is actually running an apply. Breaking a live lock is how state gets corrupted.

### The bootstrap problem

The backend needs an S3 bucket. You want to manage the bucket with Terraform.
But Terraform needs the backend to store its state. Chicken and egg.

Two solutions:

**A. Bootstrap then migrate.** Create the bucket with a small Terraform config
using local state, then migrate that config's state into the bucket it just
created. This is what the lab does — it is neat and it teaches migration.

**B. Create it by hand, once.** Two console clicks or two CLI commands, done
once per organisation, never touched again. Many teams do this deliberately: the
state bucket is foundational and deliberately outside Terraform's reach, so no
Terraform mistake can delete it.

Both are defensible. B is arguably more robust; A teaches you more.

### Migrating to a backend

Add the `backend` block, then:

```bash
terraform init -migrate-state
```

```text
Initializing the backend...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend
  to the newly configured "s3" backend. No existing state was found in the
  newly configured "s3" backend.

  Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes
```

Say `yes`. Terraform uploads your local state to S3. The local file remains as a
leftover — delete it once you have confirmed the migration worked.

Related flags:

| Flag | Purpose |
|---|---|
| `-migrate-state` | Copy state to the new backend |
| `-reconfigure` | Ignore existing backend config, start fresh (**does not migrate**) |
| `-backend-config=FILE` | Supply backend settings externally (Module 9) |

Do not confuse them. `-reconfigure` on a project with existing state means
Terraform starts with an empty state and wants to create everything again.

---

## The `state` subcommands

You will need these rarely, but when you need them you really need them.

### `terraform state list`

Every resource address in state:

```bash
terraform state list
```

```text
random_id.suffix
aws_s3_bucket.lab
aws_s3_bucket_versioning.lab
module.vpc.aws_vpc.this[0]
```

First thing to run when you are confused about what Terraform thinks it owns.

### `terraform state show`

Every attribute of one resource:

```bash
terraform state show aws_s3_bucket.lab
```

```text
# aws_s3_bucket.lab:
resource "aws_s3_bucket" "lab" {
    arn                         = "arn:aws:s3:::tf-lab-01-a1b2c3d4"
    bucket                      = "tf-lab-01-a1b2c3d4"
    id                          = "tf-lab-01-a1b2c3d4"
    region                      = "ap-southeast-2"
}
```

Better than `cat terraform.tfstate | jq` and it works with remote backends.

### `terraform state mv`

Change a resource's address without touching infrastructure:

```bash
terraform state mv aws_instance.web aws_instance.api
```

This is the fix for the rename problem from Module 2. Terraform now associates
the real instance with the new address, and the plan is empty.

Also used to move resources into modules:

```bash
terraform state mv aws_vpc.main module.network.aws_vpc.main
```

Modern Terraform prefers `moved` blocks — declarative, reviewable in a PR,
applied by `terraform apply` rather than run by hand (Module 13). `state mv` is
the imperative fallback.

### `terraform state rm`

Remove a resource from state **without destroying it**:

```bash
terraform state rm aws_s3_bucket.legacy
```

Terraform forgets the bucket exists. The bucket is still there in AWS — it is
now unmanaged.

Use when handing a resource to another team or another state file. There is a
`removed` block for this too (Module 13).

> `state rm` does not delete infrastructure. `terraform destroy` does. Do not confuse them under pressure.

### `terraform state pull` / `push`

```bash
terraform state pull > backup.tfstate       # download current state
terraform state push backup.tfstate          # upload state
```

`pull` is a good habit before any state surgery. `push` is genuinely dangerous —
it overwrites remote state wholesale. Terraform checks `lineage` and `serial` to
stop obvious mistakes, and `-force` bypasses those checks. Do not use `-force`
without a very specific reason.

### Hand-editing state

Almost never. The commands exist so you do not have to.

If you truly must:

1. `terraform state pull > backup.tfstate` — keep the original untouched
2. Copy it, edit the copy
3. Increment `serial`
4. `terraform state push` the edited copy
5. `terraform plan` — an empty plan means you got it right

Every step is a chance to break things. Reach for `state mv`, `state rm`,
`import`, or a `moved` block first.

---

## Real-World Example

A team of five, three environments, this state layout:

```text
s3://acme-tf-state/
├── prod/network/terraform.tfstate
├── prod/data/terraform.tfstate
├── prod/app/terraform.tfstate
├── staging/network/terraform.tfstate
├── staging/data/terraform.tfstate
├── staging/app/terraform.tfstate
└── dev/...
```

Nine states, all locked, all versioned, all encrypted.

**Why split by layer, not just by environment?** Blast radius and plan time. The
network layer changes twice a year; the app layer changes daily. Keeping them
separate means a routine app deploy runs a plan over twelve resources, not two
hundred, and a mistake in the app configuration cannot produce a plan that
deletes a VPC.

**What it looks like on a normal day.** An engineer opens an MR changing the app
layer. CI runs `plan` against `staging/app`, posts the output as a comment, and
a reviewer reads it. On merge, CI applies to staging, then to prod behind a
manual gate. GitLab's `resource_group` keyword serialises the apply jobs so two
pipelines cannot fight over the same state — the pipeline-level equivalent of the
DynamoDB lock.

**When it went wrong.** Someone ran `terraform apply` locally against prod with
an out-of-date branch, reverting a change made two hours earlier. The fix took
ten minutes because S3 versioning meant they could see exactly what state looked
like before. The policy afterwards: nobody applies to prod from a laptop, and
the prod state bucket policy denies write access to individual engineer roles —
only the CI role can write. That is a state-level control, and it exists because
state is where the authority lives.

---

## Common Mistakes Beginners Make

**1. Committing `terraform.tfstate` to git.**

The worst one. Every database password, private key, and generated secret, in a
repository. Delete the file and rotate the secrets — removing the commit is not
enough, the history persists in clones and forks.

**2. Confusing `state rm` with `destroy`.**

`state rm` forgets. `destroy` deletes. Running `state rm` expecting cleanup
leaves orphaned billable resources; running `destroy` expecting to just detach
deletes production.

**3. Manually deleting a resource in the console.**

Terraform's next plan wants to recreate it, which is usually fine — but if
something else came to depend on it in the meantime, you get a confusing cascade.
Use `terraform destroy -target` or remove it from config properly.

**4. Two people, two local state files.**

Both plans want to create everything. Both applies produce duplicates or
"already exists" errors. Set up a remote backend before the second person joins,
not after.

**5. No versioning on the state bucket.**

You will eventually need to roll state back. Versioning costs nothing and is the
only thing that makes recovery straightforward.

**6. `-reconfigure` when you meant `-migrate-state`.**

`-reconfigure` discards the existing backend association. Terraform sees an
empty state and plans to create your entire production estate a second time.
Read the plan, cancel, use the right flag.

**7. Assuming `sensitive = true` protects the state file.**

It hides values from CLI output. That is all. The state file is plaintext.

**8. Force-unlocking without checking.**

If someone really is mid-apply, breaking their lock is how you corrupt state.
Ask first.

**9. One giant state for everything.**

Every plan takes ten minutes and every change risks the whole estate. Split by
environment and by layer.

---

## Hands-On Lab — Local State to a Locked S3 Backend

**Cost: ~$0.** S3 storage for a few KB and an on-demand DynamoDB table are both
inside free tier. You will destroy everything at the end.

### Goal

Build a backend with local state, migrate into it, then practise the state
commands — including deliberately breaking things and recovering.

### Step 1: Build the backend infrastructure

```bash
mkdir -p ~/terraform-labs/03-state/bootstrap
cd ~/terraform-labs/03-state/bootstrap
```

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
      Project   = "tf-learning"
      ManagedBy = "Terraform"
      Ephemeral = "true"
    }
  }
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "state" {
  bucket = "tf-lab-state-${random_id.suffix.hex}"
}

# Versioning is your undo button for state. Never skip it.
resource "aws_s3_bucket_versioning" "state" {
  bucket = aws_s3_bucket.state.id

  versioning_configuration {
    status = "Enabled"
  }
}

# State holds secrets in plaintext. Encrypt it.
resource "aws_s3_bucket_server_side_encryption_configuration" "state" {
  bucket = aws_s3_bucket.state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "state" {
  bucket = aws_s3_bucket.state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# The classic locking mechanism. LockID is just a partition key.
resource "aws_dynamodb_table" "locks" {
  name         = "tf-lab-locks-${random_id.suffix.hex}"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}

output "state_bucket" {
  value = aws_s3_bucket.state.bucket
}

output "lock_table" {
  value = aws_dynamodb_table.locks.name
}
```

```bash
terraform init
terraform apply
```

Note the outputs — you need both names in a moment.

```bash
terraform output -raw state_bucket
terraform output -raw lock_table
```

Note that this bootstrap config is itself using **local state**. That is the
chicken-and-egg problem, and you are about to resolve it.

### Step 2: A project to migrate

```bash
mkdir -p ~/terraform-labs/03-state/app
cd ~/terraform-labs/03-state/app
```

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
      Project   = "tf-learning"
      ManagedBy = "Terraform"
      Ephemeral = "true"
    }
  }
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "app_data" {
  bucket = "tf-lab-appdata-${random_id.suffix.hex}"
}

resource "aws_cloudwatch_log_group" "app" {
  name              = "/tf-lab/app-${random_id.suffix.hex}"
  retention_in_days = 1
}

# Deliberately a "secret", to prove a point about state later.
resource "random_password" "db" {
  length  = 24
  special = true
}

output "bucket" {
  value = aws_s3_bucket.app_data.bucket
}
```

```bash
terraform init
terraform apply
```

### Step 3: Look at local state, and find the password

```bash
ls -la
```

You have `terraform.tfstate`. Now the important part:

```bash
grep -o '"result": "[^"]*"' terraform.tfstate
```

There is your generated password, in plaintext, in a file sitting in a directory.
`sensitive` was never going to help.

```bash
terraform state list
```

```text
random_id.suffix
random_password.db
aws_cloudwatch_log_group.app
aws_s3_bucket.app_data
```

### Step 4: Migrate to the remote backend

Add to `main.tf`, inside the `terraform` block:

```hcl
terraform {
  required_version = ">= 1.5.0"

  backend "s3" {
    bucket         = "REPLACE-WITH-YOUR-STATE-BUCKET"
    key            = "lab/app/terraform.tfstate"
    region         = "ap-southeast-2"
    encrypt        = true
    dynamodb_table = "REPLACE-WITH-YOUR-LOCK-TABLE"
  }

  required_providers {
    # ... unchanged
  }
}
```

Substitute the two names from Step 1.

```bash
terraform init -migrate-state
```

Answer `yes`.

```text
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
```

Verify it landed in S3:

```bash
aws s3 ls s3://$(cd ../bootstrap && terraform output -raw state_bucket)/lab/app/
```

And confirm Terraform is reading it:

```bash
terraform state list
```

Same four resources, now from S3.

```bash
terraform plan
```

`No changes.` — the migration preserved everything.

Clean up the leftover local file:

```bash
rm terraform.tfstate terraform.tfstate.backup
terraform plan
```

Still `No changes.` State is genuinely remote now.

### Step 5: Watch the lock

Open a second terminal in the same directory. In terminal one:

```bash
terraform apply
```

Stop at the `Enter a value:` prompt — do not answer. The lock is held.

In terminal two:

```bash
terraform plan
```

```text
Error: Error acquiring the state lock
...
  Who:       rahman@laptop
```

That is the mechanism that stops two applies corrupting state.

Go back to terminal one and answer `no` to release it.

### Step 6: Force-unlock

Simulate a cancelled CI job. In terminal one:

```bash
terraform apply
```

At the prompt, kill it with `Ctrl+C` twice — hard enough that the lock is not
released cleanly. Then:

```bash
terraform plan
```

If the lock is stuck, take the ID from the error and:

```bash
terraform force-unlock <LOCK_ID>
```

In real life, confirm nobody is mid-apply first.

### Step 7: `state rm` and re-import

Remove the log group from state without destroying it:

```bash
terraform state rm aws_cloudwatch_log_group.app
terraform state list
```

Three resources now. But:

```bash
aws logs describe-log-groups --log-group-name-prefix /tf-lab/
```

The log group still exists — Terraform just forgot it.

```bash
terraform plan
```

Terraform wants to create it, and will fail on apply because the name is taken.
That is the "resource already exists" error, and now you know exactly what causes
it.

Fix by importing it back:

```bash
terraform import aws_cloudwatch_log_group.app /tf-lab/app-<YOUR_SUFFIX>
```

Get the exact name from the `describe-log-groups` output.

```bash
terraform plan
```

`No changes.` Recovered. Module 13 covers import properly.

### Step 8: Drift and `-refresh-only`

Change something outside Terraform:

```bash
aws logs put-retention-policy \
  --log-group-name /tf-lab/app-<YOUR_SUFFIX> \
  --retention-in-days 7
```

```bash
terraform plan
```

Terraform wants to set it back to 1, because that is what your config says.

Now try the other direction:

```bash
terraform plan -refresh-only
```

This shows state being updated to match reality — no infrastructure change
proposed. If 7 were the correct value, you would `apply -refresh-only` and then
update your config to say 7.

For the lab, just let the normal plan revert it:

```bash
terraform apply
```

### Step 9: State versioning as an undo button

```bash
BUCKET=$(cd ../bootstrap && terraform output -raw state_bucket)
aws s3api list-object-versions \
  --bucket $BUCKET \
  --prefix lab/app/terraform.tfstate \
  --query 'Versions[].[VersionId,LastModified]' \
  --output table
```

Every state write is a separate S3 version. This is what recovery from a bad
state operation looks like — you can download any previous version and push it
back.

### Step 10: Tear down, in the right order

The app stack first, because its state lives in the bootstrap stack's bucket:

```bash
cd ~/terraform-labs/03-state/app
terraform destroy
```

Then move its state back to local so the bucket can be emptied — or simply empty
the bucket by hand:

```bash
BUCKET=$(cd ../bootstrap && terraform output -raw state_bucket)
aws s3api delete-objects --bucket $BUCKET \
  --delete "$(aws s3api list-object-versions --bucket $BUCKET \
    --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json)"
```

Then the bootstrap stack:

```bash
cd ~/terraform-labs/03-state/bootstrap
terraform destroy
```

Verify nothing is left:

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=tf-learning \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

Empty output means you are clean. **Keep this command** — it is how you find
anything a failed destroy leaves behind, and it works because you set
`default_tags`.

### What you should have at the end

- A backend built, migrated into, and destroyed
- A plaintext password found in a state file with your own eyes
- A lock observed, and force-unlocked
- A resource removed from state and re-imported
- Drift created and resolved both ways
- An empty AWS account

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **State** | Maps config addresses to real resource IDs |
| **Why it exists** | The API cannot say which resource belongs to which config block |
| **`serial`** | Increments on every write; detects stale writes |
| **`lineage`** | UUID identifying the state's ancestry |
| **Secrets in state** | Plaintext. Always. `sensitive` does not change this. |
| **Refresh** | Comparing state against reality |
| **Drift** | Reality no longer matching state |
| **`-refresh-only`** | Update state to match reality, change nothing |
| **Local state** | Fine for labs. Breaks with a second person or CI. |
| **Backend** | Where state lives and whether it locks |
| **S3 backend `key`** | What separates one state from another |
| **Locking** | Prevents concurrent applies corrupting state |
| **`use_lockfile`** | S3-native locking, TF 1.10+. Preferred. |
| **`dynamodb_table`** | Classic locking. `LockID` partition key. Still common. |
| **`force-unlock`** | Break a stuck lock. Check nobody is applying first. |
| **Bucket versioning** | Your undo button. Never skip it. |
| **`-migrate-state`** | Copy state to a new backend |
| **`-reconfigure`** | Start fresh — **does not migrate**. Dangerous by accident. |
| **`state list`** | What Terraform thinks it owns |
| **`state show`** | Every attribute of one resource |
| **`state mv`** | Rename an address without touching infrastructure |
| **`state rm`** | Forget a resource. **Does not delete it.** |
| **`state pull`** | Back up before surgery |
| **Blast radius** | Split state by environment and layer |

---

## Checkpoint (answer briefly)

1. Why can Terraform not just query the AWS API instead of keeping a state file?
2. You set `sensitive = true` on an RDS password variable. Is the password safe in the state file?
3. What is the difference between `terraform state rm` and `terraform destroy`?
4. Two engineers each have their own local `terraform.tfstate` for the same infrastructure. What goes wrong?
5. What does `terraform apply -refresh-only` do that `terraform apply` does not?
6. Why enable versioning on the S3 state bucket?
7. You add a `backend "s3"` block to an existing project. What is the difference between `terraform init -migrate-state` and `terraform init -reconfigure`, and which one could ruin your afternoon?

---

## Checkpoint — model answers

### 1. Why not just query the API?

Because the API tells Terraform *what exists*, not *which config block owns what*.

Given three buckets in an account and one `aws_s3_bucket.logs` block, the API
cannot say which bucket that block refers to — or whether the other two are
resources Terraform used to manage and should now delete, or someone else's
resources it must never touch.

State provides three things the API cannot: the **identity mapping** from address
to real ID, the **ownership boundary** marking which resources Terraform is
responsible for, and **metadata** like dependency edges recorded at create time,
which Terraform needs to compute a correct destroy order even after the config
has been deleted.

This is the same thing CloudFormation does when it keeps logical and physical
resources in sync. CFN just stores it inside AWS where you never see it.

### 2. Is the password safe in state?

**No.** `sensitive = true` only redacts values from CLI output — plan output,
apply output, `terraform output`. It has no effect on the state file.

The state file records every attribute of every resource in plaintext JSON,
including RDS passwords, generated secrets, and private keys. You can prove it
with `grep` in about two seconds, which the lab has you do.

The real protections are: encrypt state at rest (`encrypt = true`, ideally with a
customer-managed KMS key), restrict read access to the state bucket as tightly as
you would a password vault, never commit state to git, and treat any leaked state
file as a credential compromise requiring rotation.

### 3. `state rm` vs `destroy`

`terraform state rm` removes the resource from **state only**. The real
infrastructure is untouched — it continues to exist and continues to bill, it is
simply no longer managed by Terraform.

`terraform destroy` **deletes the real infrastructure** and then removes it from
state.

The failure modes are opposite and both bad. Using `state rm` when you meant
`destroy` leaves orphaned billable resources nobody is tracking. Using `destroy`
when you meant `state rm` deletes production.

`state rm` is for handing a resource to another team or another state file. The
declarative equivalent is a `removed` block (Module 13).

### 4. Two engineers, two local states

Neither state knows about the other's resources, so both plans propose to create
everything.

Whoever applies second either gets "already exists" errors (for globally unique
names like S3 buckets) or silently creates a **duplicate set** of infrastructure
(for anything AWS will happily create twice, like EC2 instances or security
groups). Now there are two VPCs, two databases, and two state files each
convinced it owns "the" infrastructure.

Untangling this means importing resources into one state, removing them from the
other, and destroying the duplicates — a bad afternoon.

There is also no locking, so even sharing a state file over Dropbox would let two
simultaneous applies interleave writes and corrupt it.

The fix is a remote backend with locking, set up **before** the second person
joins.

### 5. `apply -refresh-only`

Normal `terraform apply` makes **reality match your config** — if someone changed
something in the console, it changes it back.

`apply -refresh-only` makes **state match reality** and changes no infrastructure
at all. It updates Terraform's record of what exists to reflect what is actually
there.

Use it when the out-of-band change was correct and your config is the thing that
is stale. Accept reality into state, then update the config to match, then verify
with an empty plan.

It replaced the old `terraform refresh` command, which did the same thing without
showing you a diff first — which is exactly why it was deprecated.

### 6. Why version the state bucket?

Because state is the one file whose loss or corruption is genuinely hard to
recover from, and versioning turns that into a one-command fix.

Every write creates a new S3 version, so a botched `state rm`, a bad
`state push`, a failed migration, or a corrupted write is recoverable: list the
versions, download the last good one, push it back.

Without versioning, S3 overwrites in place and the previous state is gone. Your
only remaining copies are whatever `terraform.tfstate.backup` files happen to be
on someone's laptop.

It costs a few cents for a file measured in kilobytes. There is no argument
against it.

### 7. `-migrate-state` vs `-reconfigure`

`terraform init -migrate-state` **copies your existing state into the new
backend**. It prompts for confirmation, uploads the state, and your resources
stay managed. This is what you want when adding a backend to an existing project.

`terraform init -reconfigure` **discards the existing backend association and
starts fresh**, without migrating anything. Terraform initialises the new backend
empty.

`-reconfigure` is the one that ruins your afternoon. With an empty state,
Terraform believes none of your infrastructure exists — so the next plan proposes
to **create your entire production estate a second time**. Apply that and you get
duplicate resources, name collisions, and a genuinely difficult recovery.

`-reconfigure` is legitimate when you are deliberately pointing at a different
state, or fixing a corrupted backend configuration. It is not the flag for
migrating.

The safety net, as always: read the plan. `Plan: 47 to add` on a project where
nothing changed is the warning.

---

## Next lesson

**Module 4 — Variables, Outputs & Locals** (`04-variables-outputs-locals.md`)

Everything so far has been hardcoded. Module 4 parameterises it: typed input
variables with validation, the six ways to set a variable and which one wins,
outputs as a module's public API, and locals for computed values.

That is the last piece needed before modules become possible, and it is where
`dev` and `prod` stop being copy-pasted directories.

---

## Where we are now

Modules 1–3 are the irreducible core. With these you can:

- Explain what Terraform does and why, and place it against CloudFormation
- Read and write HCL, and read a plan properly
- Understand why a resource is being replaced
- Set up state the way a real team does, and recover when it goes wrong

Everything after this is leverage: making configuration reusable, safe, and
automated. But the model you need to reason about Terraform is complete.

> If a Terraform error confuses you, ask "what does state think is true?" first. It is the answer most of the time.
