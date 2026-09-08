# Module 5 — Expressions & Functions

## Where Notely is right now

```text
  Notely so far:

    S3 bucket (attachments) + versioning
    CloudWatch log group
    VPC + 1 public subnet + internet gateway + route table
    All driven by variables and locals          <- Module 4
    State in S3 with locking
```

By the end of this module Notely has security groups built from a list, a
computed set of names, and a real startup script that installs Node.js on a
server.

Full picture: `notely-architecture.md`.

---

## Why this module exists

Variables give you inputs. But real infrastructure needs those inputs
*transformed*.

You have a list of ports and you need a security group rule for each. You have a
CIDR block and you need to carve four subnets out of it. You have a template
script and you need to fill in the database URL before handing it to a server.

None of that is possible with plain variable references. You need expressions.

The good news: **you already know most of this.** Terraform's `for` expression is
JavaScript's `.map()`. Its `if` filter is `.filter()`. `flatten` is `.flat()`.
If you can write array methods, you can write HCL expressions.

---

## The core idea (one sentence)

Expressions let you compute values instead of typing them, and Terraform ships
about 100 built-in functions to do the computing.

> If you find yourself copy-pasting a resource block and changing one word, an expression can probably do it for you.

---

## Mental model (from JavaScript)

This table is the whole module in miniature. Come back to it.

| JavaScript | Terraform |
|---|---|
| `arr.map(x => x.id)` | `[for x in arr : x.id]` |
| `arr.filter(x => x.active)` | `[for x in arr : x if x.active]` |
| `arr.map(...).filter(...)` | `[for x in arr : x.id if x.active]` |
| `arr.flat()` | `flatten(arr)` |
| `Object.keys(obj)` | `keys(obj)` |
| `Object.values(obj)` | `values(obj)` |
| `Object.entries(obj)` | `{ for k, v in obj : ... }` |
| `{...a, ...b}` | `merge(a, b)` |
| `arr.join(", ")` | `join(", ", arr)` |
| `str.split(",")` | `split(",", str)` |
| `str.toUpperCase()` | `upper(str)` |
| `arr.length` | `length(arr)` |
| `[...new Set(arr)]` | `distinct(arr)` |
| `a ?? b` | `coalesce(a, b)` |
| `cond ? a : b` | `cond ? a : b` (identical) |
| `JSON.stringify(x)` | `jsonencode(x)` |
| `JSON.parse(s)` | `jsondecode(s)` |
| `try { f() } catch { d }` | `try(f(), d)` |
| Template literal `` `a${b}` `` | `"a${b}"` |

The syntax is different. The ideas are the same ones you use every day.

---

## Your best friend: `terraform console`

Before anything else, open the REPL. It is a Node REPL for Terraform.

```bash
cd ~/terraform-labs/notely
terraform console -var-file=dev.tfvars
```

```text
> 1 + 1
2

> upper("notely")
"NOTELY"

> [for n in [1, 2, 3] : n * 10]
[
  10,
  20,
  30,
]
```

Everything in this module can be tried there in two seconds. **Use it as you
read.** Guessing what a function does and then running `terraform apply` to find
out is the slow way to learn.

Type `exit` to leave.

---

## Part 1 — Operators

### Arithmetic

```hcl
2 + 3      # 5
10 - 4     # 6
3 * 4      # 12
10 / 3     # 3.3333...
10 % 3     # 1
-5         # negation
```

### Comparison

```hcl
1 == 1     # true
1 != 2     # true
3 > 2      # true
3 >= 3     # true
"a" == "a" # true
```

### Logical

```hcl
true && false   # false   (and)
true || false   # true    (or)
!true           # false   (not)
```

Same symbols as JavaScript. Nothing to learn.

### The conditional (ternary)

```hcl
condition ? value_if_true : value_if_false
```

Identical to JavaScript:

```hcl
var.environment == "prod" ? "t3.large" : "t3.micro"
```

Three real examples:

```hcl
# 1. Sizing by environment
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

# 2. Turning a feature on only in production
multi_az = var.environment == "prod"

# 3. Choosing how many servers
desired_capacity = var.environment == "prod" ? 4 : 1
```

**One gotcha:** both branches are type-checked, even the one that will not run.

```hcl
var.enabled ? "yes" : 42     # ERROR - one branch is a string, the other a number
```

Make both branches the same type.

---

## Part 2 — `for` expressions

This is the most useful thing in this module.

### List → list (`.map()`)

```hcl
[for item in collection : expression]
```

```hcl
> [for name in ["alice", "bob"] : upper(name)]
[
  "ALICE",
  "BOB",
]
```

JavaScript equivalent:

```javascript
["alice", "bob"].map(name => name.toUpperCase())
```

### With a filter (`.filter()`)

```hcl
[for item in collection : expression if condition]
```

```hcl
> [for n in [1, 2, 3, 4, 5, 6] : n if n % 2 == 0]
[
  2,
  4,
  6,
]
```

JavaScript:

```javascript
[1,2,3,4,5,6].filter(n => n % 2 === 0)
```

### Map and filter together

```hcl
> [for n in [1, 2, 3, 4, 5, 6] : n * 100 if n % 2 == 0]
[
  200,
  400,
  600,
]
```

JavaScript:

```javascript
[1,2,3,4,5,6].filter(n => n % 2 === 0).map(n => n * 100)
```

Note the order reads differently — HCL puts the transform first and the filter
last. The result is the same.

### Producing a map instead of a list

Swap the square brackets for curly braces, and use `=>`:

```hcl
{for item in collection : key_expression => value_expression}
```

```hcl
> {for name in ["alice", "bob"] : name => upper(name)}
{
  "alice" = "ALICE"
  "bob" = "BOB"
}
```

JavaScript:

```javascript
Object.fromEntries(["alice","bob"].map(n => [n, n.toUpperCase()]))
```

This form matters a lot in Module 6, because `for_each` wants a map.

### Iterating a map

When the source is a map, you get two loop variables:

```hcl
> {for key, value in { a = 1, b = 2 } : key => value * 10}
{
  "a" = 10
  "b" = 20
}
```

```hcl
# Swap keys and values
> {for k, v in { a = 1, b = 2 } : v => k}
{
  "1" = "a"
  "2" = "b"
}
```

### Iterating a list with the index

```hcl
> [for i, name in ["alice", "bob"] : "${i}: ${name}"]
[
  "0: alice",
  "1: bob",
]
```

Like `.map((name, i) => ...)`, but the index comes first.

### Four Notely examples

**Example 1 — build subnet names from a list.**

```hcl
locals {
  azs = ["ap-southeast-2a", "ap-southeast-2b"]

  public_subnet_names = [for az in local.azs : "notely-public-${substr(az, -1, 1)}"]
}
```

```text
["notely-public-a", "notely-public-b"]
```

**Example 2 — turn a list of subnet objects into a map keyed by name.**

```hcl
variable "subnets" {
  type = list(object({
    name       = string
    cidr_block = string
    public     = bool
  }))
}

locals {
  subnets_by_name = { for s in var.subnets : s.name => s }
}
```

```text
{
  "public-a" = { name = "public-a", cidr_block = "10.0.1.0/24", public = true }
  "app-a"    = { name = "app-a",    cidr_block = "10.0.11.0/24", public = false }
}
```

**Example 3 — filter to just the public ones.**

```hcl
locals {
  public_subnets  = { for s in var.subnets : s.name => s if s.public }
  private_subnets = { for s in var.subnets : s.name => s if !s.public }
}
```

Two maps out of one list. Module 6 feeds these straight into `for_each`.

**Example 4 — compute CIDR blocks instead of typing them.**

```hcl
locals {
  # Carve /24 subnets out of the VPC's /16
  public_cidrs = [for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 1)]
  app_cidrs    = [for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 11)]
}
```

With `vpc_cidr = "10.0.0.0/16"`:

```text
public_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
app_cidrs    = ["10.0.11.0/24", "10.0.12.0/24"]
```

Change the VPC to `10.1.0.0/16` and every subnet recalculates. No typos possible.

---

## Part 3 — Splat expressions

A shorthand for the most common `for` expression.

```hcl
# These two are identical
[for s in aws_subnet.public : s.id]
aws_subnet.public[*].id
```

The `[*]` means "give me this attribute from every element".

```hcl
# All subnet IDs
subnet_ids = aws_subnet.public[*].id

# All instance private IPs
ips = aws_instance.app[*].private_ip

# All availability zones
azs = aws_subnet.public[*].availability_zone
```

Use splat when you just want one attribute from a list. Use a full `for`
expression when you need to transform or filter.

> Splat works on lists (from `count`). For maps (from `for_each`) use `values(aws_subnet.public)[*].id`.

---

## Part 4 — The function library

There are about 100 functions. You need roughly 25. Here they are, grouped.

### Strings

| Function | Does | Example |
|---|---|---|
| `upper` / `lower` | Case | `upper("abc")` → `"ABC"` |
| `title` | Capitalise words | `title("hello world")` → `"Hello World"` |
| `trimspace` | Strip whitespace | `trimspace("  hi  ")` → `"hi"` |
| `substr` | Slice | `substr("notely", 0, 4)` → `"note"` |
| `replace` | Find and replace | `replace("a-b", "-", "_")` → `"a_b"` |
| `split` | String → list | `split(",", "a,b")` → `["a","b"]` |
| `join` | List → string | `join("-", ["a","b"])` → `"a-b"` |
| `format` | printf-style | `format("%s-%03d", "app", 7)` → `"app-007"` |
| `startswith` / `endswith` | Test | `endswith("a.tf", ".tf")` → `true` |
| `regex` | Extract with a pattern | `regex("[0-9]+", "abc123")` → `"123"` |
| `regexall` | All matches | `regexall("[0-9]+", "a1b22")` → `["1","22"]` |

```hcl
> format("notely-%s-%s", "prod", "alb")
"notely-prod-alb"

> substr("ap-southeast-2a", -1, 1)
"a"

> join(",", ["10.0.1.0/24", "10.0.2.0/24"])
"10.0.1.0/24,10.0.2.0/24"
```

### Collections

| Function | Does | JS equivalent |
|---|---|---|
| `length` | Count | `.length` |
| `keys` | Map keys | `Object.keys()` |
| `values` | Map values | `Object.values()` |
| `merge` | Combine maps | `{...a, ...b}` |
| `lookup` | Get with a fallback | `obj[k] ?? d` |
| `concat` | Join lists | `[...a, ...b]` |
| `flatten` | Un-nest | `.flat()` |
| `distinct` | Remove duplicates | `[...new Set()]` |
| `contains` | Is it in there | `.includes()` |
| `element` | Get by index, wrapping | — |
| `slice` | Sub-list | `.slice()` |
| `sort` | Sort strings | `.sort()` |
| `reverse` | Reverse | `.reverse()` |
| `zipmap` | Two lists → map | — |
| `range` | Generate numbers | — |
| `setproduct` | Every combination | — |
| `toset` / `tolist` / `tomap` | Convert types | — |

```hcl
> merge({ a = 1 }, { b = 2 }, { a = 99 })
{
  "a" = 99      # later wins
  "b" = 2
}

> lookup({ a = 1 }, "missing", "fallback")
"fallback"

> flatten([[1, 2], [3, [4]]])
[1, 2, 3, 4]

> zipmap(["a", "b"], [1, 2])
{
  "a" = 1
  "b" = 2
}

> range(3)
[0, 1, 2]

> distinct(["a", "b", "a"])
["a", "b"]
```

`setproduct` is worth knowing — it makes every combination of two lists:

```hcl
> setproduct(["a", "b"], ["public", "private"])
[
  ["a", "public"],
  ["a", "private"],
  ["b", "public"],
  ["b", "private"],
]
```

That is exactly how you build "a subnet of each type in each AZ" (Module 6).

### Networking — the ones that save you from CIDR maths

| Function | Does |
|---|---|
| `cidrsubnet(prefix, newbits, num)` | Carve a subnet out of a bigger block |
| `cidrhost(prefix, num)` | Get the nth address in a block |
| `cidrnetmask(prefix)` | Convert to a netmask |

`cidrsubnet` is the important one:

```hcl
> cidrsubnet("10.0.0.0/16", 8, 1)
"10.0.1.0/24"

> cidrsubnet("10.0.0.0/16", 8, 2)
"10.0.2.0/24"

> cidrsubnet("10.0.0.0/16", 8, 11)
"10.0.11.0/24"
```

Reading it: start with `10.0.0.0/16`, add 8 bits to the prefix (so `/16` becomes
`/24`), and give me block number N.

```hcl
> cidrhost("10.0.1.0/24", 1)
"10.0.1.1"

> cidrhost("10.0.1.0/24", 10)
"10.0.1.10"
```

> `cidrsubnet` means you never hand-calculate a subnet range again. Module 0 taught you what `/24` means; this function does the arithmetic.

### Encoding

| Function | Does |
|---|---|
| `jsonencode` | Value → JSON string |
| `jsondecode` | JSON string → value |
| `yamlencode` / `yamldecode` | Same for YAML |
| `base64encode` / `base64decode` | Base64 |
| `urlencode` | URL-escape |

`jsonencode` is how you write IAM policies:

```hcl
resource "aws_iam_role_policy" "app_s3" {
  name = "${local.name_prefix}-s3-access"
  role = aws_iam_role.app.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject",
        ]
        Resource = "${aws_s3_bucket.attachments.arn}/*"
      },
    ]
  })
}
```

Much better than a heredoc full of JSON: Terraform checks the structure,
interpolation works naturally, and you cannot produce invalid JSON.

### Files and templates

| Function | Does |
|---|---|
| `file(path)` | Read a file as a string |
| `fileexists(path)` | Does it exist |
| `filebase64(path)` | Read as base64 |
| `templatefile(path, vars)` | **Read a file and fill in variables** |
| `fileset(path, pattern)` | List files matching a glob |
| `abspath` / `dirname` / `basename` | Path manipulation |

### Safety

| Function | Does |
|---|---|
| `try(a, b, c)` | Return the first that does not error |
| `can(expr)` | `true` if it works, `false` if it errors |
| `coalesce(a, b, c)` | First non-null, non-empty value |
| `one(list)` | The single element, or null if empty |

```hcl
> try(var.optional_thing.name, "default")
"default"

> coalesce(null, "", "fallback")
"fallback"

> can(cidrnetmask("nonsense"))
false
```

`try` is your `try/catch`. `coalesce` is your `??` chain.

---

## Part 5 — `templatefile()`

This is how you generate the script that runs when a server boots.

### The problem

Notely's EC2 servers need to install Node.js, fetch the app, and start it. That
script needs values Terraform knows — the S3 bucket name, the log group, the
environment.

You could build it with string interpolation, but a 40-line bash script inside a
`.tf` file is horrible.

### The solution

Put the script in its own file with placeholders, and let Terraform fill them in.

Create `templates/user-data.sh.tftpl`:

```bash
#!/bin/bash
set -euo pipefail

# ------------------------------------------------------------------
# Notely app server bootstrap
# Generated by Terraform - do not edit on the instance.
# Environment: ${environment}
# ------------------------------------------------------------------

# Install Node.js 20
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
yum install -y nodejs

# Install the CloudWatch agent so logs reach ${log_group}
yum install -y amazon-cloudwatch-agent

mkdir -p /opt/notely
cd /opt/notely

# Configuration the app reads at startup
cat > /opt/notely/.env <<'ENVFILE'
NODE_ENV=${environment}
PORT=${app_port}
AWS_REGION=${region}
ATTACHMENTS_BUCKET=${attachments_bucket}
LOG_GROUP=${log_group}
ENVFILE

# A placeholder API so the load balancer health check passes.
# In real life you would pull a build artifact from S3 here.
cat > /opt/notely/server.js <<'APPFILE'
const http = require('http');
const port = process.env.PORT || 3000;

http.createServer((req, res) => {
  if (req.url === '/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    return res.end(JSON.stringify({ status: 'ok' }));
  }
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    app: 'notely',
    env: process.env.NODE_ENV,
    host: require('os').hostname(),
  }));
}).listen(port, () => console.log(`notely listening on ${port}`));
APPFILE

# Run it as a service so it restarts if it crashes
cat > /etc/systemd/system/notely.service <<'SVCFILE'
[Unit]
Description=Notely API
After=network.target

[Service]
ExecStart=/usr/bin/node /opt/notely/server.js
Restart=always
EnvironmentFile=/opt/notely/.env

[Install]
WantedBy=multi-user.target
SVCFILE

systemctl daemon-reload
systemctl enable --now notely
```

The `.tftpl` extension is convention. Any extension works.

Then in Terraform:

```hcl
locals {
  user_data = templatefile("${path.module}/templates/user-data.sh.tftpl", {
    environment        = var.environment
    region             = var.region
    app_port           = var.app_port
    attachments_bucket = aws_s3_bucket.attachments.bucket
    log_group          = aws_cloudwatch_log_group.app.name
  })
}
```

Every `${name}` in the template is replaced with the matching value from that
map.

### The escaping gotcha

Look at this line inside the template:

```bash
.listen(port, () => console.log(`notely listening on ${port}`));
```

That `${port}` is JavaScript, not Terraform — but Terraform will try to
interpolate it and fail, because there is no `port` variable passed in.

Two fixes:

**1. Escape it with `$${`:**

```bash
console.log(`notely listening on $${port}`)
```

Terraform turns `$${` into a literal `${`.

**2. Or use a quoted heredoc**, as the example above does with `<<'APPFILE'` —
though note this only stops *bash* from expanding it, not Terraform. Terraform
processes the whole file before bash ever sees it, so you still need `$${` for
anything Terraform should leave alone.

> Any `${...}` in a template file is Terraform's, unless you write `$${...}`. This catches everyone once.

### `templatefile` vs `file`

| | `file()` | `templatefile()` |
|---|---|---|
| Reads the file | Yes | Yes |
| Substitutes variables | **No** | Yes |
| Use for | Static content | Anything with values in it |

---

## Part 6 — `dynamic` blocks

### The problem

Some arguments are nested blocks, and you cannot use a `for` expression to
generate blocks — only values.

```hcl
resource "aws_security_group" "alb" {
  name = "notely-alb"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Two ports, two nearly identical blocks. Ten ports would be ten.

### The solution

```hcl
variable "alb_ingress_ports" {
  description = "Ports the load balancer accepts traffic on"
  type        = list(number)
  default     = [80, 443]
}

resource "aws_security_group" "alb" {
  name        = "${local.name_prefix}-alb"
  description = "Notely load balancer - accepts public web traffic"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.alb_ingress_ports

    content {
      description = "Web traffic on port ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    description = "Forward requests to the app servers"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Reading the `dynamic` block:

| Part | Meaning |
|---|---|
| `dynamic "ingress"` | Generate `ingress` blocks |
| `for_each` | The collection to loop over |
| `content { }` | The body of each generated block |
| `ingress.value` | The current item (named after the block) |
| `ingress.key` | The index, or map key |

### A richer example

Ports alone are rarely enough — you want a description and a source per rule:

```hcl
variable "app_ingress_rules" {
  description = "Inbound rules for the Notely app servers"

  type = list(object({
    description = string
    port        = number
    cidr_blocks = optional(list(string))
    source_sg   = optional(string)
  }))

  default = [
    {
      description = "App traffic from the load balancer"
      port        = 3000
    },
    {
      description = "Metrics scraping from inside the VPC"
      port        = 9090
      cidr_blocks = ["10.0.0.0/16"]
    },
  ]
}

resource "aws_security_group" "app" {
  name        = "${local.name_prefix}-app"
  description = "Notely app servers"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.app_ingress_rules

    content {
      description     = ingress.value.description
      from_port       = ingress.value.port
      to_port         = ingress.value.port
      protocol        = "tcp"
      cidr_blocks     = ingress.value.cidr_blocks
      security_groups = ingress.value.source_sg != null ? [ingress.value.source_sg] : null
    }
  }
}
```

### When NOT to use dynamic blocks

They are harder to read than plain blocks. Use them when:

- The number of blocks is genuinely variable
- You have more than about three near-identical blocks
- The list comes from a variable

Do **not** use them when:

- You have two blocks that differ meaningfully
- You are just showing off

```hcl
# Don't do this. Two clear blocks beat one clever one.
dynamic "ingress" {
  for_each = [
    { port = 443, desc = "HTTPS from the internet" },
    { port = 3000, desc = "App traffic from the ALB only" },
  ]
  ...
}
```

Those two rules mean completely different things. Write them out.

> A `dynamic` block trades readability for less repetition. Make sure you are getting a good deal.

---

## Building it into Notely

Three additions this module.

### 1. Computed subnet CIDRs

Replace the hardcoded subnet CIDR in `locals.tf`:

```hcl
locals {
  name_prefix   = "${var.project_name}-${var.environment}"
  is_production = var.environment == "prod"

  # The AZs we spread across. Module 7 looks these up instead of hardcoding.
  azs = ["${var.region}a", "${var.region}b"]

  # Carve subnets out of the VPC CIDR instead of typing them.
  # vpc_cidr = "10.0.0.0/16" gives:
  #   public  -> 10.0.1.0/24,  10.0.2.0/24
  #   app     -> 10.0.11.0/24, 10.0.12.0/24
  #   data    -> 10.0.21.0/24, 10.0.22.0/24
  public_subnet_cidrs = [for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 1)]
  app_subnet_cidrs    = [for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 11)]
  data_subnet_cidrs   = [for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 21)]

  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Ephemeral   = local.is_production ? "false" : "true"
  }
}
```

Check it in the console:

```bash
terraform console -var-file=dev.tfvars
```

```text
> local.public_subnet_cidrs
[
  "10.0.1.0/24",
  "10.0.2.0/24",
]

> local.app_subnet_cidrs
[
  "10.0.11.0/24",
  "10.0.12.0/24",
]
```

Then with prod's `10.1.0.0/16`:

```bash
terraform console -var-file=prod.tfvars
```

```text
> local.public_subnet_cidrs
[
  "10.1.1.0/24",
  "10.1.2.0/24",
]
```

The whole address plan recalculated from one variable.

Use the first one in the subnet resource for now — Module 6 creates all of them:

```hcl
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = local.public_subnet_cidrs[0]
  availability_zone       = local.azs[0]
  map_public_ip_on_launch = true

  tags = { Name = "${local.name_prefix}-public-a" }
}
```

### 2. Security groups

Create `security.tf`:

```hcl
variable "alb_ingress_ports" {
  description = "Ports the load balancer accepts public traffic on"
  type        = list(number)
  default     = [80, 443]
}

variable "app_port" {
  description = "Port the Node.js app listens on"
  type        = number
  default     = 3000
}

# ---------------------------------------------------------------
# Load balancer: open to the world on 80 and 443.
# This is the ONLY thing in Notely the internet can reach.
# ---------------------------------------------------------------
resource "aws_security_group" "alb" {
  name        = "${local.name_prefix}-alb"
  description = "Notely load balancer"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.alb_ingress_ports

    content {
      description = "Public web traffic on ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    description = "Anywhere - the ALB needs to reach the app servers"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${local.name_prefix}-alb-sg" }
}

# ---------------------------------------------------------------
# App servers: reachable ONLY from the load balancer.
# Note the security_groups line - we reference the ALB's group,
# not an IP range. Servers come and go; the group does not.
# ---------------------------------------------------------------
resource "aws_security_group" "app" {
  name        = "${local.name_prefix}-app"
  description = "Notely app servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "App traffic from the load balancer only"
    from_port       = var.app_port
    to_port         = var.app_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    description = "Outbound - AWS APIs, package registries"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${local.name_prefix}-app-sg" }
}

# ---------------------------------------------------------------
# Database: reachable ONLY from the app servers.
# Same trick again - reference the app's security group.
# ---------------------------------------------------------------
resource "aws_security_group" "db" {
  name        = "${local.name_prefix}-db"
  description = "Notely Postgres database"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "Postgres from the app servers only"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  tags = { Name = "${local.name_prefix}-db-sg" }
}
```

Security groups are **free**. Apply this and nothing costs money.

Notice the chain from Module 0, now in code:

```text
  internet -> alb_sg -> app_sg -> db_sg
```

Each layer only accepts traffic from the one above it.

### 3. The user-data template

Create `templates/user-data.sh.tftpl` with the script from Part 5, then add to
`locals.tf`:

```hcl
locals {
  user_data = templatefile("${path.module}/templates/user-data.sh.tftpl", {
    environment        = var.environment
    region             = var.region
    app_port           = var.app_port
    attachments_bucket = aws_s3_bucket.attachments.bucket
    log_group          = aws_cloudwatch_log_group.app.name
  })
}
```

You cannot use it yet — there are no servers until Module 6. But you can look at
it:

```bash
terraform console -var-file=dev.tfvars
```

```text
> local.user_data
```

You will see the rendered script with all the values filled in. That exact text
is what a server would run at boot.

---

## Real-World Example

A platform team maintains the VPC module used by 30 services.

Before expressions, their subnet configuration was 180 lines of hardcoded CIDR
blocks, and adding a region meant a day of careful arithmetic and a code review
nobody enjoyed.

After:

```hcl
locals {
  azs = slice(data.aws_availability_zones.available.names, 0, var.az_count)

  subnet_specs = flatten([
    for tier_index, tier in ["public", "app", "data"] : [
      for az_index, az in local.azs : {
        key               = "${tier}-${substr(az, -1, 1)}"
        tier              = tier
        availability_zone = az
        cidr_block        = cidrsubnet(var.vpc_cidr, 8, tier_index * 10 + az_index + 1)
        public            = tier == "public"
      }
    ]
  ])

  subnets = { for s in local.subnet_specs : s.key => s }
}
```

Twelve lines replaced 180. Adding a third AZ is changing `az_count` from 2 to 3.
Moving to a new region is changing `vpc_cidr`.

The nested `for` with `flatten` is the pattern worth stealing: the outer loop
walks the tiers, the inner walks the AZs, and `flatten` turns the resulting
list-of-lists into one flat list. It is exactly `.flatMap()` in JavaScript.

Then `{ for s in ... : s.key => s }` turns that list into a map, because Module
6's `for_each` needs a map.

---

## Common Mistakes Beginners Make

**1. Forgetting to escape `${}` in a template file.**

```bash
console.log(`port ${port}`)     # Terraform tries to interpolate this
console.log(`port $${port}`)    # correct - stays literal
```

**2. Using a `for` expression where you need `for_each`.**

A `for` expression makes a *value*. It cannot make *resources*. Creating many
resources is `count` or `for_each`, which is Module 6.

**3. Indexing into a set.**

```text
Error: Invalid index
```

Sets have no order. Use `tolist()`, or better, restructure to iterate.

**4. Mismatched types in a ternary.**

```hcl
var.enabled ? "yes" : 42     # ERROR
```

Both branches are type-checked, even the unused one.

**5. Overusing `dynamic`.**

Two different rules are clearer written out. Save `dynamic` for genuinely
repeated, genuinely variable configuration.

**6. Reaching for `jsonencode` for everything.**

For IAM policies it is right. For a config file, `templatefile` is usually
clearer.

**7. Not using `terraform console`.**

People write an expression, run `terraform apply`, get an error, edit, apply
again. Ten minutes per attempt. The console gives the answer in two seconds.

**8. Building CIDRs by hand when `cidrsubnet` exists.**

Hand-typed subnet ranges are where overlapping-CIDR errors come from.

---

## Hands-On Lab — Expressions in Practice

**Cost: free.** Security groups, subnets and console work cost nothing.

### Part A: Console drills

```bash
cd ~/terraform-labs/notely
terraform console -var-file=dev.tfvars
```

Work through these. Predict the answer before pressing enter.

```hcl
# 1. Map over a list
[for n in ["alpha", "beta"] : upper(n)]

# 2. Filter
[for n in [1,2,3,4,5,6,7,8] : n if n % 3 == 0]

# 3. Map and filter together
[for n in [1,2,3,4,5,6] : n * n if n > 3]

# 4. List -> map
{for az in ["ap-southeast-2a", "ap-southeast-2b"] : az => substr(az, -1, 1)}

# 5. Map -> map, transforming values
{for k, v in { small = 1, large = 10 } : k => v * 100}

# 6. CIDR arithmetic
[for i in range(4) : cidrsubnet("10.0.0.0/16", 8, i)]

# 7. Nested loop, flattened
flatten([for tier in ["web", "app"] : [for az in ["a", "b"] : "${tier}-${az}"]])

# 8. Build a map from that
{for s in flatten([for t in ["web","app"] : [for az in ["a","b"] : { key = "${t}-${az}", tier = t }]]) : s.key => s}

# 9. Merge
merge({ a = 1, b = 2 }, { b = 99, c = 3 })

# 10. Safe lookup
lookup({ dev = "t3.micro" }, "prod", "t3.small")

# 11. Coalesce
coalesce(null, "", "first real value")

# 12. Try
try(jsondecode("not json"), "fell back")

# 13. Conditional
"prod" == "prod" ? "big" : "small"

# 14. setproduct
setproduct(["a", "b"], ["public", "private"])

# 15. Your own locals
local.public_subnet_cidrs
local.common_tags
```

**Answers to check yourself against:**

```text
1.  ["ALPHA", "BETA"]
2.  [3, 6]
3.  [16, 25, 36]
4.  { "ap-southeast-2a" = "a", "ap-southeast-2b" = "b" }
5.  { "large" = 1000, "small" = 100 }
6.  ["10.0.0.0/24", "10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
7.  ["web-a", "web-b", "app-a", "app-b"]
8.  { "app-a" = {...}, "app-b" = {...}, "web-a" = {...}, "web-b" = {...} }
9.  { "a" = 1, "b" = 99, "c" = 3 }
10. "t3.small"
11. "first real value"
12. "fell back"
13. "big"
14. [["a","public"], ["a","private"], ["b","public"], ["b","private"]]
```

### Part B: Add computed CIDRs

Update `locals.tf` with `azs`, `public_subnet_cidrs`, `app_subnet_cidrs` and
`data_subnet_cidrs` as shown above.

```bash
terraform console -var-file=dev.tfvars <<< 'local.app_subnet_cidrs'
terraform console -var-file=prod.tfvars <<< 'local.app_subnet_cidrs'
```

Two different address plans, one expression.

### Part C: Add the security groups

Create `security.tf` as shown.

```bash
terraform fmt
terraform validate
terraform plan -var-file=dev.tfvars
```

`Plan: 3 to add` — three security groups.

```bash
terraform apply -var-file=dev.tfvars
```

Check the dynamic block worked:

```bash
aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=notely-dev-alb" \
  --query 'SecurityGroups[0].IpPermissions[].{Port:FromPort,Cidr:IpRanges[0].CidrIp}' \
  --output table
```

Two rules, 80 and 443, generated from a list of two numbers.

Now add a third port and see how little work it is:

```bash
terraform plan -var-file=dev.tfvars -var 'alb_ingress_ports=[80,443,8443]'
```

`1 to change` — one new rule, no new code.

### Part D: The user-data template

Create `templates/user-data.sh.tftpl` and the `user_data` local.

```bash
terraform console -var-file=dev.tfvars <<< 'local.user_data'
```

Read the rendered output. Every `${...}` is filled in.

Now break it deliberately. Add an unescaped JavaScript template literal to the
script:

```bash
console.log(`this will break: ${someVar}`)
```

```bash
terraform validate
```

```text
Error: Invalid function argument
... vars map does not contain key "someVar"
```

Fix it with `$${someVar}` and validate again. That error, once experienced, is
never confusing again.

### Part E: Tear down

Everything here is free, so you can leave it for Module 6. Otherwise:

```bash
terraform destroy -var-file=dev.tfvars
```

### What you should have at the end

- Comfort with `for` expressions in both list and map form
- Subnet CIDRs computed, never typed
- Three chained security groups, one built with a `dynamic` block
- A rendered user-data script, and the `$${}` lesson learned the hard way

---

## Summary Table

| Concept | The one-line version |
|---|---|
| **`terraform console`** | The REPL. Use it constantly. |
| **`cond ? a : b`** | Ternary, same as JavaScript. Both branches type-checked. |
| **`[for x in xs : y]`** | `.map()` |
| **`[for x in xs : y if c]`** | `.filter().map()` |
| **`{for x in xs : k => v}`** | Produces a map, not a list |
| **`{for k, v in m : ...}`** | Iterating a map gives you both |
| **`xs[*].id`** | Splat — one attribute from every element |
| **`length`** | `.length` |
| **`merge`** | `{...a, ...b}`; later wins |
| **`flatten`** | `.flat()` |
| **`distinct`** | Remove duplicates |
| **`lookup(m, k, default)`** | Safe map access |
| **`coalesce`** | First non-null, non-empty |
| **`try(a, b)`** | try/catch that returns a fallback |
| **`can(expr)`** | true/false instead of an error |
| **`range(n)`** | `[0, 1, ... n-1]` |
| **`setproduct`** | Every combination of two lists |
| **`cidrsubnet(p, bits, n)`** | Carve a subnet. Never do CIDR maths by hand. |
| **`cidrhost(p, n)`** | The nth address in a block |
| **`jsonencode`** | Value → JSON. The right way to write IAM policies. |
| **`file()`** | Read a file, no substitution |
| **`templatefile()`** | Read a file **and** fill in variables |
| **`$${...}`** | Escape — tells Terraform to leave `${...}` alone |
| **`dynamic "block"`** | Generate repeated nested blocks |
| **`block.value`** | The current item inside a `dynamic` |
| **`for` vs `for_each`** | `for` makes values. `for_each` makes resources (Module 6). |

---

## Checkpoint (answer briefly)

1. Write the Terraform equivalent of `names.filter(n => n.length > 3).map(n => n.toUpperCase())`.
2. What is the difference between `[for x in xs : ...]` and `{for x in xs : ... => ...}`, and when do you need the second one?
3. Your template file contains `` console.log(`port ${port}`) `` and Terraform errors. Why, and what are the two ways to fix it?
4. What does `cidrsubnet("10.0.0.0/16", 8, 11)` return, and why would you use it instead of typing the answer?
5. When should you use a `dynamic` block, and when is a plain repeated block better?
6. `try()` and `coalesce()` both provide fallbacks. What is the difference?
7. Why can't you use a `for` expression to create three EC2 instances?

---

## Checkpoint — model answers

### 1. Filter then map

```hcl
[for n in names : upper(n) if length(n) > 3]
```

Two things to notice.

The **order is different** from JavaScript. HCL puts the transform immediately
after the colon and the condition at the end, even though the filter is applied
first. Read it as: "for each n in names, give me `upper(n)`, but only if
`length(n) > 3`".

There is **no chaining**. JavaScript builds an intermediate array between
`.filter()` and `.map()`. HCL does both in one pass, which is why there is only
one set of brackets.

### 2. List form vs map form

`[for x in xs : ...]` produces a **list**. Square brackets in, list out.

`{for x in xs : key => value}` produces a **map**. Curly braces, and you must
supply both a key and a value separated by `=>`.

```hcl
[for s in subnets : s.cidr_block]
# ["10.0.1.0/24", "10.0.2.0/24"]

{for s in subnets : s.name => s.cidr_block}
# { "public-a" = "10.0.1.0/24", "public-b" = "10.0.2.0/24" }
```

**You need the map form for `for_each`** (Module 6). `for_each` requires a map or
a set, and it uses the keys as resource addresses. A list will not do, because
list positions shift when you remove an element and every resource after it gets
recreated.

The map form is also useful whenever you want to look something up by name rather
than by position.

One rule: map keys must be unique. If two items produce the same key, Terraform
errors — which is usually a real bug you wanted to know about.

### 3. The template escaping error

**Why:** Terraform processes the entire template file before anything else sees
it. It finds `${port}`, treats it as an interpolation, looks for `port` in the
variables map you passed to `templatefile()`, does not find it, and errors:

```text
Error: Invalid function argument
vars map does not contain key "port"
```

It does not know or care that this is JavaScript. Any `${...}` in the file is
Terraform's.

**Fix 1 — escape it.** `$${` tells Terraform to emit a literal `${`:

```bash
console.log(`port $${port}`)
```

The rendered file contains `${port}`, and Node interprets it at runtime as you
intended.

**Fix 2 — pass it in.** If the value is actually known to Terraform, just supply
it:

```hcl
templatefile("...", { port = var.app_port })
```

Then `${port}` resolves at render time and the script contains the literal number.

Which to choose depends on when you want the value decided: escape it if it is
runtime JavaScript, pass it in if Terraform knows it now.

(The same escape exists for the `%{...}` directive syntax: write `%%{` to make it
literal.)

### 4. `cidrsubnet("10.0.0.0/16", 8, 11)`

It returns **`"10.0.11.0/24"`**.

Reading the arguments: start from `10.0.0.0/16`, add `8` bits to the prefix
length so `/16` becomes `/24`, and return subnet number `11` from that series.

Why use it instead of typing `"10.0.11.0/24"`:

**It follows the VPC.** Change `var.vpc_cidr` from `10.0.0.0/16` to
`10.1.0.0/16` and the function returns `10.1.11.0/24` automatically. A hardcoded
string would silently point at the wrong network — or fail with a "CIDR not
inside VPC" error, which is the better outcome of the two.

**It cannot overlap.** Different indices always produce non-overlapping blocks.
Hand-typed ranges are exactly where overlapping-CIDR errors come from.

**It scales.** `[for i in range(6) : cidrsubnet(var.vpc_cidr, 8, i)]` gives six
correct subnets. Six hand-typed strings gives six chances to make a typo.

### 5. `dynamic` vs a plain block

Use `dynamic` when:

- The **number** of blocks is genuinely variable — driven by a variable, a
  lookup, or a computed list
- The blocks are **structurally identical** and only their values differ
- You have more than roughly three of them

Use plain repeated blocks when:

- There are only two or three
- They mean **different things** — "HTTPS from the internet" and "app traffic
  from the load balancer only" are different policies, not two instances of one
  policy
- Someone reading the file needs to see the actual rules at a glance

The trade is readability against repetition. A `dynamic` block hides its contents
behind a level of indirection: to know what rules exist you have to go and read a
variable's default, possibly in another file. That is a real cost, and it is only
worth paying when the repetition is worse.

A good heuristic: if you cannot answer "what ports are open?" by looking at the
resource, think twice.

### 6. `try()` vs `coalesce()`

They handle different failures.

**`try(a, b)`** catches **errors**. It evaluates each argument in turn and
returns the first that does not blow up.

```hcl
try(var.config.optional_field, "default")   # the field may not exist at all
try(jsondecode(var.raw), {})                # the string may not be valid JSON
```

It is a try/catch.

**`coalesce(a, b)`** catches **null and empty values**. Every argument must be
valid — if one errors, `coalesce` errors too. It returns the first that is
neither `null` nor `""`.

```hcl
coalesce(var.custom_name, local.generated_name)   # both exist; use the first that is set
```

It is the `??` operator.

The distinction in one line: `try` is for "this might not work", `coalesce` is
for "this might not be set". Reaching for `coalesce` when the expression could
error will not help you, and reaching for `try` to handle nulls will silently
return the null, because returning null is not an error.

### 7. Why a `for` expression cannot create instances

Because a `for` expression produces a **value**, and resources are not values.

```hcl
locals {
  names = [for i in range(3) : "server-${i}"]   # a list of three strings
}
```

That gives you three strings. It does not give you three EC2 instances — there is
nowhere for a `resource` block to go inside a `for` expression, and HCL has no
syntax for producing blocks from an expression.

Resources are declared, not computed. To declare many of them you use a
**meta-argument** on the resource block itself:

```hcl
resource "aws_instance" "app" {
  for_each = toset(["a", "b", "c"])
  # ...
}
```

The two work together constantly, though: a `for` expression builds the map, and
`for_each` consumes it.

```hcl
locals {
  subnets = { for s in var.subnet_specs : s.name => s }
}

resource "aws_subnet" "this" {
  for_each   = local.subnets
  cidr_block = each.value.cidr_block
}
```

That is exactly what Module 6 does to give Notely its second availability zone.

---

## Next lesson

**Module 6 — Meta-Arguments** (`06-meta-arguments.md`)

You can now compute any value you need. Module 6 uses those values to create
*many resources at once* — `count` and `for_each`, and why choosing wrong causes
one of the nastiest surprises in Terraform.

This is the module where Notely finally grows up: six subnets across two
availability zones, a real load balancer, and two EC2 servers running the Node.js
app you templated in Part 5.
