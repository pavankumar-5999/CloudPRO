# Amazon S3 — Advanced


**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. S3 Performance — Throughput Limits

S3 auto-scales but has **per-prefix throughput limits:**

| Operation | Limit per prefix per second |
|-----------|-----------------------------|
| **Reads** (GET/HEAD) | **5,500 requests** |
| **Writes** (PUT/POST/DELETE/COPY) | **3,500 requests** |

> 💡 These limits are **per prefix** — spread objects across multiple prefixes to multiply throughput.

**Bad pattern — all hits one prefix:**
```
/logs/app-error-1.log
/logs/app-error-2.log   ← all under /logs/ → hits limit fast ❌
/logs/app-error-3.log
```

**Good pattern — spread across prefixes:**
```
/logs/2026-01-01/app-error-1.log   ← 5,500 reads/sec
/logs/2026-01-02/app-error-2.log   ← 5,500 reads/sec
/logs/2026-01-03/app-error-3.log   ← 5,500 reads/sec
                                      ─────────────────
                                      16,500 reads/sec total ✅
```

---

## 2. Multipart Upload (Advanced Detail)

| Rule | Value |
|------|-------|
| Recommended for | Files **> 100 MB** |
| Mandatory for | Files **> 5 GB** |
| Max parts | 10,000 parts |
| Min part size | 5 MB (except last part) |

> ⚠️ **Hidden cost trap:** Incomplete Multipart Uploads — if an upload fails midway and is never completed, the partial parts are **still stored and billed**. Set a **Lifecycle Rule to delete incomplete uploads** after X days.

---

## 3. S3 Byte-Range Fetches

Download **specific byte ranges** of an object in parallel instead of the whole file.

```
Full File (500 MB)
├── Bytes 0–100MB      ← Thread 1 ─┐
├── Bytes 100–200MB    ← Thread 2  │
├── Bytes 200–300MB    ← Thread 3  ├──► All downloaded in parallel ✅
├── Bytes 300–400MB    ← Thread 4  │
└── Bytes 400–500MB    ← Thread 5 ─┘
```

**Two use cases:**

| Use Case | How |
|----------|-----|
| **Speed up downloads** | Download different byte ranges in parallel |
| **Fetch partial file** | Read just the header/first few bytes without downloading the full file |

> ⚠️ **Exam Tip:** Speed up downloads → **Byte-Range Fetches**. Speed up uploads → **Multipart Upload**.

---

## 4. S3 Transfer Acceleration

**Problem:** Uploading from far away (e.g. Australia → S3 bucket in us-east-1) is slow over the public internet.

**Solution:** Route through the **nearest AWS Edge Location** → then use AWS's private network to reach the bucket.

```
User in Australia
      │
      ▼ public internet (short hop)
Nearest Edge Location (Sydney)
      │
      ▼ AWS private network (ultra fast)
S3 Bucket in us-east-1 ✅
```

- Compatible with **Multipart Upload**
- Great for globally distributed users uploading to one central bucket

---

## 5. S3 Pre-Signed URLs

Give **temporary access** to a private S3 object without needing AWS credentials.

```
App Backend ──generates──► Pre-Signed URL (expires in 20 mins)
                                    │
                                    ▼
                    User uploads/downloads directly ✅
                    (no IAM account needed)
```

**Key facts:**

| Detail | Value |
|--------|-------|
| Generated via | S3 Console, AWS CLI, SDK |
| Max expiry (Console) | 12 hours |
| Max expiry (CLI) | 168 hours (7 days) |
| Works for | Both upload AND download |
| Inherits permissions of | The IAM user/role that generated it |

**Common use cases:**
- Share a private file temporarily (e.g. premium video download)
- Allow users to upload directly into a private bucket

> ⚠️ **Exam Trap:** If the IAM role generating the URL has no write access, the Pre-Signed URL for upload won't work either — it inherits the generator's permissions.

---

## 6. S3 Select & Glacier Select

Instead of downloading an entire object and filtering in your app, let **S3 do the filtering server-side**.

```
Without S3 Select:
S3 downloads 5GB CSV → Your app filters it → Gets 100KB of data needed ❌ (slow + expensive)

With S3 Select:
S3 filters server-side → Returns only 100KB of data needed ✅ (fast + cheap)
```

- Uses simple **SQL expressions** to filter data
- Works on CSV, JSON, Parquet files
- Up to **400% faster** and **80% cheaper** than downloading the full object

---

## 7. S3 Event Notifications

S3 can automatically **trigger actions** when events happen on objects.

```
Object Created / Deleted / Restored
            │
            ▼
    S3 Event Notification
            │
      ┌─────┴──────┐
      ▼            ▼            ▼
   Lambda        SNS          SQS
  (run code)  (notify)    (queue job)
```

**Common use cases:**
- Auto-generate thumbnails when an image is uploaded (Lambda)
- Send a notification when a file is deleted (SNS)
- Queue a processing job when a new file arrives (SQS)

> 💡 Events can also be sent to **Amazon EventBridge** for more advanced routing to 18+ AWS services.

---

## 8. S3 Requester Pays

Normally the **bucket owner** pays for storage AND data transfer costs.

With **Requester Pays** enabled:
- Bucket owner pays for **storage only**
- The person downloading pays for **data transfer**

**Use when:** Sharing large datasets publicly (e.g. research data, open datasets) — you don't want to pay for everyone downloading it.

> ⚠️ Requesters must be **authenticated AWS users** — anonymous access not allowed with Requester Pays.

---

## 9. S3 Object Lambda

Run a **Lambda function** to transform an object **as it is being downloaded** — no need to store a modified copy.

```
App requests object
      │
      ▼
S3 Object Lambda Access Point
      │
      ▼ Lambda transforms data on the fly
      │
      ▼
Modified object returned to app ✅
(original in S3 is unchanged)
```

**Use cases:**
- Strip **PII data** before delivering to analytics teams
- **Watermark images** on the fly
- Convert **XML to JSON** dynamically
- Enrich data with info from external databases

---

## 10. S3 Access Points

**Problem:** One massive bucket policy for 100 teams = chaos.

**Solution:** Each team gets their own **Access Point** with their own policy.

```
Finance Team ──► Access Point (read/write /finance/*)  ─┐
Sales Team   ──► Access Point (read-only  /sales/*)   ─┼──► Same S3 Bucket
HR Team      ──► Access Point (read-only  /hr/*)      ─┘
```

**Key facts:**
- Each Access Point has a **unique DNS name**
- Each has its own **access policy**
- Can be configured for **VPC-only access** (no public internet)
- Simplifies bucket policy management significantly

---

## 11. S3 Batch Operations

Run **bulk operations** on millions of existing S3 objects with a single request.

**Common operations:**
- Copy objects between buckets
- Encrypt unencrypted objects
- Restore objects from Glacier
- Invoke Lambda on each object
- Modify ACLs or tags

> 💡 **S3 Batch Replication** uses this to replicate existing objects that were in a bucket **before replication was enabled**.

---

## 12. Amazon Athena + S3

**Athena** is a **serverless SQL query engine** that runs queries directly on files stored in S3 — no database needed.

```
S3 Bucket (CSV / JSON / Parquet files)
      │
      ▼
Amazon Athena (run SQL queries)
      │
      ▼
Results / Insights ✅
```

- Works with CSV, JSON, Parquet, ORC files
- Pay per query (per TB scanned)
- No infrastructure to manage

**Real-world (Sysco):**
> Sysco stores business data in S3 → Athena queries it → insights on delivery routes, demand, product performance — no traditional database needed.

**Exam scenario:**
> "A startup stores logs as JSON in S3 and needs ad-hoc queries without a database."
> ✅ Answer: **Amazon Athena**

---

## 13. S3 Strong Consistency

S3 provides **strong read-after-write consistency** for all operations.

```
PUT object ──► immediately visible on next GET ✅
DELETE object ──► immediately reflected on next GET ✅
```

- No eventual consistency delays
- Updates are **atomic** — you either see the full update or nothing
- Applies to all AWS regions

> 💡 This used to be an issue (eventual consistency) but AWS fixed it in December 2020. Now all S3 reads are strongly consistent.

---

## 14. Real Exam Scenarios

**Scenario 1 — Large unstable upload**
> "Backup 50 TB of on-premise data over an unstable connection to S3."
> ✅ **Multipart Upload** — only failed chunks need retrying

**Scenario 2 — Speed up global uploads**
> "Users across the world uploading files to one S3 bucket in us-east-1 are experiencing slow speeds."
> ✅ **S3 Transfer Acceleration** — routes via nearest Edge Location

**Scenario 3 — Speed up downloads**
> "Application is slow downloading 5 GB files from S3."
> ✅ **Byte-Range Fetches** — download in parallel chunks

**Scenario 4 — Temporary access without IAM**
> "Allow users to upload profile photos directly to S3 without permanent AWS accounts."
> ✅ **Pre-Signed URL** with short expiry

**Scenario 5 — Query S3 files without database**
> "Run SQL queries on JSON log files stored in S3."
> ✅ **Amazon Athena**

**Scenario 6 — Strip PII before delivering data**
> "Deliver S3 objects to analytics team without sensitive personal data, without storing a copy."
> ✅ **S3 Object Lambda**

**Scenario 7 — Unpredictable access patterns**
> "Reduce storage costs without changing app code — access patterns are unknown."
> ✅ **S3 Intelligent-Tiering**

---

*→ See also: S3-Basics.md*
*→ See also: S3-Security.md*

---

## 15. S3 Inventory

Generates a **scheduled report** of all objects in your bucket — their metadata, encryption status, replication status etc.

```
S3 Bucket
    │
    ▼ (daily or weekly)
S3 Inventory Report (CSV / ORC / Parquet)
    │
    ▼
Saved to another S3 bucket
    │
    ▼
Analyse with Athena ✅
```

**Key facts:**
- Reports generated **daily or weekly**
- Output formats: **CSV, ORC, Parquet**
- Covers: object size, last modified, encryption status, replication status, storage class

**Use cases:**
- Audit if all objects in a bucket are encrypted
- Check replication status across objects
- Compliance reporting

> ⚠️ **Exam Tip:** "How to audit encryption status of all S3 objects?" → **S3 Inventory**

---

## 16. S3 Storage Lens

Organisation-wide **analytics dashboard** for S3 usage and activity across all accounts and regions.

```
AWS Organisation
    │
    ├── Account A (S3 buckets)
    ├── Account B (S3 buckets)  ──► S3 Storage Lens Dashboard ✅
    └── Account C (S3 buckets)
```

**What it shows:**
- Total storage used across all accounts
- Object count trends
- Cost optimisation recommendations
- Data protection insights (versioning, encryption coverage)
- Activity metrics (requests, downloads)

**Two tiers:**

| Tier | Cost | What you get |
|------|------|-------------|
| **Free** | Free | 28 usage metrics, 14-day retention |
| **Advanced** | Paid | 35+ metrics including activity, 15-month retention |

> ⚠️ **Exam Tip:** "Get a single view of S3 storage usage across all AWS accounts in an Organisation" → **S3 Storage Lens**

