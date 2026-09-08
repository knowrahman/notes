# Notely — The Running Example

Every module in this course builds one more piece of the same system. This file
is the map. Come back to it whenever you want to see where you are.

---

## What we are building

**Notely** is a note-taking API written in Node.js.

Think of it as a small backend you might actually build at work:

- People sign up and write notes
- Notes are stored in a PostgreSQL database
- People can attach files to a note, and those files go in S3
- It runs on more than one server, so if one dies the site stays up
- It has a real domain name
- When something breaks, you get an email

That is it. Nothing exotic. But building it properly touches almost every
Terraform concept there is, which is exactly why we use it.

> You do not need to write the Node.js app. We only build the infrastructure it runs on. Wherever a server needs code, we use a tiny placeholder API so you can see it working.

---

## The finished picture

This is where we end up in Module 14. Do not worry about understanding it yet.

```text
                              Internet
                                 |
                                 |  someone types notely.example.com
                                 v
                        +------------------+
                        |    Route 53      |   DNS: turns the name into an address
                        +------------------+
                                 |
                                 v
                 +-------------------------------+
                 |  Application Load Balancer    |   public subnets
                 |  (spreads traffic, checks     |   this is the only thing
                 |   which servers are alive)    |   the internet can reach
                 +-------------------------------+
                        /                 \
                       /                   \
                      v                     v
            +---------------+       +---------------+
            |  EC2 server   |       |  EC2 server   |   private subnets
            |  Node.js API  |       |  Node.js API  |   no direct internet access
            |  (AZ a)       |       |  (AZ b)       |
            +---------------+       +---------------+
                      \                     /
                       \                   /
                        v                 v
                 +-------------------------------+
                 |     RDS PostgreSQL            |   private subnets
                 |     (the database)            |   only the servers can reach it
                 +-------------------------------+

        Supporting pieces, used by the servers:

        S3 bucket ............. file attachments people upload
        CloudWatch Logs ....... where the app's logs go
        CloudWatch Alarms ..... notice when something is wrong
        SNS topic ............. email you when an alarm fires
        Secrets Manager ....... stores the database password
        IAM role .............. lets the servers use S3 and read the secret
```

---

## Why it is shaped like this

Every choice above has a reason. You will meet all of these properly in
**Module 0**, but here they are in one place.

| Choice | Why |
|---|---|
| **Two servers, not one** | If one server dies, the site stays up. One server means one bad afternoon takes you offline. |
| **Two Availability Zones** | An AZ is a physical group of data centres. They fail sometimes. Putting one server in each means a whole AZ can go dark and you survive. |
| **A load balancer in front** | Something has to decide which server gets each request, and stop sending traffic to a server that has died. |
| **Servers in private subnets** | Nothing on the internet can connect to them directly. The only way in is through the load balancer. Much smaller attack surface. |
| **Database in private subnets** | A database should never be reachable from the internet. Ever. |
| **Route 53 in front** | People type names, not IP addresses. And the load balancer's address changes; the name does not. |
| **S3 for attachments** | Files should not live on the servers. If a server is replaced, the files must survive. |
| **Secrets Manager for the password** | So the database password is never written in a file you might commit. |
| **SNS for alerts** | So you find out something broke from an email, not from a customer. |

---

## How we get there

Each module adds one piece. Nothing is thrown away — Notely only grows.

| Module | What we add | What it teaches |
|---|---|---|
| **0** | Nothing. We draw the network on paper. | VPC, subnets, routing, security groups |
| **1** | The S3 attachments bucket | Your first resource, the four commands |
| **2** | VPC + one public subnet + internet gateway + route table | HCL syntax, the dependency graph |
| **3** | (No new resources — we move the state file) | State, backends, locking |
| **4** | Same resources, now driven by variables | Variables, outputs, locals |
| **5** | Security groups, computed names, the server's startup script | Expressions, functions, `templatefile` |
| **6** | Subnets in **both** AZs, the load balancer, both servers | `count`, `for_each`, `lifecycle` |
| **7** | Stop hardcoding AMI IDs, AZ names, account IDs | Data sources |
| **8** | Reorganise into `modules/network`, `modules/web`, `modules/database` | Writing and using modules |
| **9** | Three copies: dev, staging, prod | Workspaces vs directories |
| **10** | The database, with its password in Secrets Manager | Secrets, IAM, provider auth |
| **11** | Tests and scanners over everything built so far | `terraform test`, tflint, checkov |
| **12** | A pipeline that plans on a PR and applies on merge | Terraform in CI/CD |
| **13** | Adopt a bucket somebody made by hand in the console | `import`, `moved`, drift |
| **14** | Route 53, CloudWatch alarms, SNS — and it all runs | Putting it together |

---

## Notely's network layout

This is the shape we settle on. Module 0 explains every line of it.

```text
  VPC   10.0.0.0/16          <- our own private slice of AWS
                                65,536 addresses to play with

  +---------------------------------------------------------------+
  |                                                               |
  |   AZ ap-southeast-2a              AZ ap-southeast-2b          |
  |   ---------------------           ---------------------       |
  |                                                               |
  |   PUBLIC   10.0.1.0/24            PUBLIC   10.0.2.0/24        |
  |   +-------------------+           +-------------------+       |
  |   | load balancer     |           | load balancer     |       |
  |   +-------------------+           +-------------------+       |
  |            |                               |                  |
  |   PRIVATE  10.0.11.0/24           PRIVATE  10.0.12.0/24       |
  |   +-------------------+           +-------------------+       |
  |   | EC2  Node.js API  |           | EC2  Node.js API  |       |
  |   +-------------------+           +-------------------+       |
  |                                                               |
  |   PRIVATE  10.0.21.0/24           PRIVATE  10.0.22.0/24       |
  |   +-------------------+           +-------------------+       |
  |   | RDS  Postgres     |           | RDS  standby      |       |
  |   +-------------------+           +-------------------+       |
  |                                                               |
  +---------------------------------------------------------------+
                            |
                      Internet Gateway
                            |
                        Internet
```

Three tiers, two of everything, one internet door.

| Tier | Subnets | What lives there | Reachable from internet? |
|---|---|---|---|
| Public | `10.0.1.0/24`, `10.0.2.0/24` | Load balancer | **Yes** |
| Private app | `10.0.11.0/24`, `10.0.12.0/24` | EC2 servers | No |
| Private data | `10.0.21.0/24`, `10.0.22.0/24` | RDS database | No |

---

## What each piece costs

Be honest about this from the start. Most of Notely is free. A few pieces are not.

| Piece | Cost | Used in labs? |
|---|---|---|
| VPC, subnets, route tables, internet gateway | **Free** | Yes, freely |
| Security groups | **Free** | Yes, freely |
| S3 bucket (a few files) | **Free tier** | Yes, freely |
| CloudWatch log group | **Free tier** | Yes, freely |
| SNS topic | **Free tier** | Yes, freely |
| Secrets Manager | ~$0.40/month per secret | Yes — delete after |
| EC2 `t3.micro` | Free tier for 12 months, else ~$0.01/hour | Yes, with a warning |
| Application Load Balancer | **~$0.022/hour (~$16/month)** | Only where flagged |
| RDS `db.t3.micro` | Free tier for 12 months, else ~$0.017/hour | Only where flagged |
| Route 53 hosted zone | **$0.50/month** | Capstone only, optional |
| **NAT Gateway** | **~$0.045/hour (~$32/month)** | **Never applied.** We explain it and use a free S3 gateway endpoint instead. |

> The NAT Gateway is the single most common way people get a surprise Terraform bill. We show you the code, we never leave one running.

Where a lab uses something billable, it says so in bold at the top, and offers a
`terraform plan` only version so you can learn the code without spending money.

---

## The cleanup command

Every Notely lab tags its resources like this:

```hcl
provider "aws" {
  region = "ap-southeast-2"

  default_tags {
    tags = {
      Project   = "notely"
      ManagedBy = "Terraform"
      Ephemeral = "true"
    }
  }
}
```

Which means one command finds anything a failed `destroy` left behind:

```bash
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=notely \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

Empty output means you are clean. Run it at the end of every session.

---

## Where to keep the lab code

Not in this notes repo. Use a scratch directory:

```bash
mkdir -p ~/terraform-labs/notely
cd ~/terraform-labs/notely
```

Each module tells you what to add. By Module 14 this directory holds a complete,
working infrastructure project you can reuse at work.

---

## Quick links

| Module | File |
|---|---|
| 0 — AWS networking primer | `00-aws-networking-primer.md` |
| 1 — IaC & Terraform basics | `01-iac-and-terraform-basics.md` |
| 2 — HCL & the core workflow | `02-hcl-and-core-workflow.md` |
| 3 — Terraform state | `03-terraform-state.md` |
| 4 — Variables, outputs, locals | `04-variables-outputs-locals.md` |
| 5 — Expressions & functions | `05-expressions-and-functions.md` |
| 6 — Meta-arguments | `06-meta-arguments.md` |
| 7 — Data sources | `07-data-sources.md` |
| 8 — Modules | `08-modules.md` |
| 9 — Environments & layout | `09-environments-and-layout.md` |
| 10 — Secrets & authentication | `10-secrets-and-auth.md` |
| 11 — Testing & validation | `11-testing-and-validation.md` |
| 12 — Terraform in CI/CD | `12-terraform-in-cicd.md` |
| 13 — Import & drift | `13-import-and-drift.md` |
| 14 — The complete build | `14-real-world-project.md` |

Roadmap and progress tracker: `README.md`
