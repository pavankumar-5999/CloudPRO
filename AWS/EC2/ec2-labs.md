# Amazon EC2 — Hands-On Labs


> 💡 Complete labs in order — each one builds on the previous.
> Use `lab-template.md` to document every lab you complete.
> All labs use **AWS Free Tier** (t2.micro) unless mentioned otherwise.
> ⚠️ Always **stop or terminate** instances after each lab to avoid charges.

---

## Labs Overview

| Lab | Topic | Level |
|-----|-------|-------|
| Lab 01 | Launch Your First EC2 Instance | 🟢 Beginner |
| Lab 02 | EC2 User Data — Auto Install Web Server | 🟢 Beginner |
| Lab 03 | Security Groups — Control Traffic | 🟢 Beginner |
| Lab 04 | Connect to EC2 via SSH | 🟢 Beginner |
| Lab 05 | EC2 Instance Connect (Browser) | 🟢 Beginner |
| Lab 06 | EC2 Instance Types — Explore & Compare | 🟢 Beginner |
| Lab 07 | IAM Role on EC2 Instance | 🟡 Intermediate |
| Lab 08 | Private vs Public vs Elastic IP | 🟡 Intermediate |
| Lab 09 | EC2 Placement Groups | 🟡 Intermediate |
| Lab 10 | Elastic Network Interface (ENI) | 🟡 Intermediate |
| Lab 11 | EC2 Hibernate | 🟡 Intermediate |
| Lab 12 | EBS Volume — Create, Attach & Mount | 🟡 Intermediate |
| Lab 13 | EBS Snapshots & Cross-AZ Migration | 🟡 Intermediate |
| Lab 14 | Create a Custom AMI | 🟡 Intermediate |
| Lab 15 | EBS Volume Types & Modification | 🟡 Intermediate |
| Lab 16 | EBS Encryption | 🔴 Advanced |
| Lab 17 | EBS Multi-Attach | 🔴 Advanced |
| Lab 18 | Amazon EFS — Shared File System | 🔴 Advanced |
| Lab 19 | EC2 Purchasing Options — Spot Instance | 🔴 Advanced |
| Lab 20 | Full EC2 Architecture (Capstone) | 🔴 Advanced |

---

# SECTION 1 — EC2 Basics Labs

---

## Lab 01 — Launch Your First EC2 Instance

**Goal:** Launch an EC2 instance and understand all the settings you choose during launch.

**Steps:**
1. Go to AWS Console → EC2 → **Launch Instance**
2. Name: `my-first-ec2`
3. AMI: **Amazon Linux 2023** (free tier eligible)
4. Instance type: **t2.micro** (free tier)
5. Key pair: **Create new key pair**
   - Name: `my-ec2-key`
   - Type: RSA
   - Format: `.pem` (Mac/Linux) or `.ppk` (Windows PuTTY)
   - Download and save it safely — you only get it once!
6. Network settings: leave default VPC and subnet
7. Security group: **Create new** → allow SSH (port 22) from My IP
8. Storage: default 8 GB gp3
9. Click **Launch Instance**
10. Go to Instances → watch status go from Pending → Running

**Explore:**
- What is the **Public IP** of your instance?
- What is the **Private IP**?
- What **AZ** is it in?
- What is the **instance ID**?
- Stop the instance → start it again → did the Public IP change?

**Expected result:** Instance running, visible in EC2 dashboard

---

## Lab 02 — EC2 User Data — Auto Install Web Server

**Goal:** Use User Data to automatically install and start a web server on launch.

**Steps:**
1. Launch a new EC2 instance (same settings as Lab 01)
2. In **Advanced details** → **User Data** → paste:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2 - $(hostname -f)</h1>" > /var/www/html/index.html
```

3. Security group: allow **HTTP (port 80)** from Anywhere (0.0.0.0/0)
4. Launch the instance
5. Wait 2–3 minutes → copy the **Public IP** → open in browser
6. You should see your Hello message ✅

**Explore:**
- What happens if you **stop and start** the instance — does the web server still run?
- SSH into the instance → check: `cat /var/log/cloud-init-output.log` → see User Data execution logs
- Does User Data run again on restart? (Answer: No — only on first boot)

**Expected result:** Web page loads in browser showing hostname

---

## Lab 03 — Security Groups — Control Traffic

**Goal:** Understand how security groups control inbound and outbound traffic.

**Steps:**
1. Use the EC2 instance from Lab 02 (web server running)
2. Go to the instance → **Security** tab → click on the security group
3. **Remove the HTTP (port 80) inbound rule** → Save
4. Try to open the web page in browser → should time out ❌
5. Add the HTTP rule back → try again → works ✅

**Now test SSH:**
1. Remove the SSH (port 22) inbound rule
2. Try to SSH → connection times out ❌ (not refused — times out)
3. Add SSH rule back

**Create a second security group:**
1. EC2 → Security Groups → Create Security Group
2. Name: `web-sg` → allow HTTP from anywhere
3. Attach both security groups to the instance
4. Verify web still works

**Explore:**
- What is the difference between **timeout** vs **connection refused**?
- Can you add a **Deny rule** to a security group? (No — only Allow rules exist)
- What happens if you have two security groups — one allows, one doesn't have the rule?

**Expected result:** Traffic controlled exactly by security group rules

---

## Lab 04 — Connect to EC2 via SSH

**Goal:** SSH into your EC2 instance from your terminal.

**Steps:**

**On Mac/Linux:**
```bash
# Fix key permissions first (required)
chmod 400 my-ec2-key.pem

# SSH into instance
ssh -i "my-ec2-key.pem" ec2-user@YOUR-PUBLIC-IP
```

**On Windows (PowerShell):**
```powershell
ssh -i "my-ec2-key.pem" ec2-user@YOUR-PUBLIC-IP
```

**Once connected — explore inside:**
```bash
# Who am I?
whoami

# What OS is this?
cat /etc/os-release

# Check User Data ran
cat /var/log/cloud-init-output.log

# Check web server is running
systemctl status httpd

# Check disk
df -h

# Check memory
free -m

# Exit
exit
```

**Explore:**
- What happens if you SSH without `chmod 400`? Note the error.
- What happens if you use the wrong username? (try `ubuntu` instead of `ec2-user`)
- What port does SSH use? Can you change it?

**Expected result:** Successfully connected to instance via SSH

---

## Lab 05 — EC2 Instance Connect (Browser)

**Goal:** Connect to EC2 directly from the browser without a key pair.

**Steps:**
1. Go to EC2 → select your running instance
2. Click **Connect** (top right)
3. Choose **EC2 Instance Connect** tab
4. Username: `ec2-user`
5. Click **Connect**
6. Terminal opens in browser ✅

**Explore inside:**
```bash
# Check AWS CLI is available
aws --version

# Try listing S3 buckets (will fail — why?)
aws s3 ls

# Check instance metadata
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/instance-id
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

**Explore:**
- Why did `aws s3 ls` fail? (No IAM role attached — fixed in Lab 07)
- What is `169.254.169.254`? (Instance Metadata Service — IMDS)
- Does EC2 Instance Connect require port 22 open? (Yes — to AWS IP ranges)

**Expected result:** Browser terminal opens, metadata queries work

---

## Lab 06 — EC2 Instance Types — Explore & Compare

**Goal:** Understand instance type naming and categories.

**Steps:**
1. Go to EC2 → Launch Instance → Instance type → click **Compare instance types**
2. Search and compare:
   - `t2.micro` vs `t3.micro` — what's different?
   - `c5.xlarge` — what category? vCPUs? RAM?
   - `r5.large` — what category? vCPUs? RAM?
   - `i3.large` — what category?
3. Go to [ec2instances.info](https://ec2instances.info) → filter by free tier
4. Note the naming convention: class + generation + size

**Explore:**
- What does the `t` in `t2.micro` stand for? (Burstable)
- What does the `c` in `c5.xlarge` stand for? (Compute optimised)
- What does the `r` in `r5.large` stand for? (RAM/Memory optimised)
- Find the cheapest instance with 4 vCPUs

**Expected result:** Clear understanding of instance naming and categories

---

# SECTION 2 — EC2 Intermediate Labs

---

## Lab 07 — IAM Role on EC2 Instance

**Goal:** Attach an IAM Role to EC2 so it can access AWS services securely.

**Steps:**

**Create the IAM Role:**
1. Go to IAM → Roles → **Create role**
2. Trusted entity: **EC2**
3. Policy: attach **AmazonS3ReadOnlyAccess**
4. Role name: `ec2-s3-read-role` → Create

**Attach to EC2:**
1. Go to EC2 → select your instance
2. Actions → Security → **Modify IAM role**
3. Select `ec2-s3-read-role` → Update

**Test:**
1. Connect via EC2 Instance Connect
2. Run:
```bash
# Now this should work!
aws s3 ls

# List objects in a specific bucket
aws s3 ls s3://your-bucket-name

# Check which identity EC2 is using
aws sts get-caller-identity
```

**Explore:**
- What happens if you run `aws configure` and add keys manually? (Works but dangerous — never do this in production)
- What role is attached? Check via metadata:
```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

**Expected result:** EC2 can list S3 buckets without any access keys

---

## Lab 08 — Private vs Public vs Elastic IP

**Goal:** See how IPs behave and assign a fixed Elastic IP.

**Steps:**

**Observe IP change on restart:**
1. Note your instance's current Public IP
2. **Stop** the instance → **Start** it again
3. Check Public IP → it changed! ❌
4. Private IP → stayed the same ✅

**Allocate an Elastic IP:**
1. EC2 → **Elastic IPs** → **Allocate Elastic IP address** → Allocate
2. Select the new Elastic IP → Actions → **Associate Elastic IP**
3. Select your instance → Associate
4. Note the Elastic IP address
5. Stop → Start the instance
6. Check Public IP → now matches Elastic IP ✅ — didn't change!

**Cleanup (important):**
1. Disassociate the Elastic IP from the instance
2. **Release the Elastic IP** → otherwise AWS charges you!

**Explore:**
- What is the charge if Elastic IP is not associated to a running instance?
- Can you have more than one Elastic IP per instance?
- Can you transfer Elastic IP to another instance?

**Expected result:** Elastic IP stays fixed across stop/start cycles

---

## Lab 09 — EC2 Placement Groups

**Goal:** Create and test different placement group strategies.

**Steps:**

**Create a Cluster Placement Group:**
1. EC2 → **Placement Groups** → Create
2. Name: `cluster-pg`
3. Strategy: **Cluster**
4. Launch 2 instances into this group:
   - Launch Instance → Advanced → Placement group → select `cluster-pg`
5. Note both instances are in the **same AZ**

**Create a Spread Placement Group:**
1. Create another placement group
2. Name: `spread-pg`
3. Strategy: **Spread**
4. Launch 3 instances → each goes to a different rack
5. Note the AZ distribution

**Create a Partition Placement Group:**
1. Name: `partition-pg`
2. Strategy: **Partition** → Number of partitions: 3
3. Launch instances → assign to different partitions

**Explore:**
- Can you launch a t2.micro in a Cluster placement group?
- What happens if you try to launch 8 instances in a Spread group in one AZ? (Fails — max 7)
- Check the partition number via instance metadata:
```bash
curl http://169.254.169.254/latest/meta-data/placement/partition-number
```

**Expected result:** All 3 placement group types created and instances launched

---

## Lab 10 — Elastic Network Interface (ENI)

**Goal:** Create an ENI and move it between instances to simulate failover.

**Steps:**

**Create a standalone ENI:**
1. EC2 → **Network Interfaces** → **Create Network Interface**
2. Description: `failover-eni`
3. Subnet: same subnet as your instances
4. Security group: your existing SG
5. Create

**Attach to Instance A:**
1. Select the ENI → Actions → **Attach**
2. Choose Instance A → Attach
3. Note the private IP assigned to the ENI

**Simulate failover — move to Instance B:**
1. Actions → **Detach** from Instance A
2. Actions → **Attach** to Instance B
3. Note: same private IP now on Instance B ✅

**Explore:**
- Does the ENI keep the same private IP when moved? (Yes!)
- Does the ENI keep its security groups? (Yes!)
- Can you move an ENI to an instance in a different AZ? (No — AZ locked)
- What is the primary ENI (eth0) — can you detach it? (No — primary ENI can't be detached)

**Expected result:** ENI moved between instances with same private IP

---

## Lab 11 — EC2 Hibernate

**Goal:** Hibernate an EC2 instance and verify RAM state is preserved.

> ⚠️ Hibernate must be enabled at launch — cannot enable on existing instance.

**Steps:**

**Launch instance with Hibernate enabled:**
1. Launch new instance → **Advanced details**
2. Stop/Hibernate behaviour: ✅ **Enable**
3. Storage: Root volume must be **encrypted** → enable encryption
4. Launch

**Test hibernate:**
1. Connect via SSH
2. Run a process that will keep running:
```bash
# Start a background process
sleep 1000 &
echo "Process PID: $!"

# Note the uptime
uptime
```
3. Note the process ID and uptime
4. Go to EC2 Console → Instance state → **Hibernate**
5. Wait for instance to stop (takes a minute)
6. **Start** the instance again
7. Connect via SSH
```bash
# Check uptime — should show it was running before hibernate!
uptime

# Check process is still running
ps aux | grep sleep
```

**Explore:**
- Is the instance ID the same after hibernate? (Yes!)
- Does the Public IP change after hibernate? (Yes — unless Elastic IP)
- Check EBS volume — find the file where RAM was saved (hiberfil on Windows, swap on Linux)

**Expected result:** Instance resumes with same processes running, uptime continues

---

# SECTION 3 — EBS & Storage Labs

---

## Lab 12 — EBS Volume — Create, Attach & Mount

**Goal:** Create an EBS volume, attach it to EC2 and use it.

**Steps:**

**Create EBS volume:**
1. EC2 → **Volumes** → **Create Volume**
2. Type: **gp3**
3. Size: **10 GB**
4. AZ: **same AZ as your EC2 instance** (important!)
5. Create Volume

**Attach to EC2:**
1. Select the volume → Actions → **Attach Volume**
2. Select your instance → Device: `/dev/sdf` → Attach

**Mount inside EC2 (SSH in):**
```bash
# Check the new volume is visible
lsblk

# Create a file system on it (first time only)
sudo mkfs -t ext4 /dev/xvdf

# Create mount point
sudo mkdir /data

# Mount the volume
sudo mount /dev/xvdf /data

# Verify it's mounted
df -h

# Create a test file
echo "Hello from EBS volume!" | sudo tee /data/test.txt

# Read it back
cat /data/test.txt
```

**Explore:**
- What happens to your file if you **unmount** and **re-mount** the volume? (Still there)
- What happens if you **detach** the volume and attach to another instance? (Data still there)
- Can you attach this volume to an instance in a different AZ? (No — create snapshot first)

**Expected result:** EBS volume mounted, file created and persists

---

## Lab 13 — EBS Snapshots & Cross-AZ Migration

**Goal:** Take a snapshot and use it to move data to another AZ.

**Steps:**

**Take a snapshot:**
1. EC2 → Volumes → select your `/data` volume
2. Actions → **Create Snapshot**
3. Description: `my-data-snapshot`
4. Wait for snapshot to complete (Snapshots section → Status: completed)

**Create volume in different AZ from snapshot:**
1. Snapshots → select your snapshot → Actions → **Create Volume from Snapshot**
2. AZ: choose a **different AZ** from original (e.g. eu-west-2b instead of eu-west-2a)
3. Create

**Launch instance in new AZ and attach:**
1. Launch a new EC2 instance in the **new AZ**
2. Attach the new volume → mount it
3. Check your file:
```bash
sudo mount /dev/xvdf /data
cat /data/test.txt
```
→ Your file is there ✅ — data migrated across AZs!

**Cross-region copy:**
1. Snapshots → select snapshot → Actions → **Copy Snapshot**
2. Choose a different region (e.g. us-east-1)
3. Note: now you can create an EBS volume in that region from this snapshot

**Explore:**
- How long did the snapshot take?
- Is the snapshot incremental? (Second snapshot of same volume is faster)
- Enable Recycle Bin — delete snapshot — can you recover it?

**Expected result:** Data migrated from one AZ to another via snapshot

---

## Lab 14 — Create a Custom AMI

**Goal:** Build a custom AMI with a pre-installed web server so new instances launch faster.

**Steps:**

**Prepare the instance:**
1. Launch a fresh EC2 instance
2. SSH in and install + configure:
```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Launched from Custom AMI</h1>" | sudo tee /var/www/html/index.html
```

**Create the AMI:**
1. Go to EC2 → select the instance
2. Actions → Image and templates → **Create Image**
3. Image name: `my-webserver-ami`
4. Description: `Amazon Linux with Apache pre-installed`
5. No reboot: leave unchecked (safer for data consistency)
6. Create Image → wait for AMI status: **Available**

**Launch from your AMI:**
1. EC2 → AMIs → select your AMI → **Launch Instance from AMI**
2. Choose t2.micro → configure and launch
3. Open Public IP in browser → web server already running ✅ (no User Data needed!)

**Explore:**
- How long did the AMI instance take to serve the web page vs the User Data method?
- Can you use this AMI in another region? (No — copy it first)
- Copy AMI to another region → Launch there

**Expected result:** New instances launch with web server pre-installed — no setup needed

---

## Lab 15 — EBS Volume Types & Modification

**Goal:** Change EBS volume type and size without stopping the instance (Elastic Volumes).

**Steps:**

**Change volume type gp2 → gp3:**
1. Launch instance with a **gp2** root volume
2. EC2 → Volumes → select root volume
3. Actions → **Modify Volume**
4. Volume type: change from **gp2** to **gp3**
5. Increase size: from 8 GB → **20 GB**
6. IOPS: set to **4000** (gp3 allows independent IOPS)
7. Click **Modify** → Confirm

**Watch the modification:**
- Status: `modifying` → `optimizing` → `completed`
- Instance is still running throughout ✅

**Extend file system (required after resize):**
```bash
# Check current disk size (still shows old size!)
df -h

# Check new device size (shows new size)
lsblk

# Extend partition
sudo growpart /dev/xvda 1

# Resize file system
sudo resize2fs /dev/xvda1

# Check again — now shows new size ✅
df -h
```

**Explore:**
- Can you DECREASE volume size? (No — only increase)
- Can you change from gp3 to io2? (Yes!)
- How long did the modification take?

**Expected result:** Volume resized from 8 GB to 20 GB with no downtime

---

# SECTION 4 — Advanced Labs

---

## Lab 16 — EBS Encryption

**Goal:** Encrypt an existing unencrypted EBS volume.

**Steps:**

**Check current encryption:**
1. EC2 → Volumes → check if your volume shows Encrypted: No

**Encrypt via snapshot route:**
1. Select the unencrypted volume → Create Snapshot
2. Snapshots → select snapshot → Actions → **Copy Snapshot**
3. ✅ Tick **Encrypt this snapshot**
4. KMS key: use default `aws/ebs` key → Copy
5. Create Volume from the **encrypted snapshot**
6. Attach the encrypted volume to your instance

**Enable encryption by default (account level):**
1. EC2 → Settings → **EBS Encryption**
2. Enable → all new volumes will be encrypted automatically

**Via CLI:**
```bash
# Create encrypted volume
aws ec2 create-volume \
  --volume-type gp3 \
  --size 10 \
  --encrypted \
  --kms-key-id alias/aws/ebs \
  --availability-zone eu-west-2a
```

**Explore:**
- Can you directly encrypt an existing volume? (No — snapshot route only)
- Does encryption affect performance? (Minimal — transparent to the app)
- What KMS key is used by default? (`aws/ebs` — AWS managed key)

**Expected result:** Encrypted EBS volume created and attached

---

## Lab 17 — EBS Multi-Attach

**Goal:** Attach one io2 EBS volume to two EC2 instances simultaneously.

> ⚠️ io2 volumes are NOT free tier — small cost involved. Delete immediately after lab.

**Steps:**

**Create io2 volume:**
1. EC2 → Create Volume → Type: **io2**
2. Size: 4 GB · IOPS: 100 · AZ: choose one AZ
3. Enable: ✅ Multi-Attach

**Launch 2 instances in the same AZ:**
1. Instance A — same AZ as volume
2. Instance B — same AZ as volume

**Attach to both:**
1. Volume → Attach → Instance A
2. Volume → Attach again → Instance B

**SSH into each and check:**
```bash
# On Instance A
lsblk  # volume should appear

# On Instance B
lsblk  # same volume appears ✅
```

**Explore:**
- Can you format and mount the volume on both with ext4? (Dangerous without cluster FS)
- What file systems support Multi-Attach safely? (GFS2, OCFS2)
- Can you Multi-Attach across AZs? (No — same AZ only)

**Cleanup:** Detach from both instances → Delete the io2 volume immediately

**Expected result:** Single io2 volume attached to two instances in same AZ

---

## Lab 18 — Amazon EFS — Shared File System

**Goal:** Mount the same EFS file system on two EC2 instances and share files.

**Steps:**

**Create EFS:**
1. Go to EFS → **Create file system**
2. Name: `shared-efs`
3. VPC: default
4. Storage class: **Standard**
5. Create → note the **File System ID**

**Install EFS mount helper on both instances:**
```bash
sudo yum install -y amazon-efs-utils
```

**Create mount point and mount on Instance A:**
```bash
sudo mkdir /shared
sudo mount -t efs -o tls fs-YOUR-ID:/ /shared
```

**Mount on Instance B (same EFS, same command):**
```bash
sudo mkdir /shared
sudo mount -t efs -o tls fs-YOUR-ID:/ /shared
```

**Test sharing:**
```bash
# On Instance A — create a file
echo "Hello from Instance A!" | sudo tee /shared/hello.txt

# On Instance B — read the file
cat /shared/hello.txt
# Should see: Hello from Instance A! ✅
```

**Explore:**
- Can Instance A and B see each other's changes in real time?
- What security group must EFS have? (Allow NFS — port 2049 — from EC2 SG)
- Check EFS storage used: EFS Console → Metered Size
- Try mounting from an instance in a different AZ — does it work? (Yes!)

**Expected result:** Files written on one instance instantly visible on the other

---

## Lab 19 — Spot Instance Request

**Goal:** Launch a Spot instance and understand interruption behaviour.

**Steps:**

**Request a Spot Instance:**
1. EC2 → Spot Requests → **Request Spot Instances**
2. AMI: Amazon Linux 2023
3. Instance type: t2.micro (or t3.micro)
4. Max price: leave as default (current spot price)
5. Request type: **One-time**
6. Launch

**Check Spot pricing history:**
1. EC2 → Spot Requests → **Pricing History**
2. Select instance type → see price variation over time

**Via CLI:**
```bash
# Request a spot instance
aws ec2 request-spot-instances \
  --instance-count 1 \
  --type "one-time" \
  --launch-specification '{
    "ImageId": "ami-XXXXXXXX",
    "InstanceType": "t2.micro",
    "KeyName": "my-ec2-key"
  }'

# Check spot request status
aws ec2 describe-spot-instance-requests
```

**Explore:**
- What is the current Spot price for t2.micro in your region?
- What happens when spot price exceeds your max? (Instance terminated with 2 min warning)
- What workloads are suitable for Spot? (Batch jobs, ML training, image processing)
- What is a Spot Fleet? (Group of spot instances from multiple pools)

**Expected result:** Spot instance launched at reduced cost

---

## Lab 20 — Full EC2 Architecture (Capstone)

**Goal:** Build a complete, real-world EC2 architecture combining everything from this section.

```
Architecture:

Internet
    │
    ▼
Security Group (allow HTTP 80, HTTPS 443)
    │
    ▼
EC2 Instance (Custom AMI — web server pre-installed)
    │           │
    │           ├── Elastic IP (fixed public IP)
    │           ├── IAM Role (S3 read access)
    │           ├── Root Volume: gp3 (encrypted)
    │           └── Data Volume: gp3 (separate, encrypted)
    │                     │
    │               EBS Snapshot (weekly backup)
    │
    └── EFS Mount (/shared) ← shared with other instances
```

**Steps:**

1. **Create custom AMI** — install Apache, configure index.html (Lab 14)
2. **Launch EC2** from your custom AMI with:
   - t2.micro
   - Security group: HTTP (80) + SSH (22) from your IP only
   - Root volume: gp3, 10 GB, encrypted ✅
   - IAM Role: S3 read access
3. **Allocate Elastic IP** → Associate with instance
4. **Create a second gp3 EBS volume** → attach → mount at `/data`
5. **Create EFS** → mount at `/shared`
6. **Test everything:**
```bash
# Web server running?
curl http://YOUR-ELASTIC-IP

# IAM role working?
aws s3 ls

# Data volume mounted?
ls /data

# EFS mounted?
ls /shared
echo "test" > /shared/test.txt
```
7. **Take EBS snapshot** of the data volume
8. **Verify Elastic IP** doesn't change after stop/start
9. **Document your architecture** in your lab template

**What to document:**
- Architecture diagram
- Every service used and why
- Cost estimate for running 24/7 for 1 month
- What you would add for production (Load Balancer, Auto Scaling, Multi-AZ)

**Expected result:** Full working EC2 setup with all components connected

---

## Labs Summary

| Lab | Topic | Level |
|-----|-------|-------|
| Lab 01 | Launch First EC2 Instance | 🟢 Beginner |
| Lab 02 | User Data — Auto Install Web Server | 🟢 Beginner |
| Lab 03 | Security Groups — Control Traffic | 🟢 Beginner |
| Lab 04 | Connect via SSH | 🟢 Beginner |
| Lab 05 | EC2 Instance Connect (Browser) | 🟢 Beginner |
| Lab 06 | Instance Types — Explore & Compare | 🟢 Beginner |
| Lab 07 | IAM Role on EC2 | 🟡 Intermediate |
| Lab 08 | Private vs Public vs Elastic IP | 🟡 Intermediate |
| Lab 09 | Placement Groups | 🟡 Intermediate |
| Lab 10 | Elastic Network Interface (ENI) | 🟡 Intermediate |
| Lab 11 | EC2 Hibernate | 🟡 Intermediate |
| Lab 12 | EBS — Create, Attach & Mount | 🟡 Intermediate |
| Lab 13 | EBS Snapshots & Cross-AZ Migration | 🟡 Intermediate |
| Lab 14 | Custom AMI | 🟡 Intermediate |
| Lab 15 | EBS Volume Types & Elastic Volumes | 🟡 Intermediate |
| Lab 16 | EBS Encryption | 🔴 Advanced |
| Lab 17 | EBS Multi-Attach | 🔴 Advanced |
| Lab 18 | EFS — Shared File System | 🔴 Advanced |
| Lab 19 | Spot Instance | 🔴 Advanced |
| Lab 20 | Full EC2 Architecture (Capstone) | 🔴 Advanced |

---

> 💡 **Pro tip:** After each lab fill in `lab-template.md` and save as e.g. `ec2-lab-01-launch-instance.md` in your `EC2/labs/` folder on GitHub. Document every error you hit — those are gold for interviews! 🎯
