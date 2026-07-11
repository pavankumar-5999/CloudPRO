## Labs Overview

| Lab | Topic | Level |
|-----|-------|-------|
| Lab 01 | Create First S3 Bucket | 🟢 Beginner |
| Lab 02 | Upload, Download & Delete Objects | 🟢 Beginner |
| Lab 03 | Static Website Hosting | 🟢 Beginner |
| Lab 04 | Versioning | 🟢 Beginner |
| Lab 05 | Storage Classes | 🟢 Beginner |
| Lab 06 | Lifecycle Rules | 🟡 Intermediate |
| Lab 07 | Replication (CRR) | 🟡 Intermediate |
| Lab 08 | Pre-Signed URLs | 🟡 Intermediate |
| Lab 09 | Event Notifications | 🟡 Intermediate |
| Lab 10 | Transfer Acceleration | 🟡 Intermediate |
| Lab 11 | Multipart Upload (CLI) | 🟡 Intermediate |
| Lab 12 | Access Points | 🟡 Intermediate |
| Lab 13 | S3 Inventory | 🟡 Intermediate |
| Lab 14 | Bucket Policies | 🔴 Advanced |
| Lab 15 | Encryption (SSE-S3 & SSE-KMS) | 🔴 Advanced |
| Lab 16 | MFA Delete (CLI) | 🔴 Advanced |
| Lab 17 | Access Logs | 🔴 Advanced |
| Lab 18 | Object Lock (WORM) | 🔴 Advanced |
| Lab 19 | CORS Configuration | 🔴 Advanced |
| Lab 20 | Full S3 Architecture (Capstone) | 🔴 Advanced |

---

# SECTION 1 — Basics Labs

---

## Lab 01 — Create Your First S3 Bucket

**Goal:** Understand bucket creation, naming rules and region selection.

**Steps:**
1. Go to AWS Console → S3 → **Create bucket**
2. Bucket name: `yourname-saa-lab-01` (must be globally unique)
3. Region: select your nearest region
4. Leave all settings default → **Create bucket**
5. Verify bucket appears in S3 dashboard

**Explore:**
- Try creating another bucket with the **same name** → note the error
- Try a bucket name with **UPPERCASE letters** → note the error
- Check which region your bucket is in

**Expected result:** Bucket created successfully with a unique lowercase name

---

## Lab 02 — Upload, Download & Delete Objects

**Goal:** Understand how objects work in S3.

**Steps:**
1. Open your bucket from Lab 01
2. Click **Upload** → upload any file (photo, PDF, text file)
3. Click on the uploaded object → explore the **Properties** tab
4. Copy the **Object URL** → try opening it in browser (should give 403 — note why)
5. Click **Download** to download it
6. Upload the **same file** again → what happens?
7. Delete the object

**Explore:**
- Upload a file with a folder path in the name e.g. `photos/2026/profile.jpg`
- Does a real folder get created? (Answer: No — it's just the key name)
- What is the full key of your object?

**Expected result:** Object uploaded, downloaded and deleted successfully

---

## Lab 03 — Static Website Hosting

**Goal:** Host a real website using S3 Static Website Hosting.

**Steps:**
1. Create new bucket: `yourname-static-website`
2. Go to **Properties** → Static website hosting → **Enable**
3. Index document: `index.html` · Error document: `error.html`
4. Create and upload `index.html`:

```html
<!DOCTYPE html>
<html>
<head><title>My AWS S3 Website</title></head>
<body>
  <h1>Hello from Amazon S3!</h1>
  <p>This website is hosted on S3 Static Website Hosting.</p>
</body>
</html>
```

5. Create and upload `error.html`:

```html
<!DOCTYPE html>
<html>
<body><h1>404 — Page Not Found</h1></body>
</html>
```

6. Go to **Permissions** → Block Public Access → **Turn OFF** → Confirm
7. Go to **Permissions** → Bucket Policy → paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::yourname-static-website/*"
  }]
}
```

8. Copy **Bucket website endpoint URL** → open in browser

**Explore:**
- Open a URL that doesn't exist e.g. `/abc.html` → does error.html show?
- What is the difference between Object URL and Website URL?

**Expected result:** Website loads in browser with your HTML content

---

## Lab 04 — Versioning

**Goal:** See how versioning protects against accidental deletion and overwrite.

**Steps:**
1. Create new bucket: `yourname-versioning-lab`
2. Properties → Versioning → **Enable**
3. Upload `notes.txt` with content: `Version 1`
4. Upload same file `notes.txt` with content: `Version 2`
5. Upload again with content: `Version 3`
6. Toggle **Show versions** → see all 3 versions with different IDs
7. Download each version → verify content
8. Delete `notes.txt` (without selecting a version)
9. Toggle Show versions → see the **Delete Marker**
10. Delete the Delete Marker → file comes back ✅

**Explore:**
- Permanently delete one specific version — can you recover it?
- What is the version ID before versioning was enabled? (null)

**Expected result:** All 3 versions visible, delete and restore works correctly

---

## Lab 05 — Storage Classes

**Goal:** Understand how to assign and change storage classes on objects.

**Steps:**
1. Upload a file to your bucket
2. Click on the object → **Properties** → Storage class → note the default (**S3 Standard**)
3. Click **Edit storage class** → change to **S3 Standard-IA** → Save
4. Change again to **S3 Intelligent-Tiering**
5. Upload a NEW file → during upload assign **S3 Glacier Instant Retrieval**

**Explore:**
- What storage classes are available in the dropdown?
- Can you change directly from Standard to Glacier Deep Archive?

**Expected result:** Storage class changed successfully on existing and new objects

---

## Lab 06 — Lifecycle Rules

**Goal:** Automatically transition objects between storage classes.

**Steps:**
1. Go to your bucket → **Management** → **Lifecycle rules** → **Create rule**
2. Rule name: `auto-transition`
3. Apply to: All objects in bucket
4. Add transitions:
   - After **30 days** → S3 Standard-IA
   - After **90 days** → S3 Glacier Flexible Retrieval
   - After **365 days** → S3 Glacier Deep Archive
5. Add expiration: After **730 days** → Delete object → Save

6. Create second rule: `clean-old-versions`
   - Apply to: **Noncurrent versions only**
   - After **30 days** → delete noncurrent versions

**Explore:**
- What is a noncurrent version lifecycle rule and why is it important?
- What is the difference between transition and expiration actions?

**Expected result:** Two lifecycle rules created and visible in Management tab

---

## Lab 07 — Replication (CRR)

**Goal:** Automatically replicate objects to a bucket in another region.

**Steps:**
1. Create source bucket: `yourname-source-crr` → Region: `eu-west-2`
2. Enable **Versioning** on source bucket
3. Create destination bucket: `yourname-dest-crr` → Region: `us-east-1`
4. Enable **Versioning** on destination bucket
5. Source bucket → **Management** → **Replication rules** → **Create rule**
6. Rule name: `crr-rule`
7. Destination: your destination bucket
8. IAM Role: **Create new role**
9. Save → upload a new file to source bucket
10. Wait 1–2 minutes → check destination bucket

**Explore:**
- Upload a file BEFORE enabling replication — does it replicate? (No)
- Does deleting from source replicate the delete to destination by default?

**Expected result:** New objects in source appear in destination bucket

---

# SECTION 2 — Advanced Labs

---

## Lab 08 — Pre-Signed URLs

**Goal:** Generate a temporary URL to share a private object.

**Steps:**
1. Upload a private file (Block Public Access ON)
2. Try the Object URL in browser → 403 ❌
3. Object → **Object actions** → **Share with a presigned URL**
4. Expiry: **5 minutes** → Generate URL
5. Open the Pre-Signed URL → file loads ✅
6. Wait 5 minutes → try again → 403 ❌

**Via CLI:**
```bash
aws s3 presign s3://your-bucket/your-file.txt --expires-in 300
```

**Explore:**
- What does the Pre-Signed URL look like? (notice signature + expiry in the URL)
- What happens after it expires?

**Expected result:** URL works for 5 mins then expires automatically

---

## Lab 09 — Event Notifications

**Goal:** Trigger an email notification when an object is uploaded.

**Steps:**
1. Go to SNS → Create topic → Standard → Name: `s3-upload-alert`
2. Create subscription → Email → enter your email → confirm
3. Go to your S3 bucket → **Properties** → **Event notifications** → **Create**
4. Event name: `upload-notification`
5. Event type: ✅ PUT (object created)
6. Destination: SNS topic → select your topic → Save
7. Upload any file → check your email ✅

**Explore:**
- Add a DELETE event — what does that notification look like?
- What fields appear in the SNS notification JSON? (bucket name, key, size etc.)

**Expected result:** Email received within seconds of uploading a file

---

## Lab 10 — Transfer Acceleration

**Goal:** Enable and test S3 Transfer Acceleration.

**Steps:**
1. Bucket → **Properties** → **Transfer acceleration** → **Enable**
2. Note the new accelerated endpoint:
   ```
   yourname-bucket.s3-accelerate.amazonaws.com
   ```
3. Open AWS speed comparison tool:
   ```
   https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html
   ```
4. Run the speed test — compare results from your location

**Via CLI:**
```bash
aws s3 cp large-file.zip s3://your-bucket/ \
  --endpoint-url https://s3-accelerate.amazonaws.com
```

**Explore:**
- How much faster was Transfer Acceleration vs direct upload from your location?
- When would you NOT use Transfer Acceleration? (nearby region = no benefit)

**Expected result:** Accelerated endpoint enabled, speed comparison completed

---

## Lab 11 — Multipart Upload (CLI)

**Goal:** Upload a large file using Multipart Upload via CLI.

**Steps:**
1. Create a 100MB test file:
```bash
# Mac / Linux
dd if=/dev/zero of=testfile.bin bs=1M count=100
```

2. Upload using CLI:
```bash
aws s3 cp testfile.bin s3://your-bucket/ --storage-class STANDARD
```

3. Check for incomplete multipart uploads:
```bash
aws s3api list-multipart-uploads --bucket your-bucket
```

4. Create lifecycle rule to auto-clean incomplete uploads:
   - Management → Lifecycle → Create rule
   - Action: **Delete incomplete multipart uploads**
   - Days after initiation: **7**

**Explore:**
- What happens to incomplete multipart uploads if you don't clean them?
- How much would incomplete parts cost over time?

**Expected result:** 100MB file uploaded in parts successfully

---

## Lab 12 — Access Points

**Goal:** Create separate access points for different teams on the same bucket.

**Steps:**
1. Create bucket: `yourname-access-points-lab`
2. Upload 3 files:
   - `finance/report.csv`
   - `sales/data.csv`
   - `hr/employees.csv`
3. Bucket → **Access Points** → **Create access point**
4. Access Point 1: `finance-ap` → policy: Allow GET on `finance/*` only
5. Access Point 2: `sales-ap` → policy: Allow GET on `sales/*` only
6. Copy Access Point ARN → test access via each point

**Explore:**
- Can you access `finance/report.csv` via the sales access point? (No!)
- What is the DNS name of your access point?

**Expected result:** Each access point only allows access to its own prefix

---

## Lab 13 — S3 Inventory

**Goal:** Generate an inventory report of all objects in your bucket.

**Steps:**
1. Bucket → **Management** → **Inventory configurations** → **Create**
2. Name: `weekly-audit`
3. Destination: create new bucket `yourname-inventory-reports`
4. Frequency: **Weekly** · Output format: **CSV**
5. Optional fields: ✅ Size ✅ Last modified ✅ Encryption status ✅ Storage class ✅ Replication status
6. Save → wait up to 48 hours for first report

**Explore:**
- How would you query this CSV report using Athena?
- How would you use this to audit if all objects are encrypted?

**Expected result:** Inventory configuration created, report appears in destination bucket

---

# SECTION 3 — Security Labs

---

## Lab 14 — Bucket Policies

**Goal:** Write and test bucket policies for different scenarios.

**Policy 1 — Public read:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::your-bucket/*"
  }]
}
```

**Policy 2 — Force HTTPS only:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::your-bucket",
      "arn:aws:s3:::your-bucket/*"
    ],
    "Condition": {
      "Bool": { "aws:SecureTransport": "false" }
    }
  }]
}
```

**Policy 3 — Deny upload without encryption:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::your-bucket/*",
    "Condition": {
      "StringNotEquals": {
        "s3:x-amz-server-side-encryption": "AES256"
      }
    }
  }]
}
```

**Steps:**
1. Apply each policy one at a time
2. Test by uploading with and without encryption header
3. Note the exact error message you get when blocked

**Expected result:** All 3 policies behave as expected

---

## Lab 15 — Encryption (SSE-S3 & SSE-KMS)

**Goal:** Encrypt objects using different encryption methods.

**Steps:**

**SSE-S3:**
1. Upload file → Properties → Encryption → **SSE-S3 (AES-256)** → Upload
2. Click object → check Server-side encryption settings

**SSE-KMS:**
1. KMS → Create key → Symmetric → name: `s3-lab-key`
2. Upload file → Encryption → **SSE-KMS** → choose your key → Upload
3. Click object → check which KMS key was used

**Default Encryption:**
1. Bucket → Properties → Default encryption → **Edit** → SSE-KMS → choose key → Save
2. Upload a new file WITHOUT specifying encryption
3. Check object → encrypted with KMS automatically ✅

**Via CLI:**
```bash
# SSE-S3
aws s3 cp file.txt s3://your-bucket/ --sse AES256

# SSE-KMS
aws s3 cp file.txt s3://your-bucket/ \
  --sse aws:kms --sse-kms-key-id your-key-id
```

**Expected result:** Objects encrypted with SSE-S3 and SSE-KMS successfully

---

## Lab 16 — MFA Delete (CLI Only)

**Goal:** Enable MFA Delete to protect object versions from permanent deletion.

> ⚠️ Must be done via CLI with root account. MFA device required.

**Steps:**
1. Configure CLI with root account credentials
2. Enable MFA Delete:
```bash
aws s3api put-bucket-versioning \
  --bucket your-bucket \
  --versioning-configuration Status=Enabled,MFADelete=Enabled \
  --mfa "arn:aws:iam::ACCOUNT-ID:mfa/root-account-mfa-device TOTP-CODE"
```
3. Try to permanently delete a version WITHOUT MFA → should fail ❌
4. Try WITH MFA code → should succeed ✅
5. Disable MFA Delete when done (same command with `MFADelete=Disabled`)

**Explore:**
- Why can't MFA Delete be configured from the console?
- Which operations require MFA?

**Expected result:** Permanent deletes blocked without MFA code

---

## Lab 17 — S3 Access Logs

**Goal:** Enable access logging and analyse the logs.

**Steps:**
1. Create logging bucket: `yourname-access-logs` (same region as source)
2. Main bucket → **Properties** → **Server access logging** → **Enable**
3. Target bucket: your logging bucket · Target prefix: `logs/`
4. Make several requests — upload, download, delete objects
5. Wait 15–30 minutes → check logging bucket
6. Open a log file → read the log format

**Explore:**
- What information is in each log entry?
- How long did it take for logs to appear?
- What happens if you set the logging bucket = monitored bucket?

**Expected result:** Log files appear in logging bucket after requests

---

## Lab 18 — Object Lock (WORM)

**Goal:** Protect objects from deletion using Object Lock.

> ⚠️ Object Lock must be enabled at bucket creation — cannot be added later.

**Steps:**
1. Create new bucket: `yourname-object-lock-lab`
2. During creation → Advanced settings → **Enable Object Lock** ✅
3. Upload a file
4. Object actions → **Manage legal hold** → **Enable** → Save
5. Try to delete → blocked ❌
6. Try via CLI:
```bash
