# Amazon S3 — Simple Storage Service

---

## 1. What is Amazon S3?

Amazon S3 (Simple Storage Service) is AWS's **object storage service** — it lets you store, retrieve and manage any amount of data from anywhere over the internet.

Think of it like this:
- A **bucket** = a folder (container to hold your files)
- An **object** = a file stored inside the bucket

> 💡 **Fun Fact:** S3 was the second AWS service ever launched, right after Amazon SQS.

---

## 2. Why S3?

Before cloud storage, companies stored data on **physical servers in their own data centres** — expensive, complex and hard to scale.

S3 solved this by offering:

- **99.999999999% (11 nines) durability** — your data is extremely safe
- **Virtually unlimited scalability** — store as much as you want
- **High availability** — always accessible
- **Strong security** — multiple layers of access control
- **Pay-as-you-go** — only pay for what you use
- **Easy AWS integration** — works with Lambda, EC2, CloudFront and more

---

## 3. Real-World Use Cases

| Company | What they use S3 for |
|---------|----------------------|
| **Nasdaq** | Stores 7 years of financial trade records in S3 Glacier (legally required) |
| **Sysco** | Runs analytics on business data using S3 + Redshift/Athena for insights |

**Common use cases:**
- Website image and asset hosting
- Company backups and disaster recovery
- Application logs
- Data lakes and big data analytics
- Media streaming
- Software delivery
- Static website hosting
- Hybrid cloud storage

---

## 4. S3 Buckets

```
S3 looks like a global service — but buckets are created in a specific AWS Region
```

**Key rules:**
- Bucket names must be **globally unique** across all AWS accounts
- A bucket lives in **one specific region**
- You can store **virtually unlimited** data in a bucket
- Bucket names must be **lowercase, no spaces, no underscores**

---

## 5. S3 Objects

Every file you store in S3 is called an **object**.

**Key concepts:**

| Concept | Explanation |
|---------|-------------|
| **Key** | The full path of the object — e.g. `s3://my-bucket/folder/file.txt` |
| **Value** | The actual content/data of the file |
| **Max object size** | **50 TB** |
| **Max single upload** | 5 GB — if larger, use Multipart Upload |
| **Metadata** | Key/value pairs describing the object (set by system or user) |
| **Tags** | Up to 10 Unicode key/value pairs — useful for security and lifecycle |
| **Version ID** | Assigned if versioning is enabled on the bucket |

> ⚠️ **Exam Trap:** There are **no real directories** in S3 — everything is a key. A folder-like structure is just part of the key name (prefix + object name).

### Multipart Upload

If you're uploading a file **larger than 5 GB**, you must use **Multipart Upload**:
- Splits the file into smaller parts
- Uploads parts in **parallel**
- S3 combines them into one object at the end
- **Benefits:** Faster uploads, better reliability, easier recovery if a part fails

---

## 6. S3 Security

### 6.1 Types of Access Control

```
Two main ways to control access:
1. User-Based  → IAM Policies
2. Resource-Based → Bucket Policies, ACLs
```

| Type | What it is | When to use |
|------|------------|-------------|
| **IAM Policies** | Attached to a user/role — controls which API calls they can make | Control what a specific user can do |
| **Bucket Policy** | JSON policy attached to the bucket — controls who accesses the bucket | Grant public access, cross-account access |
| **Object ACL** | Fine-grained per-object control | Rarely used, can be disabled |
| **Bucket ACL** | Less common than bucket policy | Rarely used, can be disabled |

### 6.2 The Golden Rule

> ✅ An IAM principal **CAN** access an S3 object if:
> - The **IAM policy ALLOWS** it **OR** the **resource policy ALLOWS** it
> - **AND** there is **no explicit DENY**

### 6.3 Common Security Scenarios

| Scenario | Solution |
|----------|----------|
| Make bucket publicly accessible | Write a **Bucket Policy** allowing public access |
| Give an IAM user access to S3 | Assign an **IAM Policy** to the user |
| Give an EC2 instance access to S3 | Assign an **IAM Role** to the EC2 instance |
| Allow another AWS account access | Write a **Bucket Policy** for cross-account access |

### 6.4 Encryption Options

| Type | What it means |
|------|---------------|
| **SSE-S3** | AWS manages the encryption keys entirely |
| **SSE-KMS** | You use AWS KMS — more control, auditing and compliance |
| **SSE-C** | You provide and manage your own keys |
| **Client-Side** | You encrypt before uploading |

### 6.5 Block Public Access

S3 has a **Block Public Access** setting at the account and bucket level.
- On by default — prevents accidental public exposure
- Must be turned off explicitly if you want a public bucket

> ⚠️ **Exam Trap:** If you get a **403 Forbidden** error on a static website hosted on S3 — check that **Block Public Access is turned OFF** and the bucket policy allows public reads.

---

## 7. S3 Bucket Policies

Bucket policies are **JSON-based** and applied at the bucket level.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**JSON fields explained:**

| Field | What it does |
|-------|-------------|
| **Effect** | Allow or Deny |
| **Principal** | Who this applies to (`*` = everyone) |
| **Action** | Which S3 API actions are allowed/denied |
| **Resource** | Which bucket or objects this applies to |

---

## 8. Static Website Hosting

S3 can host **static websites** directly.

- The website URL depends on the region
- Ideal for: portfolio sites, landing pages, documentation

> ⚠️ **Important:** S3 only supports **static** content (HTML, CSS, JS, images).
> It **does NOT** run server-side code like Python, PHP, Java or Node.js.

---

## 9. S3 Versioning

Versioning lets you keep **multiple versions** of the same object in a bucket.

- Enabled at the **bucket level**
- Before enabling: version ID is **null** by default
- Suspending versioning does **NOT delete** previous versions

**Benefits:**
- Recover accidentally deleted files
- Restore overwritten objects
- Rollback to any previous version

> ⚠️ **Exam Trap:** Suspending versioning ≠ deleting old versions. Old versions remain.

---

## 10. S3 Replication

S3 supports **asynchronous replication** between buckets.

```
Source Bucket ──async──► Target Bucket
```

| Type | Full Name | Use Case |
|------|-----------|----------|
| **CRR** | Cross-Region Replication | Compliance, low latency, cross-account replication |
| **SRR** | Same-Region Replication | Log aggregation, live replication between prod and test |

**Requirements:**
- Versioning must be **enabled on both** source and destination buckets
- Buckets can be in **different AWS accounts**
- Proper **IAM permissions** must be given to S3

**Important rules:**
- Only **new objects** are replicated after enabling replication
- To replicate existing objects → use **S3 Batch Replication**
- Delete markers can be optionally replicated — but deletions with a **version ID are NOT replicated**
- **No chaining** — if bucket 1 replicates to bucket 2, and bucket 2 replicates to bucket 3, objects from bucket 1 do **NOT** automatically go to bucket 3

---

## 11. S3 Storage Classes

> 💡 **Key concept:** All storage classes have the **same 11 nines durability**. What changes is **retrieval speed, cost, and availability**.

You can move between storage classes **manually** or automatically using **S3 Lifecycle Rules**.

---

### Storage Classes — Quick Comparison Table

| Storage Class | Retrieval Time | Min Storage Duration | Use Case | Cost |
|---------------|---------------|---------------------|----------|------|
| **S3 Standard** | Milliseconds | None | Frequently accessed data | Highest |
| **S3 Intelligent-Tiering** | Milliseconds | None | Unknown/changing access patterns | Medium (monitoring fee applies) |
| **S3 Standard-IA** | Milliseconds | 30 days | Infrequently accessed, needs fast retrieval | ~45% less than Standard |
| **S3 One Zone-IA** | Milliseconds | 30 days | Re-creatable infrequent data, single AZ | ~20% less than Standard-IA |
| **S3 Glacier Instant Retrieval** | Milliseconds | 90 days | Rarely accessed, needs instant access | Much cheaper than Standard |
| **S3 Glacier Flexible Retrieval** | 1–5 min / 3–5 hrs / 5–12 hrs | 90 days | Archive, accessed 1–2x per year | Low |
| **S3 Glacier Deep Archive** | 12 hrs / up to 48 hrs | 180 days | Long-term compliance, almost never accessed | Lowest (~$0.001/GB) |
| **S3 Express One Zone** | Single-digit milliseconds | None | ML training, high-performance workloads | High but lowest latency |

---

### 11.1 S3 Standard (General Purpose)
- **Retrieval:** Milliseconds, no retrieval fee
- **Availability:** 99.99%
- **Use when:** Data is accessed daily or weekly — user uploads, web assets, active application data, CDN origins
- **Do NOT use for:** Backups older than 30 days, log archives, data you rarely touch

---

### 11.2 S3 Intelligent-Tiering
- **Retrieval:** Milliseconds (no retrieval fee between tiers)
- **How it works:** AWS monitors each object's access pattern and automatically moves it between tiers:
  - **Frequent Access** → accessed regularly
  - **Infrequent Access** → not accessed for 30 days
  - **Archive Instant Access** → not accessed for 90 days
  - **Deep Archive Access** → not accessed for 180 days
- **Use when:** You can't predict how often data will be accessed
- **Watch out:** A small **per-object monitoring fee** applies — not ideal for very small objects under 128 KB

---

### 11.3 S3 Standard-IA (Infrequent Access)
- **Retrieval:** Milliseconds — but has a **retrieval fee per GB**
- **Min storage duration:** 30 days (charged even if deleted earlier)
- **Use when:** Data accessed monthly — backups, disaster recovery files, older logs
- **Watch out:** Per-GB retrieval fee — don't use for data accessed multiple times a month

---

### 11.4 S3 One Zone-IA
- Same as Standard-IA but stored in **only ONE Availability Zone**
- **Retrieval:** Milliseconds
- **Min storage duration:** 30 days
- **Use when:** Data that can be **re-created** if lost — thumbnails, transcoded video, CRR replicas
- **⚠️ Risk:** If that AZ is physically destroyed, **data is lost permanently**

---

### 11.5 S3 Glacier Instant Retrieval
- **Retrieval:** Milliseconds — same speed as Standard but much cheaper storage
- **Min storage duration:** 90 days
- **Use when:** Rarely accessed data (once per quarter) that still needs **instant access** — medical records, compliance archives, archived photos
- **Real-world:** Nasdaq-style financial records that must be kept 7 years but accessed instantly when auditors ask

---

### 11.6 S3 Glacier Flexible Retrieval
- **Retrieval options:**
  - **Expedited:** 1–5 minutes (costs more)
  - **Standard:** 3–5 hours
  - **Bulk:** 5–12 hours (free)
- **Min storage duration:** 90 days
- **Use when:** Archive data accessed 1–2 times per year — backup/DR, historical datasets, tape replacement
- **⚠️ Exam Trap:** Do NOT use if someone might need the data urgently (e.g. on-call engineer at 2am needs a restore)

---

### 11.7 S3 Glacier Deep Archive
- **Retrieval options:**
  - **Standard:** ~12 hours
  - **Bulk:** up to 48 hours
- **Min storage duration:** 180 days
- **Cost:** ~$0.001/GB — cheapest storage on AWS (~23x cheaper than Standard)
- **Use when:** Data you must keep but almost never touch — 7-year financial records, healthcare data, legal holds, compliance archives
- **Real-world:** Nasdaq 7-year regulatory storage — this is the right class for it

---

### 11.8 S3 Express One Zone
- **Retrieval:** Single-digit milliseconds — the fastest S3 storage
- **Stored in:** A single AZ (Directory Bucket)
- **Use when:** ML training datasets accessed hundreds of times per day, real-time analytics, high-performance compute workloads
- **Note:** Higher cost but lowest latency of all classes

---

### Storage Class Decision Tree

```
How often do you access the data?

Daily / Weekly
└──► S3 Standard

Unknown / Unpredictable access pattern
└──► S3 Intelligent-Tiering

Monthly — but need it fast when accessed
└──► S3 Standard-IA

Monthly — data is re-creatable, want to save more
└──► S3 One Zone-IA

Quarterly — still need instant access
└──► S3 Glacier Instant Retrieval

1-2x per year — can wait minutes to hours
└──► S3 Glacier Flexible Retrieval

Yearly or never — compliance/legal requirement
└──► S3 Glacier Deep Archive

ML / High-performance — need single-digit ms latency
└──► S3 Express One Zone
```

---

## 12. S3 Lifecycle Rules

Lifecycle rules let you **automatically move objects** between storage classes based on age.

**Example lifecycle flow:**
```
Upload to S3 Standard
     │
     ▼ After 30 days
S3 Standard-IA
     │
     ▼ After 90 days
S3 Glacier Flexible Retrieval
     │
     ▼ After 365 days
S3 Glacier Deep Archive
```

> 💡 Set it once and forget it — AWS handles the transitions automatically and you save money without changing your application.

---

## 13. Important S3 Features Summary

| Feature | What it does | Key exam point |
|---------|-------------|----------------|
| **Versioning** | Keeps multiple versions of same object | Null version before enabling; suspending doesn't delete old versions |
| **Replication (CRR/SRR)** | Async copy to another bucket | Versioning required on both; no chaining |
| **Lifecycle Rules** | Auto-move objects between storage classes | Based on object age or last-access |
| **Default Encryption** | Auto-encrypts every new object | SSE-S3 (AWS managed) or SSE-KMS (you control) |
| **Server Access Logging** | Logs every request to your bucket | Used for auditing and troubleshooting |
| **Event Notifications** | Triggers on object create/delete | Can invoke Lambda, SNS, SQS |
| **Object Lock** | Prevents modify/delete for a set period | WORM compliance, ransomware protection |
| **Static Website Hosting** | Host HTML/CSS/JS websites | Static only — no server-side code |
| **Tags** | Key-value labels on objects | Cost allocation, access management, automation |
| **Multipart Upload** | Split large files, upload in parallel | Required for files > 5 GB |

---

## 14. Exam Traps — Watch Out For These!

> These come up directly in SAA-C03 questions

1. **S3 is regional, not global** — bucket names are globally unique but buckets live in a region
2. **Max object size = 50 TB**, must use Multipart Upload if uploading > 5 GB
3. **There are no real directories** in S3 — everything is a key (prefix + object name)
4. **Suspending versioning ≠ deleting old versions** — they stay
5. **Replication only applies to new objects** after enabling — use Batch Replication for existing ones
6. **Delete markers are not replicated** by default with version ID deletions
7. **No chaining in replication** — bucket 1 → bucket 2 → bucket 3 does NOT auto-replicate from 1 to 3
8. **403 Forbidden on S3 website** = bucket policy is missing or Block Public Access is still ON
9. **One Zone-IA = data can be permanently lost** if the AZ is destroyed
10. **Glacier Flexible Retrieval is NOT instant** — even Expedited takes 1–5 minutes

---

## 15. One-Line Summary

> Amazon S3 is a regional, infinitely scalable object storage service where files (objects) are stored in buckets — choose the right storage class based on how often you access your data to balance cost and speed.

---

*Notes prepared from Maarek's SAA-C03 course + AWS official documentation*

---

## 16. S3 Pre-Signed URLs

A **Pre-Signed URL** gives temporary access to a private S3 object without needing an AWS account or IAM credentials.

- Generated by your application backend using AWS SDK or CLI
- Has an **expiration time** (e.g. 20 minutes, 1 hour)
- Works for both **upload** and **download**

**Example use case:**
> A media company wants users to upload profile photos directly into S3 without giving them permanent IAM access → Generate a Pre-Signed URL with a 15-minute expiry

```
App Backend ──generates──► Pre-Signed URL (expires in 20 mins)
                                    │
                                    ▼
                         User uploads directly to S3
                         (no IAM account needed)
```

> ⚠️ **Exam Trap:** Pre-Signed URLs inherit the permissions of the IAM user/role that generated them — if that role has no write access, the URL won't work either.

---

## 17. S3 Transfer Acceleration

**Problem:** Uploading large files from a location far from your S3 bucket region is slow over the public internet.

**Solution:** S3 Transfer Acceleration routes your upload through the **nearest AWS Edge Location** first, then uses AWS's private fiber backbone to reach your bucket — much faster than the public internet.

```
User in Australia
      │
      ▼ (public internet — short hop)
Nearest Edge Location (Sydney)
      │
      ▼ (AWS private network — ultra fast)
S3 Bucket in us-east-1
```

**Use when:** Globally distributed users uploading to a single centralized bucket.

---

## 18. S3 Access Points

**Problem:** Managing one massive bucket policy for 100+ teams becomes unmanageable.

**Solution:** Create **Access Points** — dedicated endpoints with their own access policies sitting on top of the same bucket.

- Each team/application gets its own Access Point with tailored permissions
- Simplifies permission management significantly
- Each Access Point has its own DNS name

```
Team A ──► Access Point A (read-only /teamA/*)
Team B ──► Access Point B (read/write /teamB/*)
         └──────────────► Same S3 Bucket
Team C ──► Access Point C (read-only /shared/*)
```

---

## 19. S3 Object Lambda

**S3 Object Lambda** lets you run a **Lambda function** to transform an object **as it is being downloaded** — without storing a modified copy.

**Use cases:**
- Strip out **PII data** before delivering to analytics teams
- **Watermark images** on the fly
- Convert file formats based on who is requesting

```
App requests object
      │
      ▼
S3 triggers Lambda function
      │
      ▼ (Lambda modifies the data on the fly)
Modified object returned to app
(original in S3 is unchanged)
```

---

## 20. S3 Performance — Throughput Limits

S3 automatically scales but has **per-prefix throughput limits:**

| Operation | Limit per prefix per second |
|-----------|----------------------------|
| **Reads** (GET/HEAD) | **5,500 requests** |
| **Writes** (PUT/POST/DELETE/COPY) | **3,500 requests** |

> 💡 **Key insight:** These limits are **per prefix** — so spread your objects across multiple prefixes to multiply your throughput.

**Bad pattern (all hits one prefix):**
```
/logs/app-error-1.log
/logs/app-error-2.log   ← all under /logs/ → hits 5,500 limit fast
/logs/app-error-3.log
```

**Good pattern (spread across prefixes):**
```
/logs/2026-01-01/app-error-1.log
/logs/2026-01-02/app-error-2.log   ← each prefix gets 5,500 read limit
/logs/2026-01-03/app-error-3.log
```

**Also for Multipart Upload:**
- **Mandatory** for files > **5 GB**
- **Strongly recommended** for files > **100 MB** — uploads in parallel, faster and more resilient

---

## 21. S3 Object Lock

Object Lock prevents objects from being **modified or deleted** for a defined retention period.

**Two modes:**

| Mode | Who can override it? | Use case |
|------|---------------------|----------|
| **Governance Mode** | Admins with special permission can override | Internal compliance, audit trails |
| **Compliance Mode** | **Nobody can override — not even root account** | Strict regulatory compliance (banking, healthcare) |

> ⚠️ **Exam Trap:** If the question says "no user including root account can delete" → answer is **Object Lock in Compliance Mode**

**Legal Hold:** Locks an object indefinitely with no expiry date — until explicitly removed by a user with permission.

---

## 22. Encryption in Transit

S3 supports **HTTPS** to encrypt data moving between your client and S3.

- You can enforce HTTPS-only by writing a **bucket policy** that **denies HTTP requests**:

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::my-bucket/*",
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "false"
    }
  }
}
```

> This blocks any request coming over unencrypted HTTP — only HTTPS allowed.

---

## 23. Amazon Athena + S3

**Athena** is a **serverless query engine** that lets you run **SQL queries directly on files stored in S3** — no database setup needed.

- Works with CSV, JSON, Parquet, ORC files
- Pay per query (per TB scanned)
- Commonly paired with S3 for log analysis, data lakes, ad-hoc analytics

```
S3 Bucket (CSV / JSON / Parquet files)
      │
      ▼
Amazon Athena (run SQL queries)
      │
      ▼
Results / Insights
```

**Real-world example (Sysco):**
> Sysco stores business data in S3 and uses Athena to query it → get insights on delivery routes, demand forecasting, product performance — all without a traditional database.

---

## 24. Real Exam Scenarios — Recognise the Pattern

These types of scenarios come up directly in SAA-C03 questions:

---

**Scenario 1 — Large file upload over unstable connection**
> "An engineering team wants to backup 50 TB of on-premise data over an unstable internet connection into S3."

✅ **Answer:** Use **Multipart Upload** — if the connection drops, only the current chunk is lost, not the entire 50 TB transfer.

---

**Scenario 2 — Temporary upload access without IAM**
> "A global media company wants authenticated users to upload files directly into a private S3 bucket without giving them permanent AWS accounts."

✅ **Answer:** Generate an **S3 Pre-Signed URL** with a short expiry (e.g. 20 minutes) from your application backend.

---

**Scenario 3 — High latency for users far from bucket**
> "Users in Australia are experiencing slow download times for static files stored in an S3 bucket in us-west-2."

✅ **Answer:** Set up an **Amazon CloudFront Distribution** pointing to the S3 bucket — files cache at edge locations globally, reducing latency and data transfer costs.

---

**Scenario 4 — Regulatory data that cannot be deleted by anyone**
> "A banking platform must ensure records cannot be modified or deleted by any user, including the root account, for exactly 5 years."

✅ **Answer:** Enable **S3 Object Lock in Compliance Mode** with a 5-year retention period.

---

**Scenario 5 — Unpredictable access patterns, need to cut costs**
> "An application dumps logs into S3 with unpredictable usage — some files are queried multiple times an hour, others are never touched again. How to reduce costs without changing application code?"

✅ **Answer:** Move data to **S3 Intelligent-Tiering** — AWS automatically monitors access and moves objects between tiers with no retrieval fee.

---

**Scenario 6 — Need to query logs stored in S3 without a database**
> "A startup stores application logs as JSON files in S3 and needs to run ad-hoc queries on them without setting up a database."

✅ **Answer:** Use **Amazon Athena** — query S3 files directly with SQL, pay per query, no infrastructure needed.

---

**Scenario 7 — Strip PII before delivering data to another team**
> "A data team wants to deliver S3 objects to analytics teams but needs to remove sensitive personal information before delivery, without storing a separate copy."

✅ **Answer:** Use **S3 Object Lambda** — transform the object on the fly during download using a Lambda function.


---

# 25. S3 Lifecycle Rules

## What are Lifecycle Rules?

Imagine your application uploads thousands of files to Amazon S3 every day. Initially, these files are accessed frequently, but over time they become less important or are never accessed again.

Instead of manually moving or deleting old files, Amazon S3 provides **Lifecycle Rules**, which automatically manage your objects based on rules you define.

Lifecycle Rules help reduce storage costs and eliminate manual maintenance.

---

## Types of Lifecycle Actions

Amazon S3 supports two main lifecycle actions.

### 1. Transition Actions

Transition moves objects from one storage class to another as they become less frequently accessed.

Example:

```text
Day 0      → S3 Standard
Day 30     → S3 Standard-IA
Day 90     → S3 Glacier Flexible Retrieval
Day 365    → S3 Glacier Deep Archive
```

Since colder storage classes are cheaper, this helps reduce storage costs.

---

### 2. Expiration Actions

Expiration automatically deletes objects after a specified period.

Examples:

- Delete temporary files after 30 days
- Delete log files after 90 days
- Delete backups after one year
- Delete incomplete Multipart Uploads after 7 days

---

## Applying Lifecycle Rules

Lifecycle Rules can be applied to:

- Entire bucket
- Objects with a specific prefix
- Objects with specific tags

Example:

```text
images/
logs/
backups/
```

Each folder (prefix) can have its own lifecycle policy.

---

## Real-World Example

A photo-sharing application stores:

- Original images
- Thumbnail images

Original images remain in S3 Standard.

Thumbnail images are moved to Glacier after 60 days since they are rarely accessed.

---

## Benefits

- Reduce storage costs
- Automate storage management
- Remove unnecessary files
- Meet compliance requirements

> 💡 **Exam Tip:** Lifecycle Rules automatically transition or delete objects based on age.

---

# 26. S3 Requester Pays

## What is Requester Pays?

Normally, the **bucket owner pays** for:

- Storage
- Requests
- Data transfer

With **Requester Pays**, the bucket owner still pays for storage, but the **requester pays for download requests and data transfer charges**.

---

## Why use it?

Useful when sharing large datasets with many users.

Instead of the bucket owner paying for every download, each user pays for their own downloads.

---

## Real-World Example

A university publishes a **2 TB research dataset**.

Thousands of researchers download it every month.

Without Requester Pays:

- University pays for every download.

With Requester Pays:

- Researchers pay for their own downloads.

---

## Requirements

- Requester must have an AWS account.
- Anonymous users are not allowed.

> 💡 **Exam Tip:** Bucket owner pays for storage. Requester pays for download requests and data transfer.

---

# 27. S3 Event Notifications

## What are Event Notifications?

Amazon S3 can automatically notify other AWS services whenever something happens inside a bucket.

These are called **S3 Event Notifications**.

---

## Supported Events

- Object Created
- Object Deleted
- Object Restore
- Replication Events

---

## Event Destinations

Notifications can be sent to:

- AWS Lambda
- Amazon SNS
- Amazon SQS
- Amazon EventBridge

---

## Event Filtering

You can trigger notifications only for certain files.

Example:

```text
Only *.jpg
Only *.png
Only files inside images/
```

---

## Real-World Example

A user uploads a profile picture.

S3 automatically triggers a Lambda function.

Lambda creates a thumbnail and stores it back into S3.

---

## EventBridge Integration

Amazon EventBridge provides:

- Advanced filtering
- Routing to many AWS services
- Event-driven architectures

---

## Permissions

Before sending notifications:

- SNS requires an SNS Resource Policy.
- SQS requires an SQS Resource Policy.
- Lambda requires a Lambda Resource Policy.

---

> 💡 **Exam Tip:** S3 Event Notifications can trigger Lambda, SNS, SQS or EventBridge when objects are created, deleted or restored.

---

# 28. Amazon S3 Performance

Amazon S3 automatically scales to support very high request rates.

No manual capacity planning is required.

---

## Request Limits

Each prefix supports approximately:

- **3,500 PUT/COPY/POST/DELETE requests per second**
- **5,500 GET/HEAD requests per second**

Using multiple prefixes increases throughput automatically.

---

## Latency

Typical first-byte latency is around **100–200 ms**.

---

## Performance Features

Amazon S3 provides:

- Multipart Upload
- Transfer Acceleration
- Byte Range Fetches

These features improve upload and download performance.

---

# 29. Multipart Upload

## What is Multipart Upload?

Multipart Upload splits a large file into multiple smaller parts.

Each part uploads independently.

Once every part finishes, Amazon S3 combines them into one object.

---

## When should you use it?

Recommended:

- Files larger than **100 MB**

Required:

- Files larger than **5 GB**

Maximum object size:

- **5 TB**

---

## Benefits

- Faster uploads
- Parallel uploads
- Better bandwidth utilization
- Recover easily if upload fails

---

## Real-World Example

Uploading a **4 TB backup**.

Instead of uploading one massive file, Amazon S3 uploads hundreds or thousands of smaller chunks simultaneously.

If one chunk fails, only that chunk needs to be uploaded again.

---

# 30. S3 Transfer Acceleration

## What is Transfer Acceleration?

Transfer Acceleration speeds up uploads and downloads for users located far away from the S3 bucket.

Instead of sending data directly to the bucket, uploads first reach the nearest AWS Edge Location.

AWS then transfers the data across its private global network.

```text
User
   ↓
Nearest AWS Edge Location
   ↓
AWS Global Network
   ↓
S3 Bucket
```

---

## Best Use Cases

- Global users
- Large uploads
- Media files
- Backup uploads

---

## Benefits

- Faster uploads
- Lower latency
- Better long-distance performance

Works especially well with **Multipart Upload**.

---

# 31. Byte Range Fetches

## What are Byte Range Fetches?

Normally, downloading an object downloads the entire file.

Byte Range Fetches allow you to download only the required portion of an object.

---

## Example

Suppose you have a **5 GB video**.

Instead of downloading the whole file, your application downloads only the first few MB required for playback.

---

## Benefits

- Faster downloads
- Lower bandwidth usage
- Parallel downloads
- Better streaming performance

---

## Common Use Cases

- Video streaming
- Reading large log files
- Reading file headers
- Scientific datasets

---

# 32. S3 Batch Operations

## What are S3 Batch Operations?

S3 Batch Operations allow you to perform the same action on millions or even billions of S3 objects using a single managed job.

---

## Common Operations

- Copy objects
- Add tags
- Change ACLs
- Encrypt objects
- Restore Glacier objects
- Invoke Lambda for every object

---

## How does it work?

A Batch Operation consists of:

- List of objects
- Action to perform
- Optional parameters

Amazon S3 performs the work automatically.

---

## Creating the Object List

The object list can be generated using:

- S3 Inventory
- CSV files
- Amazon Athena queries

---

## Benefits

Compared to custom scripts:

- Automatic retries
- Progress tracking
- Completion reports
- Error reporting

---

## Real-World Example

Encrypt one million existing S3 objects without writing custom scripts.

---

> 💡 **Exam Tip:** Batch Operations perform bulk actions on millions of objects using one managed job.

---

# 33. S3 Storage Lens

## What is S3 Storage Lens?

S3 Storage Lens helps you monitor, analyze and optimize storage usage across your AWS environment.

It provides dashboards and recommendations to improve storage efficiency.

---

## What does it monitor?

Storage Lens provides metrics such as:

- Total storage
- Number of objects
- Average object size
- Storage growth
- Cost optimization opportunities
- Data protection status

---

## Scope

Metrics can be viewed for:

- AWS Organization
- AWS Account
- AWS Region
- Bucket
- Prefix

---

## Dashboard

Amazon provides a built-in dashboard showing:

- Storage trends
- Object growth
- Cost insights
- Security recommendations

---

## Free vs Advanced Metrics

### Free

- Storage usage
- Object count
- Basic dashboards

### Advanced (Paid)

- Activity metrics
- Cost optimization recommendations
- Prefix-level insights
- CloudWatch publishing

---

## Export Reports

Metrics can be exported to S3 in:

- CSV
- Parquet

for analysis using Amazon Athena.

---

## Benefits

- Reduce storage costs
- Improve security
- Monitor storage growth
- Optimize storage classes

> 💡 **Exam Tip:** Storage Lens provides organization-wide visibility into S3 usage, helping optimize cost, storage efficiency and data protection.
