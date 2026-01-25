# Trusted Advisor - Best Practice Checklist

- **What is it? (1-Sentence Pitch):** An automated online tool that provides real-time guidance to help you provision your resources following AWS best practices across five pillars.

---

## Overview

Trusted Advisor acts as your automated cloud consultant, continuously analyzing your AWS environment and comparing it against AWS best practices. Think of it as having an AWS Solutions Architect constantly reviewing your account.

**Key Analogy:** Trusted Advisor is like a "health checkup" for your AWS account - it scans for issues and recommends improvements.

---

## The Five Pillars (Categories)

Trusted Advisor organizes its checks into **five categories** that align with AWS Well-Architected Framework principles:

| **Pillar** | **Focus** | **Example Checks** |
|---|---|---|
| **Cost Optimization** | Reduce unnecessary spending | Idle EC2 instances, underutilized EBS volumes |
| **Performance** | Improve speed and efficiency | Overutilized resources, CloudFront configuration |
| **Security** | Strengthen security posture | Open security groups, MFA on root, S3 permissions |
| **Fault Tolerance** | Increase resilience | RDS Multi-AZ, ELB health checks, backup coverage |
| **Service Limits** | Avoid hitting quotas | VPC limits, EC2 instance limits, EBS volume limits |

---

## Pillar 1: Cost Optimization

**Goal:** Identify ways to reduce your AWS bill by eliminating waste.

### Key Checks

| **Check Name** | **What It Detects** | **Recommendation** |
|---|---|---|
| **Idle EC2 Instances** | EC2 instances with < 10% average CPU utilization over 14 days | Stop or terminate idle instances |
| **Underutilized EBS Volumes** | Volumes with < 1 IOPS per day over 14 days | Delete unattached/unused volumes |
| **Unassociated Elastic IPs** | Elastic IPs not attached to running instances | Release unused Elastic IPs ($0.005/hour charge!) |
| **Idle Load Balancers** | ELBs with no registered targets or minimal traffic | Delete unused load balancers |
| **Reserved Instance Optimization** | RI coverage vs. On-Demand usage | Purchase RIs for steady-state workloads |
| **Savings Plans Coverage** | Compute usage not covered by Savings Plans | Consider Savings Plans for flexible compute |
| **Amazon RDS Idle DB Instances** | RDS instances with no connections for 7+ days | Stop or delete idle databases |
| **Route 53 Latency Resource Record Sets** | Incomplete latency routing configurations | Complete or remove partial configurations |
| **Low Utilization Amazon EC2 Instances** | Instances where downsizing would save money | Right-size instances |

### Cost Optimization Best Practices

- **Review weekly:** Cost waste adds up quickly
- **Set up alerts:** Use EventBridge to trigger notifications for new findings
- **Automate responses:** Use Lambda to stop idle resources automatically

**Exam Tip:** "Reduce costs" + "identify idle resources" = Trusted Advisor Cost Optimization

---

## Pillar 2: Performance

**Goal:** Identify configuration issues that may degrade performance.

### Key Checks

| **Check Name** | **What It Detects** | **Recommendation** |
|---|---|---|
| **High Utilization EC2 Instances** | CPU utilization consistently > 90% | Scale up or add instances |
| **EBS Throughput Optimization** | Volumes that could benefit from gp3 provisioned throughput | Upgrade or optimize volume configuration |
| **CloudFront Alternate Domain Names** | Missing SSL certificates for custom domains | Configure proper SSL certificates |
| **CloudFront Header Forwarding** | Unnecessary headers reducing cache hit ratio | Minimize forwarded headers |
| **CloudFront Content Delivery Optimization** | Content that could benefit from caching | Increase TTL, optimize cache behavior |
| **EC2 to EBS Throughput Optimization** | Instance/volume throughput mismatch | Match instance type to volume capabilities |
| **Large Number of EC2 Security Group Rules** | Security groups with excessive rules (>50) | Consolidate or simplify rules |
| **Overutilized EBS Magnetic Volumes** | Magnetic volumes with high IOPS | Migrate to SSD-backed volumes |

### Performance Best Practices

- **Right-size resources:** Don't over-provision or under-provision
- **Use appropriate storage types:** Match EBS volume types to workload needs
- **Optimize caching:** CloudFront configuration impacts performance significantly

**Exam Tip:** "Improve performance" + "identify bottlenecks" = Trusted Advisor Performance

---

## Pillar 3: Security

**Goal:** Identify security vulnerabilities and misconfigurations.

### Key Checks

| **Check Name** | **What It Detects** | **Recommendation** |
|---|---|---|
| **Security Groups - Unrestricted Access** | Security groups with 0.0.0.0/0 on sensitive ports (SSH, RDP) | Restrict to known IP ranges |
| **MFA on Root Account** | Root account without MFA enabled | Enable MFA immediately |
| **IAM Access Key Rotation** | Access keys not rotated in 90+ days | Rotate access keys regularly |
| **IAM Use** | Root account being used for daily operations | Create IAM users instead |
| **IAM Password Policy** | Weak password policy settings | Strengthen password requirements |
| **S3 Bucket Permissions** | S3 buckets with public access | Remove public access unless required |
| **S3 Bucket Logging** | S3 buckets without access logging | Enable logging for audit trails |
| **EBS Public Snapshots** | EBS snapshots shared publicly | Make snapshots private |
| **RDS Public Snapshots** | RDS snapshots shared publicly | Make snapshots private |
| **RDS Security Group Access Risk** | RDS security groups with unrestricted access | Restrict database access |
| **CloudTrail Logging** | CloudTrail not enabled | Enable CloudTrail for all regions |
| **Exposed Access Keys** | Access keys exposed in public repositories | Rotate immediately, revoke exposed keys |
| **IAM Root Access Key** | Root account has active access keys | Delete root access keys |

### Security Priority Order

1. **Critical (Fix Immediately):**
   - MFA on root account
   - Exposed access keys
   - Root access keys

2. **High (Fix Within 24 Hours):**
   - Security groups with unrestricted access
   - Public S3 buckets (if unintentional)
   - Public EBS/RDS snapshots

3. **Medium (Fix Within 1 Week):**
   - IAM access key rotation
   - Password policy improvements
   - CloudTrail not enabled

**Exam Tip:** Security checks are ALWAYS available in the **free tier** (even Basic support)

---

## Pillar 4: Fault Tolerance

**Goal:** Identify configurations that could impact availability and resilience.

### Key Checks

| **Check Name** | **What It Detects** | **Recommendation** |
|---|---|---|
| **Amazon RDS Multi-AZ** | RDS instances without Multi-AZ enabled | Enable Multi-AZ for production databases |
| **Amazon RDS Backups** | RDS instances without automated backups | Enable automated backups |
| **Amazon EBS Snapshots** | EBS volumes without recent snapshots | Create regular snapshots |
| **EC2 Availability Zone Balance** | Unbalanced instance distribution across AZs | Distribute instances evenly |
| **ELB Connection Draining** | Load balancers without connection draining | Enable connection draining |
| **ELB Cross-Zone Load Balancing** | Load balancers without cross-zone enabled | Enable cross-zone load balancing |
| **ELB Health Check Configuration** | Missing or misconfigured health checks | Configure proper health checks |
| **Auto Scaling Group Health Check** | ASG not using ELB health checks | Use ELB health checks for web apps |
| **Auto Scaling Group Resources** | ASG resources approaching limits | Monitor and increase limits |
| **VPN Tunnel Redundancy** | Single VPN tunnel instead of redundant pair | Create redundant VPN tunnels |
| **Route 53 DNS Failover** | Missing health checks for failover routing | Configure health checks |
| **Aurora DB Instance Accessibility** | Aurora instances in single AZ | Use Multi-AZ for production |

### Fault Tolerance Best Practices

- **Multi-AZ everything:** Databases, instances, load balancers
- **Automate backups:** EBS snapshots, RDS automated backups
- **Test failover:** Regular DR testing
- **Health checks:** Configure proper health monitoring

**Exam Tip:** "High availability" + "disaster recovery" = Trusted Advisor Fault Tolerance

---

## Pillar 5: Service Limits (Quotas)

**Goal:** Identify when you're approaching AWS service limits.

### Key Checks

| **Check Name** | **Default Limit** | **Action When Approaching** |
|---|---|---|
| **VPCs** | 5 per region | Request limit increase |
| **EC2 On-Demand Instances** | Varies by instance type | Request limit increase |
| **EBS General Purpose SSD (gp2/gp3)** | 50 TB per region | Request limit increase |
| **EBS Provisioned IOPS SSD (io1/io2)** | 200 TB per region | Request limit increase |
| **Elastic IPs** | 5 per region | Request limit increase |
| **Auto Scaling Groups** | 200 per region | Request limit increase |
| **ELB Classic Load Balancers** | 20 per region | Request limit increase |
| **IAM Users** | 5,000 per account | Review and consolidate |
| **IAM Roles** | 1,000 per account | Request limit increase |
| **IAM Groups** | 300 per account | Request limit increase |
| **RDS DB Instances** | 40 per region | Request limit increase |
| **RDS Storage Quota** | 100 TB per region | Request limit increase |
| **S3 Buckets** | 100 per account | Request limit increase |
| **Lambda Concurrent Executions** | 1,000 per region | Request limit increase |
| **CloudFormation Stacks** | 200 per region | Request limit increase |

### Service Limits Best Practices

- **Monitor proactively:** Don't wait until you hit a limit
- **Request increases early:** Limit increases can take time
- **Use Service Quotas:** New console for managing limits
- **Automate monitoring:** EventBridge + Lambda for alerts

**Exam Tip:** Trusted Advisor monitors service limits; **Service Quotas** is where you request increases

---

## Support Plan Comparison

### What's Available by Support Plan

| **Check Category** | **Basic (Free)** | **Developer** | **Business** | **Enterprise** |
|---|---|---|---|---|
| **Service Limits** | 7 checks | 7 checks | All checks | All checks |
| **Security (Core)** | 6 checks | 6 checks | All checks | All checks |
| **Cost Optimization** | ❌ Limited | ❌ Limited | ✅ All | ✅ All |
| **Performance** | ❌ Limited | ❌ Limited | ✅ All | ✅ All |
| **Fault Tolerance** | ❌ Limited | ❌ Limited | ✅ All | ✅ All |
| **API Access** | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| **CloudWatch Integration** | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| **Programmatic Refresh** | ❌ No | ❌ No | ✅ Yes | ✅ Yes |

### Free Tier Checks (Always Available)

**Security (6 Checks):**
1. S3 Bucket Permissions
2. Security Groups - Unrestricted Access (Specific Ports)
3. IAM Use
4. MFA on Root Account
5. EBS Public Snapshots
6. RDS Public Snapshots

**Service Limits (7 Checks):**
1. Auto Scaling Groups
2. EBS Active Volumes
3. EC2 On-Demand Instances
4. Elastic IPs
5. IAM Users
6. RDS DB Instances
7. VPCs

**Exam Tip:** Memorize that **Basic/Developer support = 7 Service Limits + 6 Security checks ONLY**

---

## API Access & Automation

### Trusted Advisor API (Business/Enterprise Only)

```
# Describe all checks
aws support describe-trusted-advisor-checks --language en

# Get check results
aws support describe-trusted-advisor-check-result --check-id <check-id>

# Refresh a check
aws support refresh-trusted-advisor-check --check-id <check-id>
```

### Refresh Frequency

| **Support Plan** | **Manual Refresh** | **Automatic Refresh** |
|---|---|---|
| Basic/Developer | Every 24 hours (console only) | Weekly |
| Business/Enterprise | Every 5 minutes (API) | Varies by check |

**Note:** Some checks cannot be refreshed on-demand (they run on AWS's schedule)

---

## Integration with EventBridge

### Architecture

```
Trusted Advisor ──> EventBridge ──> SNS/Lambda/CloudWatch
     │                                      │
     │                                      ▼
     └── Check status changes ──> Trigger automation
```

### Sample EventBridge Rule Pattern

```json
{
  "source": ["aws.trustedadvisor"],
  "detail-type": ["Trusted Advisor Check Item Refresh Notification"],
  "detail": {
    "status": ["WARN", "ERROR"]
  }
}
```

### Common Automation Patterns

| **Pattern** | **Trigger** | **Action** |
|---|---|---|
| Alert on security issues | Status = ERROR | SNS notification to security team |
| Auto-remediate idle resources | Cost check = WARN | Lambda to stop idle EC2 |
| Track in dashboard | Any status change | CloudWatch custom metric |
| Create tickets | Status = ERROR | Lambda to create Jira/ServiceNow ticket |

---

## Integration with CloudWatch

### Trusted Advisor Metrics in CloudWatch

**Namespace:** `AWS/TrustedAdvisor`

| **Metric** | **Description** |
|---|---|
| `GreenChecks` | Number of checks with no issues |
| `YellowChecks` | Number of checks with warnings |
| `RedChecks` | Number of checks with errors |

### Setting Up CloudWatch Alarms

```
CloudWatch Alarm: RedChecks > 0
Action: SNS notification to ops team
```

**Exam Tip:** Trusted Advisor + CloudWatch = Real-time monitoring of best practice compliance

---

## Organizational View (AWS Organizations)

### Features

- **Aggregated view:** See Trusted Advisor across all accounts
- **Priority recommendations:** Focus on highest-impact issues
- **Downloadable reports:** CSV exports for analysis

### Requirements

- AWS Organizations with all features enabled
- Business or Enterprise support on management account
- Trusted Advisor enabled in member accounts

---

## Exam Scenarios

### Scenario 1: Cost Reduction

**Question:** "Your company wants to identify underutilized resources to reduce AWS costs. Which service should you use?"

**Answer:** Trusted Advisor (Cost Optimization pillar)

**Why:** Identifies idle EC2, underutilized EBS, unassociated Elastic IPs

---

### Scenario 2: Security Audit

**Question:** "You need to ensure MFA is enabled on your root account and identify overly permissive security groups. Which service provides this?"

**Answer:** Trusted Advisor (Security pillar)

**Why:** Free tier includes MFA on root + Security Group checks

---

### Scenario 3: Approaching Limits

**Question:** "Your development team is concerned about hitting AWS service limits during a product launch. How can you proactively monitor this?"

**Answer:** Trusted Advisor (Service Limits pillar)

**Why:** Monitors usage vs. limits for key services

---

### Scenario 4: Automated Remediation

**Question:** "You want to automatically receive notifications when Trusted Advisor detects security issues. What should you configure?"

**Answer:** EventBridge rule to capture Trusted Advisor events → SNS topic

**Why:** Business/Enterprise support enables EventBridge integration

---

### Scenario 5: Support Plan Selection

**Question:** "Your company needs programmatic access to Trusted Advisor checks for all five categories. What's the minimum support plan required?"

**Answer:** Business Support

**Why:** API access and full checks require Business or Enterprise

---

### Scenario 6: High Availability

**Question:** "You want to verify that your RDS instances are configured for high availability and that proper backups are in place. Which Trusted Advisor category?"

**Answer:** Fault Tolerance pillar

**Why:** Checks for Multi-AZ, backups, ELB health checks

---

## Trusted Advisor vs. Similar Services

| **Service** | **Focus** | **Key Difference** |
|---|---|---|
| **Trusted Advisor** | Best practices across 5 pillars | Proactive recommendations |
| **AWS Config** | Resource compliance rules | Custom rules, configuration history |
| **Security Hub** | Security posture (aggregates findings) | Consolidates GuardDuty, Inspector, Macie |
| **Cost Explorer** | Cost analysis and forecasting | Historical cost data, not recommendations |
| **Compute Optimizer** | Right-sizing EC2, EBS, Lambda | ML-based recommendations for specific services |
| **Well-Architected Tool** | Workload review against best practices | Manual review process |

---

## Key Exam Tips

1. **Free tier = 7 Service Limits + 6 Security checks** (memorize this)

2. **Business/Enterprise support unlocks:**
   - All checks across all 5 pillars
   - API access
   - EventBridge/CloudWatch integration
   - 5-minute refresh (vs. 24-hour)

3. **Five pillars mnemonic: "C-P-S-F-S"**
   - **C**ost Optimization
   - **P**erformance
   - **S**ecurity
   - **F**ault Tolerance
   - **S**ervice Limits

4. **Automation:** EventBridge + Lambda = automated remediation

5. **Real-time monitoring:** CloudWatch integration for dashboards/alarms

6. **Organizations:** Aggregated view across all accounts (Business/Enterprise)

7. **Service Limits vs. Service Quotas:**
   - Trusted Advisor = monitors limits
   - Service Quotas = request increases

8. **Security checks are critical:** Root MFA, public snapshots, open security groups

9. **Cost savings:** Idle resources, unassociated Elastic IPs, Reserved Instance coverage

10. **Fault Tolerance:** Multi-AZ, backups, health checks, AZ balance

---

## Summary

| **Aspect** | **Details** |
|---|---|
| **What it does** | Automated best practice recommendations |
| **Categories** | Cost, Performance, Security, Fault Tolerance, Service Limits |
| **Free tier** | 7 Service Limits + 6 Security checks |
| **Full access** | Business or Enterprise support |
| **Integration** | EventBridge, CloudWatch, Organizations |
| **Refresh** | 24 hours (free) / 5 minutes (Business+) |
| **API access** | Business/Enterprise only |

**Bottom Line:** Trusted Advisor is your automated AWS consultant. Free tier covers basic security and limits; Business/Enterprise unlocks everything including automation capabilities.
