# Module 14 — The Complete Build

## Where Notely is right now

```text
  Everything except the finishing touches:

    VPC, 6 subnets, 2 AZs, routing, S3 endpoint     Modules 2, 6, 8
    3 chained security groups                       Modules 5, 8
    2 EC2 servers running Node.js                   Module 6
    Application Load Balancer                       Module 6
    RDS Postgres with Secrets Manager               Module 10
    S3 attachments, CloudWatch logs                 Modules 1, 8
    3 environments, independent state               Module 9
    16 tests, tflint, checkov                       Module 11
    CI/CD pipeline with OIDC                        Module 12

  Missing: Route 53, alarms, SNS notifications.
```

This module adds those three, then steps back and covers everything that only
matters once a system is real.

Full picture: `notely-architecture.md`.

---

## Why this module exists

You can now build infrastructure. This module is about **running** it.

The difference is everything that happens after the first apply: how the
repository is laid out for a team, what a reviewer looks for, what happens when
state is corrupted at 3am, how you upgrade a provider without breaking
production, and how you avoid the bill nobody expected.

None of it is Terraform syntax. All of it is why some teams ship infrastructure
changes daily and others are frightened of their own repository.

---

## The core idea (one sentence)

Production infrastructure is judged by how safely it can be changed, not by how
cleverly it was written.

> Optimise for the person who has to fix this at 3am, and who might be you.

---

## Part 1 — Completing Notely

Three additions.

### DNS with Route 53

**Cost: $0.50/month per hosted zone.** Skip this part if you do not have a domain
— everything else works with the load balancer's own DNS name.

```hcl
# modules/dns/variables.tf
variable "domain_name" {
  description = "The domain, e.g. notely.example.com"
  type        = string
}

variable "hosted_zone_id" {
  description = "Route 53 zone that already exists for the parent domain"
  type        = string
}

variable "alb_dns_name" {
  type = string
}

variable "alb_zone_id" {
  type = string
}
```

```hcl
# modules/dns/main.tf

# An alias record points at the ALB. Unlike a CNAME it works at the
# zone apex, it is free to query, and it follows the ALB if its
# address changes.
resource "aws_route53_record" "app" {
  zone_id = var.hosted_zone_id
  name    = var.domain_name
  type    = "A"

  alias {
    name                   = var.alb_dns_name
    zone_id                = var.alb_zone_id
    evaluate_target_health = true
  }
}

# A certificate so we can serve HTTPS.
resource "aws_acm_certificate" "app" {
  domain_name       = var.domain_name
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}

# ACM asks you to prove you own the domain by creating a DNS record.
# Terraform can do that automatically.
resource "aws_route53_record" "cert_validation" {
  for_each = {
    for dvo in aws_acm_certificate.app.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  zone_id         = var.hosted_zone_id
  name            = each.value.name
  type            = each.value.type
  records         = [each.value.record]
  ttl             = 60
  allow_overwrite = true
}

# This resource does nothing except wait for validation to finish.
resource "aws_acm_certificate_validation" "app" {
  certificate_arn         = aws_acm_certificate.app.arn
  validation_record_fqdns = [for r in aws_route53_record.cert_validation : r.fqdn]
}
```

Then an HTTPS listener, and a redirect from HTTP:

```hcl
# modules/web/main.tf
resource "aws_lb_listener" "https" {
  count = var.certificate_arn == null ? 0 : 1

  load_balancer_arn = aws_lb.main.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = var.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"

  # With a certificate, redirect to HTTPS. Without one, just forward.
  dynamic "default_action" {
    for_each = var.certificate_arn == null ? [] : [1]

    content {
      type = "redirect"

      redirect {
        port        = "443"
        protocol    = "HTTPS"
        status_code = "HTTP_301"
      }
    }
  }

  dynamic "default_action" {
    for_each = var.certificate_arn == null ? [1] : []

    content {
      type             = "forward"
      target_group_arn = aws_lb_target_group.app.arn
    }
  }
}
```

The `count = var.certificate_arn == null ? 0 : 1` is Module 6's on/off switch,
making DNS genuinely optional.

### Alarms and notifications

**Cost: free.** Ten CloudWatch alarms and SNS are within free tier.

```hcl
# modules/monitoring/main.tf

resource "aws_sns_topic" "alerts" {
  name = "${var.name_prefix}-alerts"
}

resource "aws_sns_topic_subscription" "email" {
  for_each = toset(var.alert_emails)

  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = each.value
}

# --- Is anything actually serving traffic? ---
resource "aws_cloudwatch_metric_alarm" "unhealthy_hosts" {
  alarm_name          = "${var.name_prefix}-unhealthy-hosts"
  alarm_description   = "One or more Notely app servers are failing health checks."
  namespace           = "AWS/ApplicationELB"
  metric_name         = "UnHealthyHostCount"
  statistic           = "Maximum"
  period              = 60
  evaluation_periods  = 2
  threshold           = 0
  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    LoadBalancer = var.alb_arn_suffix
    TargetGroup  = var.target_group_arn_suffix
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
  ok_actions    = [aws_sns_topic.alerts.arn]
}

# --- Are users seeing errors? ---
resource "aws_cloudwatch_metric_alarm" "http_5xx" {
  alarm_name          = "${var.name_prefix}-http-5xx"
  alarm_description   = "Notely is returning server errors."
  namespace           = "AWS/ApplicationELB"
  metric_name         = "HTTPCode_Target_5XX_Count"
  statistic           = "Sum"
  period              = 300
  evaluation_periods  = 1
  threshold           = var.error_threshold
  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    LoadBalancer = var.alb_arn_suffix
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# --- Is the site slow? ---
resource "aws_cloudwatch_metric_alarm" "response_time" {
  alarm_name          = "${var.name_prefix}-slow-responses"
  alarm_description   = "Notely's 95th percentile response time is above target."
  namespace           = "AWS/ApplicationELB"
  metric_name         = "TargetResponseTime"
  extended_statistic  = "p95"
  period              = 300
  evaluation_periods  = 2
  threshold           = var.latency_threshold_seconds
  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    LoadBalancer = var.alb_arn_suffix
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# --- Is the database about to fall over? ---
resource "aws_cloudwatch_metric_alarm" "db_cpu" {
  alarm_name          = "${var.name_prefix}-db-cpu"
  alarm_description   = "Notely's database CPU is sustained above 80%."
  namespace           = "AWS/RDS"
  metric_name         = "CPUUtilization"
  statistic           = "Average"
  period              = 300
  evaluation_periods  = 3
  threshold           = 80
  comparison_operator = "GreaterThanThreshold"

  dimensions = {
    DBInstanceIdentifier = var.db_instance_id
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

resource "aws_cloudwatch_metric_alarm" "db_storage" {
  alarm_name          = "${var.name_prefix}-db-storage"
  alarm_description   = "Notely's database is running out of disk."
  namespace           = "AWS/RDS"
  metric_name         = "FreeStorageSpace"
  statistic           = "Minimum"
  period              = 300
  evaluation_periods  = 1
  threshold           = 2147483648    # 2 GB in bytes
  comparison_operator = "LessThanThreshold"

  dimensions = {
    DBInstanceIdentifier = var.db_instance_id
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# --- Is anyone about to get a surprise bill? ---
resource "aws_budgets_budget" "monthly" {
  count = var.monthly_budget_usd == null ? 0 : 1

  name         = "${var.name_prefix}-monthly"
  budget_type  = "COST"
  limit_amount = tostring(var.monthly_budget_usd)
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  cost_filter {
    name   = "TagKeyValue"
    values = ["user:Project$${var.project_name}"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "FORECASTED"
    subscriber_sns_topic_arns  = [aws_sns_topic.alerts.arn]
  }
}
```

Five alarms answering five real questions: is anything serving, are users seeing
errors, is it slow, is the database struggling, and is it costing more than
expected.

Note `treat_missing_data = "notBreaching"` — without it, an alarm with no data
goes to `INSUFFICIENT_DATA` and can fire spuriously.

And `$${var.project_name}` in the budget filter — the `$$` escapes it so AWS
receives a literal `$` in the tag filter syntax.

### The complete `modules/notely`

```hcl
module "network" {
  source      = "../network"
  name_prefix = local.name_prefix
  vpc_cidr    = var.vpc_cidr
  az_count    = var.az_count
}

module "security" {
  source      = "../security"
  name_prefix = local.name_prefix
  vpc_id      = module.network.vpc_id
  app_port    = var.app_port
}

module "storage" {
  source             = "../storage"
  name_prefix        = local.name_prefix
  enable_versioning  = var.enable_versioning
  log_retention_days = var.log_retention_days
}

module "database" {
  source                = "../database"
  name_prefix           = local.name_prefix
  subnet_ids            = module.network.data_subnet_ids
  security_group_id     = module.security.db_security_group_id
  instance_class        = var.db_instance_class
  multi_az              = var.db_multi_az
  backup_retention_days = var.db_backup_retention_days
  deletion_protection   = var.db_deletion_protection
  skip_final_snapshot   = !var.db_deletion_protection
}

module "iam" {
  source                 = "../iam"
  name_prefix            = local.name_prefix
  attachments_bucket_arn = module.storage.attachments_bucket_arn
  log_group_arn          = module.storage.log_group_arn
  db_secret_arn          = module.database.secret_arn
}

module "dns" {
  count  = var.domain_name == null ? 0 : 1
  source = "../dns"

  domain_name    = var.domain_name
  hosted_zone_id = var.hosted_zone_id
  alb_dns_name   = module.web.alb_dns_name
  alb_zone_id    = module.web.alb_zone_id
}

module "web" {
  source                = "../web"
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
  certificate_arn       = try(module.dns[0].certificate_arn, null)
}

module "monitoring" {
  source = "../monitoring"

  name_prefix             = local.name_prefix
  project_name            = var.project_name
  alert_emails            = var.alert_emails
  alb_arn_suffix          = module.web.alb_arn_suffix
  target_group_arn_suffix = module.web.target_group_arn_suffix
  db_instance_id          = module.database.instance_id
  error_threshold         = var.error_threshold
  latency_threshold_seconds = var.latency_threshold_seconds
  monthly_budget_usd      = var.monthly_budget_usd
}
```

Eight modules. Read the block names top to bottom and you have the architecture.

---

## Part 2 — Repository layout for a team

```text
notely-infra/
├── README.md                    what this is, how to run it
├── CONTRIBUTING.md              how to make a change
├── .gitignore
├── .gitlab-ci.yml               or .github/workflows/
├── .tflint.hcl
├── .checkov.yaml
├── .terraform-version           tfenv picks this up
│
├── docs/
│   ├── architecture.md          the diagram and why
│   ├── runbooks/
│   │   ├── state-recovery.md
│   │   ├── stuck-lock.md
│   │   └── rollback.md
│   └── decisions/
│       ├── 001-directory-per-environment.md
│       └── 002-no-nat-gateway-in-dev.md
│
├── scripts/
│   ├── check.sh                 everything CI runs, locally
│   └── sweep.sh                 find untagged or orphaned resources
│
├── modules/
│   ├── network/
│   │   ├── main.tf variables.tf outputs.tf versions.tf README.md
│   │   └── tests/
│   ├── security/  storage/  database/  iam/  web/  dns/  monitoring/
│   └── notely/                  composes the above
│
└── envs/
    ├── dev/       backend.tf main.tf outputs.tf
    ├── staging/
    └── prod/
```

Things worth calling out:

**`docs/decisions/`** — architecture decision records. Three paragraphs each:
what we decided, why, what we gave up. When someone asks in a year "why is there
no NAT gateway in dev", the answer is written down.

**`docs/runbooks/`** — what to do when things break. Written *before* you need
them, because 3am is not the time to work out how to recover state.

**`scripts/check.sh`** — the same commands CI runs. If it passes locally, CI
passes.

**Module `README.md`** — what it does, what it needs, what it returns. Generate
the tables with `terraform-docs`:

```bash
terraform-docs markdown table --output-file README.md modules/network
```

---

## Part 3 — Naming and tagging

### Naming

Pick a pattern and never deviate:

```text
<project>-<environment>-<component>[-<qualifier>]

notely-prod-vpc
notely-prod-app-sg
notely-prod-db
notely-dev-public-a
```

Notely does this with `local.name_prefix = "${var.project_name}-${var.environment}"`.

Two rules:

**Environment goes in the name.** In a console full of resources you need to know
instantly whether you are looking at prod.

**Lowercase and hyphens.** Some AWS resources reject uppercase, some reject
underscores. Lowercase and hyphens work everywhere.

### Tagging

```hcl
provider "aws" {
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
      Repository  = "github.com/knowrahman/notely-infra"
      Owner       = "platform-team"
      CostCentre  = "engineering"
    }
  }
}
```

Every tag earns its place:

| Tag | Answers |
|---|---|
| `Project` | What is this for? Drives cost allocation. |
| `Environment` | Can I safely delete this? |
| `ManagedBy` | Am I allowed to click this in the console? |
| `Repository` | Where is the code? |
| `Owner` | Who do I ask? |
| `CostCentre` | Whose budget? |

`ManagedBy = "Terraform"` is the most useful one day to day. It tells anyone in
the console that changing this by hand will be reverted.

`Repository` is the one people thank you for. Finding the code that created a
mystery resource is otherwise genuinely hard.

Enforce it with checkov or a policy:

```yaml
# .checkov.yaml
check:
  - CKV_AWS_RESOURCE_TAGS
```

---

## Part 4 — Reviewing Terraform changes

Reviewing HCL is not like reviewing application code. **The plan is the diff that
matters.**

### The checklist

```markdown
## Terraform review checklist

### The plan
- [ ] Is the plan attached or rendered in the PR?
- [ ] Does the destroy count match the intent? Anything unexpected?
- [ ] Any `-/+` replacements? Is losing that resource acceptable?
- [ ] Any `# forces replacement` on a stateful resource?
- [ ] Does the resource count roughly match the description?

### The code
- [ ] Does it follow the naming convention?
- [ ] Are new variables typed, described and validated?
- [ ] Are secrets handled properly - nothing hardcoded, nothing output?
- [ ] Is `prevent_destroy` on anything stateful?
- [ ] Is `create_before_destroy` on anything serving traffic?
- [ ] Are security group rules as narrow as they can be?
- [ ] Any `0.0.0.0/0` on a port that is not 80 or 443?
- [ ] Any new `ignore_changes` - and is there a comment saying why?

### Cost
- [ ] Does this add a NAT gateway, ALB, RDS instance or anything hourly?
- [ ] Is the cost mentioned in the PR description?

### Process
- [ ] Do the tests pass? Are there tests for new module behaviour?
- [ ] Is tflint clean?
- [ ] Are new checkov findings fixed or explained?
- [ ] If resources were renamed or moved, are there `moved` blocks?
- [ ] Was the plan empty before this change?
```

### The three lines that matter most

```text
Plan: 2 to add, 1 to change, 0 to destroy.
```

**`0 to destroy` on a refactor.** If the PR says "move things into a module" and
the plan destroys anything, a `moved` block is missing.

**`# forces replacement`.** Find it, read the attribute, decide whether losing
that resource is acceptable.

**The count.** "Add a security group rule" should not produce 23 changes.

> Review the plan before the code. Twelve lines of HCL can replace a production database, and the HCL diff will not tell you.

---

## Part 5 — Cost awareness

### The usual suspects

| Resource | Cost | Notes |
|---|---|---|
| **NAT Gateway** | ~$32/month each | The number one accidental bill |
| **EKS control plane** | ~$73/month | Per cluster, even if idle |
| **ALB / NLB** | ~$16/month | Plus per-LCU charges |
| **RDS** | varies | Multi-AZ doubles it |
| **Interface VPC endpoints** | ~$7/month each | Add up fast |
| **Unattached Elastic IP** | ~$3.60/month | Classic zombie |
| **Provisioned IOPS** | expensive | Easy to over-specify |
| **CloudWatch Logs** | $0.50/GB ingested | Chatty apps get expensive |
| **Data transfer** | varies | Cross-AZ traffic is not free |

### Habits that prevent surprises

**Tag everything and set a budget:**

```hcl
resource "aws_budgets_budget" "project" {
  name         = "${var.project_name}-monthly"
  budget_type  = "COST"
  limit_amount = "50"
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  cost_filter {
    name   = "TagKeyValue"
    values = ["user:Project$${var.project_name}"]
  }
}
```

**Run `infracost` in CI:**

```bash
infracost breakdown --path envs/prod
```

```text
 Name                                  Monthly Qty  Unit    Monthly Cost
 module.web.aws_lb.main
 └─ Application load balancer                  730  hours         $16.43
 module.database.aws_db_instance.this
 └─ Database instance (db.t3.micro)            730  hours         $12.41

 OVERALL TOTAL                                                    $28.84
```

It comments on pull requests with the cost delta. A reviewer sees "+$32/month"
next to a NAT gateway and asks whether it is needed.

**Sweep for orphans:**

```bash
# scripts/sweep.sh
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text

# Untagged, which usually means console-created and forgotten
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?!not_null(Tags)].InstanceId' --output text

aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].PublicIp' --output text
```

---

## Part 6 — Runbooks

Write these before you need them.

### State is locked and nobody is running anything

```text
Error: Error acquiring the state lock
  ID:        7c4b9e2a-...
  Who:       runner@gitlab-runner-abc
  Created:   2026-09-08 03:14:22 UTC
```

1. **Check nobody is applying.** Look at the CI pipeline list. Ask in chat.
2. If the lock is from a cancelled job (the `Who` is a runner that has finished):
   ```bash
   terraform force-unlock 7c4b9e2a-...
   ```
3. Re-plan before applying — state may have partially changed.

**Never force-unlock a lock held by a running apply.** That is how state gets
corrupted.

### State is corrupted or was overwritten

This is why the bucket has versioning.

```bash
BUCKET=notely-tfstate-a1b2c3d4
KEY=notely/prod/terraform.tfstate

# 1. Find the versions
aws s3api list-object-versions --bucket $BUCKET --prefix $KEY \
  --query 'Versions[].[VersionId,LastModified,Size]' --output table

# 2. Download the last good one
aws s3api get-object --bucket $BUCKET --key $KEY \
  --version-id <VERSION_ID> recovered.tfstate

# 3. Check it looks right
python3 -c "
import json
s = json.load(open('recovered.tfstate'))
print('serial:', s['serial'])
print('resources:', len(s['resources']))
"

# 4. Push it back
terraform state push recovered.tfstate

# 5. Verify
terraform plan
```

An empty plan means you recovered correctly.

### An apply failed halfway

Terraform has no rollback. Some resources exist, some do not, and state records
what was created.

1. **Read the error.** Usually a permissions problem, a quota, or a name clash.
2. **Do not panic and destroy.** State is accurate — it knows what was created.
3. Fix the cause.
4. `terraform plan` — it will propose creating only what is missing.
5. `terraform apply`.

Terraform is designed for this. The partial state is correct, not corrupt.

### Someone destroyed something in production

1. **Check state:**
   ```bash
   terraform state list | grep <thing>
   ```
2. **If it is still in state**, it was deleted outside Terraform. `terraform apply`
   recreates it.
3. **If it is gone from state**, someone ran `destroy` or `state rm`. Restore the
   previous state version (above), then apply.
4. **For data**, state does not help. You need the RDS snapshot or S3 version.

This is why `prevent_destroy` exists:

```hcl
lifecycle {
  prevent_destroy = true
}
```

### Rolling back a bad change

Terraform has no `terraform rollback`. The rollback is **git revert plus apply**:

```bash
git revert <commit>
git push
# pipeline plans, someone reviews, someone approves, it applies
```

Which is why every change goes through the pipeline. A change applied from a
laptop cannot be reverted this way, because there is nothing to revert.

Caveat: reverting is not always safe. If the change deleted data, reverting
recreates an empty resource. Read the revert plan as carefully as the original.

---

## Part 7 — Upgrading safely

### Terraform versions

```hcl
terraform {
  required_version = ">= 1.5.0, < 2.0.0"
}
```

```text
1.9.5     .terraform-version, picked up by tfenv
```

Upgrade deliberately: bump the version in one pull request, let CI plan every
environment, check no plan changed unexpectedly, merge.

### Provider versions

```hcl
aws = {
  source  = "hashicorp/aws"
  version = "~> 5.0"
}
```

```bash
terraform init -upgrade
```

This updates `.terraform.lock.hcl`. Commit it in its own pull request, so the
plan diff shows exactly what the new provider version changed.

**Read the changelog for major versions.** AWS provider 5.0 changed S3 bucket
resources significantly; upgrading blind produced enormous, alarming plans.

### The upgrade routine

```text
1. One PR, one version bump
2. CI plans dev, staging and prod
3. Read all three plans
4. Unexpected changes? Read the changelog before proceeding
5. Merge, apply to dev, wait a day
6. Then staging, then prod
```

---

## Part 8 — Capstone lab: the complete Notely

**Cost, honestly:**

| Resource | Rate | 4 hours | 24 hours |
|---|---|---|---|
| ALB | $0.0225/hr | $0.09 | $0.54 |
| 2 × EC2 `t3.micro` | $0.0104/hr each | $0.08 | $0.50 |
| RDS `db.t3.micro` | $0.017/hr | $0.07 | $0.41 |
| Secrets Manager | $0.40/month | $0.00 | $0.01 |
| Everything else | free | — | — |
| **Total** | | **~$0.24** | **~$1.46** |

**Under $2 if you destroy it the same day.** Free tier makes it less.

There is a cheaper variant at the end that skips the ALB and RDS.

### Step 1: Set the guard rail first

```bash
ACCOUNT=$(aws sts get-caller-identity --query Account --output text)

aws budgets create-budget --account-id $ACCOUNT --budget '{
  "BudgetName": "notely-capstone",
  "BudgetLimit": {"Amount": "10", "Unit": "USD"},
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}'
```

### Step 2: Assemble the modules

```bash
cd ~/terraform-labs/notely
mkdir -p modules/{dns,monitoring}
```

Create `modules/monitoring` from Part 1. Skip `modules/dns` unless you have a
domain.

Add the module blocks to `modules/notely/main.tf`.

### Step 3: Check before you spend

```bash
./scripts/check.sh
```

Everything must pass before you apply anything.

```bash
cd envs/dev
terraform init
terraform plan
```

**Read the whole plan.** Count the resources. Around 35–40 for the full stack.

**If you want zero cost, stop here.** The plan shows you everything the apply
would do.

### Step 4: Apply

```bash
terraform apply
```

Roughly 8–12 minutes, mostly RDS.

```bash
terraform output
```

```text
alb_dns_name       = "notely-dev-alb-1234567890.ap-southeast-2.elb.amazonaws.com"
attachments_bucket = "notely-dev-attachments-123456789012"
db_endpoint        = "notely-dev-db.abc.ap-southeast-2.rds.amazonaws.com:5432"
db_secret_arn      = "arn:aws:secretsmanager:...:secret:notely-dev/db-credentials-AbCd"
url                = "http://notely-dev-alb-1234567890.ap-southeast-2.elb.amazonaws.com"
```

### Step 5: Confirm it works

Wait two or three minutes for health checks, then:

```bash
curl $(terraform output -raw url)
```

```json
{"app":"notely","env":"dev","host":"ip-10-0-11-42.ap-southeast-2.compute.internal"}
```

Run it several times — the `host` alternates between your two servers.

```bash
curl $(terraform output -raw url)/health
```

```json
{"status":"ok"}
```

### Step 6: Verify the whole architecture

**Both targets healthy:**

```bash
TG_ARN=$(aws elbv2 describe-target-groups \
  --query "TargetGroups[?starts_with(TargetGroupName, 'ntly')].TargetGroupArn" \
  --output text | head -1)

aws elbv2 describe-target-health --target-group-arn $TG_ARN \
  --query 'TargetHealthDescriptions[].{Target:Target.Id,Health:TargetHealth.State}' \
  --output table
```

**Servers have no public IP:**

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=notely" "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].{Id:InstanceId,AZ:Placement.AvailabilityZone,Private:PrivateIpAddress,Public:PublicIpAddress}' \
  --output table
```

`Public` should be `None`. They are in private subnets.

**Database is unreachable from outside:**

```bash
nc -zv -w 5 $(terraform output -raw db_address) 5432
```

Times out. Correct.

**The secret exists and is readable by exactly one role:**

```bash
aws secretsmanager get-secret-value \
  --secret-id $(terraform output -raw db_secret_arn) \
  --query SecretString --output text | python3 -m json.tool | grep -v password
```

**Alarms are configured:**

```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix notely-dev \
  --query 'MetricAlarms[].{Name:AlarmName,State:StateValue}' --output table
```

### Step 7: Break it and watch it recover

This is the best part. Terminate one server:

```bash
INSTANCE=$(aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=notely" "Name=instance-state-name,Values=running" \
  --query 'Reservations[0].Instances[0].InstanceId' --output text)

aws ec2 terminate-instances --instance-ids $INSTANCE
```

Now keep polling:

```bash
for i in $(seq 1 20); do
  curl -s $(terraform output -raw url) | python3 -c "import sys,json; print(json.load(sys.stdin)['host'])" 2>/dev/null || echo "no response"
  sleep 10
done
```

The site **stays up**, served entirely by the surviving server. That is the
two-AZ design from Module 0 doing exactly what it was for.

Then:

```bash
terraform plan
```

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

Terraform noticed the missing server.

```bash
terraform apply
```

It comes back. **That is the whole promise of infrastructure as code** — the
system is described, so it can be rebuilt.

### Step 8: Destroy

**Do this. Things are billing.**

```bash
terraform destroy
```

Several minutes, mostly RDS.

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text

aws secretsmanager list-secrets \
  --query 'SecretList[?starts_with(Name, `notely`)].Name' --output text

aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].PublicIp' --output text
```

All three empty. You are clean.

### The cheap variant

To run the capstone for essentially nothing, set in `envs/dev/main.tf`:

```hcl
module "notely" {
  source = "../../modules/notely"

  environment = "dev"
  vpc_cidr    = "10.0.0.0/16"
  az_count    = 2

  create_load_balancer = false   # skip the ALB, ~$16/month
  create_database      = false   # skip RDS
  app_instance_count   = 1       # one t3.micro
}
```

with `count` switches on the ALB and database modules. You still build the VPC,
subnets, routing, security groups, S3, IAM, CloudWatch and a working server —
everything except the two billable pieces.

### What you should have at the end

- The complete Notely architecture applied and working
- A URL returning JSON from a private server through a load balancer
- Both servers healthy, neither reachable directly
- A database nothing outside the VPC can reach
- Five alarms and a budget
- A terminated server, a site that stayed up, and Terraform rebuilding it
- An empty AWS account

---

## Common Mistakes At This Stage

**1. No cost visibility.** Add `infracost` or at least a budget. "It's only a
NAT gateway" is $32/month, twelve times.

**2. Writing runbooks after the incident.** 3am is not when you work out how to
recover state.

**3. No `prevent_destroy` on stateful resources.** One line, and it prevents the
worst possible afternoon.

**4. Reviewing HCL without the plan.** Twelve lines of HCL can replace a
database.

**5. Upgrading providers without reading the changelog.** AWS provider 5.0
restructured S3 resources.

**6. One state file for everything.** Slow plans and enormous blast radius.

**7. Tags nobody enforces.** Untagged resources cannot be swept, budgeted, or
traced back to code.

**8. Applying to prod from a laptop.** No audit trail, no review, no rollback.

**9. Never testing recovery.** Restore a state file in dev at least once, so you
have done it before you need to.

**10. Treating infrastructure code as second-class.** It creates the thing your
application runs on. It deserves the same review, tests and care.

---

## Summary Table

| Area | The one-line version |
|---|---|
| **Route 53 alias** | Points at an ALB; works at the apex, free to query |
| **ACM + DNS validation** | Terraform can create the validation records itself |
| **`treat_missing_data`** | Set it, or alarms fire on no data |
| **Repository layout** | `modules/`, `envs/`, `docs/`, `scripts/` |
| **ADRs** | Write down why, not just what |
| **Runbooks** | Written before the incident |
| **`terraform-docs`** | Generates module READMEs |
| **Naming** | `<project>-<env>-<component>`, lowercase and hyphens |
| **`default_tags`** | `ManagedBy` and `Repository` are the useful ones |
| **Review the plan first** | It is the diff that matters |
| **`0 to destroy` on a refactor** | Otherwise a `moved` block is missing |
| **`# forces replacement`** | Always find it and decide |
| **NAT Gateway** | ~$32/month. The usual surprise. |
| **`infracost`** | Cost delta on every pull request |
| **Stuck lock** | Confirm nobody is applying, then `force-unlock` |
| **Corrupted state** | S3 versioning. `state push` the last good version. |
| **Failed apply** | State is accurate. Fix, re-plan, re-apply. |
| **Rollback** | `git revert` plus apply. There is no other kind. |
| **`prevent_destroy`** | On every stateful resource |
| **Upgrades** | One PR, read all three plans, dev first |

---

## Checkpoint (answer briefly)

1. In a code review, what do you look at before the HCL, and what three things do you look for?
2. Terraform has no rollback. So how do you undo a bad change?
3. Your state file is corrupted. What saves you, and what are the steps?
4. Why is `ManagedBy = "Terraform"` one of the most useful tags?
5. An apply fails halfway through. Why should you not immediately run `terraform destroy`?
6. Why does the capstone's alarm module set `treat_missing_data = "notBreaching"`?
7. You terminated one of Notely's two servers and the site stayed up. Which design decisions made that work?

---

## Checkpoint — model answers

### 1. Reviewing: the plan first

**Look at the plan before the code.**

HCL is not a reliable indicator of impact. A three-character change to an RDS
`name` argument produces a plan that replaces a production database. Reading the
diff you would see a rename; reading the plan you see `1 to destroy`.

The three things:

**The counts.** `Plan: X to add, Y to change, Z to destroy.` Does `Z` match the
intent? On a pure refactor it must be zero. Does the total roughly match the
description — "add one security group rule" should not be 23 changes.

**Any `-/+` replacement.** These destroy and recreate. For stateless things that
is fine; for a database, a bucket with objects, or anything with an IP other
things depend on, it is not.

**The `# forces replacement` comment.** When there is a replacement, this names
the attribute that caused it. Read it and decide deliberately whether losing that
resource is acceptable.

Then read the code — naming, validation, secrets, `prevent_destroy`, security
group scope. But the plan comes first, because that is the actual change.

### 2. Undoing a bad change

**`git revert` plus a normal apply.** There is no other kind of rollback.

```bash
git revert <commit>
git push
# the pipeline plans, someone reviews, someone approves, it applies
```

The revert restores the previous configuration; Terraform reconciles reality back
to it. Which is why every change should go through the pipeline — a change
applied from a laptop has no commit to revert.

Two important caveats.

**Reverting is not always safe.** If the change deleted something, reverting
recreates it *empty*. An S3 bucket comes back without its objects; an RDS
instance comes back without its data. Read the revert plan as carefully as the
original.

**For data loss, code cannot help.** State and configuration describe
infrastructure, not contents. Recovery there means an RDS snapshot, an S3 object
version, or a backup. That is why `prevent_destroy` matters more than any
rollback procedure — the cheapest recovery is the deletion that never happened.

### 3. Recovering corrupted state

**S3 bucket versioning saves you.** Every write to the state file creates a new
version, so the previous good state is one command away.

```bash
# 1. List the versions
aws s3api list-object-versions --bucket notely-tfstate-a1b2c3d4 \
  --prefix notely/prod/terraform.tfstate \
  --query 'Versions[].[VersionId,LastModified]' --output table

# 2. Download the last good one
aws s3api get-object --bucket notely-tfstate-a1b2c3d4 \
  --key notely/prod/terraform.tfstate \
  --version-id <VERSION_ID> recovered.tfstate

# 3. Sanity check it
python3 -c "
import json; s=json.load(open('recovered.tfstate'))
print('serial:', s['serial'], 'resources:', len(s['resources']))"

# 4. Push it back
terraform state push recovered.tfstate

# 5. Verify
terraform plan
```

**An empty plan is your proof.** If the recovered state matches reality, nothing
needs changing.

Without versioning, S3 overwrites in place and the previous state is gone — your
only copies would be whatever `terraform.tfstate.backup` files happen to be on
someone's laptop. That is why Module 3 insisted on it, and why it is worth
practising this in dev before you need it in prod.

### 4. `ManagedBy = "Terraform"`

**Because it tells anyone in the console whether they are allowed to touch
something.**

An AWS account accumulates resources from many sources: Terraform, CloudFormation,
console clicks, other tools, things nobody remembers creating. Looking at a
security group, there is no built-in way to know which.

That matters because the answer changes what you should do:

- **Managed by Terraform** — do not edit it here. Your change will be reverted on
  the next apply, and in the meantime it is undocumented drift. Change the code.
- **Not managed** — editing it in the console is the normal way to change it.

Without the tag, well-meaning people make console changes that get silently
reverted, and nobody understands why.

It also enables the sweep command Notely uses throughout, and makes drift
investigation much faster: if a resource is tagged, you know a `plan` will show
the difference.

The companion tag `Repository = "github.com/knowrahman/notely-infra"` completes
it — now you know not just that it is managed, but *where the code is*. Tracking
down which of forty repositories created a mystery resource is otherwise a
genuinely annoying afternoon.

### 5. Why not destroy after a failed apply

**Because state is accurate, and destroying throws away information you need.**

Terraform has no rollback: when an apply fails partway, the resources it already
created stay created — and, importantly, **state records every one of them**. It
knows exactly what exists.

That means recovery is simple:

1. Read the error. It is usually a permissions problem, a service quota, or a
   name collision.
2. Fix the cause.
3. `terraform plan` — it proposes creating only what is missing.
4. `terraform apply`.

The partial state is correct, not corrupt. Terraform is designed for exactly this.

Running `destroy` instead is bad for three reasons. It deletes the resources that
did succeed, so you start from scratch and pay for the time again. If the failure
was mid-way through a stateful resource, you may destroy data. And if `destroy`
itself fails — quite likely, since whatever blocked the apply may block the
destroy — you are left in a genuinely messy state with fewer options.

The instinct to wipe and retry comes from environments where partial state is
untrustworthy. Terraform's is not.

### 6. `treat_missing_data = "notBreaching"`

**Because without it, an alarm with no data can fire spuriously.**

CloudWatch alarms have three states: `OK`, `ALARM`, and `INSUFFICIENT_DATA`. Many
metrics are only published when something happens — `HTTPCode_Target_5XX_Count`
reports nothing at all when there are no errors, rather than reporting zero.

So a perfectly healthy Notely publishes no 5xx data points. The default handling
(`missing`) leaves the alarm in `INSUFFICIENT_DATA`, and depending on
configuration and evaluation periods that can transition to `ALARM` — paging
someone because nothing is wrong.

`notBreaching` tells CloudWatch to treat a missing data point as "fine". No
errors reported means no errors, which is what you actually mean.

The options:

| Setting | Missing data treated as |
|---|---|
| `missing` | Ignored (default; can cause `INSUFFICIENT_DATA`) |
| `notBreaching` | Good |
| `breaching` | Bad |
| `ignore` | State unchanged |

`notBreaching` is right for error-count and low-traffic alarms. `breaching` is
occasionally right when *absence of data is itself the problem* — a heartbeat
metric that stops arriving means the thing died.

Alarms that page people when nothing is wrong get muted, and a muted alarm
protects nothing. This one setting is most of the difference.

### 7. Why the site survived losing a server

Four decisions, all made earlier in the course.

**Two servers in two availability zones (Module 6).** `for_each` over the app
subnets created one instance per AZ. There was a second server to serve traffic —
and it was in a different physical data centre, so it would survive an AZ failure
too, not just an instance failure.

**A load balancer with health checks (Module 6).** The ALB polls `/health` every
30 seconds and needs three consecutive failures to mark a target unhealthy. When
the instance was terminated, the ALB stopped routing to it within about 90
seconds and sent everything to the survivor.

**Stateless servers (Modules 5, 8, 10).** Nothing important lived on the instance.
Attachments are in S3, notes are in RDS, configuration comes from Secrets Manager
at boot. Losing a server lost nothing, so the survivor could serve every request
identically.

**The ALB spanning both public subnets (Module 8).** The `precondition` requiring
at least two subnets ensures the load balancer itself is not single-homed — it
has nodes in both AZs, so it survives the same failure.

And then the Terraform part: `terraform plan` showed `1 to add`, and `apply`
rebuilt the missing server from the same configuration. The system is described,
so it can be recreated — which is the entire argument for infrastructure as code,
demonstrated in about four minutes.

---

## Where we are now

You have finished the curriculum. Working through Modules 0–14 you have:

- Learned what a VPC, subnet, route table and security group are, from zero
- Built a complete three-tier AWS architecture entirely in code
- Understood state well enough to recover from problems with it
- Written reusable modules with clean interfaces
- Run three environments from one codebase
- Handled secrets so that nobody ever types a password
- Written tests that run with no cloud account
- Built a pipeline that plans on review and applies on approval
- Adopted infrastructure somebody else created by hand
- Terminated a production-shaped server and watched the system survive

That is a genuinely useful skill set. It is roughly what a mid-level
infrastructure engineer does day to day.

### What to learn next

| Topic | Why |
|---|---|
| **Kubernetes** | Terraform builds the cluster; something has to run on it. Nothing in this repo covers it yet. |
| **Packer** | Bake AMIs instead of installing Node.js at boot. Faster and more reliable. |
| **Ansible** | Terraform provisions, Ansible configures. The other half of the story. |
| **AWS Organizations** | Multi-account setups, which is how real companies structure things. |
| **HCP Terraform** | If your team wants managed state, policy enforcement and a private module registry. |
| **OpenTofu** | The open-source fork. Nearly drop-in compatible. Worth knowing it exists. |

### The habits worth keeping

1. **Read the plan.** Every time. Especially the destroy count.
2. **`terraform fmt` before every commit.**
3. **An empty plan proves a refactor was safe.**
4. **`for_each`, not `count`**, unless the answer is genuinely 0 or 1.
5. **`prevent_destroy` on anything stateful.**
6. **Never commit state, `.tfvars`, or credentials.**
7. **Tag everything.** `ManagedBy` and `Repository` especially.
8. **Destroy your labs.**
9. **Write down why, not just what.**
10. **The console is read-only** for anything Terraform manages.

> You do not need to remember every function or argument. You need to know what to look up, and to read the plan before you type yes.

---

## The whole course, in one table

| Module | The one thing to remember |
|---|---|
| **0 — Networking** | A subnet is public because its route table says `0.0.0.0/0 → igw` |
| **1 — Basics** | Terraform reconciles your description with reality |
| **2 — HCL** | References build the dependency graph; you never declare order |
| **3 — State** | State maps config addresses to real resources, and holds secrets in plaintext |
| **4 — Variables** | Variables are inputs, locals are computed. `-var` beats everything. |
| **5 — Expressions** | `for` expressions are `.map()` and `.filter()` |
| **6 — Meta-arguments** | `for_each`, not `count` — index shifting destroys more than you asked |
| **7 — Data sources** | `data` reads, `resource` owns. Always set `owners` on `aws_ami`. |
| **8 — Modules** | Outputs are the only public interface. `moved` makes refactoring safe. |
| **9 — Environments** | An environment is a separate state file |
| **10 — Secrets** | `sensitive` is a display filter. Give the app the ARN, not the secret. |
| **11 — Testing** | The best infrastructure test creates no infrastructure |
| **12 — CI/CD** | Apply the plan that was reviewed, not a fresh one |
| **13 — Import & drift** | An empty plan is the acceptance test for everything |
| **14 — Production** | Optimise for the person fixing this at 3am |

---

Roadmap and progress tracker: `terraform-roadmap.md`
Running example: `notely-architecture.md`
