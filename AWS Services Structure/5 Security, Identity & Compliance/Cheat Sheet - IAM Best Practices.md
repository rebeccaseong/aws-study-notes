# Cheat Sheet: IAM Best Practices

Quick reference for AWS Identity and Access Management best practices.

---

## Core Security Principles

### **1. Least Privilege**
✅ Grant only the permissions required to perform a task
✅ Start with minimum permissions, add more as needed
✅ Use specific actions, not `"*"` (all actions)
✅ Use specific resources, not `"*"` (all resources)

❌ **Don't:** Give `AdministratorAccess` to everyone
✅ **Do:** Create specific policies for each role/user

---

### **2. Use Roles, Not Users**
✅ **Applications on EC2:** Use **IAM Roles** (instance profiles), not access keys
✅ **Lambda functions:** Use **execution roles**
✅ **Cross-account access:** Use **roles with trust policies**

❌ **Don't:** Embed access keys in application code
✅ **Do:** Attach IAM role to EC2 instance, application gets credentials automatically

---

### **3. Enable MFA (Multi-Factor Authentication)**
✅ **Root account:** **ALWAYS** enable MFA (exam favorite!)
✅ **IAM users:** Enable MFA for privileged users
✅ **Require MFA for sensitive operations** (delete resources, change security settings)

**MFA Options:**
- Virtual MFA (Google Authenticator, Authy)
- Hardware MFA (YubiKey)
- SMS-based MFA (not recommended for root)

---

### **4. Protect Root Account**
✅ **Never use root account for daily tasks**
✅ **Enable MFA on root account**
✅ **Delete root access keys** (don't create them)
✅ **Use root only for:** Billing, account settings, closing account

❌ **Don't:** Give root credentials to anyone
✅ **Do:** Create IAM users for all human access (including administrators)

---

### **5. Use IAM Policies Effectively**

| Policy Type | Scope | Use Case |
|-------------|-------|----------|
| **Identity-based** | Attached to users, groups, roles | Define what a user/role can do |
| **Resource-based** | Attached to resources (S3 bucket, SQS queue) | Define who can access resource |
| **Permission Boundaries** | Maximum permissions for users/roles | Limit what a user can do (even with other policies) |
| **SCPs (Service Control Policies)** | AWS Organizations (account-level) | Restrict what accounts can do |

---

## IAM Best Practices Checklist

### **User Management**

✅ **Create individual IAM users** (don't share credentials)
✅ **Use IAM groups** to assign permissions to multiple users
✅ **Delete unused users and credentials**
✅ **Rotate credentials regularly** (passwords, access keys)
✅ **Use strong password policy** (length, complexity, expiration)

---

### **Permission Management**

✅ **Start with AWS managed policies** (AmazonS3ReadOnlyAccess, etc.)
✅ **Create custom policies** when managed policies are too broad
✅ **Use conditions** to restrict access (time, IP, MFA, etc.)
✅ **Avoid inline policies** (use managed policies for reusability)

**Policy Condition Example:**
```json
{
  "Condition": {
    "IpAddress": {"aws:SourceIp": "203.0.113.0/24"},
    "Bool": {"aws:MultiFactorAuthPresent": "true"}
  }
}
```

---

### **Access Key Management**

✅ **Use roles instead of access keys** (for applications on AWS)
✅ **Rotate access keys regularly** (every 90 days)
✅ **Delete unused access keys**
✅ **Never commit access keys to code repositories**
✅ **Use temporary credentials** (STS, IAM roles) instead of long-term keys

❌ **Don't:** Hardcode access keys in application code
✅ **Do:** Use IAM roles for EC2, Lambda, ECS (automatic credential rotation)

---

### **Monitoring & Auditing**

✅ **Enable CloudTrail** (log all API calls)
✅ **Use IAM Access Analyzer** (identify resources shared with external entities)
✅ **Review IAM credential reports** (who has access, last used)
✅ **Use CloudWatch** to monitor unusual activity
✅ **Enable AWS Config** to track IAM changes

**IAM Credential Report:**
- Shows all IAM users and credential status
- Last password used, last access key used
- MFA enabled or not

---

### **Cross-Account Access**

✅ **Use IAM roles** for cross-account access (not IAM users)
✅ **Use trust policies** to define which accounts can assume role
✅ **Require external ID** for third-party access (prevent confused deputy problem)

**Cross-Account Role Example:**
1. Account A creates role with trust policy (allows Account B to assume)
2. User in Account B assumes role using `AssumeRole` API
3. User gets temporary credentials to access Account A resources

---

## IAM Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Components:**
- **Effect:** `Allow` or `Deny`
- **Action:** What operations (e.g., `s3:GetObject`)
- **Resource:** Which resources (ARN)
- **Condition:** When to apply (IP, time, MFA, etc.)

---

## Common IAM Scenarios (Exam)

| Scenario | Solution |
|----------|----------|
| EC2 needs to access S3 | **IAM Role** attached to EC2 instance |
| Grant temporary access to contractor | **IAM User** with temporary credentials or **STS** |
| Prevent users from deleting S3 buckets | **IAM Policy** with `Deny` on `s3:DeleteBucket` |
| Require MFA for deleting resources | **IAM Policy** with `"aws:MultiFactorAuthPresent": "true"` condition |
| Cross-account access (Account A → Account B) | **IAM Role** in Account B with trust policy for Account A |
| Restrict access to specific IP range | **IAM Policy** with `"aws:SourceIp"` condition |
| Grant read-only access to entire AWS account | **ReadOnlyAccess** managed policy |
| Limit what users can do (even with Admin permissions) | **Permission Boundaries** |

---

## Exam Tips - IAM Keywords

| Exam Keyword | Solution |
|--------------|----------|
| "Root account" | Enable MFA, don't use for daily tasks |
| "Application on EC2 needs S3 access" | IAM Role (instance profile) |
| "Temporary credentials" | STS AssumeRole, IAM roles |
| "Cross-account access" | IAM Role with trust policy |
| "Third-party access" | IAM Role with external ID |
| "Least privilege" | Specific policies, not `*` |
| "MFA required" | IAM Policy condition `"aws:MultiFactorAuthPresent"` |
| "Audit who accessed what" | CloudTrail |
| "Rotate credentials" | Every 90 days, IAM credential report |

---

## IAM Access Evaluation Logic

**Order of evaluation:**
1. **Deny by default** (implicit deny)
2. **Explicit Allow?** (IAM policy grants permission)
3. **Explicit Deny?** (IAM policy denies permission) → **Deny wins**
4. **Permission Boundary?** (restricts maximum permissions)
5. **SCP?** (Service Control Policy at Organization level)

**Rule:** **Explicit Deny always wins** (cannot be overridden)

---

## Summary - Top 10 IAM Best Practices

1. **Enable MFA for root account** (mandatory!)
2. **Use IAM roles, not access keys** (for AWS applications)
3. **Grant least privilege** (start minimal, add as needed)
4. **Delete unused users and credentials**
5. **Rotate credentials regularly** (every 90 days)
6. **Use IAM groups** (don't assign policies directly to users)
7. **Monitor with CloudTrail and IAM Access Analyzer**
8. **Create individual IAM users** (don't share)
9. **Use strong password policy**
10. **Never use root account for daily tasks**
