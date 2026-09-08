# Module 0 — AWS Networking, From Zero

## Why this module exists

This module is not about Terraform. Not one line of HCL appears in it.

It exists because every Terraform tutorial in the world starts like this:

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-southeast-2a"
  map_public_ip_on_launch = true
}
```

...and then moves on, as if you already knew what a subnet is, what
`10.0.1.0/24` means, why it says `/24` and not `/16`, what an availability zone
is, and what "map public IP on launch" does.

If you do not know those things, you are not learning Terraform. You are
copying and pasting and hoping.

So: this module explains all of it. Slowly, in plain words, with pictures.

By the end you will be able to draw Notely's network from memory and explain
every line. Then Module 1 starts, and Terraform will make sense.

> You cannot describe infrastructure in code until you understand the infrastructure. This module is that understanding.

---

## The core idea (one sentence)

A VPC is your own private network inside AWS, and everything else in this module
is about dividing it up and controlling what can talk to what.

---

## The big analogy: an office building

Hold this picture in your head for the whole module. It maps almost perfectly.

| AWS thing | The building | In one line |
|---|---|---|
| **AWS Region** | A city | Sydney, Virginia, Frankfurt |
| **Availability Zone** | A building in that city | There are several; they can burn down independently |
| **VPC** | Your company's floors, across all the buildings | Your own private space |
| **CIDR block** | How many rooms your company leased | Decides how many addresses you get |
| **Subnet** | One floor of one building | A slice of your space, in one AZ |
| **Public subnet** | Ground floor with a door to the street | Traffic can come in from outside |
| **Private subnet** | Fifth floor, no street door | No way in from outside |
| **Internet Gateway** | The street door | The building's connection to the world |
| **NAT Gateway** | A staff-only exit door | People can go out, nobody can come in |
| **Route table** | The signs in the corridors | "Post room this way, exit that way" |
| **Security group** | The lock on each office door | Protects one room |
| **NACL** | The guard at the lift on each floor | Protects the whole floor |
| **Elastic IP** | A permanent street address | Does not change when you move furniture |

Every time something confuses you later, come back to this table.

---

## Part 1 — IP addresses, from nothing

Skip this part if you already know what `10.0.1.5` means. Otherwise, read it —
everything else depends on it.

### What an IP address is

Every device on a network needs an address, so other devices can send it things.
Like a house needs a street address for post to arrive.

An IP address (the common kind, IPv4) looks like this:

```text
10.0.1.5
```

Four numbers, separated by dots. Each number can be **0 to 255**.

That is it. That is the whole format.

Some addresses you have probably seen:

| Address | What it is |
|---|---|
| `127.0.0.1` | Your own machine. "localhost". |
| `192.168.0.1` | Usually your home router |
| `8.8.8.8` | Google's public DNS server |
| `10.0.1.5` | A typical private address inside a company network |

### Why 0 to 255?

Because each of those four numbers is stored in 8 bits. Eight bits can hold 256
different values, and we count from zero, so: 0 to 255.

Four numbers × 8 bits each = **32 bits total**. You will see the number 32
everywhere in networking. That is where it comes from.

You do not need to do binary maths in this course. You just need to know that
an address is 32 bits, arranged as four numbers of 0–255.

### Public vs private addresses

Some address ranges are reserved for **private networks**. They are not routable
on the internet — anyone can use them inside their own network, and they never
clash, because they never leave.

There are three private ranges:

| Range | Addresses in it | Commonly used for |
|---|---|---|
| `10.0.0.0` – `10.255.255.255` | ~16.7 million | Big company networks. **AWS VPCs almost always use this one.** |
| `172.16.0.0` – `172.31.255.255` | ~1 million | Docker uses this by default |
| `192.168.0.0` – `192.168.255.255` | ~65,000 | Home routers |

Everything else is a public address, and those are owned and allocated
globally — you cannot just pick one.

**This is why every Terraform tutorial starts with `10.0.0.0/16`.** It is a
private range, it is huge, and it is the convention.

> Your VPC uses private addresses. That is why nothing inside it is reachable from the internet by default — those addresses do not exist on the internet.

---

## Part 2 — CIDR, or "what does /16 mean?"

This is the part that scares people. It should not.

### The problem CIDR solves

You need to say "I want this block of addresses". Writing out every address is
absurd. So you write the **start** and **how big**.

```text
10.0.0.0/16
   ^      ^
   |      +--- how big (sort of)
   +---------- where it starts
```

### The rule

The number after the slash says: **how many bits are fixed**. The rest are free
to vary.

Remember there are 32 bits in total. So:

```text
/16  →  16 bits fixed, 16 bits free
/24  →  24 bits fixed,  8 bits free
/32  →  32 bits fixed,  0 bits free  (exactly one address)
```

Each free bit doubles the number of addresses. So:

| CIDR | Bits free | Addresses | Rough size |
|---|---|---|---|
| `/8` | 24 | 16,777,216 | Enormous |
| `/16` | 16 | 65,536 | A whole VPC |
| `/20` | 12 | 4,096 | A big subnet |
| `/24` | 8 | 256 | A normal subnet |
| `/26` | 6 | 64 | A small subnet |
| `/28` | 4 | 16 | Tiny |
| `/32` | 0 | 1 | One single address |

### The shortcut you actually need

Forget the bits. Learn these three, and you will handle 95% of real work:

**`/16` — a whole VPC.** 65,536 addresses. You can change the second, third and
fourth numbers.

```text
10.0.0.0/16  covers  10.0.0.0  through  10.0.255.255
             ^  ^
             fixed   these two can be anything
```

**`/24` — a normal subnet.** 256 addresses. Only the last number varies.

```text
10.0.1.0/24  covers  10.0.1.0  through  10.0.1.255
             ^ ^ ^
             all fixed        only this one varies
```

**`/32` — exactly one address.** Used in firewall rules to mean "just this one
machine".

```text
203.0.113.45/32  covers  only  203.0.113.45
```

### Examples, so it sticks

| CIDR | First address | Last address | Count |
|---|---|---|---|
| `10.0.0.0/16` | `10.0.0.0` | `10.0.255.255` | 65,536 |
| `10.0.1.0/24` | `10.0.1.0` | `10.0.1.255` | 256 |
| `10.0.2.0/24` | `10.0.2.0` | `10.0.2.255` | 256 |
| `10.0.11.0/24` | `10.0.11.0` | `10.0.11.255` | 256 |
| `172.16.0.0/12` | `172.16.0.0` | `172.31.255.255` | 1,048,576 |
| `0.0.0.0/0` | `0.0.0.0` | `255.255.255.255` | **everything** |

### The one you must recognise instantly

```text
0.0.0.0/0
```

Zero bits fixed. Everything varies. This means **"any address anywhere"** — in
practice, "the entire internet".

You will see it in two places, and they mean very different things:

```hcl
# In a route table: "for anywhere else, go to the internet gateway"
route {
  cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.main.id
}

# In a security group: "let ANYONE in the world connect"
ingress {
  from_port   = 22
  to_port     = 22
  cidr_blocks = ["0.0.0.0/0"]   # <-- SSH open to the entire internet. Bad.
}
```

> `0.0.0.0/0` in a route table is normal and correct. `0.0.0.0/0` on an SSH port in a security group is how servers get hacked.

### AWS takes five addresses from every subnet

A small gotcha that trips people up. In a `/24` subnet you get 256 addresses, but
only **251** are usable. AWS reserves five:

| Address in `10.0.1.0/24` | Reserved for |
|---|---|
| `10.0.1.0` | Network address (always reserved, everywhere) |
| `10.0.1.1` | The VPC router |
| `10.0.1.2` | AWS DNS |
| `10.0.1.3` | Reserved for future use |
| `10.0.1.255` | Broadcast address (always reserved) |

This only matters when you make subnets very small. A `/28` gives you 16
addresses, minus 5, leaves **11 usable**. That runs out faster than you expect.

> Do not make subnets smaller than `/24` unless you have counted. Addresses are free; running out is not.

---

## Part 3 — Regions and Availability Zones

### Region = a city

AWS has data centres all over the world, grouped into **regions**.

| Region code | Where |
|---|---|
| `ap-southeast-2` | Sydney |
| `us-east-1` | Northern Virginia |
| `eu-west-1` | Ireland |
| `ap-south-1` | Mumbai |

You pick one. Everything you build lives there unless you say otherwise.

Pick the one closest to your users. Every kilometre is latency.

### Availability Zone = a building in that city

Each region is made of several **Availability Zones**, usually three.

```text
Region: ap-southeast-2  (Sydney)
   |
   +-- ap-southeast-2a   <- separate data centre
   +-- ap-southeast-2b   <- separate data centre, kilometres away
   +-- ap-southeast-2c   <- separate data centre, kilometres away
```

They are physically separate. Different buildings, different power, different
cooling, different flood risk. Connected to each other by very fast private
fibre, so talking between them is quick.

### Why you care

**AZs fail.** Not often, but they do. A power event, a cooling failure, a fibre
cut, a fire.

If everything you own is in `ap-southeast-2a` and that AZ goes down, you are
down. If you have one server in `2a` and one in `2b`, you stay up.

That is the entire reason Notely has two of everything.

```text
BAD                                GOOD

 AZ a          AZ b                 AZ a          AZ b
+------+      +------+             +------+      +------+
|server|      |      |             |server|      |server|
|server|      |      |             |  DB  |      |  DB  |
|  DB  |      |      |             |      |      |standby
+------+      +------+             +------+      +------+

AZ a dies = you are down        AZ a dies = you are fine
```

> One AZ is a demo. Two AZs is a system. This is the cheapest reliability you will ever buy.

---

## Part 4 — The VPC

**VPC** stands for Virtual Private Cloud. Ignore the name; it tells you nothing.

### What it actually is

A VPC is **your own private network inside AWS**.

Before VPCs existed, everyone's servers sat on a shared network and you hoped
for the best. A VPC gives you a walled-off space that only you can see into.

Two facts that define it:

1. **It is regional.** A VPC spans all the AZs in one region. One VPC, several
   buildings.
2. **It has a CIDR block.** You choose it when you create the VPC, and it is the
   pool of addresses everything inside will use.

```text
VPC: 10.0.0.0/16   in region ap-southeast-2

+---------------------------------------------------+
|                                                   |
|   AZ 2a          AZ 2b          AZ 2c             |
|   ------         ------         ------            |
|                                                   |
|   (empty until you add subnets)                   |
|                                                   |
+---------------------------------------------------+
```

A VPC on its own does nothing. It is an empty plot of land with a fence round it.
You cannot put a server "in a VPC" — you put it in a **subnet**, and subnets live
inside the VPC.

### Choosing the CIDR

Almost everyone uses `10.0.0.0/16`.

- It is private, so it never clashes with the internet
- 65,536 addresses is plenty
- If you later connect two VPCs together, you want them not to overlap — so a
  second one becomes `10.1.0.0/16`, a third `10.2.0.0/16`

> You cannot change a VPC's main CIDR block after creating it. Pick `/16` and stop worrying.

---

## Part 5 — Subnets

### What a subnet is

A subnet is a **slice of your VPC's addresses, pinned to one AZ**.

Two things define it:

1. A CIDR block that is **inside** the VPC's CIDR
2. Exactly **one** availability zone

```text
VPC 10.0.0.0/16
+---------------------------------------------------+
|                                                   |
|  AZ 2a                    AZ 2b                   |
|  +------------------+     +------------------+    |
|  | subnet           |     | subnet           |    |
|  | 10.0.1.0/24      |     | 10.0.2.0/24      |    |
|  | 256 addresses    |     | 256 addresses    |    |
|  +------------------+     +------------------+    |
|                                                   |
+---------------------------------------------------+
```

Note: `10.0.1.0/24` and `10.0.2.0/24` are both inside `10.0.0.0/16`, and they do
not overlap. Both rules are mandatory.

### A subnet cannot span two AZs

This is the rule people forget. One subnet = one AZ. Always.

So if you want servers in two AZs, you need **two subnets**. If you want public
and private in two AZs, you need **four subnets**.

That is why Notely has six subnets — three tiers × two AZs.

### Public and private subnets

Here is the thing that surprises everyone:

> There is no "public subnet" checkbox. A subnet is public **only** because of what its route table says.

That is it. That is the whole difference.

| | Public subnet | Private subnet |
|---|---|---|
| **Route table has a route to an internet gateway?** | Yes | No |
| **Can things inside reach the internet?** | Yes | Not directly |
| **Can the internet reach things inside?** | Yes, if they have a public IP | No |
| **What goes here** | Load balancers, NAT gateways, bastion hosts | App servers, databases, caches |

We will come back to this once route tables are explained. For now, hold onto:
**public vs private is a routing decision, not a setting on the subnet.**

### Naming your subnet CIDRs

There is no rule, but this convention is common and worth copying:

| Purpose | AZ a | AZ b | Pattern |
|---|---|---|---|
| Public | `10.0.1.0/24` | `10.0.2.0/24` | 1–10 |
| Private app | `10.0.11.0/24` | `10.0.12.0/24` | 11–20 |
| Private data | `10.0.21.0/24` | `10.0.22.0/24` | 21–30 |

You can read the tier straight off the address. `10.0.12.5` is an app server in
AZ b. That is genuinely useful at 3am.

---

## Part 6 — The Internet Gateway

An **Internet Gateway** (IGW) is the door between your VPC and the internet.

Facts:

- **One per VPC.** You do not need more.
- **It costs nothing.** Free. Always attach one.
- **It does not do anything by itself.** Attaching it changes nothing until a
  route table points at it.
- It handles translation between private and public addresses automatically.

```text
                    Internet
                        |
                        |
              +------------------+
              | Internet Gateway |     <- attached to the VPC
              +------------------+
                        |
    +---------------------------------------+
    |            VPC 10.0.0.0/16            |
    |                                       |
    +---------------------------------------+
```

That picture is misleading in one way, and the way matters: attaching the IGW
does **not** connect your subnets to the internet. It just puts a door in the
wall. Until a route table says "go through that door", nothing uses it.

---

## Part 7 — Route tables (this is the important one)

If you only properly understand one thing in this module, make it this.

### What a route table is

A route table is a list of rules that says: **"traffic for this destination goes
that way."**

Every subnet is associated with exactly one route table. When something in that
subnet sends a packet, AWS looks at the table to decide where to send it.

That is all it is. A list of signposts.

### Reading one

Here is the route table of a **public** subnet:

| Destination | Target | Means |
|---|---|---|
| `10.0.0.0/16` | `local` | "Anything inside our own VPC — deliver it directly" |
| `0.0.0.0/0` | `igw-abc123` | "Anything else — send it to the internet gateway" |

And a **private** subnet:

| Destination | Target | Means |
|---|---|---|
| `10.0.0.0/16` | `local` | "Anything inside our own VPC — deliver it directly" |

That is the entire difference between public and private. **One row.**

The private table has no route to the internet gateway, so traffic for the
internet has nowhere to go, and it is dropped.

### The `local` route

Every route table automatically gets a `local` route covering the VPC's CIDR.
You cannot remove it, and you should not want to. It is what lets your servers
talk to your database.

### Most specific wins

If two routes could match, the more specific one (bigger slash number) wins.

| Destination | Target |
|---|---|
| `0.0.0.0/0` | internet gateway |
| `10.0.0.0/16` | local |

Traffic to `10.0.5.20` matches both — but `/16` is more specific than `/0`, so it
goes `local`. Traffic to `142.250.70.78` (Google) only matches `/0`, so it goes
out the internet gateway.

You will rarely think about this. It just works the way you would want.

### Three worked examples

Take a server sitting in the **public** subnet `10.0.1.0/24`.

**Example 1 — it talks to the database at `10.0.21.15`.**
Look up `10.0.21.15`. Matches `10.0.0.0/16` → `local`. Stays inside the VPC.
Never touches the internet. Fast and free.

**Example 2 — it calls `https://api.stripe.com`.**
That resolves to a public address. No `local` match. Falls to `0.0.0.0/0` →
internet gateway. Goes out to the internet. Works.

**Example 3 — the same server, but now in the private subnet `10.0.11.0/24`.**
It calls `https://api.stripe.com` again. No `local` match. And there is no
`0.0.0.0/0` row at all. **The packet is dropped.** The call times out.

That third example is the single most common "why doesn't my server have
internet?" question in all of AWS. The answer is always the route table.

> A subnet is public because its route table has `0.0.0.0/0 → internet gateway`. Nothing else makes it public.

---

## Part 8 — The NAT Gateway

### The problem it solves

Your app servers are in a private subnet. Good — the internet cannot reach them.

But they need to reach *out*. To install packages with `npm install`. To call a
payment API. To download OS security updates.

Right now they cannot, because there is no `0.0.0.0/0` route.

You cannot just add one pointing at the internet gateway, because then they
would be reachable *from* the internet too, and you have thrown away the whole
point of a private subnet.

### What a NAT Gateway does

A **NAT Gateway** allows outbound connections but not inbound ones.

Think of it as a one-way valve. Your server can start a conversation with the
outside world; the outside world cannot start a conversation with your server.

It lives in a **public** subnet (it needs internet access itself), and private
subnets route to it.

```text
      Internet
          |
   Internet Gateway
          |
   PUBLIC subnet  10.0.1.0/24
   +-------------------+
   |   NAT Gateway     |
   +-------------------+
          ^
          |   route:  0.0.0.0/0  ->  nat-abc123
          |
   PRIVATE subnet  10.0.11.0/24
   +-------------------+
   |   EC2 server      |    can call out, cannot be called
   +-------------------+
```

The private route table now looks like:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `nat-abc123` |

### The cost warning

**A NAT Gateway costs about $0.045 per hour, plus data charges.**

That is roughly **$32 per month**, per NAT gateway, whether you use it or not.
And doing it properly means one per AZ, so two AZs means ~$64/month.

This is the number one accidental Terraform bill. Someone runs a tutorial, gets
distracted, and finds a $32 charge next month.

> **We never leave a NAT Gateway running in this course.** You will see the code. You will not apply it.

### The free alternative

If your private servers only need to reach **AWS services** (S3, DynamoDB), you
do not need a NAT gateway at all. Use a **VPC gateway endpoint** — a private
doorway straight to that service, which never touches the internet.

| | NAT Gateway | S3 Gateway Endpoint |
|---|---|---|
| **Cost** | ~$32/month + data | **Free** |
| **Reaches** | Anything on the internet | Only S3 (or DynamoDB) |
| **Good for** | `npm install`, third-party APIs | Reading and writing your own buckets |

Notely's servers need to reach S3 for attachments, so a gateway endpoint covers
the important case for free. For real production you would add a NAT gateway too;
for learning, we do not.

---

## Part 9 — Security groups

Route tables decide **where traffic can go**. Security groups decide **what is
allowed to connect**.

### What a security group is

A firewall that wraps around a single resource — a server, a database, a load
balancer.

Key facts:

| Fact | Meaning |
|---|---|
| **Default deny** | Nothing is allowed unless a rule allows it |
| **Allow rules only** | You cannot write a "deny" rule |
| **Stateful** | If you allow traffic in, the reply is automatically allowed out |
| **Attached to resources** | Not to subnets |
| **Multiple allowed** | A server can have several; the rules add up |

### Stateful — why it matters

This saves you from a whole class of confusion.

You allow inbound port 443 to your load balancer. Someone loads the page. The
load balancer needs to send a response back on a random high port.

You did **not** write a rule for that outbound response. It works anyway,
because security groups are stateful: they remember the connection and allow the
reply automatically.

> Write rules for connections being *started*. The replies take care of themselves.

### Reading a security group

Notely's load balancer:

| Direction | Port | Source / Destination | Why |
|---|---|---|---|
| Inbound | 443 | `0.0.0.0/0` | Anyone on the internet can load the site over HTTPS |
| Inbound | 80 | `0.0.0.0/0` | So we can redirect them to HTTPS |
| Outbound | 3000 | the app servers' security group | Forward requests to the app |

Notely's app servers:

| Direction | Port | Source / Destination | Why |
|---|---|---|---|
| Inbound | 3000 | **the load balancer's security group** | Only the load balancer may talk to the app |
| Outbound | 5432 | the database's security group | Talk to Postgres |
| Outbound | 443 | `0.0.0.0/0` | Call AWS APIs and third parties |

Notely's database:

| Direction | Port | Source / Destination | Why |
|---|---|---|---|
| Inbound | 5432 | **the app servers' security group** | Only the app may talk to the database |

### The best trick: groups referencing groups

Look at those tables again. The app's inbound rule does not say an IP address. It
says **the load balancer's security group**.

This is the single most useful thing about security groups.

```text
  ALB security group
        |
        |  "allow inbound 3000 from sg-alb"
        v
  App security group
        |
        |  "allow inbound 5432 from sg-app"
        v
  DB security group
```

Why this is so much better than using IP addresses:

- Servers come and go. Auto-scaling replaces them. Their IPs change constantly.
  A rule that names a security group never needs updating.
- It reads like a sentence: *"the database accepts connections from the app
  servers."* That is exactly the policy you wanted to express.
- Nothing else can connect, even if it is in the same subnet with a similar IP.

> Reference security groups, not IP ranges, whenever both ends are inside your VPC.

### A worked chain

Someone loads a note in Notely:

1. Browser → load balancer on port 443. Allowed: the ALB's group allows 443 from
   anywhere.
2. Load balancer → app server on 3000. Allowed: the app's group allows 3000 from
   the ALB's group.
3. App server → database on 5432. Allowed: the DB's group allows 5432 from the
   app's group.
4. Responses travel back. All allowed automatically, because stateful.

Now try to break in. You are on the internet and you know the app server's
private address:

- You cannot reach it. Its subnet is private — there is no route from the
  internet to it.
- Even if you were inside the VPC, its security group only accepts port 3000
  from the ALB's group. You are not in that group.

Two independent layers had to fail for you to get in. That is the point.

---

## Part 10 — NACLs (and why you can mostly ignore them)

A **Network ACL** is a second firewall, at the subnet level.

| | Security group | NACL |
|---|---|---|
| **Wraps** | A resource | A whole subnet |
| **Rules** | Allow only | Allow **and** deny |
| **Stateful?** | Yes | **No** |
| **Rule order** | All evaluated | Numbered; first match wins |
| **Default** | Deny everything | Allow everything |

**Stateless** is the painful bit. Because a NACL does not remember connections,
you must write a rule for the reply too — including for the random high ports
responses come back on (1024–65535, the "ephemeral" range). Forget that, and
things break in ways that are miserable to debug.

### Do you need them?

Usually no. The default NACL allows everything, which effectively means security
groups are doing all the work. That is fine and normal.

Use a NACL when you need to **block** something specific — a security group
cannot express "deny", so blocking one abusive IP range is a genuine NACL job.

> Use security groups for everything. Reach for a NACL only when you specifically need to deny something.

---

## Part 11 — The whole path, one request at a time

Everything above, assembled. Someone in a café opens `notely.example.com` and
loads their notes.

```text
 1. Browser asks DNS: "where is notely.example.com?"
        |
        v
 2. Route 53 answers with the load balancer's public address
        |
        v
 3. Browser opens an HTTPS connection to that address, port 443
        |
        v
 4. Traffic arrives at the Internet Gateway
        |     the IGW is attached to the VPC
        v
 5. Routed to the PUBLIC subnet where the load balancer lives
        |     public subnet: route table has 0.0.0.0/0 -> igw
        v
 6. ALB security group check: inbound 443 from 0.0.0.0/0?  ALLOWED
        |
        v
 7. Load balancer picks a healthy app server and forwards to port 3000
        |
        v
 8. App server is in a PRIVATE subnet
        |     traffic goes VPC-internal: matches the 10.0.0.0/16 local route
        v
 9. App security group check: inbound 3000 from the ALB's group?  ALLOWED
        |
        v
10. Node.js app needs the user's notes, connects to Postgres on 5432
        |     again VPC-internal, local route
        v
11. DB security group check: inbound 5432 from the app's group?  ALLOWED
        |
        v
12. Database returns rows
        |     stateful: the reply is allowed automatically
        v
13. App needs an attachment from S3
        |     goes via the S3 gateway endpoint - never touches the internet
        v
14. App builds the JSON response, sends it to the load balancer
        |     stateful: allowed automatically
        v
15. Load balancer sends it out through the Internet Gateway
        |     stateful: allowed automatically
        v
16. Browser renders the notes
```

Sixteen steps. Every single one is something you now understand.

Read it again in a week. If any step is fuzzy, the section explaining it is
above.

---

## Part 12 — Notely's network, fully drawn

This is the picture we build in code, starting in Module 2.

```text
REGION  ap-southeast-2
VPC     10.0.0.0/16                                 65,536 addresses

+-------------------------------------------------------------------+
|                                                                   |
|   AZ ap-southeast-2a                AZ ap-southeast-2b            |
|   ==================                ==================            |
|                                                                   |
|   PUBLIC  10.0.1.0/24               PUBLIC  10.0.2.0/24           |
|   +----------------------+          +----------------------+      |
|   |  Load balancer node  |          |  Load balancer node  |      |
|   +----------------------+          +----------------------+      |
|   route: 0.0.0.0/0 -> igw           route: 0.0.0.0/0 -> igw       |
|              |                                 |                  |
|              v                                 v                  |
|   PRIVATE APP  10.0.11.0/24         PRIVATE APP  10.0.12.0/24     |
|   +----------------------+          +----------------------+      |
|   |  EC2  Node.js API    |          |  EC2  Node.js API    |      |
|   |  port 3000           |          |  port 3000           |      |
|   +----------------------+          +----------------------+      |
|   route: local only                 route: local only             |
|              |                                 |                  |
|              v                                 v                  |
|   PRIVATE DATA  10.0.21.0/24        PRIVATE DATA  10.0.22.0/24    |
|   +----------------------+          +----------------------+      |
|   |  RDS Postgres        |          |  RDS standby         |      |
|   |  port 5432           |          |                      |      |
|   +----------------------+          +----------------------+      |
|   route: local only                 route: local only             |
|                                                                   |
|                        S3 Gateway Endpoint  (free)                |
|                                                                   |
+-------------------------------------------------------------------+
                              |
                    +--------------------+
                    | Internet Gateway   |
                    +--------------------+
                              |
                          Internet
```

### The inventory

| Thing | How many | Notes |
|---|---|---|
| VPC | 1 | `10.0.0.0/16` |
| Availability zones | 2 | `2a` and `2b` |
| Public subnets | 2 | one per AZ |
| Private app subnets | 2 | one per AZ |
| Private data subnets | 2 | one per AZ |
| Internet gateway | 1 | free |
| Public route table | 1 | shared by both public subnets |
| Private route table | 1 | shared by all private subnets |
| S3 gateway endpoint | 1 | free, so servers reach S3 without a NAT |
| Security groups | 3 | ALB, app, database |
| NAT gateways | **0** | explained but never applied — $32/month each |

### Why one route table for both public subnets?

Because they need the same rules. A route table can be associated with many
subnets. Both public subnets want `0.0.0.0/0 → igw`, so one table serves both.

Same for the private ones. Two route tables total, six subnets.

(In real production with NAT gateways you would need one private route table per
AZ, because each AZ's traffic should go to its own AZ's NAT gateway. We do not
have NAT gateways, so one table is fine.)

---

## Common Mistakes Beginners Make

**1. Thinking "public subnet" is a setting.**
It is not. It is whether the route table has a `0.0.0.0/0` route to an internet
gateway. That single row is the whole difference.

**2. Overlapping subnet CIDRs.**
`10.0.1.0/24` and `10.0.1.128/25` overlap. AWS rejects it. Keep a list of which
ranges you have used.

**3. Making subnets too small.**
`/28` gives you 16 addresses, and AWS takes 5. Eleven usable. A load balancer
alone wants several. Use `/24` and stop thinking about it.

**4. Putting the database in a public subnet.**
It will work. It will also be scanned by bots within hours. Databases go in
private subnets, always.

**5. Opening SSH to `0.0.0.0/0`.**
```text
inbound  22  from  0.0.0.0/0     <- the whole internet can try to log in
```
Restrict to your office IP, or better, use AWS Systems Manager Session Manager
and open no SSH port at all.

**6. Leaving a NAT Gateway running.**
$32/month. The most common accidental AWS charge there is.

**7. Using IP addresses in security groups instead of group references.**
Servers get replaced and their IPs change. Reference the security group.

**8. Forgetting a subnet lives in exactly one AZ.**
Want two AZs? You need two subnets. Every time.

**9. Expecting an internet gateway to do something on its own.**
Attaching it changes nothing. A route table has to point at it.

**10. Not knowing why a private server has no internet.**
It has no `0.0.0.0/0` route. That is by design. Add a NAT gateway (costs money)
or a VPC endpoint (free, AWS services only).

---

## Summary Table

| Term | In one line |
|---|---|
| **IP address** | Four numbers 0–255. A device's address. |
| **Private ranges** | `10.x`, `172.16–31.x`, `192.168.x`. Not routable on the internet. |
| **CIDR** | `10.0.0.0/16` — where a block starts and how big it is |
| **`/16`** | 65,536 addresses. A whole VPC. |
| **`/24`** | 256 addresses. A normal subnet. |
| **`/32`** | Exactly one address. |
| **`0.0.0.0/0`** | Everything. The internet. |
| **Region** | A city. `ap-southeast-2` is Sydney. |
| **Availability Zone** | A physically separate data centre in that city |
| **VPC** | Your own private network. Regional. Has a CIDR. |
| **Subnet** | A slice of the VPC, in exactly one AZ |
| **Public subnet** | Its route table has `0.0.0.0/0 → internet gateway` |
| **Private subnet** | It does not |
| **Internet Gateway** | The door to the internet. One per VPC. Free. |
| **Route table** | The signposts: "for this destination, go there" |
| **`local` route** | Automatic. Lets everything in the VPC talk to everything else. |
| **NAT Gateway** | Outbound-only door for private subnets. **~$32/month.** |
| **VPC gateway endpoint** | Free private path to S3 or DynamoDB |
| **Security group** | Firewall on a resource. Stateful. Allow-only. |
| **Stateful** | Replies to allowed traffic are allowed automatically |
| **SG referencing an SG** | "Allow the app servers", not "allow 10.0.11.42" |
| **NACL** | Subnet firewall. Stateless. Can deny. Usually leave alone. |

---

## Checkpoint (answer briefly)

1. What actually makes a subnet "public"?
2. How many addresses are in `10.0.1.0/24`, and how many can you use?
3. Why does Notely have two of everything?
4. Your app server in a private subnet cannot run `npm install`. Why, and what are your two options?
5. What does `0.0.0.0/0` mean, and why is it fine in a route table but dangerous on port 22 of a security group?
6. Why is it better for the database's security group to allow port 5432 from the app's security group, rather than from `10.0.11.0/24`?
7. Security groups are "stateful". What does that save you from writing?

---

## Checkpoint — model answers

### 1. What makes a subnet public?

**Its route table has a route sending `0.0.0.0/0` to an internet gateway.**

That is the whole difference. There is no checkbox, no property, no flag on the
subnet itself called "public".

A public subnet's route table:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-abc123` |

A private subnet's route table:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |

One row. Delete it and the subnet is private. Add it and the subnet is public.

(There is a related setting, `map_public_ip_on_launch`, which decides whether
instances launched there automatically get a public IP. It is a convenience, not
the definition — a subnet with that on but no internet gateway route is still
private in every way that matters.)

### 2. Addresses in `10.0.1.0/24`

**256 total, 251 usable.**

`/24` means 24 bits fixed, 8 free. Eight bits gives 256 combinations, so the
range is `10.0.1.0` through `10.0.1.255`.

AWS reserves five of them in every subnet:

- `10.0.1.0` — network address
- `10.0.1.1` — the VPC router
- `10.0.1.2` — AWS DNS
- `10.0.1.3` — reserved for future use
- `10.0.1.255` — broadcast

256 − 5 = **251 usable**.

This only bites on small subnets. A `/28` has 16 addresses and gives you 11,
which runs out faster than you would think.

### 3. Why two of everything?

**Because an Availability Zone can fail.**

An AZ is a physically separate group of data centres — its own power, cooling and
network. AWS designs them to fail independently, and occasionally they do.

If Notely ran one server in `ap-southeast-2a` and that AZ lost power, Notely
would be down. With one server in `2a` and one in `2b`, the load balancer notices
the dead one, stops sending it traffic, and everything keeps working on the
survivor.

The same logic applies to the database, which is why RDS Multi-AZ keeps a standby
in the second AZ and fails over automatically.

It is the cheapest reliability available: the same number of servers you would
have run anyway, just placed in two buildings instead of one.

### 4. Private server cannot `npm install`

**Why:** its route table has no `0.0.0.0/0` route. Traffic aimed at the internet
has nowhere to go, so it is dropped and the command hangs, then times out.

That is not a bug. It is the definition of a private subnet.

**Option 1 — NAT Gateway.** Put one in a public subnet, and point the private
route table's `0.0.0.0/0` at it. Servers can then start outbound connections
while remaining unreachable from outside. Costs about **$32/month** per gateway,
so this is the expensive option.

**Option 2 — VPC endpoint.** If the server only needs AWS services, a gateway
endpoint gives it a private path to S3 or DynamoDB for **free**. No internet
involved at all.

The catch: an endpoint only reaches the one service it is for. `npm install`
talks to the public npm registry, so an endpoint will not help there — you would
need the NAT gateway, or bake the dependencies into the machine image before
launch, which is what production systems usually do.

### 5. `0.0.0.0/0`

It means **every address that exists** — zero bits fixed, everything varies. In
practice, "anywhere on the internet".

**In a route table it is normal and correct.** It is the catch-all rule: "for any
destination I don't have a more specific route for, send it to the internet
gateway." Without it a public subnet would not be public. It is a statement about
*where traffic goes*.

**On port 22 of a security group it is dangerous.** There it means "any machine
on the internet may attempt to open an SSH connection to this server". It is a
statement about *who may connect to you*. Bots scan the entire IPv4 internet for
open port 22 continuously and start guessing credentials within minutes.

Same notation, opposite kind of statement: one is about your outbound path, the
other is about your inbound door.

For SSH, restrict to a known IP range, or use Session Manager and leave port 22
closed entirely.

### 6. Referencing a security group instead of a CIDR

Three reasons, and the first is the practical one.

**Addresses change.** Auto-scaling replaces servers constantly, and each new one
gets a different private IP. A rule naming a security group keeps working; a rule
naming an IP does not. Even a rule naming the whole subnet `10.0.11.0/24` keeps
working, but see the next point.

**It is more precise.** `10.0.11.0/24` allows *anything* in that subnet to reach
the database — including something you put there later for an unrelated reason.
Referencing the app's security group allows only resources you have explicitly
placed in that group.

**It documents intent.** `allow 5432 from sg-notely-app` reads as "the database
accepts connections from the app servers", which is the actual policy. `allow
5432 from 10.0.11.0/24` reads as a fact about addressing, and a reader has to go
and find out what lives there.

Rule of thumb: if both ends are inside your VPC, reference the security group.
Use CIDR blocks for things outside it — your office IP, a partner's network.

### 7. Stateful

**It saves you from writing rules for return traffic.**

When a security group allows a connection in, it remembers that connection and
automatically allows the response back out — and vice versa. You only ever write
rules for the direction a connection is *started* in.

Concretely: you allow inbound 443 on the load balancer. The response goes back to
the browser on some random high port, typically in the 1024–65535 range. You
never wrote an outbound rule for that, and you do not need one.

NACLs are **stateless**, which is exactly why they are annoying. There you must
add an explicit rule allowing that whole ephemeral port range back out, and
forgetting it produces failures that look like a broken application rather than a
firewall problem.

This is the main reason the advice is "use security groups, leave NACLs alone".

---

## Next lesson

**Module 1 — IaC & Terraform Basics** (`01-iac-and-terraform-basics.md`)

You now understand the network Notely will run on. Module 1 steps back to ask
what Infrastructure as Code is and why Terraform exists, then has you create and
destroy your first real AWS resource — Notely's S3 bucket for file attachments.

From Module 2 onwards, everything you drew in this module gets built in code.

> Keep this module open in a tab. Every time a later module says "subnet", "route table" or "security group", the explanation is here.
