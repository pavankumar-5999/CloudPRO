# EC2 Storage — EBS, Snapshots, AMI, Instance Store & EFS

**Course:** Stephane Maarek · SAA-C03 · Section 6 (Lectures 57–68)
**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. What is EBS?

EBS = **Elastic Block Store** — a network-attached storage drive for your EC2 instance.

Think of it like a **USB stick** but over the network — you plug it into an EC2 instance and use it as a drive.

```
EC2 Instance
     │
     │ (network connection)
     ▼
EBS Volume ← your data lives here
```

**Key facts:**
- It is a **network drive** — not physically attached to the instance
- Because it's over the network, there can be a tiny bit of latency
- Can be **detached from one instance and attached to another** quickly
- **Locked to one AZ** — an EBS in eu-west-2a cannot attach to an instance in eu-west-2b
- You are **billed for provisioned capacity** — even if you don't use it all
- Capacity can be increased over time

---

## 2. EBS — Delete on Termination

When you terminate an EC2 instance, what happens to the EBS?

| Volume | Default behaviour |
|--------|------------------|
| Root EBS volume | ✅ Deleted when instance terminates |
| Additional EBS volumes | ❌ NOT deleted by default — they persist |

> 💡 You can change this behaviour during instance launch — uncheck "Delete on termination" for root volume if you want to keep data after termination.

> ⚠️ **Exam Tip:** "Preserve root volume after instance termination" → Disable Delete on Termination for root EBS.

---

## 3. EBS Snapshots

A snapshot is a **backup of your EBS volume** at a point in time — stored in S3.

```
EBS Volume → Snapshot (stored in S3) → Restore to new EBS in any AZ/Region
```

**Key facts:**
- You don't need to detach the volume to take a snapshot (but recommended)
- Snapshots are **incremental** — only changed blocks are saved after the first snapshot
- Can copy snapshots **across regions** — useful for DR and migration
- Can create a new EBS volume from a snapshot **in any AZ**

### EBS Snapshot Features

**Snapshot Archive:**
- Move snapshot to an **archive tier** — 75% cheaper
- Restoring from archive takes **24–72 hours**
- Use when you rarely need the snapshot

**Recycle Bin:**
- Instead of permanently deleting snapshots, send to Recycle Bin
- Can recover deleted snapshots — retention: 1 day to 1 year
- Protects against accidental deletion

**Fast Snapshot Restore (FSR):**
- Force full initialisation of snapshot so there's **no latency on first use**
- Useful when you need to restore quickly and can't wait for lazy loading
- Costs more — only enable when needed

---

## 4. AMI — Amazon Machine Image

An AMI is a **template** used to launch EC2 instances — it includes the OS, configuration and software.

```
EC2 Instance (configured with your apps)
      │
      ▼ Create AMI
Custom AMI
      │
      ▼ Launch new instances from AMI
Multiple identical EC2 instances ✅
```

**Types of AMIs:**
- **AWS managed AMIs** — Amazon Linux 2, Ubuntu, Windows etc. (provided by AWS)
- **AWS Marketplace AMIs** — third-party software pre-installed (e.g. WordPress, Nginx)
- **Custom AMIs** — your own image with your apps pre-installed

**Why create a custom AMI?**
- Faster boot time — everything is pre-installed
- No need for User Data scripts to install software on launch
- Consistent environment across all instances
- AMIs are **region-specific** — must copy to another region to use there

### AMI Creation Process

```
1. Launch EC2 instance
2. Customise it — install apps, configure settings
3. Stop the instance (recommended for data integrity)
4. Create AMI (EBS snapshot is taken automatically)
5. Launch new instances from your AMI ✅
```

> ⚠️ **Exam Trap:** AMIs are region-locked. To use in another region → copy the AMI to that region first.

---

## 5. EC2 Instance Store

Some EC2 instance types come with a **physical hard drive** directly attached to the server — this is called Instance Store.

```
EC2 Instance
├── EBS Volume    ← network attached, persistent
└── Instance Store ← physically attached, temporary
```
##Note: EC2 Instance Store lose their storage if they're stopped (ephemeral)
**Key differences from EBS:**

| | EBS | Instance Store |
|--|-----|----------------|
| Connection | Network | Physical |
| Speed | Fast | **Extremely fast** (highest IOPS) |
| Persistence | ✅ Persists after stop/terminate | ❌ Lost when instance stops/terminates |
| Use case | Persistent data | Temporary data, cache, buffer |
| Backup | Snapshots | Your responsibility (no snapshot support) |

**Use Instance Store for:**
- Temporary files
- Cache data
- Buffer / scratch space
- Data you can afford to lose

> ⚠️ **Exam Trap:** "Highest IOPS possible on EC2" → **Instance Store** (not EBS). But data is lost when instance stops — so only use for temporary data.

---

## 6. EBS Volume Types

EBS has 6 volume types — split into SSD and HDD.

```
EBS Volume Types
├── SSD (random I/O — good for databases, boot volumes)
│   ├── gp3 — General Purpose (latest, recommended)
│   ├── gp2 — General Purpose (older)
│   ├── io2 — Provisioned IOPS (highest performance)
│   └── io1 — Provisioned IOPS (older)
│
└── HDD (sequential I/O — good for big data, logs)
    ├── st1 — Throughput Optimized HDD
    └── sc1 — Cold HDD (cheapest)
```

> ⚠️ **Only SSD volumes (gp2, gp3, io1, io2) can be used as boot volumes** — HDD volumes cannot.

### Volume Types — Full Comparison

| Volume | Type | Max IOPS | Max Throughput | Size | Use Case | Cost |
|--------|------|----------|---------------|------|----------|------|
| **gp3** | SSD | 16,000 | 1,000 MB/s | 1GB–16TB | Boot, web apps, dev | $ |
| **gp2** | SSD | 16,000 | 250 MB/s | 1GB–16TB | Boot, general (legacy) | $ |
| **io2** | SSD | 256,000 | 4,000 MB/s | 4GB–64TB | Critical DBs, mission-critical | $$$ |
| **io1** | SSD | 64,000 | 1,000 MB/s | 4GB–16TB | High perf DB (legacy) | $$$ |
| **st1** | HDD | 500 | 500 MB/s | 125GB–16TB | Big data, Kafka, logs | ¢ |
| **sc1** | HDD | 250 | 250 MB/s | 125GB–16TB | Cold data, infrequent access | ¢ |

---

### gp3 — General Purpose SSD (Recommended)

- **Default and recommended** for most workloads
- Baseline: **3,000 IOPS + 125 MB/s** — regardless of volume size
- Can scale up to **16,000 IOPS** and **1,000 MB/s** independently (pay extra)
- **~20% cheaper** than gp2

**Use for:** Boot volumes, web servers, dev/test, small-medium databases

> 💡 **gp3 vs gp2:** In gp2, IOPS scales with size (3 IOPS per GB). In gp3, IOPS is independent of size — you always get 3,000 baseline. **Always prefer gp3 over gp2.**

---

### gp2 — General Purpose SSD (Legacy)

- Older generation — **gp3 is better in almost every way**
- IOPS linked to volume size: **3 IOPS per GB** — max 16,000 IOPS
- Can burst to 3,000 IOPS for small volumes using I/O credits

> ⚠️ If you exhaust burst credits → performance drops to baseline (as low as 100 IOPS for tiny volumes). This is a classic production issue with gp2.

---

### io2 / io1 — Provisioned IOPS SSD

Use when you need **guaranteed, consistent IOPS** — great for critical databases.

| | io2 | io1 |
|--|-----|-----|
| Max IOPS | 256,000 | 64,000 |
| Max throughput | 4,000 MB/s | 1,000 MB/s |
| Durability | **99.999%** (5 nines) | 99.9% |
| IOPS per GB ratio | 1,000 IOPS/GB | 50 IOPS/GB |
| Multi-Attach | ✅ Yes | ✅ Yes |

**Use for:** Large SQL/NoSQL databases, SAP HANA, Oracle, mission-critical workloads

> 💡 io2 and io1 cost the same but **io2 is far superior** — always choose io2 over io1.

---

### st1 — Throughput Optimized HDD

- HDD-based — good for **large sequential reads/writes**
- Baseline: 40 MB/s per TB — burst up to 250 MB/s per TB — max 500 MB/s
- **Cannot be used as a boot volume**

**Use for:** Big data, Hadoop, Kafka, log processing, data warehouses, ETL

---

### sc1 — Cold HDD

- **Cheapest EBS volume** — lowest cost per GB
- Baseline: 12 MB/s per TB — burst up to 80 MB/s per TB — max 250 MB/s
- **Cannot be used as a boot volume**

**Use for:** Infrequently accessed data, cold archival, lowest-cost storage requirement

---

### Volume Type Decision Tree

```
Do you need a boot volume?
├── YES → gp3 (or io2 if high IOPS needed)
└── NO
    │
    ├── Need guaranteed high IOPS (database, mission-critical)?
    │   └── io2
    │
    ├── General workload — web app, dev, moderate DB?
    │   └── gp3
    │
    ├── Big data, logs, sequential reads — price matters?
    │   └── st1
    │
    └── Infrequent access, lowest cost?
        └── sc1
```

---

## 7. EBS Multi-Attach

Normally an EBS volume can only attach to **one EC2 instance** at a time.

With **Multi-Attach** you can attach one EBS volume to **up to 16 EC2 instances** in the same AZ.

```
EBS io2 Volume
   ├──► EC2 Instance 1  ┐
   ├──► EC2 Instance 2  │ All in same AZ
   ├──► EC2 Instance 3  │
   └──► ...up to 16     ┘
```

**Requirements:**
- Only available on **io1 and io2** volumes
- All instances must be in the **same AZ**
- Must use a **cluster-aware file system** (not regular ext4 or XFS)

**Use for:** High-availability apps, clustered databases (Oracle RAC), Teradata

> ⚠️ **Exam Trap:** Multi-Attach needs a cluster-aware file system — regular file systems like ext4 don't support concurrent writes from multiple instances.

---

## 8. EBS Encryption

When you create an encrypted EBS volume:

- Data **at rest** is encrypted
- Data **in transit** between EC2 and EBS is encrypted
- Snapshots are encrypted
- Volumes created from encrypted snapshots are encrypted

**Behind the scenes:** Uses AWS KMS (AES-256) — you don't manage the keys yourself.

### How to encrypt an existing unencrypted volume:

```
Unencrypted EBS Volume
      │
      ▼ Take snapshot
Unencrypted Snapshot
      │
      ▼ Copy snapshot with encryption enabled
Encrypted Snapshot
      │
      ▼ Create volume from encrypted snapshot
Encrypted EBS Volume ✅
```

> 💡 There is **no direct way to encrypt an existing volume** — you must go through the snapshot route.

> ⚠️ **Exam Trap:** Encryption has **minimal performance impact** — AWS handles it transparently. No reason NOT to encrypt.

---

## 9. Amazon EFS — Elastic File System

EFS is a **managed network file system** — it can be mounted on **many EC2 instances at the same time**, across multiple AZs.

```
EFS File System
   ├──► EC2 in us-east-1a
   ├──► EC2 in us-east-1b  ← all share the same files ✅
   └──► EC2 in us-east-1c
```

**Key facts:**
- Works with **Linux only** (POSIX file system) — NOT Windows
- Scales **automatically** — no capacity planning needed
- Pay per GB used (not provisioned) — more expensive than EBS
- Uses **NFS protocol** (port 2049)
- Highly available — data stored across multiple AZs

### EFS Storage Classes

| Class | Use case | Cost |
|-------|----------|------|
| **EFS Standard** | Frequently accessed files | Higher |
| **EFS Standard-IA** | Infrequent access | ~92% cheaper |
| **EFS One Zone** | Single AZ, frequently accessed | |
| **EFS One Zone-IA** | Single AZ, infrequent access | Cheapest |

> 💡 Enable **EFS Lifecycle Management** to auto-move files between Standard and IA based on last access time — saves cost automatically.

### EFS Performance Modes

| Mode | When to use |
|------|-------------|
| **General Purpose** (default) | Most use cases — web serving, CMS |
| **Max I/O** | Hundreds of EC2 instances — big data, media processing |

### EFS Throughput Modes

| Mode | How it works |
|------|-------------|
| **Bursting** (default) | Throughput scales with storage size |
| **Provisioned** | Set throughput regardless of storage size |
| **Elastic** | Auto-scales based on workload — recommended |

---

## 10. EBS vs EFS vs Instance Store

This is one of the most common exam topics — know this cold.

| | EBS | EFS | Instance Store |
|--|-----|-----|----------------|
| **Type** | Block storage | File storage (NFS) | Block storage |
| **Attached to** | One instance (Multi-Attach: up to 16 with io2) | Many instances across AZs | One instance |
| **AZ scope** | Locked to one AZ | Multi-AZ | Locked to one AZ |
| **Persistence** | ✅ Persistent | ✅ Persistent | ❌ Lost on stop |
| **Performance** | High | Moderate | Highest (physical) |
| **OS support** | Linux + Windows | Linux only | Linux + Windows |
| **Scaling** | Manual | Automatic | Fixed |
| **Cost** | Pay per provisioned GB | Pay per used GB | Free (included) |
| **Use case** | Boot volumes, databases | Shared content, CMS, home dirs | Cache, temp files, buffer |

---

## 11. EBS RAID Configurations

RAID = combining multiple EBS volumes together for more performance or redundancy.

### RAID 0 — Performance

```
App writes data
      │
      ├──► EBS Volume 1 (stripe 1) ─┐
      └──► EBS Volume 2 (stripe 2) ─┴──► Combined IOPS + Throughput ✅
```

- Combines volumes → **doubles IOPS and throughput**
- If **one volume fails → all data is lost** ❌
- **Use for:** When you need more IOPS than a single volume can give — databases, gaming

### RAID 1 — Fault Tolerance

```
App writes data
      │
      ├──► EBS Volume 1 (mirror) ─┐
      └──► EBS Volume 2 (mirror) ─┴──► Same data on both ✅
```

- Mirrors data across 2 volumes → **fault tolerance**
- If one volume fails → other keeps running ✅
- No performance gain — same IOPS as a single volume
- **Use for:** Critical data that must survive a disk failure

> ⚠️ **Exam Tip:** "Need more IOPS than a single EBS volume can provide" → **RAID 0**. "Need fault tolerance at disk level" → **RAID 1**.

---

## 12. EFS — Security

EFS uses security groups and encryption to control and protect access.

**Access Control:**
- Attach a **Security Group** to your EFS mount target — controls which EC2 instances can connect
- Use **IAM policies** to control who can create/delete EFS file systems

**Encryption:**

| Type | How |
|------|-----|
| **At rest** | Uses **AWS KMS** — enable during EFS creation |
| **In transit** | Uses **TLS** — enable when mounting with the EFS mount helper |

```bash
# Mount EFS with TLS encryption in transit
mount -t efs -o tls fs-12345678:/ /mnt/efs
```

> ⚠️ **Exam Trap:** EFS encryption at rest must be enabled **at creation time** — cannot be enabled on an existing EFS file system.

---

## 13. EBS Elastic Volumes — Modify Without Downtime

You can modify an EBS volume **while it is attached and in use** — no need to stop the instance.

**What you can change on the fly:**
- Increase volume **size**
- Change **volume type** (e.g. gp2 → gp3)
- Increase **IOPS or throughput**

```
gp2 volume (100 GB, 300 IOPS)
      │
      ▼ Modify (no downtime)
gp3 volume (200 GB, 6000 IOPS) ✅
```

> ⚠️ **Important:** After resizing the EBS volume, you still need to **extend the file system** inside the OS manually:

```bash
# Check current disk size
lsblk

# Extend the partition (Linux)
sudo growpart /dev/xvda 1

# Resize the file system
sudo resize2fs /dev/xvda1
```

> ⚠️ **Exam Trap:** "Increase EBS size without downtime" → **Elastic Volumes** — but remember to resize the file system inside the OS too, otherwise the extra space won't be visible.

---

## 14. Exam Scenarios

**Scenario 1 — Highest possible disk IOPS**
> "An application needs the absolute highest disk I/O performance."
> ✅ **EC2 Instance Store** — physically attached, highest IOPS. Note: data is lost on stop.

**Scenario 2 — Database needs guaranteed 50,000 IOPS**
> "A critical Oracle database needs consistent 50,000 IOPS."
> ✅ **io2 (Provisioned IOPS SSD)** — guaranteed IOPS, mission-critical workloads

**Scenario 3 — Share files across multiple EC2 instances**
> "100 EC2 web servers need access to the same config files."
> ✅ **EFS** — shared file system, multiple instances, multiple AZs

**Scenario 4 — Big data / Hadoop workload, cost sensitive**
> "MapReduce cluster processing 10 TB of log files — large sequential reads."
> ✅ **st1 (Throughput Optimized HDD)** — sequential throughput, low cost

**Scenario 5 — Cheapest EBS storage for cold data**
> "Archive data accessed once a month."
> ✅ **sc1 (Cold HDD)** — lowest cost EBS option

**Scenario 6 — Move EBS volume to another AZ**
> "Move an EBS volume from eu-west-2a to eu-west-2b."
> ✅ **Take snapshot → Create new volume from snapshot in target AZ**

**Scenario 7 — Move EBS to another region**
> "Copy an EBS volume to us-east-1."
> ✅ **Snapshot → Copy snapshot to us-east-1 → Create volume from snapshot**

**Scenario 8 — Encrypt an existing unencrypted EBS volume**
> "An existing gp3 volume needs to be encrypted."
> ✅ **Snapshot → Copy snapshot with encryption → Create encrypted volume from snapshot**

**Scenario 9 — Faster instance launch, software pre-installed**
> "New EC2 instances take 20 mins to install software via User Data. Need faster launch."
> ✅ **Create a custom AMI** with software pre-installed → launch from AMI instantly

**Scenario 10 — Backup EBS across regions for DR**
> "Need disaster recovery in a different AWS region."
> ✅ **EBS Snapshot → Copy to another region → Restore when needed**

**Scenario 11 — Need more IOPS than a single EBS volume provides**
> "A database needs 40,000 IOPS but a single volume maxes out at 16,000."
> ✅ **RAID 0** — stripe two volumes together to combine IOPS

**Scenario 12 — Disk-level fault tolerance**
> "Critical application needs to survive a single EBS volume failure."
> ✅ **RAID 1** — mirror two volumes so data survives one disk failure

**Scenario 13 — Increase EBS size without stopping instance**
> "Production database volume is running out of space — can't afford downtime."
> ✅ **Elastic Volumes** — increase size on the fly, then extend file system in OS

**Scenario 14 — EFS encryption must be enforced**
> "All EFS data must be encrypted at rest — how to ensure this?"
> ✅ Enable **KMS encryption at EFS creation time** — cannot be added later

---

## 15. Key Takeaways

| Concept | Remember |
|---------|----------|
| EBS | Network drive, locked to one AZ, persistent |
| EBS Delete on Termination | Root volume deleted by default, extra volumes are NOT |
| Snapshots | Incremental, stored in S3, can copy across regions |
| AMI | Region-specific template, copy to use in another region |
| Instance Store | Highest IOPS, data lost on stop — temp data only |
| gp3 | Default recommended, 3,000 IOPS baseline, independent IOPS/throughput |
| gp2 | Legacy — IOPS tied to size, use gp3 instead |
| io2 | Highest performance SSD, 99.999% durability, up to 256,000 IOPS |
| st1 | HDD, sequential reads, big data — cannot boot |
| sc1 | Cheapest HDD, cold data — cannot boot |
| Multi-Attach | io1/io2 only, same AZ, up to 16 instances, needs cluster FS |
| EBS Encryption | Snapshot route to encrypt existing volume, minimal performance impact |
| EFS | NFS, Linux only, multi-AZ, shared, auto-scales |
| EFS vs EBS | EFS = shared across many instances, EBS = one instance |
| RAID 0 | Stripe volumes → more IOPS, but one failure = all data lost |
| RAID 1 | Mirror volumes → fault tolerance, no IOPS gain |
| EFS encryption | Must enable at creation — cannot add later, uses KMS + TLS |
| Elastic Volumes | Resize/retype EBS with no downtime — still need OS file system resize |

---

*→ See also: EC2-Basics.md*
*→ See also: EC2-SA-Level.md*
*→ Next: High Availability & Scalability (ELB & ASG)*
