# Amazon S3 — Basics

**Course:** Stephane Maarek · SAA-C03
**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. What is Amazon S3?

Amazon S3 (Simple Storage Service) is AWS's **object storage service** — store, retrieve and manage any amount of data from anywhere over the internet.

> 💡 **Fun Fact:** S3 was the **second AWS service** ever launched, right after Amazon SQS.

**Simple analogy:**

```
Hard Drive / SSD   →   S3 Bucket
Folder             →   Prefix (fake folder)
File               →   Object
File Path          →   Key
```

---

## 2. Why S3?

Before cloud, companies stored data on **physical servers in their own data centres** — expensive, hard to scale and risky.

S3 solved this with:

| Feature | What it means |
|---------|---------------|
| **11 nines durability** | 99.999999999% — extremely safe |
| **Unlimited scalability** | Store as much as you want |
| **High availability** | Always accessible |
| **Pay-as-you-go** | Only pay for what you use |
| **AWS integration** | Works with Lambda, EC2, CloudFront and more |

---

## 3. Real-World Examples

| Company | What they use S3 for |
|---------|----------------------|
| **Nasdaq** | Stores 7 years of financial records in S3 Glacier (legal requirement) |
| **Sysco** | Runs analytics on business data using S3 + Athena/Redshift |

**Common use cases:**
- Website assets and image hosting
- Backups and disaster recovery
- Data lakes and big data analytics
- Application logs
- Static website hosting
- Media streaming

---

## 4. Buckets

```
┌────────────────────────────────────────────┐
│             Amazon S3                      │
│                                            │
│  ┌─────────────┐    ┌─────────────┐        │
│  │  Bucket A   │    │  Bucket B   │        │
│  │ (us-east-1) │    │ (eu-west-1) │        │
│  └─────────────┘    └─────────────┘        │
│                                            │
│  Global dashboard — buckets are REGIONAL   │
└────────────────────────────────────────────┘
```

**Key rules:**
- Bucket names must be **globally unique** across all AWS accounts
- A bucket lives in **one specific AWS Region**
- Names must be **3–63 characters, lowercase, no underscores**
- You can store **virtually unlimited** data

---

## 5. Objects

Every file stored in S3 is called an **object**.

```
s3://my-bucket/photos/2026/profile.jpg
      │              │
   Bucket        Key (prefix + object name)
```

> ⚠️ There are **no real directories** in S3 — everything is a key. Folders are just part of the key name (called a prefix).

**Object components:**

| Component | Description |
|-----------|-------------|
| **Key** | Full path — e.g. `s3://bucket/folder/file.txt` |
| **Value** | Actual file content/data |
| **Metadata** | System or user key-value pairs describing the object |
| **Tags** | Up to 10 Unicode key-value pairs — used for security and lifecycle |
| **Version ID** | Unique ID assigned if versioning is enabled |

**Size limits:**

| Limit | Value |
|-------|-------|
| Max object size | **50 TB** |
| Max single upload | **5 GB** |
| Recommended Multipart Upload | Files **> 100 MB** |
| Mandatory Multipart Upload | Files **> 5 GB** |

---

## 6. Multipart Upload

For large files, split into chunks and upload in parallel.

```
Large File (10 GB)
├── Part 1 (1GB) ──► uploads in parallel ─┐
├── Part 2 (1GB) ──► uploads in parallel  │
├── Part 3 (1GB) ──► uploads in parallel  ├──► S3 combines → single object ✅
│   ...                                   │
└── Part 10 (1GB) ──► uploads in parallel ┘
```

**Benefits:**
- Faster — parallel uploads
- Resilient — if one part fails, only retry that part
- Required for files **> 5 GB**
- Recommended for files **> 100 MB**

> ⚠️ **Exam Tip:** Speeding up uploads → **Multipart Upload**. Speeding up downloads → **Byte-Range Fetches**.

---

## 7. Static Website Hosting

S3 can host **static websites** directly — no server needed.

```
Browser ──► S3 Bucket (index.html) ──► Website loads ✅
```

**Supports:** HTML, CSS, JavaScript, images
**Does NOT support:** PHP, Python, Node.js, Java (server-side code)

> ⚠️ **Exam Trap:** Getting a **403 Forbidden** error?
> 1. Check **Block Public Access is OFF**
> 2. Check **Bucket Policy allows public reads**

---

## 8. Versioning

Keeps **multiple versions** of the same object in a bucket.

```
Upload photo.jpg (v1)  →  version: null
Upload photo.jpg (v2)  →  version: abc123
Upload photo.jpg (v3)  →  version: xyz789
                                    │
Delete photo.jpg       →  adds Delete Marker (versions still safe!)
Restore               →  delete the Delete Marker → file is back ✅
```

**Key rules:**
- Enabled at the **bucket level**
- Before enabling → version ID is **null**
- Suspending versioning does **NOT delete** old versions — they stay
- Required to enable **S3 Replication**

> ⚠️ **Versioning Cost Trap:** Every version is stored and billed separately. Overwriting a 5 GB file 10 times = 50 GB in storage costs. Always set a **Lifecycle Rule for Noncurrent Versions** to clean up old ones.

---

## 9. Replication (CRR & SRR)

Automatically copy objects from one bucket to another — asynchronously.

```
Source Bucket ──── async replication ────► Target Bucket
  (Region A)                               (Region A or B)
```

| Type | Full Name | Use Cases |
|------|-----------|-----------|
| **CRR** | Cross-Region Replication | Compliance, low latency, cross-account DR |
| **SRR** | Same-Region Replication | Log aggregation, prod → test mirroring |

**Requirements:**
- Versioning must be **enabled on BOTH** buckets
- Proper **IAM permissions** must be given
- Buckets can be in **different AWS accounts**

**Important rules:**
- Only **new objects** replicate after enabling — use **S3 Batch Replication** for existing objects
- Delete markers are **optionally** replicated
- Deletions with a **Version ID are NOT replicated** (prevents accidental deletion spreading)
- **No chaining** — Bucket A → B → C does NOT auto-replicate A objects to C

---

## 10. Storage Classes

> All storage classes share the same **11 nines durability**. What changes is retrieval speed, availability and cost.

### Quick Comparison Table

| Storage Class | Retrieval Time | Min Storage | Use Case | Cost |
|---------------|---------------|-------------|----------|------|
| **S3 Standard** | Milliseconds | None | Frequently accessed data | Highest |
| **S3 Intelligent-Tiering** | Milliseconds | None | Unknown/unpredictable access | Medium + monitoring fee |
| **S3 Standard-IA** | Milliseconds | 30 days | Monthly access, fast retrieval needed | ~45% less than Standard |
| **S3 One Zone-IA** | Milliseconds | 30 days | Re-creatable data, single AZ | ~20% less than Standard-IA |
| **S3 Glacier Instant** | Milliseconds | 90 days | Quarterly access, instant retrieval | Much cheaper |
| **S3 Glacier Flexible** | 1–5 min / 3–5 hrs / 5–12 hrs | 90 days | Archive, 1–2x per year | Low |
| **S3 Glacier Deep Archive** | 12 hrs / 48 hrs | 180 days | Compliance, almost never accessed | Lowest (~$0.001/GB) |
| **S3 Express One Zone** | Microseconds | None | ML training, real-time analytics | High, lowest latency |

### Decision Tree — Which class to pick?

```
How often do you access the data?

Daily / Weekly
└──► S3 Standard

Unknown / Unpredictable
└──► S3 Intelligent-Tiering

Monthly — need fast access
└──► S3 Standard-IA

Monthly — data is re-creatable, want cheaper
└──► S3 One Zone-IA

Quarterly — need instant access
└──► S3 Glacier Instant Retrieval

1–2x per year — can wait minutes/hours
└──► S3 Glacier Flexible Retrieval

Yearly or never — legal/compliance
└──► S3 Glacier Deep Archive

ML / Real-time — need microsecond latency
└──► S3 Express One Zone
```

---

## 11. Lifecycle Rules

Automatically move objects between storage classes based on age — set once, AWS handles it.

```
Day 0   →  Upload to S3 Standard
           │
Day 30  ▼  Move to S3 Standard-IA       (saves ~45%)
           │
Day 90  ▼  Move to S3 Glacier Flexible  (saves more)
           │
Day 365 ▼  Move to Glacier Deep Archive (cheapest)
           │
Day 730 ▼  Delete permanently
```

**Two types of rules:**
- **Transition Actions** — move to a cheaper class after X days
- **Expiration Actions** — delete objects or old versions after X days

> ⚠️ **Exam Trap:** Min storage durations still apply. Moving to Standard-IA on Day 15 = still charged for 30 days.

---

## 12. Exam Traps — Quick Ref

| Trap | Correct Answer |
|------|----------------|
| S3 is global? | ❌ Regional — bucket names are globally unique but buckets live in a region |
| Max object size | ✅ **50 TB** (not 5 TB) |
| Folders in S3? | ❌ No real folders — everything is a key (prefix + object name) |
| Suspend versioning = delete old versions? | ❌ Old versions stay |
| Replication copies existing objects? | ❌ Only new objects — use Batch Replication for existing |
| Version ID deletions replicate? | ❌ Never replicated |
| 403 on S3 website | Block Public Access still ON or bucket policy missing |
| One Zone-IA risk | Data permanently lost if AZ is destroyed |
| Glacier Flexible = instant? | ❌ Expedited = 1–5 mins minimum |

---

*→ Next: S3-Advanced.md*
*→ Next: S3-Security.md*
