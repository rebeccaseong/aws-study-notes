# Comparison: CloudWatch vs CloudTrail vs Config vs X-Ray

Quick reference for choosing the right monitoring and observability service.

---

## High-Level Decision Tree

```
What do you need to monitor?

├─ Performance metrics and logs (CPU, memory, application logs)
│  └─ CloudWatch
│
├─ API activity audit ("who did what?")
│  └─ CloudTrail
│
├─ Resource configuration changes ("what changed?")
│  └─ Config
│
└─ Distributed application tracing (microservices, request flow)
   └─ X-Ray
```

---

## Comparison Table

| Aspect | CloudWatch | CloudTrail | Config | X-Ray |
|--------|------------|------------|--------|-------|
| **Purpose** | **Performance monitoring** | **API audit logging** | **Configuration compliance** | **Distributed tracing** |
| **What It Monitors** | Metrics, logs, events | API calls (who did what) | Resource configurations | Request traces across services |
| **Use Case** | "CPU is high" | "Who deleted the S3 bucket?" | "Is my infrastructure compliant?" | "Where is the latency bottleneck?" |
| **Question Answered** | **"How is it performing?"** | **"Who did what, when?"** | **"Is configuration correct?"** | **"How is my request flowing?"** |
| **Data Type** | Metrics, logs, events | API call logs | Resource configurations, snapshots | Trace data, service maps |
| **Retention** | Metrics (15 months), Logs (indefinite) | 90 days (default), S3 (long-term) | Config history | 30 days (default) |
| **Alarms** | ✅ Yes (metric-based) | ❌ No | ✅ Yes (compliance rules) | ❌ No |
| **Cost** | $ | $ | $$ | $ |

---

## When to Use Each Service

### **Use CloudWatch If:**

✅ Need to **monitor performance metrics** (CPU, memory, disk, network)
✅ Want to **set alarms** (alert when CPU >80%)
✅ Need to **collect and analyze logs** (application logs, system logs)
✅ Want **dashboards** for visualizing metrics
✅ Need **automated actions** (trigger Lambda, Auto Scaling on alarm)

**Common Use Cases:**
- Monitor EC2 CPU utilization
- Collect application logs from Lambda, ECS
- Set alarms for high latency, error rates
- Create dashboards for operations team
- Trigger Auto Scaling based on metrics

**Key Features:**
- **Metrics:** CPU, memory, disk, network, custom metrics
- **Logs:** Centralized log management (CloudWatch Logs)
- **Alarms:** Alert on metric thresholds
- **Events (EventBridge):** React to AWS events (EC2 state change, etc.)

---

### **Use CloudTrail If:**

✅ Need **audit logs of API calls** (who did what)
✅ Need to **track changes** to AWS resources
✅ Want **compliance and governance** (security auditing)
✅ Need to **investigate security incidents** (unauthorized access, deletion)
✅ Want to **track user activity** across AWS accounts

**Common Use Cases:**
- "Who deleted the S3 bucket?"
- "Who modified the security group?"
- "When was this EC2 instance terminated?"
- Compliance auditing (track all API calls)
- Security investigation (identify unauthorized access)

**Key Features:**
- **API Call Logging:** Records all API calls (console, CLI, SDK)
- **Event History:** 90 days in CloudTrail console (free)
- **Log to S3:** Long-term storage of API logs
- **CloudTrail Insights:** Detect unusual API activity

**What CloudTrail Logs:**
- Who made the API call
- When it was made
- What action was performed
- Which resource was affected
- Source IP address

---

### **Use Config If:**

✅ Need to **track resource configuration changes**
✅ Want to **ensure compliance** with organizational policies
✅ Need **configuration history** (how did my VPC look 6 months ago?)
✅ Want **automated compliance checks** (e.g., "all S3 buckets must be encrypted")
✅ Need to **audit resource relationships** (which SG is attached to which EC2?)

**Common Use Cases:**
- "Is all my data encrypted?" (compliance rule)
- "Show me all non-compliant resources"
- "What changed in my VPC yesterday?"
- Track configuration drift
- Ensure security groups don't allow 0.0.0.0/0 on port 22

**Key Features:**
- **Configuration Recording:** Tracks configuration changes over time
- **Compliance Rules:** Define rules (managed or custom)
- **Configuration Snapshots:** Point-in-time view of resources
- **Remediation:** Automatically fix non-compliant resources (via SSM Automation)

---

### **Use X-Ray If:**

✅ Need to **trace requests** across microservices
✅ Want to **identify performance bottlenecks**
✅ Need to **debug distributed applications**
✅ Want **service map** to visualize dependencies
✅ Building **serverless or container-based architectures**

**Common Use Cases:**
- Trace a request from API Gateway → Lambda → DynamoDB
- Identify which microservice is causing latency
- Debug errors in distributed applications
- Visualize service dependencies
- Analyze performance of individual service calls

**Key Features:**
- **Service Map:** Visual representation of application architecture
- **Trace Analysis:** End-to-end request tracing
- **Latency Analysis:** Identify slow components
- **Error Analysis:** Pinpoint errors in request flow

---

## Detailed Comparison

### **CloudWatch**

| Feature | Details |
|---------|---------|
| **Metrics** | System (CPU, network) + Custom (application-level) |
| **Logs** | Centralized log management (CloudWatch Logs) |
| **Alarms** | Alert on metric thresholds (CPU >80%, Lambda errors >10) |
| **Dashboards** | Visualize metrics, create custom dashboards |
| **Events** | EventBridge (react to events, schedule tasks) |

**Example Metrics:**
- EC2: `CPUUtilization`, `NetworkIn`, `DiskReadOps`
- Lambda: `Invocations`, `Errors`, `Duration`
- ALB: `RequestCount`, `TargetResponseTime`, `HealthyHostCount`

---

### **CloudTrail**

| Feature | Details |
|---------|---------|
| **API Logging** | Records all API calls (who, what, when, where) |
| **Event Types** | Management events (control plane), Data events (S3 object-level, Lambda invocations) |
| **Retention** | 90 days in console (free), indefinite in S3 (additional cost) |
| **Insights** | Detect unusual API activity (anomaly detection) |

**Example Events:**
- `RunInstances` (EC2 launched)
- `DeleteBucket` (S3 bucket deleted)
- `PutBucketPolicy` (S3 bucket policy changed)

---

### **Config**

| Feature | Details |
|---------|---------|
| **Configuration Items** | Snapshot of resource configuration |
| **Configuration History** | Track changes over time |
| **Compliance Rules** | Managed rules (AWS-provided) or custom (Lambda-based) |
| **Remediation** | Automatically fix non-compliant resources |

**Example Rules:**
- `s3-bucket-server-side-encryption-enabled` (all S3 buckets encrypted?)
- `required-tags` (all resources have required tags?)
- `restricted-ssh` (no security groups allow 0.0.0.0/0 on port 22)

---

### **X-Ray**

| Feature | Details |
|---------|---------|
| **Traces** | End-to-end request path across services |
| **Segments** | Individual service components (Lambda, API Gateway, DynamoDB) |
| **Service Map** | Visual graph of service dependencies |
| **Sampling** | Control which requests to trace (reduce cost) |

**Supported Services:**
- Lambda, API Gateway, ECS, EC2, Elastic Beanstalk
- DynamoDB, SQS, SNS, S3

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| "CPU utilization is too high, alert me" | **CloudWatch Alarms** |
| "Who deleted the S3 bucket?" | **CloudTrail** |
| "Are all my S3 buckets encrypted?" | **Config** (compliance rule) |
| "Trace a request across microservices" | **X-Ray** |
| "Monitor application logs" | **CloudWatch Logs** |
| "Track configuration changes to VPC" | **Config** |
| "Identify slow Lambda function" | **CloudWatch** (metrics) or **X-Ray** (traces) |
| "Audit all API calls for compliance" | **CloudTrail** |
| "Automatically fix non-compliant resources" | **Config** (remediation) |
| "Visualize service dependencies" | **X-Ray** (service map) |

---

## Key Exam Tips

1. **CloudWatch = Performance monitoring** (metrics, logs, alarms)
2. **CloudTrail = API auditing** (who did what, when)
3. **Config = Configuration compliance** (is it configured correctly?)
4. **X-Ray = Distributed tracing** (request flow, bottlenecks)
5. **"Who deleted it?"** → CloudTrail
6. **"Is it configured correctly?"** → Config
7. **"Is it performing well?"** → CloudWatch
8. **"Where is the bottleneck?"** → X-Ray
9. **CloudTrail logs API calls**, **Config tracks resource configurations**
10. **CloudWatch alarms can trigger actions** (Lambda, Auto Scaling, SNS)

---

## Service Combinations (Common Patterns)

### **CloudWatch + Lambda**
- CloudWatch Alarm triggers Lambda function (auto-remediation)

### **CloudTrail + CloudWatch**
- CloudTrail logs → CloudWatch Logs → Metric filter → Alarm
- Example: Alert when someone disables MFA

### **Config + Systems Manager**
- Config detects non-compliance → SSM Automation fixes it

### **X-Ray + CloudWatch**
- X-Ray traces requests, CloudWatch monitors metrics
- Combined view: ServiceLens (unified observability)

---

## Summary

| Service | What It Does | Key Question |
|---------|--------------|--------------|
| **CloudWatch** | Performance monitoring | "How is it performing?" |
| **CloudTrail** | API audit logging | "Who did what?" |
| **Config** | Configuration compliance | "Is it configured correctly?" |
| **X-Ray** | Distributed tracing | "Where is the request going?" |
