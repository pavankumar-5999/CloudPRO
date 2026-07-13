## What is EC2?

EC2 = **Elastic Compute Cloud** = renting virtual machines on AWS.

It falls under **Infrastructure as a Service (IaaS)** — meaning AWS gives you the raw compute and you manage everything on top of it.

EC2 is probably the most important service in AWS — almost everything else connects to it in some way.

---

## What Can EC2 Do?

| Capability | AWS Service |
|------------|-------------|
| Rent virtual machines | EC2 instances |
| Store data on virtual drives | EBS (Elastic Block Store) |
| Distribute load across machines | ELB (Elastic Load Balancer) |
| Scale services up and down | ASG (Auto Scaling Group) |

---

## Setting Up an EC2 Instance — What You Choose

When launching an EC2 instance you decide:

- **Operating System** — Linux, Windows or macOS
- **Compute power** — how many vCPUs
- **RAM** — how much memory
- **Storage** — EBS (network storage) or instance store (physical)
- **Network** — which VPC, subnet, public IP or not
- **Security** — security groups (firewall rules)
- **User Data** — a bootstrap script that runs once at first launch

---

## EC2 User Data (Bootstrap Script)

User Data is a script that runs **once** when the instance first starts — used to automate setup.

```bash
#!/bin/bash
# Example User Data script
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2</h1>" > /var/www/html/index.html
```

**What it can do:**
- Install software
- Download files
- Run updates
- Start services

> ⚠️ Runs as **root user** and only runs **once at first boot** — not on every restart.

---

## EC2 Instance Types

AWS has a naming convention for instances:

```
m5.2xlarge
│ │  │
│ │  └── Size (small, medium, large, xlarge, 2xlarge...)
│ └───── Generation (higher = newer, better)
└─────── Instance class (type of workload)
```

### Instance Categories

| Category | What it's optimised for | Example use case |
|----------|------------------------|------------------|
| **General Purpose** | Balanced CPU, memory, network | Web servers, code repos |
| **Compute Optimised** | High CPU performance | Batch processing, ML, gaming servers |
| **Memory Optimised** | Fast performance for large datasets in RAM | Databases, in-memory cache, real-time analytics |
| **Accelerated Computing** | GPU / hardware accelerators | ML training, video rendering |
| **Storage Optimised** | High sequential read/write on local storage | OLTP databases, data warehousing, Elasticsearch |
| **HPC Optimised** | High performance computing | Complex simulations, scientific modelling |

> 💡 Full list and comparison at [ec2instances.info](https://ec2instances.info)

### Common Instance Examples

| Instance | Category | Memory | Use case |
|----------|----------|--------|----------|
| t2.micro | General Purpose | 1 GB | Free tier, small apps |
| t3.medium | General Purpose | 4 GB | Web apps |
| c5.xlarge | Compute Optimised | 8 GB | CPU-heavy workloads |
| r5.large | Memory Optimised | 16 GB | In-memory DB |
| i3.large | Storage Optimised | 15.25 GB | NoSQL, data warehousing |

> 💡 **t2.micro** is the free tier instance — use this for all your labs.

---

## Security Groups

Security Groups act as a **virtual firewall** for your EC2 instances.

```
Internet
   │
   ▼
Security Group ← firewall rules here
   │
   ▼
EC2 Instance
```

**Key things to know:**

- Security groups contain **only Allow rules** — no Deny rules
- Rules can reference **IP addresses** or **other security groups**
- Controls **inbound** traffic (what comes IN to your instance)
- Controls **outbound** traffic (what goes OUT from your instance)

### Default behaviour

| Traffic direction | Default |
|------------------|---------|
| Inbound | ❌ All blocked by default |
| Outbound | ✅ All allowed by default |

### Security Group Rules

Each rule has:
- **Type** — e.g. SSH, HTTP, HTTPS, Custom TCP
- **Protocol** — TCP, UDP, ICMP
- **Port range** — e.g. 22, 80, 443
- **Source/Destination** — IP range (CIDR) or another security group

### Important rules to remember

| Port | Protocol | Use |
|------|----------|-----|
| 22 | SSH | Connect to Linux instances |
| 21 | FTP | Upload files |
| 22 | SFTP | Secure file upload via SSH |
| 80 | HTTP | Web traffic (not encrypted) |
| 443 | HTTPS | Web traffic (encrypted) |
| 3389 | RDP | Connect to Windows instances |

### Security Group — Key facts

- One security group can be attached to **multiple instances**
- One instance can have **multiple security groups**
- Locked to a **region / VPC combination** — can't use across regions
- Security groups are **stateful** — if inbound is allowed, response is automatically allowed

### Security Group referencing other Security Groups

Instead of allowing a specific IP, you can allow another security group:

```
EC2 A (Security Group 1)
EC2 B (Security Group 1)   ──► All three can talk to
EC2 C (Security Group 2)       your DB instance
         │
         ▼
DB Instance — allows inbound from SG1 and SG2
```

> 💡 This is better than using IPs because IPs change — security group references stay stable.

---

## Troubleshooting — Common Issues

| Problem | Likely cause | Fix |
|---------|-------------|-----|
| Connection **timeout** | Security group blocking the port | Check inbound rules — add the right port |
| Connection **refused** | App not running or wrong port | Security group is fine — check the app |
| SSH works but website doesn't load | Port 80 not open | Add HTTP inbound rule to security group |

> 💡 **Rule of thumb:** Timeout = security group issue. Connection refused = app issue.

---

## Connecting to EC2

Three ways to connect to your EC2 instance:

### 1. SSH (Linux / Mac / Windows 10+)
```bash
ssh -i "your-key.pem" ec2-user@YOUR-PUBLIC-IP
```
- Uses port **22**
- Requires your **.pem key file**
- Key file must have permissions `chmod 400 your-key.pem`

### 2. EC2 Instance Connect (Browser)
- Go to AWS Console → EC2 → select instance → **Connect** → EC2 Instance Connect
- Works directly in the browser — no key file needed
- Only works with **Amazon Linux 2** and **Ubuntu** AMIs

### 3. SSM Session Manager
- No SSH port needed — no key pair needed
- Works through IAM roles
- Best for production (no open port 22)

> ⚠️ **Never** open port 22 to `0.0.0.0/0` (all IPs) in production — big security risk.

---

## EC2 Instance Roles

Instead of putting AWS credentials on an EC2 instance, attach an **IAM Role** to it.

```
❌ Wrong way:
EC2 instance → run "aws configure" → paste access keys → works but DANGEROUS

✅ Right way:
EC2 instance → attach IAM Role with right permissions → works automatically
```

> ⚠️ **Never put your access keys on an EC2 instance** — if someone hacks the instance, they get your keys. Always use IAM Roles.

**How to attach a role:**
1. Create IAM Role with the right policy
2. EC2 → select instance → Actions → Security → **Modify IAM role**
3. Select your role → Save

---

## EC2 Purchasing Options

Different ways to pay for EC2 — big exam topic!

### 1. On-Demand
- Pay per second (Linux) or per hour (Windows)
- No commitment, no upfront cost
- Most expensive per hour
- **Use for:** short workloads, unpredictable usage, testing

### 2. Reserved Instances
- Reserve for **1 or 3 years**
- Up to **72% discount** vs On-Demand
- Pay options: No upfront / Partial upfront / All upfront (all upfront = biggest discount)
- **Use for:** steady-state, predictable workloads e.g. databases

**Convertible Reserved Instances:**
- Can change instance type, OS, tenancy during the term
- Less discount (~66%) but more flexibility

### 3. Savings Plans
- Commit to a certain **$ amount per hour** for 1 or 3 years
- More flexible than Reserved — applies across instance families
- Up to **72% discount**
- **Use for:** flexible but long-term workloads

### 4. Spot Instances
- Up to **90% discount** — cheapest option
- BUT AWS can **terminate your instance at any time** with 2-minute warning
- **Use for:** workloads that can be interrupted — batch jobs, data analysis, image processing
- **NOT for:** databases, critical apps, anything that can't be interrupted

### 5. Dedicated Hosts
- Physical server dedicated entirely to you
- Full control over physical core placement
- Most expensive option
- **Use for:** compliance requirements, Bring Your Own License (BYOL), regulatory needs

### 6. Dedicated Instances
- Your instance runs on hardware that's not shared with other AWS customers
- Less control than Dedicated Hosts
- **Use for:** compliance where you need hardware isolation

### 7. Capacity Reservations
- Reserve On-Demand capacity in a specific AZ for any duration
- No billing discount — you just guarantee capacity is available
- **Use for:** short-term critical workloads that need guaranteed capacity

### Quick comparison

```
Cheapest ──────────────────────────────────► Most Expensive
  Spot → Reserved → Savings Plans → On-Demand → Dedicated Host
  (90%) → (72%)  →    (72%)      →   (100%)  →  (most $$$)
```

---

## Spot Instances — Deep Dive

Spot instances are the cheapest but can be interrupted. Here's how they work:

```
You set a MAX PRICE you're willing to pay
      │
      ▼
If spot price < your max price → instance runs ✅
If spot price > your max price → instance terminated ❌ (2 min warning)
```

**Spot Instance strategies:**

| Strategy | How it works |
|----------|-------------|
| **Spot Instance** | Single instance, terminated if price rises |
| **Spot Fleet** | Mix of Spot + On-Demand instances, maintains target capacity |

**Spot Fleet allocation strategies:**

| Strategy | What it does |
|----------|-------------|
| `lowestPrice` | Launch from the cheapest pool |
| `diversified` | Spread across multiple pools — more resilient |
| `capacityOptimized` | Launch from pool with most available capacity |
| `priceCapacityOptimized` | Best of both — lowest price with good capacity |

> 💡 **Spot Fleet with `diversified` strategy** = if one pool is interrupted, others keep running.

> ⚠️ **Exam Trap:** Spot instances are NOT suitable for anything that can't handle interruption — databases, payment processing, anything stateful.

---

## Purchasing Options — Exam Scenarios

| Scenario | Answer |
|----------|--------|
| Short-term, unpredictable workload | On-Demand |
| Steady database running 24/7 for 1 year | Reserved Instance |
| Batch job processing that can be interrupted | Spot Instance |
| Compliance requirement — dedicated hardware | Dedicated Host |
| Need flexibility across instance types but want discount | Savings Plans |
| Critical job next week — need guaranteed capacity | Capacity Reservation |

---

## Key Takeaways

| Concept | Remember |
|---------|----------|
| EC2 = IaaS | You manage the OS and above |
| User Data | Runs once at first boot, runs as root |
| Instance naming | class + generation + size e.g. m5.2xlarge |
| Security Groups | Stateful firewall, allow rules only, default block inbound |
| Timeout = SG issue | Connection refused = app issue |
| Port 22 | SSH (Linux), Port 3389 = RDP (Windows) |
| Never use access keys on EC2 | Always use IAM Roles |
| Spot = cheapest | Can be interrupted — 2 min warning |
| Reserved = best discount for steady workloads | 1 or 3 year commitment |
| Dedicated Host = most expensive | BYOL, compliance |
