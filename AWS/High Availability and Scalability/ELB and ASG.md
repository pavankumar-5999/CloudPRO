# High Availability & Scalability — ELB & Auto Scaling

**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. Why Do We Need This?

Imagine your website is running on **one EC2 instance**. What happens if:

- That instance crashes? → Website goes down 
- Traffic suddenly spikes (1000s of visitors)? → Instance gets overwhelmed and slows down 
- You need to do maintenance on that instance? → Website goes down during that time 

This is exactly the problem **Load Balancers** and **Auto Scaling Groups** solve.

```
Without ELB/ASG:
One EC2 Instance → handles ALL traffic alone → single point of failure 

With ELB/ASG:
Multiple EC2 Instances → traffic spread across all of them → no single point of failure 
```

---

## 2. Two Important Concepts First

### Scalability
The ability to handle **more load** by adding resources.

| Type | What it means |
|------|---------------|
| **Vertical Scaling** | Make ONE instance bigger (more CPU/RAM) — e.g. t2.micro → t2.large |
| **Horizontal Scaling** | Add MORE instances — e.g. 1 instance → 5 instances |

```
Vertical Scaling:
[Small Server] ──► [Bigger Server]  (upgrade the same machine)

Horizontal Scaling:
[Server] ──► [Server] [Server] [Server]  (add more machines)
```

> 💡 Cloud computing is famous for **horizontal scaling** — it's cheaper, more flexible and has no upper limit like vertical scaling does.

### High Availability
Running your application in **more than one physical location (Availability Zone)** so that if one location fails, the others keep working.

```
High Availability Setup:

AZ 1              AZ 2              AZ 3
[EC2 Instance]    [EC2 Instance]    [EC2 Instance]
      │                 │                 │
      └─────────────────┴─────────────────┘
              All behind one Load Balancer

If AZ 1 goes down → AZ 2 and AZ 3 still serve traffic ✅
```

---

## 3. What is a Load Balancer?

A **Load Balancer** is a service that sits in front of your servers and **spreads incoming traffic** across multiple EC2 instances.

```
Users
  │
  ▼
Load Balancer  ← the "traffic cop"
  │
  ├──► EC2 Instance 1
  ├──► EC2 Instance 2
  └──► EC2 Instance 3
```

**Why use a Load Balancer instead of just giving users each server's IP directly?**

| Problem with direct access | Solution with Load Balancer |
|------------------------------|------------------------------|
| Users need to know every server's IP | Users only need ONE address — the load balancer's |
| If a server fails, traffic still goes to it | Load balancer automatically stops sending traffic to unhealthy servers |
| No way to balance load evenly | Load balancer spreads traffic evenly across all healthy servers |
| No SSL/encryption management in one place | Load balancer can handle SSL certificates centrally |

---

## 4. Why Use AWS's Managed Load Balancer (ELB)?

You could technically install your own load balancing software on an EC2 instance, but AWS provides a **managed service called ELB (Elastic Load Balancer)**.

**Benefits of using ELB instead of your own:**
- **Fully managed** by AWS — you don't patch or maintain it
- **Highly available** across multiple AZs automatically
- **Integrates with Auto Scaling, security groups, ACM (SSL certificates)**
- You only pay for what you use

---

## 5. Health Checks

Before sending traffic to any instance, the Load Balancer needs to know: **"Is this instance actually working?"**

```
Load Balancer periodically checks:
GET /health  →  Instance responds 200 OK  →  Healthy ✅  → traffic sent
GET /health  →  No response / Error       →  Unhealthy ❌ → traffic stopped
```

**How it works:**
- You configure a specific **port and path** to check (e.g. `/health` on port 80)
- If the instance fails the health check, the Load Balancer **stops sending traffic to it** until it recovers

> 💡 This is exactly how a Load Balancer avoids sending users to a broken server — it constantly checks and reacts automatically.

---

## 6. Types of Load Balancers in AWS

AWS offers **3 types** of managed load balancers, each suited for different needs.

```
Load Balancer Types
├── Application Load Balancer (ALB)  → Layer 7, smart routing (HTTP/HTTPS)
├── Network Load Balancer (NLB)      → Layer 4, ultra high performance (TCP/UDP)
└── Gateway Load Balancer (GWLB)     → Layer 3, for security appliances
```

### Application Load Balancer (ALB)
- Works at **Layer 7** (application layer) — understands HTTP/HTTPS
- Can route traffic based on the **URL path** or **hostname**

```
Request: mysite.com/images/*  ──► routed to Image-Servers Target Group
Request: mysite.com/api/*     ──► routed to API-Servers Target Group
Request: mysite.com/*         ──► routed to Web-Servers Target Group
```

**Best for:** Websites, web apps, microservices, anything using HTTP/HTTPS

### Network Load Balancer (NLB)
- Works at **Layer 4** (transport layer) — handles raw TCP/UDP traffic
- Extremely **high performance** — can handle millions of requests per second
- Very **low latency**

**Best for:** Gaming servers, real-time applications, extreme performance needs

### Gateway Load Balancer (GWLB)
- Works at **Layer 3** (network layer)
- Used to deploy and manage **third-party security appliances** (firewalls, intrusion detection)

**Best for:** Routing traffic through security/inspection appliances before it reaches your app

---

## 7. Target Groups

A Load Balancer doesn't send traffic directly to individual EC2 instances — it sends traffic to a **Target Group**, which is just a list of resources.

```
Load Balancer
      │
      ▼
Target Group ("web-servers")
      ├── EC2 Instance 1
      ├── EC2 Instance 2
      └── EC2 Instance 3
```

**Target Groups can contain:**
- EC2 Instances
- IP addresses (even outside AWS!)
- Lambda functions

> 💡 Health checks are actually done at the **Target Group level** — each target in the group is checked individually.

---

## 8. Sticky Sessions

Normally, every request from a user might go to a **different** EC2 instance behind the load balancer. But sometimes you need the **same user to always land on the same instance** — this is called **Sticky Sessions**.

```
Without Sticky Sessions:
User's Request 1 → Instance A
User's Request 2 → Instance B  (different instance each time)

With Sticky Sessions:
User's Request 1 → Instance A
User's Request 2 → Instance A  (same instance every time) ✅
```

**Use case:** A shopping cart application where the user's session data is stored locally on one instance — if the user's next request goes to a different instance, their cart might appear empty!

> ⚠️ **Downside:** Sticky sessions can cause **uneven load** — if the "sticky" instance suddenly gets a lot of returning users, it could be overloaded while others sit idle.

---

## 9. Cross-Zone Load Balancing

Your instances might be spread across multiple Availability Zones. Cross-Zone Load Balancing decides **how evenly** traffic is spread across those zones.

```
Without Cross-Zone Load Balancing:
AZ 1 (2 instances) gets 50% of traffic total → 25% per instance
AZ 2 (8 instances) gets 50% of traffic total → 6.25% per instance
(uneven load per instance!)

With Cross-Zone Load Balancing:
Traffic is spread evenly across ALL instances in ALL zones
regardless of how many are in each zone ✅
```

**Use when:** You want perfectly even distribution regardless of how instances are spread across AZs.

---

## 10. SSL/TLS on Load Balancers

Load Balancers can handle **HTTPS encryption** for you, so your EC2 instances don't have to manage SSL certificates individually.

```
User (HTTPS) ──► Load Balancer (decrypts here) ──► EC2 Instance (plain HTTP)
```

- You attach an **SSL/TLS certificate** to the Load Balancer (via AWS Certificate Manager - ACM)
- The Load Balancer handles the encryption/decryption
- Your backend instances can just talk plain HTTP — simpler setup

---

## 11. What is Auto Scaling?

An **Auto Scaling Group (ASG)** automatically adjusts the **number of EC2 instances** running, based on real-time demand.

```
Low traffic at 3 AM  → ASG runs 2 instances (saves money)
High traffic at 6 PM → ASG automatically launches 8 instances (handles the load)
Traffic drops again  → ASG automatically terminates extra instances
```

**Why this matters:**
- You don't pay for instances you don't need
- Your app can handle sudden traffic spikes without manual intervention
- Failed instances get replaced automatically

---

## 12. Auto Scaling Group — Key Settings

| Setting | Meaning |
|---------|---------|
| **Minimum size** | The smallest number of instances that should ever run |
| **Maximum size** | The largest number of instances allowed to run |
| **Desired capacity** | The number of instances ASG tries to maintain right now |

```
Min: 2   Desired: 4   Max: 10

ASG will always keep AT LEAST 2 running
ASG currently wants 4 running
ASG will NEVER go above 10, no matter the demand
```

---

## 13. Launch Template

Before an Auto Scaling Group can create new instances, it needs to know **exactly what kind of instance to create**. This blueprint is called a **Launch Template**.

**A Launch Template defines:**
- Which AMI (image) to use
- Instance type (e.g. t2.micro)
- Security groups
- User Data script (to auto-configure the instance on boot)
- Key pair
- Storage settings

```
Auto Scaling Group needs a new instance
        │
        ▼
Reads the Launch Template
        │
        ▼
Creates a new EC2 instance exactly matching the template ✅
```

---

## 14. Scaling Policies — How ASG Decides to Scale

ASG doesn't just randomly add or remove instances — it follows rules called **Scaling Policies**.

### Target Tracking Scaling
The simplest option — you just set a target metric, and AWS handles the rest.

```
"Keep average CPU usage at 40% across all instances"
        │
        ▼
CPU goes above 40% → ASG adds instances automatically
CPU drops below 40% → ASG removes instances automatically
```

### Step Scaling
More advanced — define specific actions based on how far a metric has crossed a threshold.

```
If CPU > 50% → add 1 instance
If CPU > 80% → add 3 instances
```

### Scheduled Scaling
Scale based on a **known schedule** — useful when you know traffic patterns in advance.

```
Every day at 8 AM  → scale up to 10 instances (office hours start)
Every day at 8 PM  → scale down to 2 instances (traffic drops overnight)
```

### Predictive Scaling
Uses **machine learning** to forecast future traffic and scales ahead of time automatically — instead of reacting after load increases.

---

## 15. Scaling Cooldown Period

After a scaling action happens, ASG waits a bit before evaluating whether to scale again — this is called the **Cooldown Period**.

```
ASG adds 1 instance
        │
        ▼
Cooldown Period starts (e.g. 300 seconds)
        │
        ▼
ASG waits before scaling again — gives new instance time to start handling traffic
```

**Why this matters:** Without a cooldown, ASG might add too many instances too quickly before the first new one even finishes starting up and helping with the load.

---

## 16. ELB vs ASG — How They Work Together

These two services are almost always used **together**, not separately.

```
                    ┌──────────────────┐
   Users ────────►  │   Load Balancer   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  Auto Scaling     │
                    │  Group            │
                    └─────────┬─────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        EC2 Instance    EC2 Instance    EC2 Instance
        (created/removed automatically by ASG)
```

- **Load Balancer** → spreads traffic across whatever instances currently exist
- **Auto Scaling Group** → decides how many instances should exist right now
- When ASG adds/removes an instance, it automatically registers/deregisters it with the Load Balancer's Target Group

---

## 17. Key Takeaways

| Concept | Remember |
|---------|----------|
| Vertical Scaling | Bigger instance — has a hardware limit |
| Horizontal Scaling | More instances — no real limit, cloud-native approach |
| High Availability | Spread across multiple AZs — survive one AZ failure |
| Load Balancer | Single entry point, spreads traffic, checks instance health |
| ALB | Layer 7, smart HTTP routing by path/hostname |
| NLB | Layer 4, extreme performance, TCP/UDP |
| GWLB | Layer 3, for security appliances |
| Target Group | The list of instances a Load Balancer sends traffic to |
| Sticky Sessions | Same user always goes to same instance — risk of uneven load |
| Cross-Zone Load Balancing | Spreads traffic evenly regardless of instance count per AZ |
| Auto Scaling Group | Automatically adds/removes instances based on demand |
| Launch Template | Blueprint ASG uses to create new instances |
| Target Tracking | Simplest scaling policy — maintain a target metric |
| Cooldown Period | Wait time after scaling before scaling again |

---

## 18. Real-World Scenarios

**Scenario 1 — Website crashes when one server goes down**
> Put multiple EC2 instances behind an **Application Load Balancer** across multiple AZs — if one instance or AZ fails, traffic automatically routes to the healthy ones.

**Scenario 2 — Need to route `/api/*` and `/images/*` to different backend servers**
> Use an **Application Load Balancer** with path-based routing rules — each path routes to its own Target Group.

**Scenario 3 — Gaming application needs extremely low latency and millions of connections**
> Use a **Network Load Balancer** — built for raw TCP/UDP performance at massive scale.

**Scenario 4 — Traffic is predictably high every weekday at 9 AM**
> Use **Scheduled Scaling** — configure the Auto Scaling Group to increase capacity right before the known traffic spike.

**Scenario 5 — Shopping cart data is lost when user's requests hit different servers**
> Enable **Sticky Sessions** on the Load Balancer so each user consistently lands on the same instance.

**Scenario 6 — Want to save cost during low-traffic hours automatically**
> Use an **Auto Scaling Group** with **Target Tracking** — it will scale down instances automatically when demand (like CPU usage) drops.
