# IAM — Advanced Concepts

**Blog:** [cloudpro.hashnode.dev](https://cloudpro.hashnode.dev/aws-cloud-journey)

---

## 1. Recap — Where We Left Off

In IAM Basics we covered Users, Groups, Policies, Roles and MFA. Now we go deeper into how large organisations manage IAM across **multiple AWS accounts** and more advanced permission controls.

---

## 2. IAM Policy Structure — Deeper Look

A policy has more parts than just Effect/Action/Resource. Here's the full structure:

```json
{
  "Version": "2012-10-17",
  "Id": "S3-Account-Permissions",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:root" },
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": { "aws:SourceIp": "203.0.113.0/24" }
      }
    }
  ]
}
```

**Full breakdown:**

| Field | Meaning |
|-------|---------|
| `Version` | Policy language version — always use "2012-10-17" |
| `Id` | Optional identifier for the policy |
| `Statement` | Can contain one or more individual rules |
| `Sid` | Optional statement ID — for your own reference |
| `Effect` | Allow or Deny |
| `Principal` | WHO this applies to (used in resource-based policies) |
| `Action` | WHAT actions are allowed/denied |
| `Resource` | WHICH resource this applies to |
| `Condition` | OPTIONAL — extra rules, e.g. "only from this IP address" |

> 💡 **Condition** is powerful — you can restrict access based on IP address, time of day, whether MFA was used, and more.

---

## 3. Inline Policies vs Managed Policies

There are different ways to attach permissions to a user, group or role.

### AWS Managed Policies
- Created and maintained **by AWS**
- Ready-made for common use cases — e.g. `AmazonS3ReadOnlyAccess`
- Best for: getting started quickly, standard permissions

### Customer Managed Policies
- Created and maintained **by you**
- Reusable — can attach the same policy to multiple users/groups/roles
- Best for: your organisation's specific custom permission needs

### Inline Policies
- Directly embedded into a **single** user, group or role
- **Cannot be reused** — it's a one-to-one relationship
- Best for: a very specific, one-off permission that should never be reused

```
AWS Managed    →  Ready-made by AWS, good for standard needs
Customer Managed →  You create it, reusable across many identities
Inline         →  Embedded in ONE identity only, not reusable
```

> ⚠️ **Best Practice:** Prefer Customer Managed Policies over Inline Policies — they're reusable and easier to manage at scale.

---

## 4. IAM Policy Evaluation Logic

When multiple policies apply to a user, AWS follows a strict order to decide the final outcome.

```
Step 1: Is there an explicit DENY anywhere? ──► YES → ❌ DENIED (final answer)
                                              │ NO
                                              ▼
Step 2: Is there an explicit ALLOW anywhere? ──► NO → ❌ DENIED (implicit deny)
                                              │ YES
                                              ▼
                                         ✅ ALLOWED
```

**Golden Rule:** An explicit **DENY always wins** — no matter how many ALLOW statements exist elsewhere.

> 💡 **Real Example:** A user is in a Group that allows S3 full access, but also has a direct policy that denies deleting objects. Result → the user can do everything EXCEPT delete — because DENY always overrides ALLOW.

---

## 5. IAM Roles — Advanced Use Cases

We covered basic Roles already. Here's where they get used in more advanced scenarios.

### Cross-Account Access

Company A wants to give a trusted partner (Company B) limited access to some of their AWS resources — without creating a permanent user for them.

```
Company B's AWS Account
        │
        │ assumes
        ▼
IAM Role in Company A's Account (with specific limited permissions)
        │
        ▼
Temporary access granted — no permanent credentials shared ✅
```

### Service-Linked Roles
- Special roles that are **pre-created and pre-configured by AWS**
- Used internally by AWS services to perform actions on your behalf
- You don't need to define the permissions yourself — AWS manages them

### Roles for Federated Users
Allows people who **already have an identity elsewhere** (like your company's Active Directory, Google, or Facebook login) to get **temporary AWS access** without creating a separate IAM User for every single person.

```
Your Company's Login System (Active Directory)
        │
        │ federates into
        ▼
   IAM Role in AWS
        │
        ▼
Temporary AWS credentials issued ✅
(No IAM User created for each employee!)
```

> 💡 **Real-world benefit:** A company with 5,000 employees doesn't need to create 5,000 IAM Users — instead, employees log in with their existing company credentials and get temporary AWS access through a Role.

---

## 6. STS — Security Token Service

STS is the service behind the scenes that issues **temporary security credentials**.

```
Request "AssumeRole"
        │
        ▼
    AWS STS
        │
        ▼
Temporary Credentials issued:
  - Access Key ID (temporary)
  - Secret Access Key (temporary)
  - Session Token
  - Expiration time (e.g. 1 hour)
```

**Key facts:**
- Credentials automatically **expire** — much safer than permanent Access Keys
- Used whenever a Role is "assumed" by a user, application or another AWS account
- Default session duration is usually **1 hour** (configurable)

> ⚠️ **Exam/Real Tip:** Any time you see "temporary credentials" or "assume a role" in a scenario — that's STS working behind the scenes.

---

## 7. IAM Identity Federation with SAML 2.0

For companies that already use an on-premises identity system (like Microsoft Active Directory), IAM supports **SAML 2.0** federation.

```
Employee logs into Company's internal system (Active Directory)
        │
        ▼
SAML 2.0 assertion sent to AWS
        │
        ▼
AWS trusts the assertion → issues temporary credentials
        │
        ▼
Employee accesses AWS Console — no separate AWS password needed ✅
```

**Benefit:** One login for everything — employees don't need to remember a separate AWS password, and when they leave the company, disabling their main account also cuts off AWS access.

---

## 8. IAM Identity Center (formerly AWS SSO)

A centralised way to manage access across **multiple AWS accounts** at once — common in larger organisations that use AWS Organizations.

```
IAM Identity Center (one central login)
        │
   ┌────┼────┬────────┐
   ▼         ▼         ▼
Account A  Account B  Account C
(Dev)      (Test)     (Prod)
```

**Key facts:**
- One login gives access across many AWS accounts (based on permissions granted)
- Supports integration with existing identity providers (Active Directory, Okta, Google Workspace etc.)
- The recommended modern approach for multi-account organisations

---

## 9. IAM Permission Boundaries

A **Permission Boundary** sets the **maximum possible permissions** a user or role can ever have — even if their attached policies say otherwise.

```
Permission Boundary = "You can NEVER go beyond this limit"
Attached Policy     = "Here's what you're granted"

Final Access = Whichever is MORE RESTRICTIVE of the two
```

**Real-world example:**
A team lead is allowed to create IAM Users for their own team — but you don't want them accidentally (or intentionally) creating a User with admin access to the whole account.

```
Team Lead's Policy:      "Can create IAM users with any permissions"
Permission Boundary:     "Any user created can NEVER have Admin access"
                              │
                              ▼
                Result: Team Lead can create users,
                but those users can never become Admins ✅
```

---

## 10. ABAC — Attribute-Based Access Control

Instead of writing individual policies for every single user, you control access based on **tags (attributes)**.

```
User Tag:    department = finance
Resource Tag: department = finance
                    │
                    ▼
        Tags match → Access Granted ✅

User Tag:    department = sales
Resource Tag: department = finance
                    │
                    ▼
        Tags don't match → Access Denied ❌
```

**Why use ABAC over traditional policies (RBAC)?**

| | RBAC (Role-Based) | ABAC (Attribute-Based) |
|--|---------------------|--------------------------|
| How it works | Create a specific role/policy per group | Match tags dynamically |
| Scaling | Need a new policy for every new team | Just tag new resources — no new policy needed |
| Best for | Small, fixed number of teams | Large, fast-growing organisations |

---

## 11. Web Identity Federation

This lets users log in using a **well-known external identity provider** — like Google, Facebook, Amazon, or any OIDC-compatible provider — instead of creating an IAM User for them.

```
User logs in with their Google account
        │
        ▼
Google confirms identity → issues an ID token
        │
        ▼
Amazon Cognito exchanges the ID token
        │
        ▼
STS issues temporary AWS credentials
        │
        ▼
User can now access allowed AWS resources ✅
```

**Key facts:**
- The recommended way to do this is through **Amazon Cognito** — it manages the whole flow for you
- Common in **mobile apps and web apps** — "Sign in with Google/Facebook"
- Uses the STS API `AssumeRoleWithWebIdentity` behind the scenes

> 💡 **Real-world example:** A photo-sharing mobile app lets users log in with their Google account and then upload photos directly to an S3 bucket — no AWS-specific login needed.

---

## 12. Custom Identity Broker

Sometimes your company's identity system **doesn't support SAML 2.0** (the standard federation method). In that case, you build your own **Identity Broker** application.

```
Employee logs into Company's Identity System (non-SAML)
        │
        ▼
Your custom-built Identity Broker app checks credentials
        │
        ▼
Broker calls STS → AssumeRole or GetFederationToken
        │
        ▼
Temporary AWS credentials returned to the employee ✅
```

**When to use this vs SAML 2.0:**

| Scenario | Solution |
|----------|----------|
| Company identity system supports SAML 2.0 | Use standard **SAML 2.0 federation** |
| Company identity system does NOT support SAML 2.0 | Build a **Custom Identity Broker** |

> ⚠️ **Exam Tip:** If a question mentions an identity provider that "doesn't support SAML" → the answer is a **Custom Identity Broker**, not standard SAML federation.

---

## 13. IAM Access Analyzer

A tool that scans your account and tells you which of your resources are **accessible from outside your own AWS account**.

```
IAM Access Analyzer scans:
   S3 Buckets, IAM Roles, KMS Keys, Lambda functions, SQS queues...
        │
        ▼
Finds anything accessible by:
   - Another AWS account
   - The public internet
   - A federated user
        │
        ▼
Generates a finding: "This S3 bucket is publicly accessible!" ⚠️
```

**Access Analyzer vs Access Advisor — don't confuse them:**

| | IAM Access Analyzer | IAM Access Advisor |
|--|---------------------|---------------------|
| **What it checks** | Resources accessible from OUTSIDE your account | What permissions a user/role has actually USED |
| **Finds** | Unintended public/cross-account access | Unused permissions to remove |
| **Question type** | "Is my S3 bucket accidentally public?" | "Has this user actually used their EC2 permissions?" |

> 💡 **Simple way to remember:** Access **Analyzer** = looks OUTWARD (who can reach my stuff). Access **Advisor** = looks INWARD (what has this user actually used).

---

## 14. AWS Organizations & Service Control Policies (SCPs)

**AWS Organizations** lets you manage **multiple AWS accounts** centrally — common in companies with separate accounts for Dev, Test, and Production.

```
AWS Organization (Management Account)
        │
   ┌────┼────────┬──────────┐
   ▼             ▼          ▼
Dev Account   Test Account  Prod Account
```

**Service Control Policies (SCPs)** set the **maximum permissions** allowed across accounts in the Organization — similar to Permission Boundaries, but applied at the **Organization level** instead of per-user.

```
SCP applied to "Dev Account" = "Nobody in this account can use EC2 instances
                                 larger than t3.medium"
        │
        ▼
Even if a Dev account user has AdministratorAccess policy,
SCP still blocks launching a large EC2 instance ❌
```

**Key facts:**
- SCPs **do not grant** permissions — they only **restrict** the maximum possible permissions
- Applies to **all users and roles** in the affected account(s), including the account's own Admins
- The **Management Account** of an Organization is NOT restricted by SCPs

**Permission Boundary vs SCP — quick comparison:**

| | Permission Boundary | SCP |
|--|---------------------|-----|
| **Applied to** | Individual IAM User or Role | Entire AWS Account or Organizational Unit |
| **Scope** | One identity | Many accounts at once |
| **Use case** | Limit what a specific admin can create | Limit what an entire team/account can ever do |

> ⚠️ **Exam Tip:** "Restrict permissions across an entire AWS account, even for account Admins" → **SCP**. "Restrict permissions for one specific user or role only" → **Permission Boundary**.

---

## 15. IAM Best Practices — Summary

| Practice | Why |
|----------|-----|
| Don't use the Root Account for daily tasks | Root has unlimited power — huge risk if compromised |
| Enable MFA everywhere, especially Root | Extra layer of protection even if password leaks |
| One IAM User per person | Never share login credentials between people |
| Assign permissions via Groups | Easier to manage than per-user policies |
| Use Roles for applications/services | Temporary, auto-rotated credentials — safer than Access Keys |
| Apply Least Privilege always | Only grant what's actually needed |
| Use Permission Boundaries for delegated admin tasks | Prevents privilege escalation |
| Rotate credentials regularly | Reduces the window of risk if leaked |
| Review with IAM Access Advisor | Remove permissions nobody actually uses |
| Use IAM Identity Center for multi-account setups | Centralised, cleaner access management |

---

## 16. Key Takeaways

| Concept | Remember |
|---------|----------|
| Policy Evaluation | Explicit DENY always wins over ALLOW |
| Inline Policy | Tied to ONE identity, not reusable |
| Managed Policy | Reusable across many identities |
| STS | Issues temporary credentials, used whenever a Role is assumed |
| Cross-Account Access | Uses Roles — no permanent credentials shared between accounts |
| Federation (SAML) | Lets existing company logins access AWS without separate IAM Users |
| IAM Identity Center | Centralised access management across multiple AWS accounts |
| Permission Boundary | Sets the maximum limit — overrides any broader policy |
| ABAC | Access controlled by matching tags — scales better than per-user policies |
| Web Identity Federation | Login with Google/Facebook/Amazon via Cognito — for mobile/web apps |
| Custom Identity Broker | Used when identity provider doesn't support SAML 2.0 |
| Access Analyzer | Finds resources accessible from OUTSIDE your account (looks outward) |
| Access Advisor | Shows what permissions a user has actually USED (looks inward) |
| SCP | Restricts max permissions for an entire AWS account/org — even blocks Admins |
| Permission Boundary vs SCP | Boundary = one identity, SCP = whole account/org |

---

## 17. Real-World Scenarios

**Scenario 1 — Partner company needs temporary access to specific S3 data**
> Set up **Cross-Account IAM Role** with specific permissions → Partner's account assumes the role via STS → temporary access granted, no permanent credentials shared.

**Scenario 2 — Company has 3,000 employees using Active Directory**
> Use **SAML 2.0 Federation** or **IAM Identity Center** → employees log in with existing company credentials → temporary AWS access issued — no need to create 3,000 individual IAM Users.

**Scenario 3 — Team lead should create IAM users but must not create Admins**
> Attach a **Permission Boundary** to limit the maximum permissions any user they create can ever have.

**Scenario 4 — Company has many departments and resources tagged by department**
> Use **ABAC** — write one policy that checks if the user's department tag matches the resource's department tag, instead of writing a separate policy for every department.

**Scenario 5 — Application on EC2 needs to call another AWS service securely**
> Attach an **IAM Role** to the EC2 instance → behind the scenes, **STS** issues temporary credentials that auto-refresh — no hardcoded keys anywhere.

**Scenario 6 — Mobile app wants users to sign in with their Google account**
> Use **Web Identity Federation** through **Amazon Cognito** → user authenticates with Google → temporary AWS credentials issued via `AssumeRoleWithWebIdentity`.

**Scenario 7 — Company's identity system doesn't support SAML 2.0**
> Build a **Custom Identity Broker** application → it authenticates users against the company system → then calls STS `AssumeRole` or `GetFederationToken` to issue temporary AWS credentials.

**Scenario 8 — Security team wants to check if any S3 buckets are publicly accessible**
> Run **IAM Access Analyzer** → it scans all resources and flags anything reachable from outside the AWS account, including public access.

**Scenario 9 — Restrict ALL accounts in an Organization from using expensive EC2 instance types, even for account Admins**
> Apply a **Service Control Policy (SCP)** at the Organization level → this restricts the maximum permissions for every account, and even Admins cannot bypass it.

---

*→ See also: IAM-Basics.md*
