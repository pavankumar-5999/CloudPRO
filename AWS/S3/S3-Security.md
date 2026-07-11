# Amazon S3 — Security


**Blog:** https://cloudpro.hashnode.dev/aws-s3-security

---

## 1. S3 Security Overview

S3 has **multiple layers** of security — you control who can access what, and how data is protected.

```
┌─────────────────────────────────────────────────────┐
│                   S3 Security Layers                │
│                                                     │
│  1. IAM Policies        → who can do what           │
│  2. Bucket Policies     → resource-level rules      │
│  3. Block Public Access → master safety switch      │
│  4. ACLs                → fine-grained (rare)       │
│  5. Encryption          → protect data at rest      │
│  6. HTTPS               → protect data in transit   │
│  7. MFA Delete          → prevent accidental delete │
│  8. Object Lock         → WORM compliance           │
└─────────────────────────────────────────────────────┘
```

---

## 2. Access Control — The Golden Rule

> ✅ A user **CAN** access an S3 object if:
> - IAM policy **ALLOWS** it **OR** Bucket policy **ALLOWS** it
> - **AND** there is **no explicit DENY**

```
Incoming Request
      │
      ▼
Explicit DENY anywhere? ──► YES → ❌ ACCESS BLOCKED
      │ NO
      ▼
Explicit ALLOW anywhere? ──► NO → ❌ ACCESS BLOCKED
      │ YES
      ▼
      ✅ ACCESS GRANTED
```

---

## 3. IAM Policies

Attached to **users, groups or roles** — controls what S3 actions they can perform.

| Scenario | Solution |
|----------|----------|
| Give a specific IAM user access to S3 | Assign **IAM Policy** to the user |
| Give an EC2 instance access to S3 | Assign **IAM Role** to the EC2 instance |
| Give a Lambda function access to S3 | Assign **IAM Role** to the Lambda function |

---

## 4. Bucket Policies

JSON-based rules attached **directly to the bucket** — controls who can access it.

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

**Fields explained:**

| Field | What it does |
|-------|-------------|
| **Effect** | Allow or Deny |
| **Principal** | Who this applies to (`*` = everyone) |
| **Action** | Which S3 API actions are allowed/denied |
| **Resource** | Which bucket or objects this applies to |

**When to use Bucket Policies:**

| Use Case | Solution |
|----------|----------|
| Make bucket publicly accessible | Bucket Policy — Allow `*` on `s3:GetObject` |
| Force HTTPS only | Bucket Policy — Deny `aws:SecureTransport: false` |
| Allow another AWS account | Bucket Policy — cross-account access |
| Enforce encryption on upload | Bucket Policy — Deny PUT without encryption header |

---

## 5. Block Public Access

A **master safety switch** — overrides all bucket policies and ACLs that allow public access.

```
Block Public Access = ON (default)
      │
      ▼
Even if bucket policy says "Allow *" → ❌ Still blocked
```

- Enabled **by default** on all new buckets
- Can be set at **account level** (blocks all buckets) or **bucket level**
- Must be **explicitly turned OFF** for a public bucket

> ⚠️ **Exam Trap:** Getting 403 on static website?
> Step 1 → Turn OFF Block Public Access
> Step 2 → Add bucket policy allowing public reads

---

## 6. ACLs (Access Control Lists)

Fine-grained per-object or per-bucket control.

| Type | Scope | Usage |
|------|-------|-------|
| **Object ACL** | Per-object | Rarely used — can be disabled |
| **Bucket ACL** | Per-bucket | Less common than Bucket Policy |

> 💡 AWS recommends using **Bucket Policies** over ACLs in most cases. ACLs are considered legacy.

---

## 7. CORS (Cross-Origin Resource Sharing)

**CORS** is a browser security feature — controls whether a website can fetch resources from a **different origin**.

**What is an Origin?**
```
https://www.example.com:443
   │           │          │
Protocol    Domain       Port

All 3 must match for "same origin"
```

**The S3 problem:**
```
Website Bucket                     Images Bucket
https://website.s3.amazonaws.com   https://images.s3.amazonaws.com
            │                                   │
            └──── different origins ────────────┘
                        │
              Browser blocks request ❌
```

**The fix — add CORS config on the Images Bucket:**

```json
[
  {
    "AllowedOrigins": ["https://website.s3.amazonaws.com"],
    "AllowedMethods": ["GET"],
    "AllowedHeaders": ["*"],
    "MaxAgeSeconds": 3000
  }
]
```

| Field | Meaning |
|-------|---------|
| `AllowedOrigins` | Which website can access this bucket |
| `AllowedMethods` | GET, PUT, POST, DELETE etc. |
| `AllowedHeaders` | Which request headers are allowed |
| `MaxAgeSeconds` | How long browser caches the CORS response |

> ⚠️ **Key Rule:** Configure CORS on the **bucket being accessed** — NOT the bucket hosting the website.

**CORS only applies to browsers:**

| Caller | CORS needed? |
|--------|-------------|
| Browser (JavaScript) | ✅ Yes |
| EC2 instance | ❌ No |
| Lambda function | ❌ No |
| Backend API | ❌ No |

---

## 8. Encryption at Rest

Every new S3 bucket has **SSE-S3 enabled by default**.

### Encryption Options

| Type | Who manages keys | Header required | Key point |
|------|-----------------|-----------------|-----------|
| **SSE-S3** | AWS manages everything | `x-amz-server-side-encryption: AES256` | Default, automatic, AES-256 |
| **SSE-KMS** | You control via AWS KMS | `x-amz-server-side-encryption: aws:kms` | Audit logs, more control |
| **SSE-C** | You provide your own key | Key sent with every request | AWS never stores your key |
| **Client-Side** | You encrypt before upload | None | Full control — AWS never sees plaintext |

```
SSE-S3:        AWS manages key ──► encrypts object ──► stores in S3
SSE-KMS:       Your KMS key   ──► encrypts object ──► stores in S3
SSE-C:         Your key       ──► AWS encrypts    ──► key discarded after
Client-Side:   You encrypt    ──► upload ciphertext ──► AWS stores as-is
```

### SSE-KMS — Watch Out

- Great for **auditing** — CloudTrail logs every key usage (who decrypted what, when)
- **High throughput warning:** Every S3 read/write = 1 KMS API call
- Can hit **KMS rate limits** → throws `ThrottlingException`
- **Fix:** Enable **S3 Bucket Keys** — creates a bucket-level key that reduces KMS API calls by up to **99%** and lowers cost

> ⚠️ **Exam Trap:** Large-scale S3 + SSE-KMS = potential throttling → solution is **S3 Bucket Keys**

---

## 9. Encryption in Transit (HTTPS)

Data moving between your client and S3 should always be encrypted using **HTTPS (SSL/TLS)**.

```
HTTP  ──► ❌ Not encrypted — data visible in transit
HTTPS ──► ✅ Encrypted with SSL/TLS — safe
```

**Enforce HTTPS only via Bucket Policy:**

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

> This **denies all HTTP requests** — only HTTPS allowed.

---

## 10. Default Encryption

By default, every new object uploaded to any S3 bucket is **automatically encrypted with SSE-S3**.

```
Upload object (no encryption specified)
      │
      ▼
Default Encryption kicks in ──► SSE-S3 applied automatically ✅
```

- You can change the default to **SSE-KMS** at bucket level
- **Bucket Policies evaluated BEFORE default encryption** — if policy enforces SSE-KMS but you upload without the right header → upload is denied

---

## 11. MFA Delete

Adds an **extra layer of protection** for critical operations by requiring MFA verification.

**MFA required for:**
- ✅ Permanently deleting an object version
- ✅ Suspending versioning on a bucket

**MFA NOT required for:**
- ❌ Enabling versioning
- ❌ Listing deleted versions

**Key rules:**
- Versioning must be **enabled first**
- Only the **root account** can enable/disable MFA Delete
- Can **only be configured via CLI** — not the AWS Console

```
Normal delete ──► adds Delete Marker (no MFA needed)
Permanent delete of version ──► MFA code required ✅
```

---

## 12. S3 Access Logs

Records **every request** made to your S3 bucket — authorized or denied.

```
Any request to Bucket A
      │
      ▼
S3 Access Logs
      │
      ▼
Saved to Logging Bucket B (same region)
      │
      ▼
Analyse with Athena ✅
```

**Key rules:**
- Logging bucket must be in the **same AWS Region**
- **NEVER set the logging bucket = monitored bucket** → creates an infinite logging loop → bucket grows forever → huge costs ❌
- Logs can be analysed with **Amazon Athena**

---

## 13. Pre-Signed URLs

Give **temporary access** to a private S3 object without permanent IAM credentials.

```
App Backend ──generates──► Pre-Signed URL (expires in 20 mins)
                                    │
                                    ▼
                    Anyone with the URL can access ✅
                    URL expires → access revoked automatically
```

| Detail | Value |
|--------|-------|
| Max expiry (Console) | 12 hours |
| Max expiry (CLI) | 168 hours (7 days) |
| Inherits permissions of | IAM user/role that generated it |
| Works for | Upload AND download |

**Use cases:**
- Share a premium/private file temporarily
- Allow users to upload directly to a private bucket

> ⚠️ **Exam Trap:** Pre-Signed URL inherits the generator's IAM permissions — if the role can't write, the URL can't write either.

---

## 14. Glacier Vault Lock

Applies a **WORM (Write Once Read Many)** policy to a Glacier vault.

```
Create Vault Lock Policy ──► Lock it ──► Policy can NEVER be changed or deleted
```

- Once locked — **nobody** can alter or delete the policy (including admins)
- Used for **long-term regulatory compliance**
- Protects data from being modified or deleted for the retention period

---

## 15. S3 Object Lock

Prevents S3 objects from being **deleted or overwritten** for a defined period.

**Requirements:**
- Versioning must be **enabled** on the bucket

**Two retention modes:**

| Mode | Who can override? | Use case |
|------|------------------|----------|
| **Governance Mode** | Admins with special IAM permission | Internal compliance, audit trails |
| **Compliance Mode** | **Nobody — not even root** | Strict regulatory requirement |

**Legal Hold:**
- Protects an object **indefinitely** — no expiry date
- Can be placed/removed by users with `s3:PutObjectLegalHold` permission
- Useful for objects needed for **active legal proceedings**

> ⚠️ **Exam Trap:** "No user including root account can delete" → **Object Lock in Compliance Mode**

---

## 16. ABAC — Attribute-Based Access Control

Control access dynamically by **matching user tags with object/bucket tags**.

```
User Tag:        department = finance
Object Tag:      department = finance
                      │
                      ▼
IAM Policy checks tag match ──► Access Granted ✅

User Tag:        department = sales
Object Tag:      department = finance
                      │
                      ▼
Tags don't match ──► Access Denied ❌
```

**Use when:** You have many users with different data access needs — instead of writing individual policies, use tags to dynamically control access.

---

## 17. Security Exam Scenarios

**Scenario 1 — Public static website**
> "Host a static website on S3 accessible to everyone."
> ✅ Turn OFF Block Public Access + add Bucket Policy allowing `s3:GetObject` for `Principal: *`

**Scenario 2 — Force HTTPS only**
> "Ensure all S3 requests use encrypted connections only."
> ✅ Bucket Policy — Deny `aws:SecureTransport: false`

**Scenario 3 — Prevent accidental deletion of versions**
> "Add extra protection so no one can accidentally delete object versions."
> ✅ Enable **MFA Delete** (root account only, via CLI)

**Scenario 4 — Regulatory data nobody can delete**
> "Banking records must be immutable for 7 years — even root account cannot delete."
> ✅ **S3 Object Lock — Compliance Mode** with 7-year retention

**Scenario 5 — SSE-KMS throttling**
> "High-throughput S3 application with SSE-KMS is throwing ThrottlingException."
> ✅ Enable **S3 Bucket Keys** — reduces KMS API calls by up to 99%

**Scenario 6 — Cross-bucket website assets blocked**
> "Website hosted on Bucket A is fetching images from Bucket B but browser blocks it."
> ✅ Configure **CORS on Bucket B** — add Bucket A's URL in AllowedOrigins

**Scenario 7 — Audit who decrypted what**
> "Compliance team needs to audit exactly who accessed/decrypted S3 objects and when."
> ✅ Use **SSE-KMS** — every key usage is logged in CloudTrail

**Scenario 8 — Temp access without permanent IAM user**
> "Allow external contractors to download specific files for 24 hours."
> ✅ **Pre-Signed URL** with 24-hour expiry

---

## 18. Security Quick Reference

| Feature | What it does | Key exam point |
|---------|-------------|----------------|
| **IAM Policies** | User/role level permissions | EC2 → S3 = use IAM Role |
| **Bucket Policies** | Resource-level JSON rules | Cross-account, public access, enforce HTTPS |
| **Block Public Access** | Master safety switch | ON by default — overrides bucket policies |
| **ACLs** | Fine-grained per-object | Legacy — use Bucket Policies instead |
| **CORS** | Browser cross-origin access | Configure on the bucket being accessed |
| **SSE-S3** | AWS managed encryption | Default, AES-256, no extra cost |
| **SSE-KMS** | KMS managed encryption | Audit logs, can throttle → use Bucket Keys |
| **SSE-C** | Customer managed keys | AWS never stores your key |
| **MFA Delete** | MFA for permanent deletes | Root only, CLI only |
| **Object Lock** | WORM — prevent deletes | Compliance Mode = even root can't delete |
| **Glacier Vault Lock** | WORM for Glacier | Policy locked forever once applied |
| **Pre-Signed URLs** | Temporary access | Inherits generator's IAM permissions |
| **Access Logs** | Log all bucket requests | Never log to same bucket |
| **S3 Bucket Keys** | Reduce KMS API calls | Use when SSE-KMS is throttling |

---

*→ See also: S3-Basics.md*
*→ See also: S3-Advanced.md*
