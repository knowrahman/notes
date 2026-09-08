# Module 12 — Terraform in CI/CD

## Where Notely is right now

```text
  Complete system across three environments, seven modules,
  sixteen tests, and a check script that runs in under a minute.

  How it gets deployed:  you, on your laptop, typing terraform apply.
```

By the end of this module Notely deploys through a pipeline: plan on every pull
request, apply on merge, production behind a manual approval, and no long-lived
AWS keys anywhere.

Full picture: `notely-architecture.md`.

---

## Why this module exists

Applying from a laptop works fine until it does not. The specific problems:

| Problem | What it looks like |
|---|---|
| **No audit trail** | Who changed the production database size, and when? Nobody knows. |
| **No review** | Infrastructure changes skip the process every code change goes through. |
| **Credentials on laptops** | Every engineer who can deploy has production AWS keys on their machine. |
| **Nothing enforces the checks** | `./scripts/check.sh` runs when someone remembers. |
| **Drift between people** | Two engineers, two Terraform versions, two different plans. |
| **The wrong directory** | `cd envs/prod` at 5pm on a Friday. |
| **Concurrent applies** | Two people apply at once and fight over the state lock. |

You already know how to fix all of this for application code — you have written
GitLab CI pipelines (`../CI_CD/Gitlab-CICD.md`) and GitHub Actions workflows
deploying to ECS (`../CI_CD/CI_CD.md`).

This module is that, applied to infrastructure. Most of it will look familiar.
Three things are genuinely new.

---

## The core idea (one sentence)

Plan on the pull request so a human can read it, save that exact plan as an
artifact, and apply *that file* after approval — so what runs is exactly what was
reviewed.

> `terraform apply -auto-approve` in CI runs a plan nobody saw. The saved plan file is the whole point of the pattern.

---

## Mental model (from your existing pipelines)

Your ECS deploy pipeline looks roughly like this:

```text
build -> test -> push image -> deploy
```

The Terraform pipeline is the same shape:

```text
validate -> lint/scan -> plan -> [approval] -> apply
```

The mapping onto things you already use:

| Your app pipeline | The Terraform pipeline |
|---|---|
| `npm ci` | `terraform init` |
| `npm run lint` | `terraform fmt -check` + `tflint` |
| `npm test` | `terraform test` |
| `npm run build` → `dist/` artifact | `terraform plan -out=tfplan` → `tfplan` artifact |
| Deploy the artifact you built | `terraform apply tfplan` |
| `when: manual` before prod | `when: manual` before prod |
| Masked CI variables | OIDC, ideally no variables at all |

**The one genuinely new idea is the plan artifact.** In an app pipeline you build
an artifact and deploy that exact artifact rather than rebuilding at deploy time.
Terraform's saved plan is the same discipline: you review a plan, save it, and
apply that file — not a fresh plan computed later.

---

## Part 1 — Why the saved plan matters

Consider the naive pipeline:

```yaml
deploy:
  script:
    - terraform init
    - terraform apply -auto-approve
```

The problem: **the plan a reviewer approved is not the plan that runs.**

Between the review and the merge, any of these can change the outcome:

- Someone else applied a change
- Someone edited something in the AWS console
- An `aws_ami` data source resolved to a newer image
- A `most_recent` lookup returned something different
- Another pull request merged first

So a reviewer approves "adds one security group rule", and the pipeline applies
"adds one security group rule and replaces both EC2 instances".

### The fix

```yaml
plan:
  script:
    - terraform plan -out=tfplan
  artifacts:
    paths: [tfplan]

apply:
  script:
    - terraform apply tfplan     # apply THAT file
```

Now:

- The applied change is **exactly** what was planned
- No confirmation prompt is needed — approval already happened, in review
- If state has moved since the plan, the apply **fails** rather than doing
  something unexpected:

```text
Error: Saved plan is stale

The given plan file can no longer be applied because the state was changed
by another operation after the plan was created.
```

That error is the feature working.

### The security caveat

> A saved plan file contains variable values, including secrets. Make it a protected artifact with a short expiry. Never a public one.

Same for the plan JSON if you post it as a PR comment — check it does not contain
sensitive values before publishing it.

---

## Part 2 — The pipeline shape

```text
┌──────────────────────────────────────────────────────┐
│  On every pull request                               │
│                                                      │
│   fmt ──┐                                            │
│  lint ──┼──► plan (dev) ──► post plan as a comment   │
│  test ──┘                                            │
│  scan ──┘                                            │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│  On merge to main                                    │
│                                                      │
│  plan (dev) ──► apply (dev, automatic)               │
│       │                                              │
│       └──► plan (prod) ──► [MANUAL] ──► apply (prod) │
└──────────────────────────────────────────────────────┘
```

Key decisions in that diagram:

**Dev applies automatically.** Fast feedback, low risk.

**Prod requires a human.** Someone reads the plan and clicks a button.

**The checks run before the plan.** No point planning code that will not lint.

---

## Part 3 — GitLab CI

Built directly on `../CI_CD/Gitlab-CICD.md`. If you have read that, this is a
short step.

```yaml
# .gitlab-ci.yml

# The official image has an entrypoint of /bin/terraform, which breaks
# GitLab's script runner. Override it to a shell.
image:
  name: hashicorp/terraform:1.9
  entrypoint: [""]

variables:
  TF_IN_AUTOMATION: "true"        # tidier output, no interactive hints
  TF_INPUT: "0"                   # never prompt - fail instead
  TF_CLI_ARGS_init: "-input=false"
  AWS_REGION: ap-southeast-2

stages:
  - validate
  - plan
  - apply

# Reused by every job
.terraform_base:
  before_script:
    - cd "${TF_DIR}"
    - terraform init -input=false
  cache:
    key: "${TF_DIR}"
    paths:
      - ${TF_DIR}/.terraform

# ---------------------------------------------------------------
# Stage 1: validate - runs on everything, needs no credentials
# ---------------------------------------------------------------
fmt:
  stage: validate
  script:
    - terraform fmt -check -recursive -diff

validate:
  stage: validate
  script:
    - |
      for dir in modules/*/ envs/*/; do
        echo "validating $dir"
        terraform -chdir="$dir" init -backend=false -input=false
        terraform -chdir="$dir" validate
      done

tflint:
  stage: validate
  image:
    name: ghcr.io/terraform-linters/tflint:latest
    entrypoint: [""]
  script:
    - tflint --init
    - tflint --recursive

checkov:
  stage: validate
  image:
    name: bridgecrew/checkov:latest
    entrypoint: [""]
  script:
    - checkov -d . --framework terraform --compact --soft-fail
  allow_failure: true      # report, don't block, until the backlog is clear

test:
  stage: validate
  script:
    - |
      for dir in modules/*/; do
        if [ -d "${dir}tests" ]; then
          echo "testing $dir"
          terraform -chdir="$dir" init -backend=false -input=false
          terraform -chdir="$dir" test
        fi
      done

# ---------------------------------------------------------------
# Stage 2: plan
# ---------------------------------------------------------------
.plan_template:
  extends: .terraform_base
  stage: plan
  script:
    - terraform plan -input=false -out=tfplan
    - terraform show -no-color tfplan > plan.txt
    - terraform show -json tfplan > plan.json
  artifacts:
    name: "plan-${TF_DIR}-${CI_COMMIT_SHORT_SHA}"
    paths:
      - ${TF_DIR}/tfplan
      - ${TF_DIR}/plan.txt
    reports:
      terraform: ${TF_DIR}/plan.json    # GitLab renders this in the MR
    expire_in: 7 days
    access: developer                    # NOT public - contains variable values

plan:dev:
  extends: .plan_template
  variables:
    TF_DIR: envs/dev
  environment:
    name: dev
    action: prepare

plan:prod:
  extends: .plan_template
  variables:
    TF_DIR: envs/prod
  environment:
    name: prod
    action: prepare
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# ---------------------------------------------------------------
# Stage 3: apply
# ---------------------------------------------------------------
apply:dev:
  extends: .terraform_base
  stage: apply
  variables:
    TF_DIR: envs/dev
  script:
    - terraform apply -input=false tfplan
  dependencies:
    - plan:dev
  environment:
    name: dev
    url: $DEV_URL
  # Serialise applies against the same state - GitLab's version of a lock
  resource_group: terraform-dev
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

apply:prod:
  extends: .terraform_base
  stage: apply
  variables:
    TF_DIR: envs/prod
  script:
    - terraform apply -input=false tfplan
  dependencies:
    - plan:prod
  environment:
    name: prod
    url: $PROD_URL
  resource_group: terraform-prod
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual              # a human must click
      allow_failure: false      # and the pipeline waits for them
```

### The GitLab-specific bits worth knowing

**`entrypoint: [""]`** — the `hashicorp/terraform` image sets its entrypoint to
the terraform binary, so GitLab's shell script never runs. This trips everyone up
once.

**`resource_group`** — GitLab will not run two jobs in the same resource group
concurrently. This prevents two pipelines fighting over the same state lock. It
is the pipeline-level equivalent of Terraform's own locking, and it produces a
much clearer failure mode (a queued job rather than a lock error).

**`reports: terraform:`** — GitLab natively renders a Terraform plan JSON in the
merge request as a summary widget. Free plan visibility for reviewers.

**`access: developer`** — the plan artifact contains variable values. Do not make
it public.

**`when: manual` + `allow_failure: false`** — the pipeline pauses and waits. With
`allow_failure: true` it would go green without anyone clicking, which defeats
the point.

### GitLab-managed state

GitLab can host your Terraform state, using the `http` backend. Convenient if you
are already on GitLab:

```hcl
terraform {
  backend "http" {}
}
```

```bash
terraform init \
  -backend-config="address=${TF_ADDRESS}" \
  -backend-config="lock_address=${TF_ADDRESS}/lock" \
  -backend-config="unlock_address=${TF_ADDRESS}/lock" \
  -backend-config="username=gitlab-ci-token" \
  -backend-config="password=${CI_JOB_TOKEN}" \
  -backend-config="lock_method=POST" \
  -backend-config="unlock_method=DELETE" \
  -backend-config="retry_wait_min=5"
```

where `TF_ADDRESS` is
`${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}`.

Locking is handled, and the state is encrypted at rest. For Notely we stay on S3,
because that is what most teams use — but it is a genuine option.

---

## Part 4 — GitHub Actions

Built on `../CI_CD/CI_CD.md`. Same pattern, different syntax.

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  pull_request:
  push:
    branches: [main]

# Required for OIDC
permissions:
  id-token: write        # request the OIDC token
  contents: read
  pull-requests: write   # post the plan as a comment

env:
  TF_IN_AUTOMATION: "true"
  TF_INPUT: "0"
  AWS_REGION: ap-southeast-2

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9.5

      - name: Format
        run: terraform fmt -check -recursive -diff

      - name: Validate
        run: |
          for dir in modules/*/ envs/*/; do
            terraform -chdir="$dir" init -backend=false -input=false
            terraform -chdir="$dir" validate
          done

      - uses: terraform-linters/setup-tflint@v4
      - name: TFLint
        run: |
          tflint --init
          tflint --recursive

      - name: Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
          soft_fail: true

      - name: Test
        run: |
          for dir in modules/*/; do
            if [ -d "${dir}tests" ]; then
              terraform -chdir="$dir" init -backend=false -input=false
              terraform -chdir="$dir" test
            fi
          done

  plan:
    needs: validate
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, prod]
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9.5

      # No AWS keys anywhere. This exchanges a short-lived
      # GitHub OIDC token for temporary AWS credentials.
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/notely-terraform-plan
          aws-region: ap-southeast-2

      - name: Plan
        id: plan
        working-directory: envs/${{ matrix.environment }}
        run: |
          terraform init -input=false
          terraform plan -input=false -out=tfplan -no-color 2>&1 | tee plan.txt
          terraform show -no-color tfplan > plan-readable.txt

      - uses: actions/upload-artifact@v4
        with:
          name: tfplan-${{ matrix.environment }}
          path: |
            envs/${{ matrix.environment }}/tfplan
            envs/${{ matrix.environment }}/plan-readable.txt
          retention-days: 5

      - name: Comment the plan on the PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync(
              'envs/${{ matrix.environment }}/plan-readable.txt', 'utf8');
            const truncated = plan.length > 60000
              ? plan.slice(0, 60000) + '\n\n... truncated, see the artifact'
              : plan;

            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `### Terraform plan — \`${{ matrix.environment }}\`\n\n<details><summary>Show plan</summary>\n\n\`\`\`terraform\n${truncated}\n\`\`\`\n\n</details>`
            });

  apply-dev:
    needs: plan
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: dev
    concurrency:
      group: terraform-dev      # serialise applies
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9.5

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/notely-terraform-apply
          aws-region: ap-southeast-2

      - uses: actions/download-artifact@v4
        with:
          name: tfplan-dev
          path: envs/dev

      - name: Apply
        working-directory: envs/dev
        run: |
          terraform init -input=false
          terraform apply -input=false tfplan

  apply-prod:
    needs: apply-dev
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    # A GitHub Environment with required reviewers = the manual gate
    environment: production
    concurrency:
      group: terraform-prod
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9.5

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::333333333333:role/notely-terraform-apply
          aws-region: ap-southeast-2

      - uses: actions/download-artifact@v4
        with:
          name: tfplan-prod
          path: envs/prod

      - name: Apply
        working-directory: envs/prod
        run: |
          terraform init -input=false
          terraform apply -input=false tfplan
```

### The GitHub-specific bits

**`permissions: id-token: write`** — without this, OIDC silently fails. It is the
most common GitHub Actions OIDC mistake.

**`environment: production`** — a GitHub Environment with required reviewers
configured. The job pauses until someone approves. That is the manual gate.

**`concurrency: group:`** — GitHub's equivalent of GitLab's `resource_group`.

**`strategy: matrix`** — plans both environments in parallel.

---

## Part 5 — OIDC: no long-lived keys

Both pipelines above use OIDC. Here is the Terraform that sets it up.

```hcl
# The OIDC provider - one per account, not per repository.
#
# About that thumbprint: you will see this exact value copied across
# hundreds of blog posts. For GitHub, AWS now validates the provider
# against its own trusted certificate store and does not rely on the
# thumbprint you supply. That matters because GitHub has rotated its
# certificates before, and people whose pipelines "depended" on a
# hardcoded thumbprint panicked when it went stale.
#
# So: supply it if the provider asks for a non-empty list, and do not
# treat it as something you need to keep up to date. Check the current
# aws_iam_openid_connect_provider docs - recent provider versions have
# made thumbprint_list optional for exactly this reason.
resource "aws_iam_openid_connect_provider" "github" {
  url            = "https://token.actions.githubusercontent.com"
  client_id_list = ["sts.amazonaws.com"]

  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

data "aws_iam_policy_document" "github_assume" {
  statement {
    effect  = "Allow"
    actions = ["sts:AssumeRoleWithWebIdentity"]

    principals {
      type        = "Federated"
      identifiers = [aws_iam_openid_connect_provider.github.arn]
    }

    condition {
      test     = "StringEquals"
      variable = "token.actions.githubusercontent.com:aud"
      values   = ["sts.amazonaws.com"]
    }

    # CRITICAL. Without this, any GitHub repository in the world
    # can assume this role. Pin the repo AND the ref.
    condition {
      test     = "StringLike"
      variable = "token.actions.githubusercontent.com:sub"
      values = [
        "repo:knowrahman/notely-infra:ref:refs/heads/main",
        "repo:knowrahman/notely-infra:pull_request",
      ]
    }
  }
}

# A read-only role for planning
resource "aws_iam_role" "terraform_plan" {
  name               = "notely-terraform-plan"
  assume_role_policy = data.aws_iam_policy_document.github_assume.json
}

resource "aws_iam_role_policy_attachment" "plan_readonly" {
  role       = aws_iam_role.terraform_plan.name
  policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
}

# Plans still need to write the state lock
resource "aws_iam_role_policy" "plan_state" {
  name = "state-access"
  role = aws_iam_role.terraform_plan.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["s3:GetObject", "s3:PutObject"]
        Resource = "arn:aws:s3:::notely-tfstate-a1b2c3d4/*"
      },
      {
        Effect   = "Allow"
        Action   = ["s3:ListBucket"]
        Resource = "arn:aws:s3:::notely-tfstate-a1b2c3d4"
      },
    ]
  })
}
```

Two roles, deliberately:

| Role | Permissions | Used by |
|---|---|---|
| `notely-terraform-plan` | Read-only + state | Pull request plans |
| `notely-terraform-apply` | Write | Apply jobs on main only |

A pull request from a fork gets, at most, the ability to read and produce a plan.
It cannot change anything.

### Why OIDC over access keys

| | Access keys | OIDC |
|---|---|---|
| Lifetime | Forever, until rotated | About an hour |
| Where stored | CI variables, and wherever they were copied | Nowhere |
| If leaked | Valid until someone notices | Expired already |
| Rotation | Manual, and forgotten | Automatic |
| Scope | Whatever the user has | Per-role, per-repo, per-branch |

> If your pipeline has `AWS_SECRET_ACCESS_KEY` in its variables, that is a credential sitting in a system with a large number of people who can read it. OIDC removes it entirely.

---

## Part 6 — Drift detection on a schedule

A pipeline that only runs on merges misses changes made outside Terraform.

```yaml
# GitLab: a scheduled pipeline, daily
drift:
  extends: .terraform_base
  stage: plan
  variables:
    TF_DIR: envs/prod
  script:
    - terraform plan -input=false -detailed-exitcode -out=tfplan || EXIT=$?
    - |
      case "${EXIT:-0}" in
        0) echo "No drift." ;;
        1) echo "Terraform failed."; exit 1 ;;
        2) echo "DRIFT DETECTED"; terraform show -no-color tfplan; exit 1 ;;
      esac
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
```

`-detailed-exitcode` gives three outcomes instead of two:

| Exit code | Meaning |
|---|---|
| `0` | No changes |
| `1` | Terraform errored |
| `2` | **There are changes** |

A daily job that fails when prod does not match its configuration is one of the
highest-value pipelines you can add. It catches console edits, manual "temporary"
fixes, and anything another tool changed.

---

## Part 7 — Things that go wrong

| Symptom | Cause | Fix |
|---|---|---|
| Job hangs waiting for input | Terraform prompting | `TF_INPUT=0` and `-input=false` |
| `/bin/sh: terraform: not found` | The image's entrypoint | `entrypoint: [""]` |
| Escape codes all over the logs | Colour output | `-no-color`, or `TF_IN_AUTOMATION` |
| `Error acquiring the state lock` | Two pipelines at once | `resource_group` / `concurrency` |
| Lock stuck after a cancelled job | Lock never released | `terraform force-unlock <ID>` |
| `Saved plan is stale` | State changed since planning | Re-run the plan. **Working as intended.** |
| OIDC "not authorized to perform sts:AssumeRoleWithWebIdentity" | Missing `id-token: write`, or the `sub` condition does not match | Check both |
| Plan differs between local and CI | Different Terraform or provider versions | Pin both; commit the lock file |
| `init` slow on every job | No cache | Cache `.terraform/` |

### The version pinning point

```yaml
image:
  name: hashicorp/terraform:1.9    # pinned, not :latest
```

and commit `.terraform.lock.hcl`. Otherwise a provider release changes your plan
overnight and nobody knows why.

---

## Building it into Notely

Add a pipeline that runs the Module 11 checks and deploys dev automatically.

### The repository layout

```text
notely-infra/
├── .gitlab-ci.yml
├── .github/workflows/terraform.yml
├── .tflint.hcl
├── .checkov.yaml
├── .gitignore
├── scripts/check.sh
├── modules/
│   ├── network/  security/  storage/  iam/  database/  web/  notely/
└── envs/
    ├── dev/  staging/  prod/
```

### Minimum viable pipeline

If you take one thing from this module, take this:

```yaml
image:
  name: hashicorp/terraform:1.9
  entrypoint: [""]

variables:
  TF_IN_AUTOMATION: "true"
  TF_INPUT: "0"

stages: [validate, plan, apply]

check:
  stage: validate
  script:
    - terraform fmt -check -recursive
    - |
      for dir in modules/*/ envs/*/; do
        terraform -chdir="$dir" init -backend=false -input=false
        terraform -chdir="$dir" validate
      done
    - |
      for dir in modules/*/; do
        [ -d "${dir}tests" ] && terraform -chdir="$dir" test || true
      done

plan:
  stage: plan
  script:
    - cd envs/dev
    - terraform init -input=false
    - terraform plan -input=false -out=tfplan
    - terraform show -no-color tfplan
  artifacts:
    paths: [envs/dev/tfplan]
    expire_in: 7 days
    access: developer

apply:
  stage: apply
  script:
    - cd envs/dev
    - terraform init -input=false
    - terraform apply -input=false tfplan
  dependencies: [plan]
  resource_group: terraform-dev
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
```

Forty lines. It validates, tests, plans, saves the plan, and applies that exact
plan behind a manual gate. Everything else in this module is refinement.

### Adding the state backend in CI

The backend needs the bucket name, which differs per environment:

```yaml
before_script:
  - cd "${TF_DIR}"
  - |
    terraform init -input=false \
      -backend-config="bucket=${TF_STATE_BUCKET}" \
      -backend-config="key=notely/${ENVIRONMENT}/terraform.tfstate" \
      -backend-config="region=${AWS_REGION}"
```

with `TF_STATE_BUCKET` as a masked, protected CI variable. This is the partial
backend configuration from Module 9, doing the job it exists for.

---

## Real-World Example

A team of six moved Notely-shaped infrastructure from laptop applies to a
pipeline over three months.

**Before:**
- Two engineers had production AWS keys on their laptops
- Applies happened whenever someone had time
- No record of who changed what
- A production outage caused by someone applying a stale branch

**After:**
- Zero AWS keys anywhere; OIDC only
- Every change is a pull request with the plan rendered in the review
- Dev applies on merge; prod needs one approval from a second engineer
- A daily drift job on prod

**Numbers after six months:**

| | Before | After |
|---|---|---|
| Infrastructure changes per week | 3 | 11 |
| Failed applies | ~1 in 6 | ~1 in 40 |
| Time from merge to deployed (dev) | hours | 4 minutes |
| Production incidents from infra changes | 4 in 6 months | 0 |
| People with prod AWS keys | 2 | 0 |

The change rate **going up** is the interesting one. Making deployment safe made
people willing to make small changes rather than batching them into a risky
quarterly release.

**The pipeline detail that mattered most.** Rendering the plan in the merge
request. Before that, reviewers read HCL and tried to imagine the effect.
Afterwards they read `Plan: 2 to add, 1 to change, 0 to destroy` and the diff
itself.

The first time it earned its place, a reviewer saw `1 to destroy` on a line
nobody expected — a `name` change on an RDS instance that would have replaced the
production database. The HCL diff was three characters.

**The mistake they made first.** They started with
`terraform apply -auto-approve` on merge, without a saved plan. It worked for two
months. Then two pull requests merged within a minute of each other, the second
pipeline planned against state the first had already changed, and the applied
result was not what either reviewer had approved.

Nothing was lost, but they added the saved plan artifact and `resource_group`
that afternoon.

---

## Common Mistakes Beginners Make

**1. `terraform apply -auto-approve` in CI without a saved plan.**

The change that runs is not the change that was reviewed.

**2. Forgetting `entrypoint: [""]` on the Terraform image.**

```text
/bin/sh: eval: line 1: terraform: not found
```

**3. Missing `permissions: id-token: write` in GitHub Actions.**

OIDC fails with an unhelpful message.

**4. An OIDC trust policy without a `sub` condition.**

Any GitHub repository in the world can assume your role.

**5. Publishing the plan artifact publicly.**

It contains variable values.

**6. No `resource_group` or `concurrency`.**

Two pipelines fight over the state lock, and one fails confusingly.

**7. Using `:latest` for the Terraform image.**

Your pipeline changes behaviour without a commit.

**8. Not committing `.terraform.lock.hcl`.**

CI resolves different provider versions than you do locally.

**9. Forgetting `-input=false`.**

The job hangs for its full timeout waiting for a prompt nobody will answer.

**10. `when: manual` with `allow_failure: true`.**

The pipeline goes green without anyone approving. The gate does nothing.

**11. Applying prod automatically because dev worked.**

Production deserves a human.

**12. Treating the OIDC thumbprint as something to maintain.**

For GitHub, AWS validates against its own certificate store. Chasing a
"correct" thumbprint after a certificate rotation is wasted effort — see the
note in Part 5.

---

## Hands-On Lab — A Pipeline for Notely

**Cost: free** for parts A–D. Part E optionally applies dev.

### Part A: Prove the stale-plan protection works

This is the most important thing in the module, and you can demonstrate it
locally.

```bash
cd ~/terraform-labs/notely/envs/dev
terraform plan -out=tfplan
```

Now change something so state moves:

```bash
terraform apply -auto-approve -var log_retention_days=14
```

Then try to apply the old plan:

```bash
terraform apply tfplan
```

```text
Error: Saved plan is stale

The given plan file can no longer be applied because the state was changed
by another operation after the plan was created.
```

**That error is the entire value of the pattern.** Terraform refused to apply
something that had been reviewed against different facts.

### Part B: Look inside a plan file

```bash
terraform plan -out=tfplan
terraform show tfplan | head -30
terraform show -json tfplan | python3 -m json.tool | head -40
```

Now check what a careless artifact would expose:

```bash
terraform show -json tfplan | grep -i -o 'password[^,]*' | head -3
```

If anything sensitive appears, that is why the artifact must be protected.

```bash
rm tfplan
```

### Part C: Write the pipeline

Create `.gitlab-ci.yml` with the minimum viable pipeline shown above. Or
`.github/workflows/terraform.yml` if you use GitHub — write whichever you
actually use, and skim the other.

Validate the YAML:

```bash
python3 -c "import yaml,sys; yaml.safe_load(open('.gitlab-ci.yml')); print('valid YAML')"
```

### Part D: Run the pipeline's logic locally

Every step in the pipeline is a command you can run:

```bash
cd ~/terraform-labs/notely

# validate stage
terraform fmt -check -recursive
for dir in modules/*/ envs/*/; do
  terraform -chdir="$dir" init -backend=false -input=false > /dev/null
  terraform -chdir="$dir" validate
done
for dir in modules/*/; do
  if [ -d "${dir}tests" ]; then
    terraform -chdir="$dir" test
  fi
done

# plan stage
cd envs/dev
terraform init -input=false
terraform plan -input=false -out=tfplan
terraform show -no-color tfplan > plan.txt
wc -l plan.txt

# apply stage
terraform apply -input=false tfplan
```

If that sequence works locally, it works in CI. A pipeline is just these commands
on someone else's computer.

### Part E: Simulate the drift job

```bash
cd ~/terraform-labs/notely/envs/dev
terraform plan -detailed-exitcode
echo "exit code: $?"
```

```text
No changes. Your infrastructure matches the configuration.
exit code: 0
```

Now create drift by hand:

```bash
aws logs put-retention-policy \
  --log-group-name /notely-dev/app \
  --retention-in-days 30
```

```bash
terraform plan -detailed-exitcode
echo "exit code: $?"
```

```text
Plan: 0 to add, 1 to change, 0 to destroy.
exit code: 2
```

**Exit code 2 means drift.** A scheduled job checking for that is how you find
out somebody changed production in the console.

```bash
terraform apply
```

### Part F: Set up OIDC (optional, free)

If you have a GitHub repository for this:

```hcl
# In a scratch directory - IAM is free
resource "aws_iam_openid_connect_provider" "github" {
  url            = "https://token.actions.githubusercontent.com"
  client_id_list = ["sts.amazonaws.com"]

  # See the note in Part 5 - this value is effectively legacy for GitHub.
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}
```

plus the role and trust policy from Part 5. Then push a workflow that just runs
`aws sts get-caller-identity` and confirm it prints the assumed role's ARN with
no keys configured anywhere.

**Then deliberately break it:** change the `sub` condition to a different
repository name and re-run. The job fails with
`Not authorized to perform sts:AssumeRoleWithWebIdentity`. That is the condition
doing its job.

### Part G: Tear down

```bash
cd ~/terraform-labs/notely/envs/dev
terraform destroy
```

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

### What you should have at the end

- A stale-plan error triggered on purpose, and understood
- A pipeline file for your CI system
- Every pipeline step run locally first
- Drift created and detected via exit code 2
- Optionally, a working OIDC role with no keys anywhere

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **The pattern** | plan → save → review → apply *that file* |
| **`-out=tfplan`** | Save the plan |
| **`terraform apply tfplan`** | Apply exactly that. No prompt. |
| **"Saved plan is stale"** | State moved since planning. Working as intended. |
| **Plan artifacts** | Contain variable values. Protect them. |
| **`TF_IN_AUTOMATION`** | Tidier output for CI |
| **`TF_INPUT=0` / `-input=false`** | Never prompt. Fail instead. |
| **`-no-color`** | No escape codes in logs |
| **`entrypoint: [""]`** | Required for the hashicorp/terraform image in GitLab |
| **`resource_group`** | GitLab: serialise jobs against one state |
| **`concurrency: group:`** | GitHub: the same thing |
| **`reports: terraform:`** | GitLab renders the plan in the MR |
| **`when: manual` + `allow_failure: false`** | The production gate |
| **GitHub `environment:`** | Required reviewers = the gate |
| **`permissions: id-token: write`** | Required for OIDC. Easy to forget. |
| **OIDC `sub` condition** | Pin the repo AND the branch. Critical. |
| **Two roles** | Read-only for plan, write for apply |
| **`-detailed-exitcode`** | 0 no change, 1 error, **2 drift** |
| **Scheduled drift job** | Finds console edits before they surprise you |
| **Pin the image and the lock file** | Or your pipeline changes without a commit |

---

## Checkpoint (answer briefly)

1. Why is `terraform plan -out=tfplan` then `terraform apply tfplan` better than `terraform apply -auto-approve`?
2. Your pipeline fails with "Saved plan is stale". Is this a bug?
3. What does `resource_group` (GitLab) or `concurrency` (GitHub) prevent?
4. Why does the plan artifact need to be protected?
5. What does `-detailed-exitcode` return, and what would you build with it?
6. Why does the pipeline use two different IAM roles?
7. What breaks if you omit the `sub` condition from an OIDC trust policy?

---

## Checkpoint — model answers

### 1. Saved plan vs `-auto-approve`

Because **`-auto-approve` runs a fresh plan that nobody has seen.**

The sequence with `-auto-approve` is: a reviewer approves a pull request based on
a plan generated during review; the branch merges; the pipeline computes a
*brand new* plan and applies it immediately.

Between those two plans the world can change: another pull request merged,
someone edited a resource in the console, an `aws_ami` data source resolved to a
newer image, another engineer applied something. The plan that runs may propose
things nobody agreed to.

With a saved plan:

```bash
terraform plan -out=tfplan     # this exact set of changes
terraform apply tfplan          # apply exactly that
```

Two guarantees follow. The applied change is byte-for-byte what was reviewed. And
if state has moved since the plan was made, the apply **fails** rather than
silently doing something else.

It is the same discipline as building an artifact once and deploying that
artifact, rather than rebuilding at deploy time.

### 2. "Saved plan is stale"

**No. It is the safety mechanism working.**

```text
Error: Saved plan is stale

The given plan file can no longer be applied because the state was changed
by another operation after the plan was created.
```

Terraform records the state serial number in the plan file. On apply it checks
that the state is still at that serial. If something else has written state since
— another pipeline, a colleague, a manual apply — the plan's assumptions no
longer hold, and Terraform refuses.

The alternative would be applying a plan computed against facts that are no
longer true, which is exactly the failure the saved-plan pattern exists to
prevent.

**The fix is to re-run the plan job**, review the new output, and apply that. If
it happens constantly, that is a signal you need `resource_group` /
`concurrency` to serialise your pipelines.

Treat it as informative, not as an error to work around. Never reach for a way to
skip the check.

### 3. What `resource_group` / `concurrency` prevents

**Two pipelines applying against the same state file at the same time.**

Terraform's own state locking already prevents actual corruption — the second
apply gets "Error acquiring the state lock" and fails. So this is not a
correctness fix; it is a usability one, and it prevents a second problem.

What it gives you:

**A clean queue instead of a failure.** The second job waits rather than failing.
Nobody has to notice, re-run, or investigate a lock error.

**No stale plans.** Without serialisation, pipeline A plans, pipeline B plans, A
applies, then B applies against state A has changed — and B gets "Saved plan is
stale". Serialising means B plans after A has finished.

**No stuck locks.** A cancelled job can leave a lock behind that needs manual
`force-unlock`. Fewer concurrent jobs means fewer opportunities for that.

Set the group per state file, not per project:

```yaml
resource_group: terraform-dev    # separate from terraform-prod
```

Dev and prod have different state, so there is no reason for them to block each
other.

### 4. Protecting the plan artifact

**Because it contains variable values, including secrets.**

A saved plan is a serialised record of everything Terraform intends to do —
including the resolved values of every variable and every resource attribute. If
a database password is passed in as a variable, or generated by
`random_password`, it is in that file.

```bash
terraform show -json tfplan | grep -i password
```

`sensitive = true` does not help. It redacts terminal output; the plan file is
not terminal output.

So the artifact must be:

- **Non-public.** GitLab's `access: developer`, or a private GitHub artifact.
  A public artifact on a public repository is a credential leak.
- **Short-lived.** `expire_in: 7 days` / `retention-days: 5`. There is no reason
  to keep it after the apply.
- **Careful when rendered.** Posting the plan as a pull request comment is
  valuable, but check that the rendered output does not contain secrets — use
  `terraform show` (which respects sensitivity markers) rather than dumping the
  JSON.

### 5. `-detailed-exitcode`

It replaces the usual two exit codes with three:

| Code | Meaning |
|---|---|
| `0` | Success, **no changes** |
| `1` | Terraform errored |
| `2` | Success, **there are changes** |

Without it, a successful plan returns `0` whether or not it found changes, so a
script cannot tell the difference.

**What you build with it: a scheduled drift-detection job.**

```yaml
drift:
  script:
    - terraform plan -input=false -detailed-exitcode
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
```

Run daily against production. Exit code 2 means production no longer matches its
configuration — someone edited something in the console, another tool changed
something, or a change was applied and never committed.

This is one of the highest-value pipelines available, because drift is otherwise
invisible until it causes a confusing plan during an unrelated deploy, often
during an incident.

The other use is a pull request gate: fail if a plan produces changes when it
should not, which is a cheap way to enforce "the main branch is always applied".

### 6. Two IAM roles

**Least privilege, and specifically to make pull requests safe.**

| Role | Permissions | Used by |
|---|---|---|
| `notely-terraform-plan` | Read-only, plus state read/write | Plan jobs, including on PRs |
| `notely-terraform-apply` | Full write | Apply jobs, on main only |

The plan job runs on **every pull request**, including — depending on your
settings — pull requests from forks, opened by people outside your team. It runs
whatever code is in that branch, which includes any `.tf` file the author wrote.

If that job had write credentials, opening a pull request would be enough to
create, modify or destroy infrastructure. A malicious PR could add a resource, or
a `local-exec` provisioner, and it would run before any human reviewed it.

With a read-only role, the worst a hostile pull request can do is produce a plan.

(Plans do need a small amount of write access — to the state file, for the lock —
which is why the plan role gets scoped S3 permissions on the state bucket and
nothing else.)

The same reasoning extends further in larger setups: a separate role per
environment, with the production apply role assumable only from the main branch
after approval.

### 7. Omitting the OIDC `sub` condition

**Any GitHub Actions workflow in the world could assume your role.**

An OIDC trust policy typically has two conditions:

```hcl
condition {
  test     = "StringEquals"
  variable = "token.actions.githubusercontent.com:aud"
  values   = ["sts.amazonaws.com"]
}

condition {
  test     = "StringLike"
  variable = "token.actions.githubusercontent.com:sub"
  values   = ["repo:knowrahman/notely-infra:ref:refs/heads/main"]
}
```

The `aud` condition only proves the token came from GitHub Actions — which is
true of every repository on GitHub. It does not identify *which* repository.

The `sub` claim is what carries the repository and ref. Without a condition on
it, the trust policy says "any valid GitHub Actions token may assume this role".
An attacker creates a repository, adds a workflow that requests an OIDC token and
calls `sts:AssumeRoleWithWebIdentity` with your role ARN, and AWS hands over
credentials — because the token genuinely is a valid GitHub token.

This is a documented, exploited misconfiguration, not a theoretical one.

Two further points:

**Pin the ref, not just the repository.** `repo:owner/name:*` allows any branch,
including a pull request branch from a fork — so an outside contributor could
obtain credentials by opening a PR.

**Be careful with wildcards.** `repo:knowrahman/*` matches every repository under
your account, including ones you create later for unrelated purposes.

---

## Next lesson

**Module 13 — Importing & Drift** (`13-import-and-drift.md`)

Notely was built entirely in Terraform, which is the easy case. Real jobs are
rarely like that: you join a company with three years of console-created
infrastructure and are asked to bring it under management.

Module 13 covers that — `import` blocks, `-generate-config-out`, the realistic
workflow for adopting existing resources, and `moved` and `removed` blocks for
restructuring without destroying anything. Plus what to do about the drift your
new pipeline is now detecting daily.
