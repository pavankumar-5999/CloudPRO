# IAM — Identity and Access Management

**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. What is IAM?

IAM = **Identity and Access Management** — it controls **who can log in** to your AWS account and **what they're allowed to do**.

Think of it like a **building with a security desk**:

```
AWS Account (the building)
      │
      ▼
IAM (the security desk)
      │
      ├── Checks your ID (Authentication)
      └── Checks what rooms you can enter (Authorization)
```

**Key facts:**
- IAM is a **global service** — not tied to any region
- It's **free** to use — no extra cost

---

## 2. Root Account

When you first create an AWS account, you get a **Root Account**.

```
Root Account = Master key to the entire building
```

- Has **full access** to everything — billing, all services, all settings
- Should **never be used for daily work**
- Should **never be shared**

> ⚠️ **Golden Rule:** Lock away the root account after setup. Create a separate IAM user for yourself and use that instead.

---

## 3. IAM Users

A **User** represents **one person or one application** that needs access to AWS.

```
AWS Account
   ├── User: Alice (works in Finance)
   ├── User: Bob (works in Engineering)
   └── User: backup-app (an application, not a person)
```

**Key facts:**
- Every new user has **zero permissions** by default
- You must explicitly grant permissions
- Users can log in via the **AWS Console** (password) or use **Access Keys** (for CLI/programmatic access)

---

## 4. IAM Groups

A **Group** is just a **collection of users** — makes managing permissions easier.

```
Group: Developers
   ├── Alice
   ├── Bob
   └── Charlie

Group: Finance
   └── Diana
```

**Why use Groups?**
Instead of setting permissions for each user one by one, you set them once on the Group — everyone inside inherits them automatically.

**Important rules:**
- A group can contain **many users**
- A user can belong to **multiple groups**
- **Groups cannot contain other groups** — only users
- Not every user needs to belong to a group

---

## 5. IAM Policies

A **Policy** is a document that says **exactly what is allowed or denied**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Breaking it down in plain English:**

| Field | Meaning |
|-------|---------|
| `Effect` | Allow or Deny |
| `Action` | What action — e.g. "read a file from S3" |
| `Resource` | Which specific thing this applies to |

**Policies can be attached to:**
- A **User** directly
- A **Group** (all users in it inherit the policy)
- A **Role** (used by AWS services)

```
Policy attached to Group
        │
        ▼
All Users in that Group get those permissions ✅
```

---

## 6. IAM Password Policy

You can set rules for how strong passwords must be for your IAM users.

**Options you can configure:**
- Minimum password length
- Require uppercase, lowercase, numbers, symbols
- Allow users to change their own password
- Force password expiry after X days
- Prevent password reuse

> 💡 **Use case:** A company security policy might require "minimum 12 characters, must include a symbol, expires every 90 days" — all configurable here.

---

## 7. MFA — Multi-Factor Authentication

MFA adds an **extra layer of security** on top of username + password.

```
Login = Something you know (password)
            +
     Something you have (MFA device) ✅
```

**Why it matters:**
If your password is stolen, the attacker still can't log in without your MFA code.

**Types of MFA devices:**

| Type | Example |
|------|---------|
| Virtual MFA | Google Authenticator, Authy app on your phone |
| Hardware Key | YubiKey |
| Hardware TOTP Device | A physical device showing a rotating code |

> ⚠️ **Best Practice:** Always enable MFA on your **root account** at minimum — this is the most important account to protect.

---

## 8. How Can People Access AWS?

There are 3 ways to interact with AWS:

| Method | Used by | Requires |
|--------|---------|----------|
| **AWS Console** | Humans, via browser | Username + Password (+ MFA) |
| **AWS CLI** | Humans/scripts, via terminal | Access Keys |
| **AWS SDK** | Applications, via code | Access Keys |

**Access Keys** consist of two parts:

```
Access Key ID       →  like a username
Secret Access Key   →  like a password (shown only ONCE when created)
```

> ⚠️ **Exam Tip / Real Tip:** Never share your Access Keys. Never commit them to GitHub. If leaked, delete and regenerate them immediately.

---

## 9. IAM Roles — Introduction

A **Role** is similar to a User, but instead of being tied to one person, it's meant to be **used by AWS services** or **assumed temporarily** by trusted people/services.

```
EC2 Instance ──assumes──► IAM Role ──has permissions for──► S3, DynamoDB etc.
```

**Why use Roles instead of Access Keys on a server?**

| Without Role (bad) | With Role (good) |
|---------------------|-------------------|
| Paste Access Keys onto EC2 | Attach a Role to EC2 |
| Keys never expire — risky if stolen | Credentials are temporary and auto-rotated |
| Manual key rotation needed | AWS handles everything |

> 💡 **Golden Rule:** Applications and AWS services should use **Roles** — humans use **Users**. Never hardcode Access Keys inside an application.

**Common role use cases:**
- EC2 instance needs to read from S3
- Lambda function needs to write to DynamoDB
- One AWS account needs to access resources in another AWS account

---

## 10. IAM Security Tools

AWS gives you tools to **check and audit** your IAM setup.

### IAM Credentials Report
- A downloadable **report (CSV file)** listing all users in your account
- Shows the status of their passwords, access keys, MFA devices
- Great for a security audit

### IAM Access Advisor
- Shows the **service permissions granted** to a user
- Shows **when those services were last accessed**
- Helps you spot and remove permissions nobody is actually using

> 💡 **Real-world use case:** "This user has permission to access 15 services but only ever uses 2" → Access Advisor helps you find this and tighten permissions (Least Privilege).

---

## 11. The Golden Rule of IAM — Least Privilege

> **Never give more permissions than what is needed.**

```
❌ Bad:  Give a user FULL access "just in case"
✅ Good: Give a user access to ONLY what their job requires
```

**Why this matters:**
If an account gets compromised, the damage is limited to only what that account could access. This is one of the most repeated security principles in cloud computing.

---

## 12. Key Takeaways

| Concept | Remember |
|---------|----------|
| IAM | Global service, controls who can do what |
| Root Account | Full access — lock it away, don't use daily |
| Users | Represents one person or app — zero permissions by default |
| Groups | Collection of users — cannot contain other groups |
| Policies | JSON documents — define Allow/Deny for actions |
| MFA | Extra security layer — always enable on root account |
| Access Keys | Access Key ID + Secret Key — never share, never commit to GitHub |
| Roles | Used by AWS services/applications — temporary credentials, auto-rotated |
| Least Privilege | Give only the access that's actually needed |
| Credentials Report | CSV audit of all users |
| Access Advisor | Shows what permissions are actually being used |

---

## 13. Real-World Scenarios

**Scenario 1 — New employee joins the Engineering team**
> Create an IAM User → Add to the "Engineers" Group → Group already has the right policies attached → Done, no need to set permissions individually.

**Scenario 2 — EC2 instance needs to read files from S3**
> Don't paste Access Keys onto the instance. Create an IAM Role with S3 read permissions → Attach the Role to the EC2 instance.

**Scenario 3 — Someone leaked their Access Keys accidentally**
> Immediately deactivate/delete those Access Keys in IAM → Generate new ones if needed → Review CloudTrail logs to see if there was unauthorized activity.

**Scenario 4 — Need to check if any users have weak or old passwords**
> Generate an **IAM Credentials Report** — shows password age, MFA status and access key status for every user.

---

*→ Next: IAM-Advanced.md*
