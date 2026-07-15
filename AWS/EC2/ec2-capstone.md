# EC2 Capstone Project — Build a Secure Web Server on AWS

**Service:** Amazon EC2 + EBS + EFS + IAM
**Date:** 15-July-2026

---

## What is this project about?

Imagine a company wants to host their website on AWS.

They need:
- A **server** to run the website
- **Storage** to save their data
- **Security** so only the right people can access it
- A **fixed IP address** so the website URL never breaks
- A way for the server to **access other AWS services** without passwords

That's exactly what we are building here — a real working web server on AWS, just like a company would do it.

By the end of this project you will have:

```
✅ A live website hosted on EC2 — accessible from any browser
✅ A fixed public IP that never changes
✅ Encrypted storage attached to your server
✅ A shared folder that multiple servers can use
✅ Your server talking to S3 without any passwords
✅ A backup of your data
```

---

## What will it look like when done?

You open your browser → type your Elastic IP → you see:

```
Hello from ip-172-31-xx-xx.eu-west-2.compute.internal ✅
```

That's YOUR website running on YOUR server on AWS 🎉

---

## Architecture — What we are building

```
You (Browser)
      │
      │ type the IP in browser
      ▼
Elastic IP (fixed public address — never changes)
      │
      ▼
Security Group (bouncer — only lets HTTP and your SSH in)
      │
      ▼
EC2 Instance (your server — running Apache web server)
      │
      ├──── IAM Role attached
      │     └── lets EC2 talk to S3 without any passwords
      │
      ├──── Root Volume (gp3, encrypted)
      │     └── this is where the OS lives (like C: drive)
      │
      ├──── Data Volume (gp3, encrypted, mounted at /data)
      │     └── extra storage for your files (like D: drive)
      │     └── EBS Snapshot taken = backup of this drive
      │
      └──── EFS mounted at /shared
            └── shared folder — multiple servers can
                read and write here at the same time
```

---

## Services Used — and WHY each one

| Service | Why we use it |
|---------|--------------|
| **EC2 t2.micro** | This IS the server — it runs our website |
| **Custom AMI** | So we don't install Apache every time — bake it once, reuse forever |
| **Elastic IP** | Without this, our server's IP changes every restart — website breaks |
| **Security Group** | Without this, anyone can access anything — this is our firewall |
| **IAM Role** | Our server needs to talk to S3 — role gives permission without passwords |
| **EBS Root Volume (encrypted)** | The OS drive — encrypted so data is safe if hardware is stolen |
| **EBS Data Volume (encrypted)** | Separate drive for our data — can detach and move to another server |
| **EBS Snapshot** | Backup of our data drive — restore if something goes wrong |
| **EFS** | Shared folder — if we have 10 servers, they all read/write the same files |

---

## Step 1 — Build a Custom AMI

**Why?** Every time we launch a new server, we'd have to install Apache manually. Custom AMI saves a pre-installed version — launch and it's ready instantly.

1. Go to EC2 → Launch Instance → Amazon Linux 2023 → t2.micro → Launch
2. SSH into the instance and run:

```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" | sudo tee /var/www/html/index.html
```

**Command breakdown:**
- `sudo yum update -y` → update all packages (like Windows Update)
- `sudo yum install -y httpd` → install Apache web server
- `sudo systemctl start httpd` → start Apache right now
- `sudo systemctl enable httpd` → make Apache start automatically on every reboot
- `echo "<h1>Hello from $(hostname -f)</h1>"` → create a simple HTML page
  - `$(hostname -f)` → fetches the private DNS name of your instance automatically
  - `"double quotes"` → needed so `$(hostname -f)` executes — single quotes print it literally
- `| sudo tee /var/www/html/index.html` → write the output to the index.html file
  - `tee` → writes to file AND shows on screen at the same time
  - We use `tee` instead of `>` because `sudo` doesn't work with `>` redirect on protected folders

3. Go to EC2 Console → select instance → Actions → Image and templates → **Create Image**
4. Name: `ec2-capstone-ami` → Create
5. Wait until AMI status shows **Available**

**What you get:** A saved template with Apache already installed ✅

---

## Step 2 — Launch EC2 from your Custom AMI

**Why?** Now we launch our real server from the AMI we just built — Apache is already there, no setup needed.

1. EC2 → Launch Instance → **My AMIs** → select `ec2-capstone-ami`
2. Instance type: **t2.micro**
3. Key pair: your existing key
4. Security Group → Create new → name: `capstone-sg`
   - Add rule: **HTTP** port 80 → Source: Anywhere (so the website is public)
   - Add rule: **SSH** port 22 → Source: **My IP only** (so only you can manage it)
5. Root volume: gp3, 10 GB → tick ✅ **Encrypted**
6. Launch

**What you get:** A running server with Apache already working ✅

---

## Step 3 — Assign Elastic IP

**Why?** Right now your server has a public IP — but every time you stop and start it, the IP changes. Elastic IP fixes this — it stays the same forever.

1. EC2 → **Elastic IPs** → Allocate Elastic IP → Allocate
2. Select the new IP → Actions → **Associate Elastic IP**
3. Select your instance → Associate
4. Open browser → type `http://YOUR-ELASTIC-IP` → you should see your webpage ✅

**Test it:**
- Stop instance → Start instance → IP is still the same ✅

> ⚠️ Always use `http://` not `https://` — we haven't set up SSL yet

**What you get:** A permanent address for your website that never changes ✅

---

## Step 4 — Attach IAM Role

**Why?** Our server might need to read files from S3. The wrong way is to paste AWS access keys inside the server — if someone hacks it, they steal your keys. IAM Role gives permission automatically without any passwords.

1. IAM → Roles → **Create Role**
2. Trusted entity: **EC2**
3. Policy: **AmazonS3ReadOnlyAccess**
4. Role name: `ec2-capstone-role` → Create
5. Go to EC2 → select instance → Actions → Security → **Modify IAM Role**
6. Select `ec2-capstone-role` → Update

**Test it — SSH into instance:**
```bash
aws s3 ls
```
→ Lists your S3 buckets with zero credentials ✅

> 💡 The role never expires — AWS automatically refreshes temporary credentials behind the scenes every 1-6 hours. You never manage this.

**What you get:** Server can securely access S3 with zero hardcoded passwords ✅

---

## Step 5 — Add a Data Volume (EBS)

**Why?** The root volume is for the OS — we don't want to mix our data with it. Think of it like an external hard drive — if we need to move data to another server, just detach and reattach.

1. EC2 → Volumes → **Create Volume**
   - Type: gp3 · Size: 10 GB
   - AZ: **same AZ as your EC2 instance** (very important!)
   - Encryption: ✅ Enabled
2. Select volume → Actions → **Attach Volume** → select your instance → /dev/sdf → Attach
3. SSH into instance:

```bash
lsblk
```

**What `lsblk` does:**
- `lsblk` = **List Block Devices** — shows all storage drives attached to your machine
- Like **Device Manager** on Windows

Output you'll see:
```
NAME          SIZE  MOUNTPOINTS
nvme0n1        8G
├─nvme0n1p1    8G   /          ← your OS drive
nvme1n1       10G              ← your new EBS volume (blank, not mounted yet)
```

```bash
sudo mkfs -t ext4 /dev/nvme1n1
```

**What `mkfs` does:**
- `mkfs` = **Make File System** — formats the drive so Linux can use it
- Like formatting a USB drive on Windows before first use
- `-t` = type of file system
- `ext4` = standard Linux file system (like NTFS on Windows)
- `/dev/nvme1n1` = which drive to format (`/dev/` is where Linux keeps all drives)

```bash
sudo mkdir /data
```
- `mkdir` = **Make Directory** — creates a new folder called `/data`

```bash
sudo mount /dev/nvme1n1 /data
```

**What `mount` does:**
- Connects the drive to the `/data` folder so you can access it
- Like Windows assigning `D:` to a USB drive
- Without mounting — drive exists but you can't open it
- After mounting — go to `/data` and all drive contents are there

```bash
echo "My data is here!" | sudo tee /data/test.txt
cat /data/test.txt
```

Run `lsblk` again — now you'll see:
```
nvme1n1    10G    /data ✅
```

**What you get:** A separate 10GB encrypted drive for your data ✅

---

## Step 6 — Take a Snapshot (Backup)

**Why?** What if the volume gets corrupted or you accidentally delete something? Snapshot = backup. You can restore from it anytime or use it to move data to another AZ.

1. EC2 → Volumes → select your data volume
2. Actions → **Create Snapshot**
3. Description: `ec2-capstone-data-backup`
4. Wait → Snapshots → Status: **completed** ✅

**What you get:** A point-in-time backup of your data volume ✅

---

## Step 7 — Mount EFS (Shared Folder)

**Why?** If we scale to 10 servers, each needs access to the same files. EBS can only attach to one server — EFS can be shared with ALL servers simultaneously across different AZs.

1. Go to EFS → **Create file system** → name: `capstone-efs` → Create
2. Note the **File System ID** — looks like `fs-0f61e8d613b301a71`

3. SSH into EC2:

```bash
sudo yum install -y amazon-efs-utils
```
- Installs the EFS helper tool — needed for TLS encryption during mount
- Without this, the `-o tls` option won't work

```bash
sudo mkdir /shared
```
- Creates the mount point folder — this is just a **door** to access EFS
- The actual data lives on AWS EFS servers, not on your EC2

```bash
sudo mount -t efs -o tls fs-0f61e8d613b301a71:/ /shared
```

**Breaking down this command:**

| Part | What it means |
|------|--------------|
| `sudo` | Run as admin |
| `mount` | Connect a drive to a folder |
| `-t efs` | `-t` means type — we're mounting an EFS file system |
| `-o tls` | `-o` means option — use TLS encryption in transit (like HTTPS for your files) |
| `fs-0f61e8d613b301a71:/` | Your EFS file system ID + `:/ ` means root of EFS |
| `/shared` | Mount it at this folder on your EC2 |

```bash
echo "Shared via EFS!" | sudo tee /shared/hello.txt
cat /shared/hello.txt
```

**Verify EFS is mounted:**
```bash
df -h | grep /shared
```

Output:
```
127.0.0.1:/    8.0E    0    8.0E    0%    /shared ✅
```

- `df -h` = **Disk Free** — shows all mounted drives and their sizes
- `|` = pipe — sends output to the next command
- `grep /shared` = filter and show only the line containing `/shared`
- `8.0E` = 8 Exabytes — EFS automatically scales, virtually unlimited!

**To access shared folder:**
```bash
cd /shared
ls
```

- `cd` = **Change Directory** — moves you into a folder
- Without `cd` into `/shared` first, plain `ls` won't show EFS files
- `ls /shared` also works from anywhere

**What you get:** A shared folder any EC2 instance can access ✅

---

## Step 8 — Mount EFS on Second Instance

**Why?** This proves EFS is truly shared — files written on Instance A appear on Instance B instantly.

On Instance B:
```bash
sudo yum install -y amazon-efs-utils
sudo mkdir /shared
sudo mount -t efs -o tls fs-0f61e8d613b301a71:/ /shared
```

Same EFS ID → same shared folder → same files ✅

**Test sharing:**
```bash
# On Instance A
echo "Hello from Instance A!" | sudo tee /shared/from-a.txt

# On Instance B
cat /shared/from-a.txt
→ Hello from Instance A! ✅
```

---

## Step 9 — Final Verification

```bash
# Website running?
curl http://YOUR-ELASTIC-IP
# Expected: Hello from ip-172-31-xx-xx ✅

# IAM role working?
aws s3 ls
# Expected: lists your S3 buckets ✅

# Data volume mounted?
df -h | grep /data
# Expected: nvme1n1 10GB at /data ✅

# EFS mounted?
df -h | grep /shared
# Expected: 8.0E at /shared ✅

# Files there?
cat /data/test.txt
cat /shared/hello.txt
# Expected: your messages ✅
```

If all 5 pass → Capstone complete! 🎉

---

## Errors I Hit & How I Fixed Them

### Error 1 — Website not loading (Timeout)

**What happened:**
Launched EC2, opened public IP in browser — page kept loading and eventually timed out.

**Why it happened:**
Security Group only had SSH (port 22) — HTTP (port 80) was missing. Browser uses port 80 to load websites — without that inbound rule, traffic was blocked.

> Timeout = Security Group issue. Connection Refused = App issue.

**How I fixed it:**
1. EC2 → Security Groups → `capstone-sg` → Edit inbound rules
2. Added: **HTTP · Port 80 · Source 0.0.0.0/0**
3. Saved → refreshed browser → website loaded ✅

---

### Error 2 — EFS Mount Timeout

**What happened:**
```
Mount attempt 1/3 failed due to timeout after 15 sec
Mount attempt 2/3 failed due to timeout after 15 sec
Mount attempt 3/3 failed due to timeout after 15 sec
```

**Why it happened:**
EFS uses **NFS port 2049** to communicate. Neither the EC2 security group nor the EFS mount target security group had port 2049 open — so the connection kept timing out.

**How I fixed it:**

Fix 1 — Added NFS rule to EC2 Security Group:
1. EC2 → Security Groups → `capstone-sg` → Edit inbound rules
2. Added: **NFS · Port 2049 · Source 0.0.0.0/0**
3. Saved

Fix 2 — Added NFS rule to EFS Security Group:
1. EFS → `capstone-efs` → Network tab → Manage
2. Edit mount target security group
3. Added: **NFS · Port 2049 · Source: capstone-sg**
4. Saved

Ran the mount command again → worked ✅

---

## What I Learned

-
-
-

---

## What Would Make This Production-Ready

| What | Why |
|------|-----|
| **Load Balancer** | Spread traffic across multiple servers |
| **Auto Scaling** | Add more servers when traffic is high |
| **Multi-AZ** | If one data centre fails, another takes over |
| **CloudWatch Alarms** | Alert when CPU spikes or server goes down |
| **HTTPS (SSL)** | Encrypt website traffic |
| **RDS** | Proper managed database |
| **Bastion Host** | SSH via jump server — don't expose port 22 directly |

---

## Cleanup — Do This After the Lab!

```
1. Terminate EC2 instances
2. Release Elastic IP ← charged if not released!
3. Delete EBS data volume
4. Delete EBS snapshot
5. Delete EFS file system
6. Deregister custom AMI
7. Delete snapshot created by AMI
```

> ⚠️ Elastic IP charges you even when not attached — always release it!

---

## One-Line Summary

> Built a live web server on AWS with a fixed IP, encrypted storage, shared file system and secure S3 access — hit real errors along the way and fixed them, just like a real cloud engineer would.
