# Module 11 — Testing & Validation

## Where Notely is right now

```text
  notely/
  ├── modules/network, security, storage, iam, web, database, notely
  └── envs/dev, staging, prod

  Complete system: VPC, 6 subnets, ALB, 2 EC2 servers, RDS Postgres,
  S3, CloudWatch, Secrets Manager, IAM.

  About 900 lines of HCL across seven modules.
  Tests: none.
```

By the end of this module Notely has a validation pipeline that runs in seconds
and creates nothing.

Full picture: `notely-architecture.md`.

---

## Why this module exists

You have written 900 lines of code that creates real, billable, security-relevant
infrastructure. How do you know it is right?

Right now the answer is "run `terraform apply` and see". That is slow, it costs
money, and by the time you find out, the mistake exists in AWS.

The mistakes worth catching are specific:

| Mistake | What it costs |
|---|---|
| An S3 bucket without a public access block | A data breach |
| A security group with `0.0.0.0/0` on port 22 | A compromised server |
| An unencrypted RDS instance | A failed compliance audit |
| An instance type that does not exist | A failed apply, ten minutes wasted |
| A module used with two subnets when it needs three | A failed apply, at the worst moment |
| Inconsistent formatting | Diffs full of whitespace noise |

Every one of these can be caught in **under a minute, without creating anything**.

---

## The core idea (one sentence)

There is a ladder of checks from "instant and free" to "slow and expensive", and
you want to catch each mistake at the cheapest rung that can catch it.

> The best infrastructure test creates no infrastructure.

---

## Mental model (from Node.js)

You already have this ladder in JavaScript:

| JavaScript | Terraform | Speed |
|---|---|---|
| `prettier --check` | `terraform fmt -check` | instant |
| `tsc --noEmit` | `terraform validate` | ~1s |
| `eslint` | `tflint` | ~2s |
| `npm audit` / Snyk | `checkov`, `tfsec` | ~10s |
| Jest unit tests | `terraform test` with `command = plan` | ~10s |
| Integration tests | `terraform test` with `command = apply` | minutes, costs money |
| Manual QA | `terraform apply` and click around | slow |

And `terraform test` will feel familiar:

```javascript
// Jest
describe('bucket', () => {
  it('blocks public access', () => {
    expect(bucket.publicAccessBlock).toBe(true);
  });
});
```

```hcl
# terraform test
run "bucket_blocks_public_access" {
  command = plan

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.block_public_acls == true
    error_message = "The attachments bucket must block public ACLs."
  }
}
```

Same shape: a named case, an action, an assertion, a message when it fails.

---

## Part 1 — `terraform fmt`

The cheapest check there is.

```bash
terraform fmt              # fix this directory
terraform fmt -recursive   # fix everything below here
terraform fmt -check       # exit non-zero if anything is unformatted
terraform fmt -diff        # show what would change
```

For CI:

```bash
terraform fmt -check -recursive
```

Exit code 3 means "some files need formatting".

There is exactly one correct format and Terraform knows what it is. Arguing about
it is wasted time, and unformatted code produces diffs full of whitespace changes
that hide the real change.

> Run `terraform fmt` before every commit. Better: make it a git pre-commit hook.

---

## Part 2 — `terraform validate`

```bash
terraform init -backend=false    # no credentials needed
terraform validate
```

**What it catches:**

- Syntax errors
- References to resources or variables that do not exist
- Type mismatches
- Missing required arguments
- Wrong argument names for a resource
- Duplicate resource addresses

**What it does not catch:**

- Anything requiring the AWS API — an invalid AMI ID, a taken bucket name,
  a quota you have hit
- Whether your security groups are sensible
- Whether the infrastructure will actually work

It is `tsc --noEmit`: it proves the code is internally consistent, not that it is
correct.

The `-backend=false` flag matters for CI — it lets you validate without
credentials or state access.

---

## Part 3 — `tflint`

`terraform validate` knows the *shape* of the AWS provider's schema. It does not
know that `t3.mciro` is not a real instance type.

```bash
brew install tflint          # macOS
# or: curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash
```

Configure it in `.tflint.hcl`:

```hcl
plugin "terraform" {
  enabled = true
  preset  = "recommended"
}

plugin "aws" {
  enabled = true
  version = "0.32.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

rule "terraform_unused_declarations" {
  enabled = true
}
```

```bash
tflint --init      # download the plugins
tflint --recursive
```

What it finds that `validate` does not:

```text
Error: "t3.mciro" is an invalid value as instance_type (aws_instance_invalid_type)

  on modules/web/main.tf line 12:
  12:   instance_type = "t3.mciro"
```

```text
Warning: variable "old_setting" is declared but not used (terraform_unused_declarations)
```

```text
Warning: `vpc_cidr` variable has no description (terraform_documented_variables)
```

That first one would have cost you a failed apply and ten minutes.

---

## Part 4 — Security scanning with `checkov`

`tflint` catches mistakes. `checkov` catches **bad decisions**.

```bash
pip install checkov
checkov -d . --framework terraform
```

Run it against Notely and you will get findings like:

```text
Check: CKV_AWS_18: "Ensure the S3 bucket has access logging enabled"
        FAILED for resource: aws_s3_bucket.attachments

Check: CKV_AWS_16: "Ensure all data stored in RDS is securely encrypted at rest"
        PASSED for resource: aws_db_instance.this

Check: CKV_AWS_23: "Ensure every security group rule has a description"
        FAILED for resource: aws_security_group.alb
```

It ships with hundreds of rules built from AWS security best practice, CIS
benchmarks and real breach patterns.

### Handling findings

Three legitimate responses:

**1. Fix it.** Usually the right answer.

```hcl
resource "aws_s3_bucket_logging" "attachments" {
  bucket        = aws_s3_bucket.attachments.id
  target_bucket = aws_s3_bucket.logs.id
  target_prefix = "attachments/"
}
```

**2. Suppress it, with a reason.**

```hcl
# checkov:skip=CKV_AWS_18:Access logging is handled centrally by the
# organisation's CloudTrail data events, not per-bucket.
resource "aws_s3_bucket" "attachments" {
  bucket = "..."
}
```

The comment is the important part. A suppression without a reason is a suppression
nobody can evaluate later.

**3. Turn the rule off globally**, if it does not apply to your organisation:

```yaml
# .checkov.yaml
skip-check:
  - CKV_AWS_18   # logging handled centrally
```

### `tfsec` / Trivy

Same job, different rule set. Many teams run both, because they find different
things.

```bash
brew install trivy
trivy config .
```

> `validate` asks "is this valid?". `tflint` asks "is this a real AWS value?". `checkov` asks "is this a good idea?"

---

## Part 5 — `terraform test`

Terraform 1.6 added a native test framework. This is the one that feels like
Jest.

### The shape

Tests live in `tests/*.tftest.hcl` next to the module.

```hcl
# modules/storage/tests/defaults.tftest.hcl

variables {
  name_prefix        = "test"
  log_retention_days = 7
  enable_versioning  = true
}

run "bucket_blocks_public_access" {
  command = plan

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.block_public_acls == true
    error_message = "The attachments bucket must block public ACLs."
  }

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.block_public_policy == true
    error_message = "The attachments bucket must block public bucket policies."
  }
}
```

```bash
terraform test
```

```text
tests/defaults.tftest.hcl... in progress
  run "bucket_blocks_public_access"... pass
tests/defaults.tftest.hcl... tearing down
tests/defaults.tftest.hcl... pass

Success! 1 passed, 0 failed.
```

### The anatomy

| Block | Purpose | Jest equivalent |
|---|---|---|
| `variables` (top level) | Default inputs for every run | `beforeEach` setup |
| `run "name"` | One test case | `it(...)` |
| `command = plan` | Plan only. Fast, free. | A unit test |
| `command = apply` | Actually create. Slow, costs money. | An integration test |
| `variables` (inside `run`) | Override inputs for this case | Per-test setup |
| `assert` | A check | `expect(...)` |
| `expect_failures` | This input **should** be rejected | `expect(...).toThrow()` |

**`command = plan` is the default and what you want almost always.** It creates
nothing, costs nothing, and runs in seconds.

### Check the attribute shape before you assert on it

One practical warning, because it will cost you time otherwise.

Whether an attribute is a **single object** or a **list** depends on the
provider's schema, and it is not always obvious from the documentation. Blocks
declared with `MaxItems: 1` are usually exposed as a one-element list, so you
reach into them with `[0]` or iterate them — but this varies between resources
and between provider versions.

Get it wrong and you see:

```text
Error: Unsupported attribute
This value does not have any attributes.
```

or

```text
Error: Iteration over non-iterable value
```

Neither message tells you the right form.

**Check before you guess.** Apply the resource once in a scratch directory, then:

```bash
terraform state show aws_s3_bucket_versioning.attachments
```

or explore it interactively:

```bash
terraform console
```

```text
> aws_s3_bucket_versioning.attachments.versioning_configuration
> type(aws_s3_bucket_versioning.attachments.versioning_configuration)
```

`type()` tells you exactly what you are dealing with — `object`, `list`, `set` —
and therefore whether you need `[0]`, a `for` expression, or a direct attribute
reference.

The assertions in this module are written for the AWS provider 5.x schema. If
one errors on your version, this is why, and the two commands above are how you
find the correct form in about thirty seconds.

> Writing an assertion is easy. Knowing the shape of the thing you are asserting on is the actual skill.

---

### Testing that validation works

This is where `expect_failures` earns its place:

```hcl
run "rejects_invalid_environment" {
  command = plan

  variables {
    environment = "production"    # should be "prod"
  }

  expect_failures = [
    var.environment,
  ]
}

run "rejects_single_az" {
  command = plan

  variables {
    az_count = 1
  }

  expect_failures = [
    var.az_count,
  ]
}
```

The test passes when the plan **fails** with that variable's validation error.
You are testing your guard rails.

### Mocking — tests with no AWS account

`command = plan` still calls AWS to read data sources. `mock_provider` removes
even that:

```hcl
mock_provider "aws" {
  mock_data "aws_availability_zones" {
    defaults = {
      names = ["ap-southeast-2a", "ap-southeast-2b", "ap-southeast-2c"]
    }
  }

  mock_data "aws_caller_identity" {
    defaults = {
      account_id = "123456789012"
    }
  }

  mock_data "aws_region" {
    defaults = {
      name = "ap-southeast-2"
    }
  }
}

run "creates_six_subnets_for_two_azs" {
  command = plan

  variables {
    name_prefix = "test"
    vpc_cidr    = "10.0.0.0/16"
    az_count    = 2
  }

  assert {
    condition     = length(aws_subnet.this) == 6
    error_message = "Expected 6 subnets (3 tiers x 2 AZs), got ${length(aws_subnet.this)}."
  }
}
```

No credentials. No network calls. Runs in about a second. This is the equivalent
of mocking your database in a Jest test.

### Testing across several modules

A `run` block can point at a different module:

```hcl
run "setup_network" {
  command = apply
  module {
    source = "./modules/network"
  }
}

run "web_uses_the_network" {
  command = plan

  variables {
    public_subnet_ids = run.setup_network.public_subnet_ids
  }
}
```

`run.<name>.<output>` reads a previous run's outputs. That is how you build
integration tests — though remember `command = apply` creates real resources.

---

## Part 6 — Assertions inside the configuration

Tests live in test files. But some checks belong in the configuration itself, so
they run on **every** plan.

### `precondition`

Checked before the resource is created:

```hcl
resource "aws_lb" "main" {
  subnets = var.public_subnet_ids

  lifecycle {
    precondition {
      condition     = length(var.public_subnet_ids) >= 2
      error_message = "A load balancer needs at least two subnets in different availability zones."
    }
  }
}
```

### `postcondition`

Checked after, on the resource's own attributes:

```hcl
resource "aws_db_instance" "this" {
  # ...

  lifecycle {
    postcondition {
      condition     = self.storage_encrypted
      error_message = "The database must be encrypted at rest."
    }
  }
}
```

`self` refers to the resource itself.

### On outputs

```hcl
output "url" {
  value = "http://${aws_lb.main.dns_name}"

  precondition {
    condition     = length(aws_instance.app) >= 2
    error_message = "Notely must run at least two servers for high availability."
  }
}
```

### `check` blocks — warnings, not errors

Terraform 1.5 added `check`, which produces a **warning** rather than failing:

```hcl
check "database_backups_configured" {
  assert {
    condition     = aws_db_instance.this.backup_retention_period > 0
    error_message = "The database has no backup retention. This is fine in dev, but check it is intended."
  }
}
```

Use `check` for "this looks wrong but might be deliberate", and `precondition`
for "this is definitely wrong".

### Variable `validation` — the cheapest test of all

Already covered in Module 4, but worth restating here: a `validation` block is a
test that runs on every plan, costs nothing, and catches the mistake before a
single API call.

```hcl
variable "az_count" {
  type = number

  validation {
    condition     = var.az_count >= 2
    error_message = "This module builds highly available infrastructure and needs at least 2 availability zones."
  }
}
```

If you write one kind of test, write these.

---

## Part 7 — Terratest, briefly

Terratest is a Go library for infrastructure integration tests:

```go
func TestNetworkModule(t *testing.T) {
    opts := &terraform.Options{
        TerraformDir: "../modules/network",
        Vars: map[string]interface{}{
            "name_prefix": "test",
            "vpc_cidr":    "10.99.0.0/16",
        },
    }

    defer terraform.Destroy(t, opts)
    terraform.InitAndApply(t, opts)

    vpcID := terraform.Output(t, opts, "vpc_id")
    assert.NotEmpty(t, vpcID)
}
```

**When it is worth it:** you need to make real HTTP requests, check that a
database is actually reachable, or test failover behaviour. Things `plan` cannot
tell you.

**When it is not:** you are a solo learner, or `terraform test` covers your
needs. It is a Go project with its own dependencies, and every test creates real
infrastructure.

For Notely, `terraform test` with mocks is the right tool.

---

## Part 8 — Policy as code

The next step up: rules enforced across *every* configuration in an
organisation.

| Tool | Where it runs | Language |
|---|---|---|
| **Sentinel** | HCP Terraform / Enterprise | Sentinel |
| **OPA / Conftest** | Anywhere, on plan JSON | Rego |
| **checkov custom policies** | Anywhere | Python or YAML |

They all work the same way: run a plan, export it as JSON, evaluate rules against
it.

```bash
terraform plan -out=tfplan
terraform show -json tfplan > plan.json
conftest test plan.json
```

A rule in Rego:

```rego
package terraform.analysis

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_db_instance"
  resource.change.after.publicly_accessible == true
  msg := sprintf("Database %v must not be publicly accessible", [resource.address])
}
```

Worth knowing exists. Not needed at Notely's size.

### The plan JSON is the universal interface

Even without a policy tool, you can assert on plan output directly:

```bash
terraform plan -out=tfplan
terraform show -json tfplan | jq '
  .resource_changes[]
  | select(.type == "aws_security_group")
  | select(.change.after.ingress[]?.cidr_blocks[]? == "0.0.0.0/0")
  | select(.change.after.ingress[]?.from_port == 22)
  | .address
'
```

That prints any security group about to open SSH to the world. Ten lines of shell
in a pipeline, no extra tooling.

---

## Building it into Notely

Tests for three modules, plus a local check script.

### 1. `modules/storage` tests

`modules/storage/tests/defaults.tftest.hcl`:

```hcl
mock_provider "aws" {
  mock_data "aws_caller_identity" {
    defaults = {
      account_id = "123456789012"
    }
  }
}

variables {
  name_prefix        = "notely-test"
  log_retention_days = 7
  enable_versioning  = true
}

run "bucket_blocks_all_public_access" {
  command = plan

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.block_public_acls
    error_message = "Attachments bucket must block public ACLs."
  }

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.block_public_policy
    error_message = "Attachments bucket must block public bucket policies."
  }

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.ignore_public_acls
    error_message = "Attachments bucket must ignore public ACLs."
  }

  assert {
    condition     = aws_s3_bucket_public_access_block.attachments.restrict_public_buckets
    error_message = "Attachments bucket must restrict public buckets."
  }
}

run "bucket_is_encrypted" {
  command = plan

  assert {
    condition = alltrue([
      for r in aws_s3_bucket_server_side_encryption_configuration.attachments.rule :
      r.apply_server_side_encryption_by_default[0].sse_algorithm == "AES256"
    ])
    error_message = "Attachments bucket must be encrypted at rest."
  }
}

run "versioning_can_be_disabled" {
  command = plan

  variables {
    enable_versioning = false
  }

  assert {
    condition = alltrue([
      for c in aws_s3_bucket_versioning.attachments.versioning_configuration :
      c.status == "Suspended"
    ])
    error_message = "Setting enable_versioning = false should suspend versioning."
  }
}

run "log_group_uses_requested_retention" {
  command = plan

  variables {
    log_retention_days = 30
  }

  assert {
    condition     = aws_cloudwatch_log_group.app.retention_in_days == 30
    error_message = "Log group retention should match the log_retention_days variable."
  }
}
```

### 2. `modules/network` tests

`modules/network/tests/subnets.tftest.hcl`:

```hcl
mock_provider "aws" {
  mock_data "aws_availability_zones" {
    defaults = {
      names = ["ap-southeast-2a", "ap-southeast-2b", "ap-southeast-2c"]
    }
  }

  mock_data "aws_region" {
    defaults = {
      name = "ap-southeast-2"
    }
  }
}

variables {
  name_prefix = "notely-test"
  vpc_cidr    = "10.0.0.0/16"
  az_count    = 2
}

run "creates_three_tiers_per_az" {
  command = plan

  assert {
    condition     = length(aws_subnet.this) == 6
    error_message = "Expected 6 subnets for 2 AZs (public, app, data in each), got ${length(aws_subnet.this)}."
  }
}

run "three_azs_gives_nine_subnets" {
  command = plan

  variables {
    az_count = 3
  }

  assert {
    condition     = length(aws_subnet.this) == 9
    error_message = "Expected 9 subnets for 3 AZs."
  }
}

run "public_subnets_are_public_private_are_not" {
  command = plan

  assert {
    condition     = aws_subnet.this["public-a"].map_public_ip_on_launch == true
    error_message = "Public subnets must assign public IPs on launch."
  }

  assert {
    condition     = aws_subnet.this["app-a"].map_public_ip_on_launch == false
    error_message = "App subnets must NOT assign public IPs - they are private."
  }

  assert {
    condition     = aws_subnet.this["data-a"].map_public_ip_on_launch == false
    error_message = "Data subnets must NOT assign public IPs - they are private."
  }
}

run "private_route_table_has_no_internet_route" {
  command = plan

  assert {
    condition     = length(aws_route_table.private.route) == 0
    error_message = "The private route table must not have an internet route. That is what makes those subnets private."
  }
}

run "subnet_cidrs_come_from_the_vpc_cidr" {
  command = plan

  variables {
    vpc_cidr = "10.7.0.0/16"
  }

  assert {
    condition     = aws_subnet.this["public-a"].cidr_block == "10.7.1.0/24"
    error_message = "Subnet CIDRs must be derived from vpc_cidr, not hardcoded."
  }
}

run "rejects_a_single_availability_zone" {
  command = plan

  variables {
    az_count = 1
  }

  expect_failures = [
    var.az_count,
  ]
}

run "rejects_a_cidr_that_is_too_small" {
  command = plan

  variables {
    vpc_cidr = "10.0.0.0/24"
  }

  expect_failures = [
    var.vpc_cidr,
  ]
}
```

Those last two are the most valuable tests in the file. They prove the module
**refuses** to build something unsafe.

### 3. `modules/security` tests

`modules/security/tests/rules.tftest.hcl`:

```hcl
mock_provider "aws" {}

variables {
  name_prefix = "notely-test"
  vpc_id      = "vpc-12345678"
  app_port    = 3000
}

run "app_is_reachable_only_from_the_load_balancer" {
  command = plan

  assert {
    condition = alltrue([
      for r in aws_security_group.app.ingress :
      length(r.cidr_blocks) == 0
    ])
    error_message = "The app security group must not allow any CIDR range directly - only the ALB's security group."
  }
}

run "database_is_reachable_only_from_the_app" {
  command = plan

  assert {
    condition = alltrue([
      for r in aws_security_group.db.ingress :
      length(r.cidr_blocks) == 0
    ])
    error_message = "The database security group must not allow any CIDR range directly - only the app's security group."
  }
}

run "no_ssh_is_open_anywhere" {
  command = plan

  assert {
    condition = alltrue(flatten([
      for sg in [aws_security_group.alb, aws_security_group.app, aws_security_group.db] : [
        for r in sg.ingress : r.from_port != 22
      ]
    ]))
    error_message = "No security group may open port 22. Use Session Manager instead."
  }
}
```

That last test is a **policy**, expressed as a test. Nobody can add an SSH rule
without the suite going red.

### 4. A local check script

`scripts/check.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "==> terraform fmt"
terraform fmt -check -recursive -diff

echo "==> terraform validate"
for dir in modules/*/ envs/*/; do
  echo "    $dir"
  terraform -chdir="$dir" init -backend=false -input=false > /dev/null
  terraform -chdir="$dir" validate
done

echo "==> tflint"
tflint --init > /dev/null
tflint --recursive

echo "==> checkov"
checkov -d . --framework terraform --compact --quiet

echo "==> terraform test"
for dir in modules/*/; do
  if [ -d "${dir}tests" ]; then
    echo "    $dir"
    terraform -chdir="$dir" test
  fi
done

echo ""
echo "All checks passed."
```

```bash
chmod +x scripts/check.sh
./scripts/check.sh
```

Under a minute, no AWS resources created, no money spent. Module 12 puts exactly
this in a pipeline.

---

## Real-World Example

A team of six adopted this ladder over about a year, adding a rung each time
something went wrong.

```text
Month 1:  fmt + validate in CI
          Trigger: a merge conflict caused by whitespace-only diffs

Month 3:  tflint
          Trigger: a typo'd instance type failed an apply during a release

Month 5:  checkov
          Trigger: a security review found three buckets without
                   public access blocks

Month 8:  terraform test on shared modules
          Trigger: a "small" change to the VPC module broke four
                   consuming services at once

Month 11: conftest policy on the plan JSON
          Trigger: someone added an RDS instance with
                   publicly_accessible = true; it was caught in review,
                   but only by luck
```

Their pipeline now runs in about 90 seconds and blocks a merge if anything
fails.

**The number that convinced their manager.** Before, roughly one in six applies
failed for a preventable reason — a typo, a missing argument, an invalid value.
Each failure cost 10–20 minutes, and the ones that failed *partway through*
sometimes left resources half-created.

After, it was about one in fifty, and those were genuine AWS problems like
capacity errors.

**The test that has paid for itself most.** The one asserting no security group
opens port 22 — the same test as Notely's `no_ssh_is_open_anywhere`. It has gone
red four times, always from someone debugging who intended to remove the rule
before merging and forgot.

> The tests that catch real problems are usually the boring ones asserting a policy you already agreed on.

---

## Common Mistakes Beginners Make

**1. Thinking `terraform validate` checks correctness.**

It checks internal consistency. An invalid AMI passes validate and fails apply.

**2. Using `command = apply` in tests by default.**

It creates real resources and costs real money. `command = plan` is the default
for a reason.

**3. Not mocking providers.**

Without `mock_provider`, tests need credentials, which means they cannot run on a
contributor's fork.

**4. Suppressing a checkov finding without a reason.**

```hcl
# checkov:skip=CKV_AWS_18
```

Six months later nobody knows whether that was a decision or laziness. Write the
reason.

**5. Only testing the happy path.**

`expect_failures` tests are often the most valuable — they prove your guard rails
work.

**6. Skipping variable `validation` because tests exist.**

Validation runs on every plan for every user. Tests run when someone remembers.

**7. Writing tests that assert Terraform works.**

```hcl
assert {
  condition     = aws_vpc.main.cidr_block == var.vpc_cidr
  error_message = "..."
}
```

That tests Terraform, not your module. Assert on *derived* values — the computed
subnet CIDRs, the count of resources, whether a flag propagated.

**8. Running `checkov` and ignoring everything.**

A scanner nobody acts on is worse than no scanner: it produces the appearance of
security review.

**9. Leaving `terraform test` out of CI.**

A test suite that only runs locally does not run.

**10. Guessing whether an attribute is a list or an object.**

`Error: Unsupported attribute` and `Error: Iteration over non-iterable value`
both mean you guessed wrong. Use `terraform console` and `type()` to check,
rather than trying `[0]` and seeing what happens.

---

## Hands-On Lab — A Validation Pipeline for Notely

**Cost: free.** Every check in this lab uses `plan` or mocks. Nothing is created.

### Part A: Formatting

```bash
cd ~/terraform-labs/notely
terraform fmt -check -recursive
```

If it reports files, look at what would change:

```bash
terraform fmt -diff -recursive
terraform fmt -recursive
```

Now break it deliberately:

```hcl
# modules/network/main.tf
resource "aws_vpc" "main" {
cidr_block=var.vpc_cidr
    enable_dns_support=true
}
```

```bash
terraform fmt -check -recursive
echo "exit code: $?"
```

```text
modules/network/main.tf
exit code: 3
```

That non-zero exit is what fails a CI job.

```bash
terraform fmt -recursive
```

### Part B: Validation

```bash
cd modules/network
terraform init -backend=false
terraform validate
```

```text
Success! The configuration is valid.
```

Break it:

```hcl
resource "aws_subnet" "this" {
  vpc_id = aws_vpc.nonexistent.id
}
```

```bash
terraform validate
```

```text
Error: Reference to undeclared resource
```

Now break it in a way validate **cannot** catch:

```hcl
variable "instance_class" {
  default = "db.t3.micrp"     # typo
}
```

```bash
terraform validate
```

```text
Success! The configuration is valid.
```

It passes. That is the gap `tflint` fills. Fix both.

### Part C: tflint

```bash
brew install tflint    # or the Linux install script
cd ~/terraform-labs/notely
```

Create `.tflint.hcl` as shown above.

```bash
tflint --init
tflint --recursive
```

Introduce a real typo:

```hcl
# modules/web/main.tf
instance_type = "t3.mciro"
```

```bash
tflint --recursive
```

```text
Error: "t3.mciro" is an invalid value as instance_type (aws_instance_invalid_type)

  on modules/web/main.tf line 12:
  12:   instance_type = "t3.mciro"
```

Caught in two seconds, with no AWS call at all. Fix it.

### Part D: checkov

```bash
pip install checkov
checkov -d . --framework terraform --compact
```

You will get a list of passes and failures. Read the failures — several will be
legitimate improvements to Notely.

Pick one and fix it. Access logging on the bucket is a good candidate:

```hcl
# modules/storage/main.tf
resource "aws_s3_bucket" "logs" {
  bucket = "${var.name_prefix}-logs-${data.aws_caller_identity.current.account_id}"
}

resource "aws_s3_bucket_public_access_block" "logs" {
  bucket = aws_s3_bucket.logs.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_logging" "attachments" {
  bucket        = aws_s3_bucket.attachments.id
  target_bucket = aws_s3_bucket.logs.id
  target_prefix = "attachments/"
}
```

Pick another and suppress it properly:

```hcl
# checkov:skip=CKV_AWS_144:Cross-region replication is not required for a
# learning project. Revisit before production.
```

```bash
checkov -d . --framework terraform --compact
```

Fewer failures, and every remaining one is either a real to-do or an explained
decision.

### Part E: Write the tests

Create the three test files shown above:

- `modules/storage/tests/defaults.tftest.hcl`
- `modules/network/tests/subnets.tftest.hcl`
- `modules/security/tests/rules.tftest.hcl`

```bash
cd modules/network
terraform init -backend=false
terraform test
```

```text
tests/subnets.tftest.hcl... in progress
  run "creates_three_tiers_per_az"... pass
  run "three_azs_gives_nine_subnets"... pass
  run "public_subnets_are_public_private_are_not"... pass
  run "private_route_table_has_no_internet_route"... pass
  run "subnet_cidrs_come_from_the_vpc_cidr"... pass
  run "rejects_a_single_availability_zone"... pass
  run "rejects_a_cidr_that_is_too_small"... pass
tests/subnets.tftest.hcl... tearing down
tests/subnets.tftest.hcl... pass

Success! 7 passed, 0 failed.
```

**No AWS credentials were used.** `mock_provider` handled the data sources.

### Part F: Watch a test catch a real bug

Break the module in a way that is easy to do by accident:

```hcl
# modules/network/main.tf
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  # Someone adds this while debugging "why can't my server reach the internet"
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}
```

```bash
terraform test
```

```text
  run "private_route_table_has_no_internet_route"... fail

Error: Test assertion failed

The private route table must not have an internet route. That is what makes
those subnets private.
```

**That change would have made every private subnet public**, exposing the app
servers and the database to the internet. A `plan` would have shown one added
route and looked harmless in review.

Remove it.

### Part G: The security test

```hcl
# modules/security/main.tf - someone needs to debug a server
ingress {
  description = "SSH - temporary, remove before merging"
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```

```bash
cd ../security
terraform init -backend=false
terraform test
```

```text
  run "no_ssh_is_open_anywhere"... fail

No security group may open port 22. Use Session Manager instead.
```

The comment said "remove before merging". The test makes sure of it.

Remove it.

### Part H: The check script

Create `scripts/check.sh` as shown, then:

```bash
chmod +x scripts/check.sh
./scripts/check.sh
```

```text
==> terraform fmt
==> terraform validate
    modules/database/
    modules/iam/
    modules/network/
    ...
==> tflint
==> checkov
==> terraform test
    modules/network/
    modules/security/
    modules/storage/

All checks passed.
```

Time it:

```bash
time ./scripts/check.sh
```

Under a minute, and it created nothing.

### What you should have at the end

- Formatting enforced, with the exit code that fails CI
- A typo caught by tflint that validate missed
- checkov findings triaged — some fixed, some explained
- Sixteen tests running with no credentials
- Two of them catching genuine security regressions you introduced yourself
- One script that runs the lot

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **`terraform fmt -check`** | Formatting gate. Exit 3 if unformatted. |
| **`terraform validate`** | Syntax and references. No API calls. |
| **`-backend=false`** | Validate without credentials or state |
| **What validate misses** | Anything needing the AWS API |
| **`tflint`** | Provider-aware linting. Catches invalid values. |
| **`.tflint.hcl`** | Its config. Enable the AWS ruleset. |
| **`checkov`** | Security scanning against best practice |
| **`# checkov:skip=ID:reason`** | Suppress — always with a reason |
| **`tfsec` / Trivy** | Another scanner, different rules |
| **`terraform test`** | Native test framework, Terraform 1.6+ |
| **`.tftest.hcl`** | Where tests live |
| **`run "name"`** | One test case. Like `it(...)`. |
| **`command = plan`** | Default. Fast, free, creates nothing. |
| **`command = apply`** | Real resources, real money |
| **`assert`** | A check, with an error message |
| **`expect_failures`** | Assert that bad input is rejected |
| **`mock_provider`** | Run tests with no AWS account at all |
| **`run.<name>.<output>`** | Read a previous run's outputs |
| **`precondition`** | Assertion before creating. Fails the plan. |
| **`postcondition`** | Assertion on the created resource. `self` refers to it. |
| **`check` block** | Produces a warning, not an error |
| **Variable `validation`** | The cheapest test. Runs on every plan. |
| **Terratest** | Go integration tests. Heavyweight. |
| **Plan JSON** | `terraform show -json` — the universal policy interface |
| **The ladder** | fmt → validate → lint → scan → unit → integration |

---

## Checkpoint (answer briefly)

1. `terraform validate` passes but `terraform apply` fails on an invalid instance type. Why, and what would have caught it?
2. What is the difference between `command = plan` and `command = apply` in a test, and which should be your default?
3. What does `mock_provider` give you that `command = plan` alone does not?
4. What is `expect_failures` for, and why are those often the most valuable tests?
5. When should you use a `precondition` instead of a test file?
6. Notely's test asserts `length(aws_security_group.app.ingress[*].cidr_blocks) == 0`. What real mistake does that prevent?
7. Why is suppressing a checkov finding without a comment worse than not running checkov?

---

## Checkpoint — model answers

### 1. Why validate misses an invalid instance type

**`terraform validate` is entirely offline.** It never contacts AWS.

It checks that your configuration is internally consistent: the syntax parses,
every reference points at something that exists, types line up, required
arguments are present, and argument names match the provider's schema.

`instance_type` is a string. `"t3.mciro"` is a valid string. As far as the schema
is concerned, nothing is wrong. Only AWS knows which instance types actually
exist, and validate never asks it.

**`tflint` would have caught it.** It ships an AWS ruleset containing the real
list of valid instance types, AMI naming rules, valid regions and so on, and
checks against them locally:

```text
Error: "t3.mciro" is an invalid value as instance_type (aws_instance_invalid_type)
```

Two seconds, no API call, no failed apply.

The mental model: `validate` is `tsc --noEmit` — it proves the code type-checks.
`tflint` is `eslint` with a rules package that knows your domain.

### 2. `plan` vs `apply` in tests

**`command = plan`** runs a plan and asserts on the values Terraform *would*
create. Nothing is created. It takes seconds and costs nothing. It is the
default.

**`command = apply`** actually creates the infrastructure, asserts against the
real thing, then destroys it during teardown. It takes minutes, costs money, and
needs credentials.

**`plan` should be your default**, and for most assertions it is entirely
sufficient. Nearly everything worth checking about a module — how many subnets it
produces, whether encryption is on, whether a flag propagates, whether validation
rejects bad input — is visible in the plan.

Reach for `apply` only when the assertion genuinely requires the resource to
exist: making an HTTP request to a load balancer, checking a database accepts
connections, testing failover.

Even then, be careful: if a test fails partway through, teardown may not complete,
and you can be left with orphaned billable resources.

### 3. What `mock_provider` adds

**It removes the need for AWS credentials entirely.**

`command = plan` does not create resources, but it still talks to AWS — every
`data` source is read during planning. Notely's network module reads
`aws_availability_zones`; the storage module reads `aws_caller_identity`; the web
module reads `aws_ami`. All of those are real API calls needing real credentials.

`mock_provider` intercepts them and returns values you supply:

```hcl
mock_provider "aws" {
  mock_data "aws_availability_zones" {
    defaults = {
      names = ["ap-southeast-2a", "ap-southeast-2b", "ap-southeast-2c"]
    }
  }
}
```

Three practical benefits:

**Tests run anywhere** — on a contributor's fork, in a pipeline with no cloud
access, on a plane.

**Tests are deterministic.** A real `aws_ami` lookup returns a different ID
whenever Amazon publishes a new image, so an assertion on it would fail randomly.
The mock always returns the same thing.

**Tests are fast.** No network round trips at all — Notely's network suite runs in
about a second.

It is exactly mocking your database in a Jest test, and for the same reasons.

### 4. `expect_failures`

It asserts that a plan **fails**, and that it fails for the specific reason you
expect.

```hcl
run "rejects_a_single_availability_zone" {
  command = plan

  variables {
    az_count = 1
  }

  expect_failures = [
    var.az_count,
  ]
}
```

This test passes when `var.az_count`'s validation block rejects the value. If
someone removes that validation, the plan succeeds, and the test **fails** —
telling you a guard rail has disappeared.

These are often the most valuable tests for two reasons.

**They test the thing that protects you.** A happy-path test proves the module
works when used correctly. An `expect_failures` test proves it *refuses* to work
when used dangerously — and the dangerous path is the one that causes incidents.

**Guard rails are easy to delete accidentally.** A validation block looks like
noise to someone in a hurry, and removing one makes an error message go away,
which feels like progress. Without a test, nothing notices until production has
one availability zone.

The Jest equivalent is `expect(() => f(bad)).toThrow()` — and the reasoning for
writing it is identical.

### 5. `precondition` vs a test file

The distinction is **when it runs and who it protects**.

**A `precondition` runs on every plan, for every user, forever.** It is part of
the configuration. Nobody can apply the module without satisfying it — not you,
not a colleague, not a pipeline. It fails the plan with your message.

```hcl
lifecycle {
  precondition {
    condition     = length(var.public_subnet_ids) >= 2
    error_message = "A load balancer needs at least two subnets in different availability zones."
  }
}
```

**A test runs when someone runs the test suite.** In CI that is on every pull
request, which is good — but it does not protect against someone applying from
their laptop, and it does not protect a consumer of your module who has never
seen your tests.

So the rule:

- **`precondition`** for invariants that must hold at apply time, where the
  consequence of violating them is broken or unsafe infrastructure. "This ALB
  needs two subnets." "This database must be encrypted."
- **Test file** for verifying behaviour and catching regressions during
  development. "Two AZs produces six subnets." "Setting `enable_versioning =
  false` suspends versioning."

They overlap deliberately. Notely has both, and the tests also verify that the
preconditions and validations themselves still work — which is what
`expect_failures` is doing.

### 6. What the security group test prevents

It prevents **anyone widening the app tier's exposure from "the load balancer
only" to "an IP range".**

Notely's design is a chain where each layer accepts traffic only from the layer
above, by security group reference:

```text
internet -> alb_sg -> app_sg -> db_sg
```

The app's ingress rule uses `security_groups = [alb_sg.id]` and has **no**
`cidr_blocks` at all. That is what the test asserts.

The realistic failure it catches: someone is debugging and cannot reach the app
directly, so they add

```hcl
ingress {
  from_port   = 3000
  to_port     = 3000
  cidr_blocks = ["10.0.0.0/16"]
}
```

That now lets *anything* in the VPC reach the app servers — including a future
resource placed there for an unrelated reason, or anything an attacker gets a
foothold on. The specific "only through the load balancer" guarantee is gone.

In a code review that looks like a small, reasonable addition. In a plan it is
one added ingress rule. It is genuinely easy to miss.

The test makes it impossible to miss, because the suite goes red with a message
explaining the intent.

### 7. Suppressing without a comment

Because it **converts a known risk into an invisible one**, while leaving the
appearance of a clean scan.

```hcl
# checkov:skip=CKV_AWS_18
```

Six months later, someone reads that line. They cannot tell whether:

- The rule was genuinely inapplicable and a thoughtful decision was made
- The finding was handled some other way, elsewhere
- Someone was in a hurry before a release and silenced it

They have no way to evaluate it, so the safe move is to leave it — and it stays
forever, unexamined.

Compare that with no scanner at all. Then the risk is at least *unknown*, and the
next security review will find it fresh.

With an unexplained suppression, the scan comes back green and everyone believes
the check happened. That is worse: it is a false assurance, and false assurance
is how things get missed.

The fix costs one sentence:

```hcl
# checkov:skip=CKV_AWS_18:Access logging is handled centrally by CloudTrail
# data events rather than per-bucket. Reviewed 2026-09 by the platform team.
```

Now a future reader can agree, disagree, or check whether it is still true. That
is what makes it a decision rather than a silence.

---

## Next lesson

**Module 12 — Terraform in CI/CD** (`12-terraform-in-cicd.md`)

You have a check script that runs in under a minute. Right now you have to
remember to run it, and you still apply Notely from your laptop.

Module 12 fixes both: a pipeline that plans on every pull request, posts the plan
where reviewers can read it, and applies only after a human approves. You already
know GitLab CI and GitHub Actions, so it is presented as a diff against pipelines
you have already written — the new parts are the saved plan artifact, state
locking under concurrency, and OIDC instead of long-lived keys.
