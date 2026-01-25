- **What is it? (1-Sentence Pitch):** A service that lets you assess, audit, and evaluate the configurations of your AWS resources over time.
- **Core Use Cases:**
    - Continuously monitoring for compliance with your desired configurations.
    - Getting a history of configuration changes for a resource.
    - Answering questions like, "Show me all S3 buckets that are publicly accessible."
- **Key Distinction:** **CloudTrail** tells you *who* made a change. **Config** tells you *what* the configuration of the resource looked like before and after the change and if it's compliant.

---

## AWS Config Core Concepts

### **What Config Does**

| Feature | Description |
|---------|-------------|
| **Configuration Recorder** | Records configuration changes for supported AWS resources |
| **Configuration History** | Timeline of all changes to a resource |
| **Configuration Snapshot** | Point-in-time view of all resources |
| **Compliance Checking** | Evaluates resources against Config Rules |
| **Remediation** | Automatically fixes non-compliant resources |

**Key Point:** Config is **not preventative** (doesn't block non-compliant changes). It **detects and remediates** after the fact.

---

### **Config vs. CloudTrail vs. CloudWatch**

| Service | What It Tracks | Use Case |
|---------|----------------|----------|
| **CloudTrail** | **API calls** (who, when, what action) | Audit trail, security analysis, "Who deleted this S3 bucket?" |
| **Config** | **Resource configurations** (what changed, compliance status) | Configuration management, compliance, "Show all non-compliant security groups" |
| **CloudWatch** | **Performance metrics** (CPU, memory, requests) | Monitoring, alarms, "Alert when CPU >80%" |

**Exam Tip:**
- "Who made the change?" → **CloudTrail**
- "What changed and is it compliant?" → **Config**
- "Is performance degraded?" → **CloudWatch**

---

## Configuration Recorder

**What It Is:** The component that records resource configuration changes.

**How It Works:**
1. Enable Config in a region
2. Configuration Recorder starts tracking supported resources
3. Changes are recorded to **S3 bucket** (for history)
4. Optionally send to **SNS topic** (for notifications)

**Supported Resources:** 100+ AWS resource types (EC2, S3, RDS, VPC, IAM, Lambda, etc.)

**Configuration Items (CI):**
- JSON document describing resource configuration
- Includes metadata (creation time, relationships, tags, configuration details)
- Created when resource is created, changed, or deleted

---

## Config Rules

**What They Are:** Desired configurations for your AWS resources. Config evaluates whether resources comply.

---

### **Rule Types**

| Rule Type | How It Works | Example |
|-----------|-------------|---------|
| **AWS Managed Rules** | Pre-built rules by AWS | `s3-bucket-public-read-prohibited`, `encrypted-volumes` |
| **Custom Rules** | You write Lambda function | Check if EC2 instances have specific tags |

**Exam Tip:** Use **Managed Rules** when possible (no code needed). Use **Custom Rules** for organization-specific policies.

---

### **Evaluation Triggers**

| Trigger Type | When Rule Runs |
|--------------|----------------|
| **Configuration Changes** | When a resource changes (default for most rules) |
| **Periodic** | On schedule (1 hour, 3 hours, 6 hours, 12 hours, 24 hours) |

**Example:**
- `s3-bucket-public-read-prohibited` → Triggered when S3 bucket configuration changes
- `ec2-instance-no-public-ip` → Triggered when EC2 instance is launched/modified

---

### **Compliance Statuses**

| Status | Meaning |
|--------|---------|
| **Compliant** | Resource meets rule requirements |
| **Non-Compliant** | Resource violates rule |
| **Not Applicable** | Rule doesn't apply to this resource |
| **Insufficient Data** | Not enough data to evaluate (e.g., resource just created) |

---

### **Popular AWS Managed Rules** (Exam-Relevant)

| Rule | What It Checks | Exam Trigger Phrase |
|------|----------------|---------------------|
| **s3-bucket-public-read-prohibited** | S3 buckets are not publicly readable | "Prevent public S3 access" |
| **s3-bucket-public-write-prohibited** | S3 buckets are not publicly writable | "Prevent public S3 writes" |
| **encrypted-volumes** | EBS volumes are encrypted | "Ensure all EBS volumes encrypted" |
| **rds-storage-encrypted** | RDS databases are encrypted | "Ensure RDS encryption" |
| **ec2-instance-no-public-ip** | EC2 instances don't have public IPs | "Private instances only" |
| **iam-user-no-policies-check** | IAM users don't have inline policies | "IAM best practices" |
| **required-tags** | Resources have required tags | "Tagging compliance" |
| **root-account-mfa-enabled** | Root account has MFA | "Secure root account" |
| **vpc-sg-open-only-to-authorized-ports** | Security groups don't allow 0.0.0.0/0 | "Prevent open security groups" |

---

## Remediation

**What It Is:** Automatically fix non-compliant resources using **SSM Automation Documents**.

---

### **How Remediation Works**

```
1. Config Rule detects non-compliance
   ↓
2. Remediation action triggers
   ↓
3. SSM Automation Document executes
   ↓
4. Resource is fixed (or retried if failed)
```

**Example Flow:**
- Rule: `s3-bucket-public-read-prohibited`
- Resource: S3 bucket becomes public
- **Config:** Detects non-compliance
- **Remediation:** Runs SSM document `AWS-PublishSNSNotification` or `AWS-DisableS3BucketPublicReadWrite`
- **Result:** Bucket made private automatically

---

### **Remediation Actions**

| Action Type | Description | Use Case |
|-------------|-------------|----------|
| **Manual Remediation** | Config alerts, you fix manually | Low-risk changes, need approval |
| **Automatic Remediation** | Config fixes automatically via SSM | High-confidence fixes, security critical |

**Automatic Remediation Settings:**

| Setting | Details |
|---------|---------|
| **SSM Document** | Automation document to run (e.g., `AWS-DisableS3BucketPublicReadWrite`) |
| **Parameters** | Input values for the document |
| **Resource ID** | Which resource to fix (auto-populated) |
| **Retry Attempts** | Number of retries if remediation fails (max 5) |
| **Retry Interval** | Time between retries (30-3600 seconds) |

---

### **Common Remediation Use Cases**

| Scenario | Config Rule | Remediation Action |
|----------|-------------|-------------------|
| S3 bucket becomes public | `s3-bucket-public-read-prohibited` | Run SSM document to disable public access |
| EC2 instance without required tags | `required-tags` | Run SSM document to add tags |
| Security group allows 0.0.0.0/0 | `vpc-sg-open-only-to-authorized-ports` | Run SSM document to restrict access |
| EBS volume unencrypted | `encrypted-volumes` | Take snapshot, create encrypted volume, notify |

**Exam Tip:** Remediation uses **SSM Automation Documents** (not Lambda). If the question asks how to auto-fix → Config + SSM Automation.

---

## Config Aggregator

**What It Is:** Aggregates Config data from **multiple accounts and regions** into a single view.

---

### **Why Use Aggregator?**

**Without Aggregator:**
- Check compliance in each account separately (50 accounts = 50 logins)
- Check compliance in each region separately (16 regions = 16 views)
- No centralized compliance view

**With Aggregator:**
- Single dashboard for all accounts and regions
- Central compliance reporting
- Ideal for multi-account organizations

---

### **How Aggregator Works**

```
Account 1 (us-east-1) → Config data
Account 2 (eu-west-1) → Config data  } → Aggregator (Central Account) → Unified View
Account 3 (ap-south-1) → Config data
```

**Setup:**
1. Create Aggregator in **central account** (typically management account)
2. Authorize source accounts to send data to Aggregator
3. View compliance across all accounts/regions in one dashboard

---

### **Aggregator Authorization**

| Method | Use Case |
|--------|----------|
| **Individual Account Authorization** | Manual approval per account (small number of accounts) |
| **AWS Organizations** | Automatic authorization for all accounts in organization (recommended) |

**Exam Tip:** For multi-account compliance → Use **Config Aggregator** with **AWS Organizations** authorization.

---

## Conformance Packs

**What It Is:** A collection of Config Rules and remediation actions packaged as a single entity.

**Purpose:** Deploy compliance-as-code at scale (e.g., "PCI-DSS compliance pack" with 50 rules).

---

### **How Conformance Packs Work**

```
Conformance Pack (YAML template)
  ├─ Config Rule 1: s3-bucket-encryption
  ├─ Config Rule 2: rds-storage-encrypted
  ├─ Config Rule 3: encrypted-volumes
  ├─ Remediation for Rule 1
  └─ Remediation for Rule 2
```

**Deployment:**
- Deploy to single account/region
- Deploy to **multiple accounts via AWS Organizations**
- Deploy to **Organizational Units (OUs)**

---

### **AWS Sample Conformance Packs**

| Pack | What It Checks | Use Case |
|------|----------------|----------|
| **Operational Best Practices for PCI-DSS** | PCI-DSS compliance (encryption, access control, logging) | Payment processing |
| **Operational Best Practices for HIPAA** | HIPAA compliance (healthcare data protection) | Healthcare apps |
| **Operational Best Practices for CIS AWS Foundations Benchmark** | CIS security benchmarks | Security baseline |
| **Operational Best Practices for NIST** | NIST cybersecurity framework | Government/federal |

**Exam Tip:** Conformance Packs = **compliance-as-code**. Deploy industry frameworks (PCI, HIPAA, CIS) as a single template.

---

### **Conformance Pack Deployment**

**Single Account:**
```bash
aws configservice put-conformance-pack \
  --conformance-pack-name my-security-pack \
  --template-s3-uri s3://my-bucket/security-pack.yaml
```

**Organization-Wide (via AWS Organizations):**
```bash
aws configservice put-organization-conformance-pack \
  --organization-conformance-pack-name org-security-pack \
  --template-s3-uri s3://my-bucket/security-pack.yaml
```

**Key Point:** Conformance Packs can be deployed to **all accounts in an Organization** automatically.

---

### **Conformance Pack Compliance**

| Compliance Score | Meaning |
|-----------------|---------|
| **100%** | All rules in pack are compliant |
| **75%** | 75% of rules are compliant |
| **0%** | No rules are compliant |

**Compliance View:** Dashboard shows compliance score per pack, per account, per region.

---

## Configuration Snapshots

**What It Is:** Point-in-time view of all resource configurations in your account.

**Use Cases:**
- Disaster recovery (restore to known good state)
- Compliance audits (prove configuration at specific date)
- Change analysis (compare configurations over time)

**Delivery:**
- Stored in **S3 bucket**
- Generated **on-demand** or **periodically**
- Can be shared across accounts

---

## Integration with Other Services

| Service | Integration | Use Case |
|---------|------------|----------|
| **CloudTrail** | Config can trigger based on CloudTrail events | Correlate "who changed what" with configuration history |
| **EventBridge** | Config sends events when compliance changes | Trigger Lambda when resource becomes non-compliant |
| **SNS** | Config sends notifications to SNS topic | Email alerts for compliance violations |
| **SSM Automation** | Config uses SSM for remediation | Auto-fix non-compliant resources |
| **Security Hub** | Config findings sent to Security Hub | Centralized security dashboard |
| **S3** | Config stores history and snapshots | Long-term audit trail |

---

## Common Exam Scenarios

| Scenario | Solution |
|----------|----------|
| "Ensure all S3 buckets are private" | Config Rule: `s3-bucket-public-read-prohibited` |
| "Automatically fix public S3 buckets" | Config Rule + **Automatic Remediation** (SSM document) |
| "Track all configuration changes for EC2 instances" | Enable **Config** with Configuration Recorder |
| "View compliance across 50 AWS accounts" | **Config Aggregator** with AWS Organizations |
| "Deploy PCI-DSS compliance rules to all accounts" | **Conformance Pack** (PCI-DSS template) via Organizations |
| "Alert when security group becomes open to 0.0.0.0/0" | Config Rule + **EventBridge** → SNS/Lambda |
| "Show S3 bucket configuration from 6 months ago" | Query **Configuration History** or **Configuration Snapshot** |
| "Require all resources to have CostCenter tag" | Config Rule: `required-tags` with parameter `CostCenter` |
| "Prevent root account usage without MFA" | Config Rule: `root-account-mfa-enabled` |
| "Auto-encrypt unencrypted EBS volumes" | Config Rule: `encrypted-volumes` + Remediation |

---

## Key Exam Tips

1. **Config = What changed + Compliance**, CloudTrail = **Who changed**
2. **Not preventative** (detects and remediates, doesn't block)
3. **Managed Rules:** Pre-built by AWS (prefer these)
4. **Custom Rules:** Lambda function (for organization-specific policies)
5. **Remediation:** Uses **SSM Automation Documents** (not Lambda)
6. **Aggregator:** Multi-account/multi-region compliance view
7. **Conformance Packs:** Deploy compliance frameworks (PCI, HIPAA, CIS) as code
8. **Integration:** Config + EventBridge + SNS = Real-time compliance alerts
9. **Configuration History:** Timeline of all changes to a resource
10. **Configuration Snapshot:** Point-in-time view of all resources
11. **Evaluation Triggers:** Configuration changes OR periodic
12. **Compliance Statuses:** Compliant, Non-Compliant, Not Applicable, Insufficient Data
13. **S3 bucket public?** → `s3-bucket-public-read-prohibited` rule
14. **EBS encrypted?** → `encrypted-volumes` rule
15. **Multi-account compliance?** → **Config Aggregator** + AWS Organizations

---

## Config Best Practices

| Best Practice | Why |
|---------------|-----|
| **Enable in all regions** | Comprehensive compliance coverage |
| **Use Aggregator for multi-account** | Centralized compliance view |
| **Use Managed Rules first** | No code required, AWS-maintained |
| **Enable automatic remediation for critical rules** | Auto-fix security issues (public S3, open security groups) |
| **Deploy Conformance Packs** | Compliance-as-code, industry frameworks |
| **Integrate with Security Hub** | Centralized security findings |
| **Store snapshots in S3** | Long-term audit trail |
| **Use required-tags rule** | Enforce tagging for cost allocation |

---

## Configuration Recorder Setup

**Step 1:** Create S3 bucket for Config history
```bash
aws s3 mb s3://my-config-bucket
```

**Step 2:** Create IAM role for Config
```bash
aws iam create-role --role-name ConfigRole --assume-role-policy-document file://trust-policy.json
aws iam attach-role-policy --role-name ConfigRole --policy-arn arn:aws:iam::aws:policy/service-role/ConfigRole
```

**Step 3:** Start Configuration Recorder
```bash
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/ConfigRole

aws configservice start-configuration-recorder --configuration-recorder-name default
```

**Step 4:** Create delivery channel (S3)
```bash
aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=my-config-bucket
```

---

## Cost Considerations

| Cost Component | Pricing |
|----------------|---------|
| **Configuration Items** | First 100K free, then $0.003 per item |
| **Config Rule Evaluations** | First 100K free, then $0.001 per evaluation |
| **Conformance Pack Evaluations** | $0.0012 per evaluation |
| **S3 Storage** | Standard S3 pricing for configuration history |

**Cost Optimization:**
- Use **Lifecycle Policies** on S3 bucket (move old configs to Glacier)
- Delete unnecessary Config Rules
- Use **periodic evaluation** instead of change-based (reduces evaluations)

---

## Summary

**AWS Config** provides:
- **Configuration history** (what changed, when)
- **Compliance checking** (Config Rules)
- **Automatic remediation** (SSM Automation)
- **Multi-account aggregation** (Config Aggregator)
- **Compliance-as-code** (Conformance Packs)

**Bottom Line:** Config is the compliance and configuration management service. Use it to ensure resources comply with organizational policies and automatically fix violations.
