# EC2 — Solution Architect Level

**Course:** Stephane Maarek · SAA-C03 · Section 6
**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. Private vs Public vs Elastic IP

When you launch an EC2 instance, it gets an IP address. There are 3 types to understand.

### Private IP

- Used for communication **inside AWS** (within VPC)
- Not reachable from the internet
- Every EC2 instance always has a private IP
- Private IP **never changes** — stays the same even after restart

### Public IP

- Used to reach your instance **from the internet**
- Assigned automatically when you launch an instance (if enabled)
- **Changes every time** you stop and start the instance
- If you stop and restart → you get a different public IP

```
Stop EC2 → Start EC2
Old Public IP: 54.12.34.56   ← gone
New Public IP: 54.98.76.54   ← new one assigned
```

> ⚠️ This is a problem if you have DNS records or scripts pointing to the public IP — it breaks every time you restart.

### Elastic IP

- A **static public IP** that you own — it doesn't change
- You allocate it from AWS and attach it to an instance
- If instance is stopped/started → Elastic IP stays the same
- You can **move it** from one instance to another (great for failover)

```
Instance A fails
      │
      ▼
Detach Elastic IP from Instance A
      │
      ▼
Attach to Instance B ← same IP, traffic redirected ✅
```

**Important billing rule:**
- Elastic IP is **free** while attached to a running instance
- Elastic IP is **charged** (~$0.005/hr) if not attached or attached to stopped instance

> ⚠️ You get max **5 Elastic IPs** per AWS account by default (can request more)

### When to use what?

| Situation | Use |
|-----------|-----|
| Internal AWS communication | Private IP |
| Temporary public access | Public IP |
| Fixed public IP that never changes | Elastic IP |
| Failover between instances | Elastic IP |

> 💡 **Best practice:** Instead of using an Elastic IP, use a **DNS name (Route 53)** or **Load Balancer** — more scalable and professional.

---

## 2. EC2 Placement Groups

Sometimes you want to control **where AWS physically places your EC2 instances**. That's what Placement Groups are for.

Three strategies available:

---

### Strategy 1 — Cluster

```
┌─────────────────────────────┐
│      Single AZ              │
│  ┌────┐ ┌────┐ ┌────┐       │
│  │EC2 │ │EC2 │ │EC2 │       │
│  └────┘ └────┘ └────┘       │
│  Same rack — ultra low      │
│  latency between instances  │
└─────────────────────────────┘
```

- All instances in the **same rack, same AZ**
- Extremely **low latency** + **high network throughput** between instances (10 Gbps)
- **Risk:** If the rack fails → ALL instances fail together
- **Use for:** HPC, big data jobs, anything needing extreme speed between nodes

---

### Strategy 2 — Spread

```
AZ 1          AZ 2          AZ 3
┌──────┐      ┌──────┐      ┌──────┐
│Rack 1│      │Rack 2│      │Rack 3│
│ EC2  │      │ EC2  │      │ EC2  │
└──────┘      └──────┘      └──────┘
  Different rack per instance = isolated failures
```

- Each instance is on a **different physical rack**
- Can span **multiple AZs**
- Max **7 instances per AZ** per placement group
- **Risk:** Lowest — rack failure only affects ONE instance
- **Use for:** Critical apps where each instance must be isolated — HA databases, critical services

---

### Strategy 3 — Partition

```
AZ 1
┌─────────────────────────────────────┐
│ Partition 1  │ Partition 2  │ Part 3│
│ ┌──┐ ┌──┐   │ ┌──┐ ┌──┐   │ ┌──┐  │
│ │EC2│ │EC2│  │ │EC2│ │EC2│  │ │EC2│  │
│ └──┘ └──┘   │ └──┘ └──┘   │ └──┘  │
│ Same rack    │ Same rack    │       │
└─────────────────────────────────────┘
```

- Instances divided into **partitions** (groups)
- Each partition is on a **different rack**
- Up to **7 partitions per AZ** — hundreds of instances per partition
- Partition failure affects only that partition — other partitions safe
- Instances can see which partition they're in (via metadata)
- **Use for:** Distributed big data systems — Hadoop, Cassandra, Kafka, HDFS

### Quick comparison

| | Cluster | Spread | Partition |
|--|---------|--------|-----------|
| **Goal** | Low latency | Max isolation | Balance both |
| **Risk** | High (one rack) | Very Low (separate racks) | Medium |
| **Instance limit** | No limit | 7 per AZ | 7 partitions, 100s of instances |
| **Spans AZs?** | ❌ No | ✅ Yes | ✅ Yes |
| **Use case** | HPC, Big Data speed | Critical HA apps | Hadoop, Kafka, Cassandra |

---

## 3. Elastic Network Interfaces (ENI)

### What is an ENI?

An ENI (Elastic Network Interface) is basically a **virtual network card** for your EC2 instance.

Every EC2 instance has at least one ENI attached — it's what gives the instance its network connectivity.

```
EC2 Instance
├── eth0 (Primary ENI) ← always attached
│     ├── Private IP (primary)
│     ├── Private IP (secondary — optional)
│     ├── Elastic IP (optional)
│     ├── Public IP (optional)
│     ├── MAC address
│     └── Security Groups
│
└── eth1 (Secondary ENI) ← optional, can attach/detach
```

### Key facts about ENIs

- An ENI can have:
  - One **primary private IPv4**
  - One or more **secondary private IPv4s**
  - One **Elastic IP** per private IPv4
  - One **public IPv4**
  - One or more **security groups**
  - One **MAC address**

- ENIs are **tied to a specific AZ** — can't move across AZs
- You can **create ENIs independently** (not tied to any instance) — useful to pre-allocate IPs before launching instances
- You can **detach and attach** ENIs to different instances
- Each ENI has its **own security groups** — different ENIs on the same instance can have different security groups

### Why would you use a secondary ENI?

**Failover use case:**

```
EC2 Instance A fails
      │
      ▼
Detach ENI (with private IP) from Instance A
      │
      ▼
Attach to EC2 Instance B ← same private IP, same MAC address ✅
Traffic flows to B without anyone noticing
```

> 💡 Moving an ENI carries the **private IP, Elastic IP and MAC address** with it — great for license software tied to MAC address.

### ENI vs Elastic IP

| | ENI | Elastic IP |
|--|-----|------------|
| What moves | Private IP + MAC | Public IP only |
| Use for | Internal failover | External/public failover |
| AZ locked? | ✅ Yes | ❌ No |

---

## 4. EC2 Hibernate

### What is Hibernate?

When you **stop** an EC2 instance:
- Instance stops
- RAM (memory) is **wiped**
- On restart → OS boots fresh → apps start from scratch → takes time

When you **hibernate** an EC2 instance:
- RAM contents are **saved to EBS** (root volume)
- Instance stops
- On restart → RAM is **restored from EBS** → apps pick up exactly where they left off

```
Normal Stop → Start:
RAM wiped → OS boots → apps start from scratch ← slow ❌

Hibernate → Start:
RAM saved to EBS → restored on next boot → picks up where it left off ✅ fast
```

### How it works internally

```
EC2 Instance (hibernating)
      │
      ▼ RAM contents written to
EBS Root Volume (encrypted)
      │
      ▼ On next start
RAM restored from EBS
      │
      ▼
Instance continues exactly where it stopped ✅
```

### Requirements for Hibernate

- **EBS root volume must be encrypted**
- Root EBS volume must be **large enough** to hold RAM contents
- RAM size must be **less than 150 GB**
- Available for: On-Demand, Reserved and Spot instances
- Supported OS: Amazon Linux 2, Linux AMIs, Ubuntu, Windows
- **Cannot hibernate for more than 60 days**

### Billing & Instance ID during Hibernate

- Instance is **not charged** while in hibernated state (same as stopped)
- But the **EBS volume is still charged** — because RAM contents are stored there
- Instance keeps the **same Instance ID** after resuming — nothing changes from the outside

### When to use Hibernate?

| Scenario | Use |
|----------|-----|
| Long-running process you don't want to restart | Hibernate |
| App takes long to warm up / initialise | Hibernate |
| Want to save state and resume later | Hibernate |
| Just want to stop and not pay | Regular Stop |

> ⚠️ **Exam Trap:** "An application takes 20 minutes to start up. How do you avoid this delay?" → **EC2 Hibernate** — instance resumes instantly from saved RAM state.

---

## 5. Key Takeaways

| Concept | Remember |
|---------|----------|
| Public IP | Changes on stop/start |
| Elastic IP | Static, you own it, charged when unused |
| Elastic IP limit | 5 per account by default |
| Cluster placement | Same rack, ultra low latency, high risk |
| Spread placement | Different racks, max 7 per AZ, max isolation |
| Partition placement | Groups of instances on separate racks — Hadoop, Kafka |
| ENI | Virtual network card, tied to AZ, can be moved between instances |
| ENI carries | Private IP + MAC address |
| ENI independent | Can be created without attaching to an instance — pre-allocate IPs |
| ENI security groups | Each ENI has its own security groups |
| Hibernate | Saves RAM to EBS, resumes from same state, max 60 days |
| Hibernate requirement | EBS must be encrypted, RAM < 150 GB |
| Hibernate billing | Instance not charged, but EBS still charged |
| Hibernate Instance ID | Same Instance ID after resume — nothing changes externally |

---

## 6. Exam Scenarios

**Scenario 1 — IP changes on restart**
> "An application hardcodes the EC2 public IP and breaks every time the instance restarts."
> ✅ Use an **Elastic IP** — stays the same across restarts

**Scenario 2 — Low latency between EC2 instances**
> "HPC application needs ultra-low latency between 10 EC2 instances."
> ✅ Use **Cluster Placement Group** — same rack, lowest latency

**Scenario 3 — Critical app — one rack failure should not take down everything**
> "7 critical EC2 instances must each be on separate hardware."
> ✅ Use **Spread Placement Group** — each instance on different rack

**Scenario 4 — Big data — Hadoop cluster with hundreds of instances**
> "Need to deploy a 200-node Hadoop cluster with rack awareness."
> ✅ Use **Partition Placement Group** — rack awareness per partition

**Scenario 5 — Failover with same private IP**
> "If primary EC2 instance fails, traffic must automatically use same private IP on backup."
> ✅ Use **ENI** — detach from primary, attach to backup, same private IP

**Scenario 6 — Long startup time**
> "Application takes 30 minutes to initialise on every boot. Need to avoid this delay."
> ✅ Use **EC2 Hibernate** — resumes from saved RAM state instantly

---

*→ See also: EC2-Basics.md*
*→ Next: EC2 Storage (EBS, EFS, Instance Store)*
