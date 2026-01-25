- **What is it? (1-Sentence Pitch):** AWS Organizations helps you centrally govern and manage multiple AWS accounts, while Control Tower provides the easiest way to set up and govern a new, secure, multi-account AWS environment based on best practices.
- **Core Use Cases:**
    - Managing billing for multiple accounts from one master account.
    - Applying security policies across all accounts (**Service Control Policies - SCPs**).
    - Creating new AWS accounts programmatically.
- **Key Features & Concepts:** Know that **SCPs** can restrict what actions are possible in member accounts, even for the root user.

---

## AWS Organizations

### **What It Is**

A service to centrally manage and govern multiple AWS accounts in a hierarchical structure.

**Key Point:** Organizations is **account management**. Control Tower is **automated governance**.

---

### **Core Components**

| Component | Description |
|-----------|-------------|
| **Management Account** | Root account that creates the organization (formerly "master account") |
| **Member Accounts** | Accounts invited to or created within the organization |
| **Organizational Units (OUs)** | Containers for grouping accounts (like folders) |
| **Service Control Policies (SCPs)** | Permissions boundaries applied to accounts/OUs |
| **Consolidated Billing** | Single bill for all accounts in organization |

---

### **Organization Structure**

```
Root (Organization)
├─ Management Account (pays the bill)
├─ OU: Production
│  ├─ Account: Prod-WebApp
│  └─ Account: Prod-Database
├─ OU: Development
│  ├─ Account: Dev-Team1
│  └─ Account: Dev-Team2
└─ OU: Security
   └─ Account: Audit-Logs
```

**Hierarchy:**
- **Root** (top level)
- **Organizational Units (OUs)** (can be nested)
- **Accounts** (leaf nodes)

---

## Organizational Units (OUs)

**What They Are:** Logical groupings of AWS accounts (like folders).

**Use Cases:**

| OU Structure | Example | Purpose |
|--------------|---------|---------|
| **Environment-Based** | Production, Development, Test | Apply different policies per environment |
| **Function-Based** | Finance, Marketing, Engineering | Segregate by business unit |
| **Compliance-Based** | PCI-Compliant, HIPAA-Compliant, General | Different compliance requirements |

**Nesting:**
- OUs can be nested **up to 5 levels deep** (Root → OU → OU → OU → OU → OU)
- Policies inherit down the hierarchy

---

## Service Control Policies (SCPs)

**What They Are:** JSON policies that define **maximum permissions** for accounts in an organization.

---

### **Key Characteristics**

| Characteristic | Details |
|----------------|---------|
| **Effect** | Limits what IAM users and roles CAN do (permission boundary) |
| **Does NOT Grant** | SCPs only restrict, never grant permissions |
| **Applies To** | All users and roles in account (even root user!) |
| **Inheritance** | Child OUs and accounts inherit SCPs from parent |
| **Evaluation** | Intersection of SCP + IAM policy = effective permissions |

**CRITICAL:** SCPs **do not affect** the **Management Account** (root account of the organization).

---

### **SCP Strategies**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Deny List** | Allow all by default, explicitly deny specific actions | Default strategy (most flexible) |
| **Allow List** | Deny all by default, explicitly allow specific actions | Highly restrictive environments |

**Default:** Organizations start with **FullAWSAccess** SCP attached (allow all).

---

### **How SCPs Work (Permission Intersection)**

```
IAM Policy (allows EC2, S3, RDS)
         ∩
SCP (allows EC2, S3 only - denies RDS)
         =
Effective Permissions: EC2, S3 only
```

**Example:**
- IAM Policy: Allows user to launch EC2, access S3, create RDS
- SCP: Denies RDS creation
- **Result:** User can launch EC2 and access S3, but **cannot create RDS**

---

### **SCP Examples**

#### **Example 1: Deny Access to Specific Regions**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["us-east-1", "us-west-2"]
        }
      }
    }
  ]
}
```
**Result:** Accounts can only use us-east-1 and us-west-2 regions.

---

#### **Example 2: Prevent Account Leaving Organization**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    }
  ]
}
```
**Result:** Member accounts cannot leave the organization.

---

#### **Example 3: Enforce MFA for Sensitive Actions**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["ec2:StopInstances", "ec2:TerminateInstances"],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
      }
    }
  ]
}
```
**Result:** Cannot stop/terminate EC2 instances without MFA.

---

#### **Example 4: Prevent Disabling CloudTrail**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```
**Result:** CloudTrail cannot be disabled in member accounts.

---

### **SCP Evaluation Logic**

1. **Explicit Deny** in SCP → Action **denied** (highest priority)
2. **Explicit Allow** in SCP + **Explicit Allow** in IAM → Action **allowed**
3. **No Allow** in SCP → Action **denied** (implicit deny)
4. **No Allow** in IAM → Action **denied** (implicit deny)

**Formula:** (SCP Allow) AND (IAM Allow) = Allowed

---

## Consolidated Billing

**What It Is:** Single payment method for all accounts in an organization (billed to Management Account).

---

### **Benefits**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Single Bill** | One bill for all accounts | Easier accounting, one invoice |
| **Volume Discounts** | Combined usage for pricing tiers | 100 TB total S3 → volume discount |
| **Reserved Instance Sharing** | RIs shared across accounts automatically | RI in Account A can cover usage in Account B |
| **Savings Plans Sharing** | Savings Plans shared across accounts | Compute Savings Plan covers EC2/Lambda in all accounts |
| **Cost Allocation Tags** | Track costs by project, department | Tag resources, view costs per tag |

---

### **How Volume Discounts Work**

**Without Consolidated Billing:**
- Account A: 30 TB S3 storage → $X price
- Account B: 30 TB S3 storage → $X price
- **Total:** 60 TB at $X per TB

**With Consolidated Billing:**
- Account A + B: 60 TB combined → Lower price tier
- **Total:** 60 TB at discounted rate

**Exam Tip:** Consolidated Billing = **automatic volume discounts** across all accounts.

---

### **Reserved Instance / Savings Plan Sharing**

**How It Works:**
- Account A buys Reserved Instance (EC2 r5.large in us-east-1)
- Account B runs EC2 r5.large in us-east-1
- **RI automatically applies to Account B** (if Account A isn't using it)

**Benefit:** Maximize RI/Savings Plan utilization across organization.

**Opt-Out:** Can disable RI/Savings Plan sharing per account (not recommended).

---

## AWS Control Tower

**What It Is:** Automated service to set up and govern a secure, multi-account AWS environment based on best practices.

**Key Point:** Control Tower = **Organizations + Guardrails + Account Factory + Dashboard**

---

### **Core Components**

| Component | Description |
|-----------|-------------|
| **Landing Zone** | Pre-configured multi-account environment (baseline setup) |
| **Guardrails** | Pre-packaged governance rules (SCPs + Config Rules) |
| **Account Factory** | Automated account provisioning with baseline configuration |
| **Dashboard** | Centralized view of compliance and accounts |

---

### **Landing Zone**

**What It Is:** Pre-configured, secure, multi-account environment that follows AWS best practices.

**Automatically Creates:**
- **Security OU** with Audit and Log Archive accounts
- **Foundational OUs** (Sandbox, Production, etc.)
- **CloudTrail** enabled in all accounts (logs to Log Archive account)
- **AWS Config** enabled in all accounts
- **AWS SSO** (IAM Identity Center) for user access

**Structure:**
```
Root
├─ Security OU
│  ├─ Audit Account (Security team access)
│  └─ Log Archive Account (Centralized logging)
└─ Sandbox OU
   └─ Member Accounts
```

---

### **Guardrails**

**What They Are:** Pre-packaged governance rules that enforce policies across accounts.

**Two Types:**

| Type | Implementation | Can Disable? | Example |
|------|----------------|--------------|---------|
| **Preventive** | **SCPs** (block actions) | ✅ For some (not mandatory ones) | Prevent deleting CloudTrail |
| **Detective** | **Config Rules** (detect violations) | ✅ For some | Detect unencrypted S3 buckets |

---

#### **Guardrail Categories**

| Category | Description | Enforcement |
|----------|-------------|-------------|
| **Mandatory** | Enforced by default, cannot be disabled | Always active |
| **Strongly Recommended** | AWS best practices, can be disabled | Optional but recommended |
| **Elective** | Additional governance, opt-in | Optional |

---

#### **Example Guardrails**

| Guardrail | Type | Category | What It Does |
|-----------|------|----------|--------------|
| **Disallow Changes to CloudTrail** | Preventive (SCP) | Mandatory | Prevents deleting/stopping CloudTrail |
| **Disallow Public Read Access to S3** | Preventive (SCP) | Strongly Recommended | Prevents S3 buckets from becoming public |
| **Detect Unencrypted EBS Volumes** | Detective (Config) | Strongly Recommended | Alerts when EBS volumes are unencrypted |
| **Disallow Root User Access Keys** | Detective (Config) | Mandatory | Detects if root user has access keys |
| **Enable MFA for Root User** | Detective (Config) | Strongly Recommended | Detects if root MFA is disabled |

---

### **Account Factory**

**What It Is:** Automated account provisioning that creates new accounts with baseline configuration.

**What It Configures:**
- Applies guardrails automatically
- Enables CloudTrail, Config, AWS SSO
- Places account in specified OU
- Configures VPCs, subnets (optional)
- Assigns administrator access

**Use Case:** Developers can request new AWS accounts via Service Catalog portal (self-service).

---

### **Control Tower Dashboard**

**What It Shows:**
- **Compliance status** of all accounts
- **Guardrail violations** (which accounts, which guardrails)
- **Drift detection** (if accounts have been modified outside Control Tower)
- **Account inventory** (all accounts in organization)

---

### **Drift Detection**

**What It Is:** Detects when accounts or Landing Zone have been modified manually (outside Control Tower).

**Types of Drift:**
- **Governance Drift:** SCPs or Config Rules modified manually
- **Account Drift:** Accounts moved to different OUs manually
- **Landing Zone Drift:** Core accounts (Audit, Log Archive) modified

**Resolution:** Control Tower provides **repair** option to restore compliant configuration.

---

## Organizations vs. Control Tower

| Feature | AWS Organizations | AWS Control Tower |
|---------|------------------|-------------------|
| **Purpose** | Account management, billing | **Automated governance** + account management |
| **Setup** | Manual (you configure everything) | **Automated** (Landing Zone + best practices) |
| **Guardrails** | You create SCPs manually | **Pre-packaged** guardrails (SCPs + Config Rules) |
| **Account Provisioning** | Manual (create accounts via console/API) | **Account Factory** (automated, self-service) |
| **Compliance Dashboard** | No built-in dashboard | ✅ **Centralized dashboard** |
| **Best Practices** | You implement | **Automatically applied** |
| **Use Case** | Fine-grained control, existing multi-account setup | **New multi-account** setup, automated governance |

**Exam Tip:** Control Tower = **Organizations + automation + best practices**. Use Control Tower for **new** multi-account environments.

---

## Common Exam Scenarios

| Scenario | Solution |
|----------|----------|
| "Centralize billing for 50 AWS accounts" | **AWS Organizations** with Consolidated Billing |
| "Prevent accounts from using services outside us-east-1" | **SCP** denying actions in other regions |
| "Automatically set up new accounts with security baseline" | **AWS Control Tower** Account Factory |
| "Prevent any account from deleting CloudTrail" | **SCP** denying `cloudtrail:StopLogging` and `cloudtrail:DeleteTrail` |
| "Get volume discounts across multiple accounts" | **AWS Organizations** Consolidated Billing (automatic) |
| "Share Reserved Instances across accounts" | **AWS Organizations** Consolidated Billing (automatic) |
| "Set up secure multi-account environment quickly" | **AWS Control Tower** Landing Zone |
| "Detect S3 buckets that are publicly accessible" | **Control Tower** Detective Guardrail (Config Rule) |
| "Prevent developers from creating resources in production account" | **SCP** attached to Development OU denying access to Production account |
| "Group accounts by department (Finance, Engineering, HR)" | **Organizational Units (OUs)** - function-based structure |
| "Prevent member accounts from leaving organization" | **SCP** denying `organizations:LeaveOrganization` |
| "Centralized logging for all accounts" | **Control Tower** Log Archive account (automatic) |

---

## Key Exam Tips

1. **Organizations = account management**, Control Tower = **automated governance**
2. **Management Account:** Not affected by SCPs (root account)
3. **Member Accounts:** All other accounts (affected by SCPs)
4. **SCPs:** Permission boundaries (restrict, never grant)
5. **SCP does NOT grant:** Only limits IAM permissions (intersection)
6. **Consolidated Billing:** Single bill, volume discounts, RI/Savings Plan sharing
7. **OUs:** Can nest up to **5 levels deep**
8. **Guardrails:** Preventive (SCPs) + Detective (Config Rules)
9. **Mandatory Guardrails:** Cannot be disabled
10. **Account Factory:** Automated account provisioning with baseline
11. **Landing Zone:** Pre-configured multi-account environment
12. **Drift Detection:** Detects manual changes outside Control Tower
13. **RI Sharing:** Automatic across accounts with Consolidated Billing
14. **Volume Discounts:** Automatic with Consolidated Billing
15. **Region Restriction:** Use SCP to deny actions in specific regions

---

## Best Practices

| Best Practice | Why |
|---------------|-----|
| **Use Control Tower for new environments** | Automated setup, best practices out-of-box |
| **Use Organizations for existing setups** | More control, integrate with existing accounts |
| **Separate Log Archive account** | Security isolation, compliance |
| **Separate Audit account** | Security team access, read-only to all accounts |
| **Use OUs for environment separation** | Production, Development, Test |
| **Apply SCPs to OUs, not individual accounts** | Easier management, policy inheritance |
| **Enable MFA for root user** | Security baseline |
| **Use CloudTrail in all accounts** | Audit trail, compliance |
| **Use Config in all accounts** | Configuration tracking, compliance |
| **Tag all resources** | Cost allocation, tracking |

---

## SCP Best Practices

| Best Practice | Example |
|---------------|---------|
| **Deny specific actions, not entire services** | Deny `s3:DeleteBucket`, not `s3:*` |
| **Use Deny List strategy** | Allow all, deny specific (more flexible) |
| **Test SCPs in non-production first** | Avoid breaking production workloads |
| **Prevent disabling CloudTrail** | SCP denying `cloudtrail:StopLogging` |
| **Enforce region restrictions** | SCP denying actions outside allowed regions |
| **Require MFA for sensitive actions** | SCP requiring MFA for `ec2:TerminateInstances` |

---

## Cost Considerations

| Cost Component | Pricing |
|----------------|---------|
| **AWS Organizations** | ❌ **Free** |
| **AWS Control Tower** | ❌ **Free** (pay only for underlying services) |
| **Consolidated Billing** | ❌ **Free** |
| **Underlying Services** | CloudTrail, Config, S3 (for logs) - standard pricing |

**Cost Savings:**
- Volume discounts (automatic with Consolidated Billing)
- RI/Savings Plan sharing (maximize utilization)
- Reduced operational overhead (automation)

---

## Summary

**AWS Organizations:**
- Centralized account management
- Service Control Policies (SCPs)
- Consolidated Billing
- Volume discounts and RI/Savings Plan sharing

**AWS Control Tower:**
- Automated multi-account setup (Landing Zone)
- Pre-packaged governance (Guardrails)
- Automated account provisioning (Account Factory)
- Centralized compliance dashboard

**Bottom Line:** Use **Organizations** for account management and billing consolidation. Use **Control Tower** to automate governance and quickly set up secure multi-account environments.
