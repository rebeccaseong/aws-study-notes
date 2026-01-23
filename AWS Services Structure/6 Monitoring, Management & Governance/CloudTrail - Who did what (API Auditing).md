# CloudTrail - Who did what (API Auditing)

- **What is it? (1-Sentence Pitch):** A service that logs every API call made in your AWS account, providing a complete audit trail of all activity.
- **Core Use Cases:** Answering "Who did what, when, and from where?" for security analysis, compliance auditing, and resource change tracking.
- **Key Distinction:** **CloudWatch** monitors resource *performance* and *operational health*. **CloudTrail** monitors account *activity* and *API usage*.

---

## What CloudTrail Does

CloudTrail records **every API call** made in your AWS account:
- **Who** made the call (IAM user, role, service)
- **What** action was performed (CreateBucket, TerminateInstances, etc.)
- **When** it happened (timestamp)
- **Where** it came from (source IP address)
- **What** resources were affected
- **What** the response was (success or failure)

**Key Point:** CloudTrail is **always enabled by default** (90-day event history in the console), but you need to create a **trail** to store events long-term in S3.

---

## CloudTrail vs CloudWatch vs Config

| Service | What It Monitors | Question It Answers | Use Case |
|---------|------------------|---------------------|----------|
| **CloudTrail** | **API calls** (account activity) | **"Who did what?"** | Security auditing, compliance, troubleshooting unauthorized changes |
| **CloudWatch** | **Metrics** (resource performance) | **"How is my app performing?"** | Monitor CPU, set alarms, view logs |
| **Config** | **Resource configuration** (compliance) | **"Is my infrastructure compliant?"** | Track configuration changes, enforce rules |

**Example Scenario:**
- **CloudTrail:** "Who deleted the S3 bucket?" → Shows IAM user `john` called `DeleteBucket` at 2:30 PM from IP 203.0.113.50
- **CloudWatch:** "Is my EC2 instance healthy?" → Shows CPU at 80%, memory at 60%
- **Config:** "Are all S3 buckets encrypted?" → Shows `my-bucket` is non-compliant (encryption disabled)

---

## Key Features

### **1. Event History (90 Days, Free)**

- **Automatically enabled** for all AWS accounts
- Stores last **90 days** of management events
- View in CloudTrail console (Event History tab)
- **No S3 bucket required**
- **No charge**

**Use Case:** Quick investigation of recent changes (e.g., "Who stopped this EC2 instance yesterday?")

---

### **2. Trails (Long-Term Storage in S3)**

To store events **beyond 90 days**, create a **trail**:

| Feature | Details |
|---------|---------|
| **What It Does** | Continuously logs events to S3 bucket |
| **Retention** | Unlimited (stored in S3, apply S3 lifecycle policies) |
| **Scope** | Single region or **all regions** (recommended) |
| **Log File Delivery** | Every 5 minutes (active events) |
| **Encryption** | S3 Server-Side Encryption (SSE-S3) or SSE-KMS |
| **Validation** | Log file integrity validation (detect tampering) |

**Best Practice:** Create an **organization trail** (logs all accounts in AWS Organizations)

---

### **3. Event Types**

CloudTrail logs three types of events:

| Event Type | What It Logs | Examples | Default in Trail |
|------------|--------------|----------|------------------|
| **Management Events** | **Control plane** operations (create, delete, modify resources) | `RunInstances`, `CreateBucket`, `DeleteDBInstance` | ✅ **Yes** (enabled by default) |
| **Data Events** | **Data plane** operations (read/write data) | S3 `GetObject`, Lambda `Invoke`, DynamoDB `PutItem` | ❌ **No** (must enable, extra cost) |
| **Insights Events** | **Anomalous activity** detection (unusual API call patterns) | Spike in `TerminateInstances` calls | ❌ **No** (must enable, extra cost) |

**Management Events (Control Plane):**
- Creating or deleting resources (EC2, S3, RDS, etc.)
- Modifying resource configurations
- IAM policy changes
- **Free** (included in trail)

**Data Events (Data Plane):**
- S3 object-level operations (`GetObject`, `PutObject`, `DeleteObject`)
- Lambda function invocations (`Invoke`)
- DynamoDB table operations (`GetItem`, `PutItem`, `DeleteItem`)
- **Charged per event** (can be high-volume)
- **Use Case:** Track who accessed sensitive S3 objects, audit Lambda invocations

**Insights Events (Anomaly Detection):**
- Detects unusual activity patterns (e.g., spike in `TerminateInstances` calls)
- Machine learning-based anomaly detection
- **Charged per event**
- **Use Case:** Detect potential security incidents or account compromise

---

### **4. CloudTrail Insights (Anomaly Detection)**

**What It Does:** Analyzes management events to detect **unusual activity** patterns

**How It Works:**
1. CloudTrail establishes a **baseline** of normal API activity
2. Detects **anomalies** (e.g., spike in `RunInstances` calls)
3. Logs **Insights event** to S3 and EventBridge
4. Can trigger alarms via CloudWatch

**Use Case:** Detect compromised credentials (e.g., attacker spinning up 100 EC2 instances)

**Cost:** $0.35 per 100,000 events analyzed

---

### **5. Multi-Region and Organization Trails**

| Trail Type | Scope | Use Case |
|------------|-------|----------|
| **Single Region Trail** | Logs events in one region only | Region-specific compliance |
| **Multi-Region Trail** | Logs events in **all regions** | Recommended for security (single trail for entire account) |
| **Organization Trail** | Logs events for **all accounts** in AWS Organizations | Centralized logging for entire organization |

**Best Practice:** Create one **multi-region organization trail** to log all activity across all accounts and regions.

---

## Common Use Cases

### **1. Security Analysis**

**Scenario:** Investigate unauthorized changes

**Example Questions:**
- "Who deleted the production database?"
- "Who modified the security group to allow SSH from 0.0.0.0/0?"
- "What API calls were made from this suspicious IP address?"

**How CloudTrail Helps:**
- Search Event History or S3 logs for specific API calls
- Filter by user, event name, resource, or time range
- Identify the IAM principal responsible

---

### **2. Compliance Auditing**

**Scenario:** Prove compliance with regulations (PCI-DSS, HIPAA, SOC 2)

**Example Requirements:**
- "Maintain audit logs for all infrastructure changes"
- "Track who accessed sensitive data"
- "Provide evidence of configuration changes"

**How CloudTrail Helps:**
- Long-term retention of API logs in S3
- Log file integrity validation (prove logs weren't tampered with)
- Export logs for compliance reports

---

### **3. Troubleshooting**

**Scenario:** Debug unexpected resource changes

**Example Questions:**
- "Why was this Lambda function deleted?"
- "What changed in my VPC configuration?"
- "Who modified this IAM policy?"

**How CloudTrail Helps:**
- View complete timeline of API calls for affected resources
- Identify the source of changes (user, role, or service)

---

### **4. Detect Compromised Credentials**

**Scenario:** Unusual activity indicates compromised IAM credentials

**How CloudTrail Helps:**
- CloudTrail Insights detects anomalous API call patterns
- Filter logs by source IP to detect access from unknown locations
- Analyze API calls to identify malicious actions (e.g., creating new IAM users, launching crypto-mining instances)

---

## CloudTrail Log File Format

CloudTrail logs are stored in **JSON format** in S3:

**Example Log Entry:**
```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAI234567890EXAMPLE",
    "arn": "arn:aws:iam::123456789012:user/john",
    "accountId": "123456789012",
    "userName": "john"
  },
  "eventTime": "2025-01-23T14:30:00Z",
  "eventSource": "ec2.amazonaws.com",
  "eventName": "TerminateInstances",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "203.0.113.50",
  "userAgent": "aws-cli/2.1.0",
  "requestParameters": {
    "instancesSet": {
      "items": [{"instanceId": "i-1234567890abcdef0"}]
    }
  },
  "responseElements": {
    "instancesSet": {
      "items": [{"instanceId": "i-1234567890abcdef0", "currentState": {"name": "shutting-down"}}]
    }
  }
}
```

**Key Fields:**
- `userIdentity`: Who made the call (IAM user, role, service)
- `eventTime`: When the call was made
- `eventSource`: AWS service (e.g., `ec2.amazonaws.com`)
- `eventName`: API action (e.g., `TerminateInstances`)
- `sourceIPAddress`: Source IP of the request
- `requestParameters`: What was requested
- `responseElements`: What was returned

---

## Integration with Other Services

### **CloudTrail → S3**
- Store logs long-term
- Apply lifecycle policies (e.g., move to Glacier after 90 days)
- Encrypt with SSE-S3 or SSE-KMS

### **CloudTrail → CloudWatch Logs**
- Stream CloudTrail events to CloudWatch Logs
- Create **metric filters** to detect specific events (e.g., "Root account login")
- Trigger **CloudWatch Alarms** (e.g., alert on unauthorized API calls)

**Example Metric Filter:**
- Detect root account usage: Filter for `userIdentity.type = "Root"`
- Alarm when root account is used outside of emergency

### **CloudTrail → EventBridge (CloudWatch Events)**
- Create **event rules** to trigger actions based on API calls
- **Example:** When `TerminateInstances` is called, send SNS notification to security team

**Event Pattern Example:**
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventName": ["TerminateInstances"]
  }
}
```

### **CloudTrail → Athena**
- Query CloudTrail logs in S3 using SQL
- Analyze patterns, generate reports, investigate security incidents

**Example Query:**
```sql
SELECT useridentity.username, eventname, sourceipaddress, eventtime
FROM cloudtrail_logs
WHERE eventname = 'DeleteBucket'
AND eventtime > '2025-01-01'
ORDER BY eventtime DESC;
```

---

## Security Best Practices

### **1. Enable Multi-Region Trail**
- Captures events from all regions (prevents attackers from hiding activity in unused regions)

### **2. Enable Log File Validation**
- Detects if logs were tampered with or deleted
- Uses cryptographic hashing to verify integrity

### **3. Encrypt Logs with KMS**
- Use SSE-KMS instead of SSE-S3 for additional security
- Control who can decrypt logs via KMS key policies

### **4. Restrict S3 Bucket Access**
- Apply least privilege S3 bucket policy
- Only CloudTrail and authorized security team can access logs

### **5. Enable CloudWatch Logs Integration**
- Real-time monitoring and alerting
- Create metric filters for critical events (root usage, IAM policy changes, security group changes)

### **6. Create Organization Trail**
- Centralized logging for all accounts in AWS Organizations
- Prevents individual account owners from disabling logging

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Need to track who deleted an S3 bucket | **CloudTrail** (logs `DeleteBucket` API call with user identity) |
| Compliance requires 7-year retention of API logs | **CloudTrail trail** → S3 with lifecycle policy (move to Glacier after 90 days) |
| Detect unusual spike in EC2 instance launches | **CloudTrail Insights** (anomaly detection) |
| Track who accessed specific S3 objects | **CloudTrail data events** for S3 (enable object-level logging) |
| Alert when root account is used | **CloudTrail → CloudWatch Logs** with metric filter + alarm |
| Investigate security incident across all regions | **Multi-region trail** (captures events from all regions) |
| Prove logs weren't tampered with | **Log file validation** (cryptographic integrity check) |
| Query CloudTrail logs with SQL | **CloudTrail → S3 → Athena** (SQL queries on log files) |
| Automatically respond to API calls | **CloudTrail → EventBridge** → Lambda/SNS/etc. |

---

## Key Exam Tips

1. **CloudTrail = API auditing** ("Who did what?")
2. **CloudWatch = Performance monitoring** ("How is my app?")
3. **Config = Configuration compliance** ("Is my infrastructure compliant?")
4. **Default:** 90-day event history (free, automatic)
5. **Trail:** Long-term storage in S3 (unlimited retention)
6. **Management events:** Control plane (enabled by default, free)
7. **Data events:** Data plane (must enable, extra cost)
8. **Insights events:** Anomaly detection (must enable, extra cost)
9. **Multi-region trail:** Logs events from all regions (recommended)
10. **Organization trail:** Logs all accounts in AWS Organizations
11. **Log file validation:** Detect tampering (cryptographic hashing)
12. **CloudTrail → CloudWatch Logs:** Real-time alerting
13. **CloudTrail → EventBridge:** Trigger actions on API calls
14. **CloudTrail → Athena:** Query logs with SQL
15. **Encryption:** SSE-S3 (default) or SSE-KMS (more secure)

---

## Common Patterns

### **Pattern 1: Security Monitoring**
```
CloudTrail → CloudWatch Logs → Metric Filter → CloudWatch Alarm → SNS
```
**Use Case:** Alert on root account usage, failed login attempts, IAM policy changes

### **Pattern 2: Automated Response**
```
API Call → CloudTrail → EventBridge → Lambda → Remediation
```
**Use Case:** Automatically respond to security group changes (e.g., remove 0.0.0.0/0 SSH rule)

### **Pattern 3: Compliance Logging**
```
CloudTrail → S3 (encrypted with KMS, log validation enabled) → Glacier (after 90 days)
```
**Use Case:** Long-term retention for compliance (PCI-DSS, HIPAA)

### **Pattern 4: Forensic Analysis**
```
CloudTrail → S3 → Athena (SQL queries) → Investigation Report
```
**Use Case:** Investigate security incidents by querying API call logs

---

## Costs

| Item | Cost |
|------|------|
| **Event History (90 days)** | **Free** |
| **First trail** | **Free** (management events only) |
| **Additional trails** | $2.00 per 100,000 management events |
| **Data events** | $0.10 per 100,000 events |
| **Insights events** | $0.35 per 100,000 events analyzed |
| **S3 storage** | Standard S3 pricing (apply lifecycle policies to reduce cost) |

**Cost Optimization:**
- Use **one multi-region trail** (captures all regions, no need for separate trails)
- Enable **data events** only for critical resources (e.g., sensitive S3 buckets)
- Apply **S3 lifecycle policies** (move old logs to Glacier or delete after retention period)

---

## Summary

**CloudTrail** is your **audit trail** for AWS account activity:

| Feature | Description |
|---------|-------------|
| **Purpose** | Log every API call made in your AWS account |
| **Question** | "Who did what, when, and from where?" |
| **Default** | 90-day event history (free, automatic) |
| **Trail** | Long-term storage in S3 (unlimited retention) |
| **Event Types** | Management (control plane), Data (data plane), Insights (anomaly detection) |
| **Scope** | Single region, multi-region, or organization-wide |
| **Integrations** | S3, CloudWatch Logs, EventBridge, Athena |
| **Use Cases** | Security auditing, compliance, troubleshooting, incident response |

**Bottom Line:** Enable a **multi-region organization trail** with **log file validation** and **KMS encryption** for comprehensive security and compliance.
