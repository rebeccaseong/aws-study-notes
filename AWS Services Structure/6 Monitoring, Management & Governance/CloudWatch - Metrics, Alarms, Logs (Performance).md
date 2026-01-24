# CloudWatch - Metrics, Alarms, Logs (Performance)

- **What is it? (1-Sentence Pitch):** The central monitoring and observability service for AWS resources and the applications you run on AWS.
- **Core Use Cases:** Monitor resource performance (CPU, memory, disk), set alarms to trigger actions, centralize logs, create dashboards, detect anomalies.
- **Key Distinction:** **CloudWatch** monitors resource *performance* and *operational health*. **CloudTrail** monitors account *activity* and *API usage*.

---

## What CloudWatch Does

CloudWatch provides visibility into your AWS resources and applications:
- **Monitor** performance metrics (CPU, memory, network, disk)
- **Collect and store** logs from applications and AWS services
- **Trigger alarms** when metrics cross thresholds
- **Visualize** data in dashboards
- **Detect anomalies** using machine learning

**Key Point:** CloudWatch answers "How is my application performing?" whereas CloudTrail answers "Who did what?"

---

## CloudWatch Components

```
┌────────────────────────────────────────────┐
│            Amazon CloudWatch               │
├──────────┬──────────┬──────────┬───────────┤
│ Metrics  │  Alarms  │   Logs   │Dashboards │
├──────────┼──────────┼──────────┼───────────┤
│ Insights │ Synthetics│ Anomaly │ RUM      │
│          │           │Detection│           │
└──────────┴──────────┴──────────┴───────────┘
```

---

## 1. CloudWatch Metrics

**What They Are:** Time-series data points representing the performance of your AWS resources and applications.

### **Default Metrics (Free)**

AWS automatically provides metrics for most services:

| Service | Key Metrics | Interval |
|---------|-------------|----------|
| **EC2** | CPUUtilization, DiskReadOps, NetworkIn/Out | 5 minutes (free), 1 minute (detailed monitoring, $) |
| **EBS** | VolumeReadBytes, VolumeWriteBytes, VolumeThroughputPercentage | 5 minutes |
| **RDS** | CPUUtilization, DatabaseConnections, FreeableMemory, ReadLatency | 1 minute |
| **Lambda** | Invocations, Duration, Errors, Throttles | 1 minute |
| **DynamoDB** | ConsumedReadCapacityUnits, ConsumedWriteCapacityUnits, UserErrors | 1 minute |
| **ELB** | RequestCount, TargetResponseTime, HTTPCode_Target_2XX_Count | 1 minute |
| **S3** | BucketSizeBytes, NumberOfObjects | Daily |

**Important:** EC2 **does NOT** provide memory or disk utilization by default (only disk I/O operations). You must use the **CloudWatch Agent** to collect these.

---

### **Custom Metrics**

You can publish your own application metrics to CloudWatch:

**Use Cases:**
- Application-level metrics (user logins, orders placed, API response times)
- Memory utilization (EC2)
- Disk space utilization (EC2)
- Business metrics (revenue, active users)

**How to Publish:**
```bash
# Using AWS CLI
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "PageLoadTime" \
  --value 245 \
  --unit Milliseconds
```

**From Code (Python):**
```python
import boto3
cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[{
        'MetricName': 'ActiveUsers',
        'Value': 1250,
        'Unit': 'Count'
    }]
)
```

**Key Concepts:**
- **Namespace:** Container for metrics (e.g., "AWS/EC2", "MyApp")
- **Dimensions:** Name/value pairs that identify metric (e.g., InstanceId=i-12345)
- **Resolution:** Standard (1 minute) or High (1 second)
- **Retention:** Automatically aggregated and retained (1 min → 15 days, 5 min → 63 days, 1 hour → 455 days)

**Exam Tip:** Custom metrics require the CloudWatch API or SDK (not automatic)

---

### **Metric Dimensions**

Dimensions identify the specific resource for a metric.

**Example:**
- Namespace: `AWS/EC2`
- MetricName: `CPUUtilization`
- Dimensions: `InstanceId=i-1234567890abcdef0`

**Result:** CPU utilization for that specific EC2 instance

---

### **Metric Math**

Combine metrics using mathematical expressions:

**Example:** Calculate error rate percentage
```
Errors / Invocations * 100
```

**Use Cases:**
- Calculate percentages (error rate, utilization rate)
- Combine metrics from multiple sources
- Create derived metrics for alarms

---

## 2. CloudWatch Alarms

**What They Are:** Automated actions triggered when a metric crosses a threshold.

### **Alarm States**

| State | Meaning |
|-------|---------|
| **OK** | Metric is within threshold |
| **ALARM** | Metric has breached threshold |
| **INSUFFICIENT_DATA** | Not enough data to determine state (new alarm, missing data) |

---

### **Alarm Actions**

When alarm enters **ALARM** state, it can:

| Action | Description | Use Case |
|--------|-------------|----------|
| **SNS Notification** | Send email, SMS, or trigger Lambda | Alert ops team when CPU > 80% |
| **Auto Scaling** | Scale out/in EC2 instances | Add instances when CPU > 70% |
| **EC2 Action** | Stop, terminate, reboot, recover instance | Terminate instance if StatusCheckFailed |
| **Systems Manager Action** | Run automation document | Restart application when memory > 90% |

---

### **Alarm Types**

#### **Metric Alarm**

Based on a single metric crossing a threshold.

**Example:**
```
Metric: EC2 CPUUtilization
Threshold: > 80%
Period: 5 minutes
Evaluation: 2 consecutive periods
Action: Send SNS notification
```

**Result:** Alarm triggers if CPU > 80% for 10 minutes (2 periods of 5 minutes)

---

#### **Composite Alarm**

Combines multiple alarms using AND/OR logic.

**Example:**
```
Alarm1: CPU > 80% (ALARM state)
Alarm2: Memory > 90% (ALARM state)
Composite: Alarm1 AND Alarm2
Action: Send SNS to ops team
```

**Use Case:** Reduce noise by only alerting when multiple conditions are met

---

### **Alarm Best Practices**

**Good Threshold Example:**
```
Metric: RDS CPUUtilization
Threshold: > 80%
Periods: 3 consecutive periods of 5 minutes
Treat missing data: As breaching
```

**Why 3 periods?** Avoids false positives from temporary spikes

**Missing Data Options:**
- **notBreaching:** Treat as OK (default for most)
- **breaching:** Treat as ALARM
- **ignore:** Maintain current alarm state
- **missing:** Insufficient data

---

## 3. CloudWatch Logs

**What It Is:** Centralized log storage and analysis service.

### **Key Concepts**

| Concept | Description |
|---------|-------------|
| **Log Group** | Container for log streams (e.g., `/aws/lambda/my-function`) |
| **Log Stream** | Sequence of log events from the same source (e.g., instance ID, date) |
| **Log Event** | Individual log entry with timestamp and message |
| **Retention Period** | How long logs are stored (1 day → Never expire) |

---

### **Log Sources**

CloudWatch Logs can collect from:

| Source | How to Enable |
|--------|---------------|
| **Lambda** | Automatic (logs to `/aws/lambda/<function-name>`) |
| **EC2** | Install CloudWatch Agent |
| **ECS/Fargate** | Configure task definition to send logs |
| **RDS** | Enable log exports (error logs, slow query logs) |
| **VPC Flow Logs** | Enable in VPC settings |
| **CloudTrail** | Send API logs to CloudWatch Logs |
| **Elastic Beanstalk** | Automatic |
| **API Gateway** | Enable execution logging |

---

### **CloudWatch Agent**

**Purpose:** Collect logs and additional metrics from EC2 instances and on-premises servers.

**What It Collects:**
- **Logs:** Application logs, system logs, custom log files
- **Metrics:** Memory utilization, disk space utilization, swap usage
- **Custom Metrics:** Any metric from your application

**How to Install:**
1. Create IAM role with `CloudWatchAgentServerPolicy`
2. Attach role to EC2 instance
3. Install agent (SSM, manual, or user data)
4. Configure agent (JSON config file)
5. Start agent

**Exam Tip:** If the question mentions "memory utilization" or "disk space utilization" on EC2 → You need the **CloudWatch Agent**

---

### **Log Insights**

**What It Is:** Query and analyze log data using a purpose-built query language.

**Use Cases:**
- Find errors in application logs
- Analyze API Gateway request patterns
- Troubleshoot Lambda failures
- Calculate statistics (average response time, error rate)

**Example Queries:**

**Find errors in logs:**
```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

**Calculate average Lambda duration:**
```
fields @duration
| stats avg(@duration), max(@duration), min(@duration)
```

**Count 5XX errors by URL:**
```
fields @timestamp, status, url
| filter status >= 500
| stats count() by url
| sort count() desc
```

**Key Fields:**
- `@timestamp`: Log event time
- `@message`: Log message
- `@duration`: Lambda duration (ms)
- `@billedDuration`: Lambda billed duration
- `@memoryUsed`: Lambda memory used

**Exam Tip:** Log Insights uses SQL-like queries to analyze logs (no need to export to S3)

---

### **Metric Filters**

**What They Are:** Extract metrics from log data and create custom CloudWatch metrics.

**Use Case:** Count occurrences of specific patterns in logs (errors, successful logins, API calls)

**Example:**
1. **Log:** Application logs contain "ERROR" when errors occur
2. **Metric Filter:** Match pattern "ERROR"
3. **Custom Metric:** `ErrorCount` in namespace `MyApp`
4. **Alarm:** Trigger SNS when `ErrorCount` > 10 in 5 minutes

**Common Patterns:**
- Count errors: `[ERROR]`
- Extract response times: `[..., status, responseTime]`
- Monitor failed logins: `Authentication failed`

---

### **Log Subscriptions**

**What They Are:** Real-time streaming of log events to destinations.

**Destinations:**

| Destination | Use Case |
|-------------|----------|
| **Lambda** | Process logs in real-time (parse, filter, transform) |
| **Kinesis Data Streams** | Stream logs for real-time analytics |
| **Kinesis Data Firehose** | Send logs to S3, Redshift, OpenSearch, or third-party (Splunk, Datadog) |

**Use Case:** Send VPC Flow Logs to Lambda for real-time analysis of network traffic patterns

---

### **Log Retention and Export**

**Retention Policies:**
- Default: Never expire
- Options: 1 day, 3 days, 5 days, 7 days, 1 week, 2 weeks, 1 month, 2 months, ... Never expire
- **Cost optimization:** Set retention to match compliance requirements (e.g., 90 days), delete old logs

**Export to S3:**
- Batch export for archival or long-term analysis
- Use S3 Lifecycle Policies to move to Glacier
- Analyze with Athena

**Exam Tip:** For long-term archival, export CloudWatch Logs to S3 → Glacier

---

## 4. CloudWatch Dashboards

**What They Are:** Customizable visual displays of metrics and alarms.

**Features:**
- Multiple widgets (line graphs, numbers, text)
- Cross-region metrics (monitor resources in all regions on one dashboard)
- Cross-account metrics (with cross-account IAM roles)
- Auto-refresh

**Use Cases:**
- NOC (Network Operations Center) displays
- Executive dashboards
- Application health monitoring

**Exam Tip:** Dashboards can display metrics from multiple regions and multiple accounts

---

## 5. CloudWatch Anomaly Detection

**What It Is:** Machine learning-based detection of unusual metric behavior.

**How It Works:**
1. Create anomaly detection model for a metric
2. CloudWatch builds baseline using historical data
3. CloudWatch creates upper/lower bounds (expected range)
4. Set alarm when metric goes outside bounds

**Use Cases:**
- Detect unusual traffic patterns (DDoS, sudden load)
- Identify performance degradation
- Catch cost anomalies (unexpected resource usage)

**Exam Trigger:**
- "Detect unusual behavior"
- "Automatically identify anomalies"
- "Machine learning-based monitoring"

---

## 6. CloudWatch Synthetics

**What It Is:** Create **canaries** (automated scripts) to monitor endpoints and APIs.

**How It Works:**
- Canaries run on a schedule (e.g., every 5 minutes)
- Check availability, latency, and functionality
- Alert if checks fail

**Use Cases:**
- Monitor website availability (HTTP checks)
- Test API endpoints
- Verify multi-step workflows (login → add to cart → checkout)

**Canary Types:**
- **Heartbeat:** Simple ping/HTTP check
- **API Canary:** Test REST API endpoints
- **Broken Link Checker:** Scan website for broken links
- **Visual Monitoring:** Take screenshots, detect UI regressions
- **GUI Workflow:** Automate user flows (Selenium-based)

**Exam Trigger:**
- "Proactive monitoring"
- "Test endpoint availability"
- "Synthetic traffic"

---

## 7. CloudWatch RUM (Real User Monitoring)

**What It Is:** Collect and analyze real user performance data from web applications.

**What It Measures:**
- Page load times
- JavaScript errors
- HTTP requests
- User sessions

**Use Case:** Understand real user experience (not synthetic tests)

**Exam Tip:** RUM = Real users, Synthetics = Simulated users

---

## CloudWatch vs CloudTrail vs Config vs X-Ray

| Service | What It Monitors | Question It Answers | Use Case |
|---------|------------------|---------------------|----------|
| **CloudWatch** | **Resource performance** (metrics) | **"How is my app performing?"** | Monitor CPU, set alarms, view logs |
| **CloudTrail** | **API calls** (account activity) | **"Who did what?"** | Security auditing, compliance |
| **Config** | **Resource configuration** (compliance) | **"Is my infrastructure compliant?"** | Track configuration changes, enforce rules |
| **X-Ray** | **Application traces** (distributed systems) | **"Where is the bottleneck?"** | Debug microservices, trace requests |

---

## Integration with Other Services

### **CloudWatch + Auto Scaling**
- CloudWatch alarms trigger scaling policies
- **Example:** CPU > 70% → Scale out (add instances)

### **CloudWatch + Lambda**
- Lambda automatically sends logs to CloudWatch Logs
- CloudWatch Events/EventBridge can trigger Lambda

### **CloudWatch + SNS**
- Alarms send notifications via SNS (email, SMS, Lambda)

### **CloudWatch + EventBridge**
- CloudWatch Events migrated to EventBridge
- Event-driven automation based on metric alarms

### **CloudWatch + Systems Manager**
- Alarms can trigger SSM automation documents
- **Example:** Memory > 90% → Run script to clear cache

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Monitor EC2 CPU utilization | **CloudWatch Metrics** (default metric, 5 min free) |
| Monitor EC2 memory utilization | **CloudWatch Agent** (custom metric) |
| Alert when RDS CPU > 80% for 10 minutes | **CloudWatch Alarm** (metric alarm with 2 periods of 5 min) |
| Centralize Lambda logs for analysis | **CloudWatch Logs** (automatic) |
| Query logs to find errors | **CloudWatch Log Insights** (SQL-like queries) |
| Count "ERROR" in logs, create metric | **Metric Filter** (extract metric from logs) |
| Stream logs to S3 for archival | **Log Subscription → Kinesis Firehose → S3** |
| Visualize metrics from multiple regions | **CloudWatch Dashboard** (cross-region) |
| Detect unusual traffic spike | **CloudWatch Anomaly Detection** (ML-based) |
| Test API availability every 5 minutes | **CloudWatch Synthetics** (canary) |
| Automatically scale based on custom metric | **Custom Metric + CloudWatch Alarm + Auto Scaling** |
| Reduce CloudWatch Logs cost | Set **retention policy** (delete after 90 days, export to S3) |

---

## Key Exam Tips

1. **CloudWatch Metrics:** Default metrics provided for most services (5 min free, 1 min detailed = $)
2. **EC2 memory/disk space:** NOT default metrics → Use **CloudWatch Agent**
3. **Custom Metrics:** Use `PutMetricData` API (1 min standard, 1 sec high-resolution)
4. **Alarms:** Trigger actions (SNS, Auto Scaling, EC2 actions) based on thresholds
5. **Composite Alarms:** Combine multiple alarms (AND/OR logic) to reduce noise
6. **CloudWatch Logs:** Centralize logs from Lambda, EC2, ECS, VPC Flow Logs, CloudTrail
7. **CloudWatch Agent:** Required for EC2 memory/disk space metrics and custom logs
8. **Log Insights:** Query logs with SQL-like syntax (no need to export to S3)
9. **Metric Filters:** Extract metrics from logs (e.g., count errors)
10. **Log Subscriptions:** Stream logs to Lambda, Kinesis, Firehose (real-time processing)
11. **Retention:** Set log retention policy to save costs (default = never expire)
12. **Dashboards:** Cross-region, cross-account visualization
13. **Anomaly Detection:** ML-based detection of unusual metric behavior
14. **Synthetics:** Automated scripts (canaries) to test endpoints
15. **RUM:** Real user monitoring (page load, errors, sessions)

---

## Cost Optimization

**Free Tier:**
- 10 custom metrics
- 10 alarms
- 1 million API requests
- 5 GB log data ingestion and storage
- 5 GB log data scanned by Log Insights

**Cost Drivers:**
- Custom metrics: $0.30/metric/month
- Detailed monitoring (1 min EC2): $2.10/instance/month
- Alarms: $0.10/alarm/month
- Logs: $0.50/GB ingested, $0.03/GB stored
- Log Insights: $0.005/GB scanned
- Dashboards: $3/dashboard/month (first 3 free)

**Optimization Tips:**
- Use default metrics when possible (free)
- Set log retention policies (delete after 90 days if not needed)
- Export old logs to S3 → Glacier
- Use metric filters sparingly (costs for each query)
- Delete unused dashboards

---

## Summary

**CloudWatch** is your **central monitoring and observability service** for AWS:

| Component | Purpose |
|-----------|---------|
| **Metrics** | Monitor resource performance (CPU, memory, network) |
| **Alarms** | Trigger actions when thresholds are breached |
| **Logs** | Centralize and analyze log data |
| **Dashboards** | Visualize metrics and alarms |
| **Log Insights** | Query logs with SQL-like syntax |
| **Anomaly Detection** | ML-based detection of unusual behavior |
| **Synthetics** | Automated endpoint monitoring (canaries) |
| **RUM** | Real user monitoring (web application performance) |

**Bottom Line:** CloudWatch monitors **performance** and **operational health**. Use CloudWatch Agent for EC2 memory/disk metrics, set alarms to trigger Auto Scaling or SNS, and use Log Insights to analyze logs.
