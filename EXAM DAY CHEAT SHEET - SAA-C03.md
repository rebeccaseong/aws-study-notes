# EXAM DAY CHEAT SHEET - AWS Solutions Architect Associate (SAA-C03)

**Purpose:** Quick reference for most-tested topics and common exam patterns.

**Exam Format:**
- 65 questions (multiple choice + multiple response)
- 130 minutes (2 hours)
- Passing score: 720/1000 (approx. 72%)
- Scenario-based questions (focus on "best" solution)

---

## Exam Domain Breakdown

| Domain | Weight | Questions (~65 total) |
|--------|--------|---------------------|
| **Design Resilient Architectures** | 30% | ~20 questions |
| **Design High-Performing Architectures** | 28% | ~18 questions |
| **Design Secure Applications** | 24% | ~16 questions |
| **Design Cost-Optimized Architectures** | 18% | ~12 questions |

---

## Critical Trigger Phrases → Solutions

### Storage & Databases

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Shared file system for Linux" | **EFS** (NFS) |
| "Shared file system for Windows" | **FSx for Windows File Server** (SMB) |
| "High Performance Computing (HPC), millions of IOPS" | **FSx for Lustre** |
| "Object storage, static website hosting" | **S3** (Standard, Intelligent-Tiering) |
| "Archive, long-term retention, retrieve within hours" | **S3 Glacier** (Flexible/Deep Archive) |
| "Block storage for EC2" | **EBS** (gp3, io2) |
| "Temporary high-performance storage for EC2" | **Instance Store** |
| "On-premises backup to cloud" | **Storage Gateway** (File, Volume, Tape) |
| "Relational database, managed" | **RDS** (MySQL, PostgreSQL, Aurora) |
| "NoSQL, single-digit millisecond latency" | **DynamoDB** |
| "In-memory cache, Redis/Memcached" | **ElastiCache** |
| "Data warehouse, analytics, petabyte-scale" | **Redshift** |
| "Lambda + RDS connection errors" | **RDS Proxy** |

---

### Compute

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Event-driven, serverless" | **Lambda** |
| "Long-running workloads, full OS control" | **EC2** |
| "Containers, orchestration" | **ECS** (Fargate for serverless, EC2 for control) |
| "Kubernetes" | **EKS** |
| "Batch processing, automatically scales" | **AWS Batch** |
| "Spot Instances for fault-tolerant workloads" | **EC2 Spot** (up to 90% savings) |
| "Predictable workloads, cost savings" | **Reserved Instances** or **Savings Plans** |
| "Gradually shift Lambda traffic" | **Lambda Aliases** (weighted routing) |
| "Shared Lambda dependencies" | **Lambda Layers** |

---

### Networking

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Distribute traffic across EC2 instances" | **ELB** (ALB for HTTP/HTTPS, NLB for TCP/UDP) |
| "Global traffic distribution, low latency" | **CloudFront** + Route 53 (latency routing) |
| "Route based on geographic location" | **Route 53** (geolocation routing) |
| "Failover routing for DR" | **Route 53** (failover routing) + health checks |
| "Weighted traffic distribution" | **Route 53** (weighted routing) or ALB |
| "Securely connect on-premises to AWS" | **VPN** (Site-to-Site VPN) or **Direct Connect** |
| "Private connectivity between VPCs" | **VPC Peering** or **Transit Gateway** |
| "Public subnet needs internet access" | **Internet Gateway** |
| "Private subnet needs internet access" | **NAT Gateway** (in public subnet) |
| "Static public IP for EC2" | **Elastic IP** |
| "DDoS protection" | **AWS Shield** (Standard = free, Advanced = paid) |
| "Web application firewall" | **AWS WAF** (with ALB/CloudFront) |

---

### Security & Identity

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Encrypt data at rest" | **KMS** (AWS-managed or customer-managed keys) |
| "Store database passwords securely" | **Secrets Manager** (automatic rotation) |
| "Store application config" | **SSM Parameter Store** (free tier) |
| "MFA for root account" | **Enable MFA** (hardware or virtual) |
| "Grant temporary AWS credentials" | **STS** (AssumeRole) |
| "Cross-account access" | **IAM Role** + AssumeRole |
| "Enforce MFA for API calls" | **IAM Policy** with Condition (aws:MultiFactorAuthPresent) |
| "Least privilege access" | **IAM Policies** (explicit Allow only) |
| "Prevent S3 bucket from being public" | **S3 Block Public Access** |
| "Detect security threats, GuardDuty" | **GuardDuty** (threat detection) |
| "Compliance scanning, vulnerabilities" | **Inspector** (EC2/container scanning) |
| "Centralized security findings" | **Security Hub** (aggregates GuardDuty, Inspector, Macie) |
| "User authentication for mobile apps" | **Cognito User Pools** |
| "Grant mobile apps access to AWS services" | **Cognito Identity Pools** |

---

### Monitoring & Management

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Monitor EC2 CPU, memory" | **CloudWatch** (Agent for memory) |
| "Trigger action when CPU >80%" | **CloudWatch Alarms** + SNS/Auto Scaling |
| "Centralize application logs" | **CloudWatch Logs** |
| "Query logs with SQL-like syntax" | **CloudWatch Logs Insights** |
| "Event-driven automation" | **EventBridge** (rules + targets) |
| "Track API calls, who did what" | **CloudTrail** |
| "Check resource compliance" | **AWS Config** (Config Rules + Remediation) |
| "Automate compliance across accounts" | **Config Conformance Packs** |
| "Identify cost savings" | **Trusted Advisor** (Cost Optimization) |
| "Prevent accounts from leaving organization" | **SCP** denying LeaveOrganization |
| "Centralize billing for multiple accounts" | **AWS Organizations** (Consolidated Billing) |
| "Set up secure multi-account environment" | **AWS Control Tower** (Landing Zone + Guardrails) |

---

### High Availability & Disaster Recovery

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Multi-AZ, automatic failover" | **RDS Multi-AZ**, **ELB**, **Aurora** |
| "Read scaling for database" | **RDS Read Replicas** |
| "Lowest RTO/RPO" | **Multi-Site Active-Active** |
| "Cost-effective DR, hours recovery" | **Backup and Restore** (S3, snapshots) |
| "DR with minimal infrastructure" | **Pilot Light** |
| "DR with scaled-down infrastructure" | **Warm Standby** |
| "Auto-scale EC2 based on demand" | **Auto Scaling Groups** |
| "Distribute across multiple AZs" | **ELB** + ASG (multi-AZ deployment) |
| "Cross-region disaster recovery" | **S3 Cross-Region Replication**, Aurora Global Database |

---

### Cost Optimization

| **Trigger Phrase** | **Solution** |
|-------------------|--------------|
| "Reduce S3 costs for infrequent access" | **S3 Intelligent-Tiering** or **S3 Standard-IA** |
| "Archive data, rarely accessed" | **S3 Glacier** (Flexible or Deep Archive) |
| "Automatically move objects to cheaper tiers" | **S3 Lifecycle Policies** |
| "Reduce EC2 costs for predictable workloads" | **Reserved Instances** or **Savings Plans** |
| "Reduce costs for fault-tolerant workloads" | **Spot Instances** |
| "Identify idle resources" | **Trusted Advisor** (Cost Optimization) |
| "Analyze and forecast costs" | **Cost Explorer** |
| "Set spending limits" | **AWS Budgets** (with CloudWatch alarms) |
| "Shared costs across accounts" | **Consolidated Billing** (volume discounts, RI sharing) |
| "Right-size EC2 instances" | **Compute Optimizer** |

---

## Most-Tested Service Comparisons

### S3 Storage Classes

| **Class** | **Use Case** | **Cost** | **Retrieval** |
|-----------|--------------|----------|---------------|
| **Standard** | Frequently accessed data | $$$ | Instant |
| **Intelligent-Tiering** | Unknown/changing access patterns | $$ | Instant |
| **Standard-IA** | Infrequent access, rapid retrieval | $$ | Instant |
| **One Zone-IA** | Infrequent, non-critical data | $ | Instant |
| **Glacier Flexible** | Archive, minutes-hours retrieval | $ | Minutes-hours |
| **Glacier Deep Archive** | Long-term archive, rarely accessed | ¢ | 12-48 hours |

**Exam Tip:** Intelligent-Tiering = Unknown access patterns, no retrieval fees

---

### RDS vs DynamoDB vs Aurora

| **Feature** | **RDS** | **DynamoDB** | **Aurora** |
|-------------|---------|--------------|------------|
| **Type** | Relational (SQL) | NoSQL (key-value) | Relational (MySQL/PostgreSQL compatible) |
| **Scaling** | Vertical (instance size) | Horizontal (auto-scale) | Auto-scale storage, read replicas |
| **Performance** | Good | Single-digit millisecond | 5x MySQL, 3x PostgreSQL |
| **Multi-AZ** | Manual enable | Global Tables | Built-in, auto-failover |
| **Use Case** | Traditional apps, complex queries | High throughput, low latency | Enterprise apps, high availability |
| **Cost** | $$ | $ (On-Demand) or $$ (Provisioned) | $$$ |

**Exam Tip:** DynamoDB = NoSQL + millisecond latency, Aurora = MySQL/PostgreSQL + high performance

---

### ELB Types

| **Type** | **Layer** | **Use Case** | **Protocols** |
|----------|-----------|--------------|---------------|
| **ALB** | Layer 7 (Application) | HTTP/HTTPS, path/host routing | HTTP, HTTPS, gRPC |
| **NLB** | Layer 4 (Transport) | TCP/UDP, extreme performance | TCP, UDP, TLS |
| **GLB** | Layer 3 (Network) | Third-party virtual appliances | IP |
| **CLB** | Layer 4/7 (Legacy) | Legacy applications | HTTP, HTTPS, TCP, SSL |

**Exam Tip:** ALB = HTTP/HTTPS routing, NLB = TCP/UDP + static IP, CLB = legacy (don't choose)

---

### Lambda vs EC2 vs Fargate

| **Feature** | **Lambda** | **EC2** | **Fargate** |
|-------------|------------|---------|-------------|
| **Management** | Fully managed (serverless) | You manage OS, patching | Managed containers (serverless) |
| **Timeout** | 15 minutes max | No limit | No limit |
| **Scaling** | Automatic (per request) | Manual (ASG) | Automatic (ECS/EKS) |
| **Pricing** | Per request + duration | Per hour (instance) | Per vCPU/memory |
| **Use Case** | Event-driven, short tasks | Long-running, full control | Containerized apps, no server management |

**Exam Tip:** Lambda = <15 min event-driven, Fargate = containers without servers, EC2 = full control

---

### VPN vs Direct Connect

| **Feature** | **Site-to-Site VPN** | **Direct Connect** |
|-------------|---------------------|-------------------|
| **Connection** | Over internet (encrypted) | Dedicated private connection |
| **Latency** | Variable (internet-dependent) | Low, predictable |
| **Bandwidth** | Up to 1.25 Gbps per tunnel | 1 Gbps - 100 Gbps |
| **Setup Time** | Minutes | Weeks (physical installation) |
| **Cost** | $ | $$$ |
| **Use Case** | Quick setup, encrypted connection | High bandwidth, low latency, consistent performance |

**Exam Tip:** VPN = quick, encrypted, Direct Connect = high bandwidth, low latency

---

## Common Exam Traps & Patterns

### Trap 1: "Most cost-effective" ≠ "Cheapest"

**Pattern:** Question asks for "most cost-effective" solution
- **Wrong:** Always choosing Spot Instances or S3 Glacier
- **Right:** Balance cost with requirements (availability, performance, retrieval time)

**Example:**
- Question: "Most cost-effective storage for frequently accessed data?"
- Wrong: S3 Glacier (cheapest but wrong use case)
- Right: S3 Standard or Intelligent-Tiering (appropriate for frequent access)

---

### Trap 2: "Highly available" requires Multi-AZ

**Pattern:** Question emphasizes "high availability" or "no downtime"
- **Wrong:** Single-AZ deployment
- **Right:** Multi-AZ (RDS, ELB, ASG across multiple AZs)

**Example:**
- Question: "Highly available database with automatic failover?"
- Wrong: RDS single-AZ + Read Replica
- Right: RDS Multi-AZ (automatic failover)

---

### Trap 3: Security = Least Privilege

**Pattern:** Security questions always prefer least privilege
- **Wrong:** Granting broad permissions (e.g., `s3:*`)
- **Right:** Specific permissions for specific resources

**Example:**
- Question: "Grant Lambda access to S3 bucket?"
- Wrong: Attach `AmazonS3FullAccess` managed policy
- Right: Custom policy with `s3:GetObject` on specific bucket ARN

---

### Trap 4: "Serverless" is often preferred

**Pattern:** When multiple solutions work, serverless is usually preferred
- **Wrong:** EC2 + cron job for scheduled task
- **Right:** Lambda + EventBridge Scheduler

**Example:**
- Question: "Run script every night to process files?"
- Wrong: EC2 with cron job
- Right: Lambda triggered by EventBridge rule

---

### Trap 5: Read the ENTIRE question

**Pattern:** Key details often in middle/end of question
- **Trap:** Answering based on first sentence
- **Right:** Read full question, identify all requirements

**Example:**
- Question starts: "You need to store data..."
- Middle: "...data must be retained for 7 years for compliance..."
- End: "...and retrieved within 12 hours when needed"
- **Right answer:** S3 Glacier (not S3 Standard)

---

## Quick Decision Trees

### Storage Selection

```
Need to store data?
├─ Structured (SQL)?
│  ├─ Managed database? → RDS (MySQL, PostgreSQL)
│  └─ High performance? → Aurora
│
├─ NoSQL (key-value)?
│  └─ DynamoDB
│
├─ File storage?
│  ├─ Linux (NFS)? → EFS
│  ├─ Windows (SMB)? → FSx for Windows
│  └─ HPC (millions IOPS)? → FSx for Lustre
│
└─ Object storage?
   ├─ Frequently accessed? → S3 Standard
   ├─ Unknown access? → S3 Intelligent-Tiering
   ├─ Infrequent access? → S3 Standard-IA
   └─ Archive? → S3 Glacier
```

---

### High Availability Strategy

```
Need high availability?
├─ Database?
│  ├─ Relational? → RDS Multi-AZ
│  └─ NoSQL? → DynamoDB (Global Tables for cross-region)
│
├─ Compute?
│  ├─ Stateless app? → ALB + ASG (multi-AZ)
│  ├─ Stateful app? → Multi-AZ with sticky sessions
│  └─ Serverless? → Lambda (inherently multi-AZ)
│
└─ Storage?
   ├─ Block? → EBS + snapshots (cross-AZ restore)
   ├─ File? → EFS (multi-AZ by default)
   └─ Object? → S3 (99.99% availability, CRR for cross-region)
```

---

### Security Best Practices

```
Securing data/access?
├─ Encrypt data at rest?
│  └─ KMS (customer-managed keys for compliance)
│
├─ Encrypt data in transit?
│  └─ SSL/TLS (enforce with ALB, CloudFront)
│
├─ Store secrets?
│  ├─ Auto-rotation needed? → Secrets Manager
│  └─ Static config? → SSM Parameter Store
│
├─ Access control?
│  ├─ AWS services? → IAM Roles
│  ├─ Users? → IAM Users + MFA
│  └─ Cross-account? → IAM Role + AssumeRole
│
└─ Prevent public access?
   ├─ S3? → S3 Block Public Access
   ├─ EC2? → Security Groups (no 0.0.0.0/0 inbound)
   └─ RDS? → Private subnet, security group rules
```

---

## Critical Numbers to Memorize

### S3
- Standard: 99.99% availability, 11 9's durability
- Lifecycle: Minimum 30 days in Standard before transition
- Glacier retrieval: Expedited (1-5 min), Standard (3-5 hrs), Bulk (5-12 hrs)
- Object size: 0 bytes - 5 TB

### EC2
- Hibernate: Max 60 days, <150 GB RAM
- Instance Store: Ephemeral, data lost on stop/terminate
- Placement Groups: Cluster (same AZ), Spread (max 7 per AZ), Partition (max 7 per AZ)

### Lambda
- Timeout: 15 minutes max
- Memory: 128 MB - 10 GB
- Deployment package: 50 MB (zipped), 250 MB (unzipped)
- Layers: Max 5 per function, 250 MB total
- Concurrency: 1000 default (can request increase)

### RDS
- Read Replicas: Up to 5
- Multi-AZ: Automatic failover (~1-2 minutes)
- Backup retention: 1-35 days (automated)
- Max storage: 64 TB (Aurora), 16 TB (RDS)

### DynamoDB
- Partition key: Max 2048 bytes
- Sort key: Max 1024 bytes
- Item size: Max 400 KB
- RCU: 1 strongly consistent read/sec (4 KB)
- WCU: 1 write/sec (1 KB)

### VPC
- CIDR block: /16 (65,536 IPs) to /28 (16 IPs)
- Default VPC: 172.31.0.0/16
- Reserved IPs per subnet: 5 (network, router, DNS, future, broadcast)

### ELB
- ALB: HTTP/HTTPS, path/host routing, WebSocket
- NLB: TCP/UDP, millions req/sec, static IP
- Health checks: Interval 5-300 sec, timeout 2-120 sec

---

## Common IAM Policy Patterns

### S3 Bucket Access (Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

---

### Lambda Execution Role (Basic)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "logs:CreateLogGroup",
      "logs:CreateLogStream",
      "logs:PutLogEvents"
    ],
    "Resource": "arn:aws:logs:*:*:*"
  }]
}
```

---

### Cross-Account Access (AssumeRole)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

---

### Enforce MFA for Sensitive Actions

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": ["ec2:StopInstances", "ec2:TerminateInstances"],
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
    }
  }]
}
```

---

## Well-Architected Framework Pillars

### 1. Operational Excellence
- **Key Concepts:** Automation, monitoring, continuous improvement
- **Services:** CloudFormation, Systems Manager, CloudWatch, X-Ray
- **Exam Focus:** Infrastructure as Code, runbooks, automated remediation

### 2. Security
- **Key Concepts:** Least privilege, defense in depth, encryption
- **Services:** IAM, KMS, Secrets Manager, GuardDuty, WAF
- **Exam Focus:** Encryption at rest/transit, MFA, security groups, NACLs

### 3. Reliability
- **Key Concepts:** Multi-AZ, auto-scaling, automatic recovery
- **Services:** RDS Multi-AZ, Auto Scaling, Route 53, ELB
- **Exam Focus:** Fault tolerance, disaster recovery (RTO/RPO)

### 4. Performance Efficiency
- **Key Concepts:** Right-sizing, caching, global distribution
- **Services:** CloudFront, ElastiCache, RDS Read Replicas, Lambda
- **Exam Focus:** Low latency, high throughput, edge locations

### 5. Cost Optimization
- **Key Concepts:** Right-sizing, Reserved Instances, S3 lifecycle
- **Services:** Cost Explorer, Trusted Advisor, S3 Intelligent-Tiering
- **Exam Focus:** Spot Instances, Savings Plans, lifecycle policies

### 6. Sustainability
- **Key Concepts:** Minimize environmental impact, efficient resource use
- **Services:** Auto Scaling, Graviton instances, serverless
- **Exam Focus:** Less emphasized on SAA-C03 (more on SAP-C02)

---

## Final Exam Day Tips

### Before the Exam
1. **Review trigger phrases** (this cheat sheet)
2. **Get good sleep** (6-8 hours)
3. **Arrive early** (15 minutes before)
4. **Bring valid ID** (government-issued)

### During the Exam
1. **Read ENTIRE question** before answering
2. **Flag uncertain questions** for review (use flag feature)
3. **Eliminate obviously wrong answers** first
4. **Look for "MOST" or "BEST"** in question (usually means multiple work, pick optimal)
5. **Don't overthink** - first instinct often correct
6. **Watch time** (~2 minutes per question, 130 min total)

### Common Question Patterns
- "Most cost-effective" = Balance cost + requirements
- "Least operational overhead" = Managed services, serverless
- "Highest availability" = Multi-AZ, multiple regions
- "Lowest latency" = CloudFront, ElastiCache, edge locations
- "Secure" = Encryption, least privilege, MFA

### Guessing Strategy
- **If unsure:** Choose serverless/managed service
- **If cost question:** Choose least expensive that meets requirements
- **If security question:** Choose most restrictive (least privilege)
- **If HA question:** Choose Multi-AZ, multi-region option

---

## Exam Question Examples

### Example 1: Storage Selection

**Question:** "A company needs to store 100 TB of data that is accessed once every 6 months for compliance audits. Retrieval must occur within 12 hours. What is the most cost-effective solution?"

**Analysis:**
- Infrequent access (6 months)
- Can tolerate 12-hour retrieval
- Cost-effective = cheapest that meets requirements

**Answer:** **S3 Glacier Flexible Retrieval** (Standard retrieval: 3-5 hours)

**Why wrong answers fail:**
- S3 Standard: Too expensive for infrequent access
- S3 Glacier Deep Archive: 12-48 hours (might not meet 12-hour requirement)
- S3 Standard-IA: More expensive than Glacier for rare access

---

### Example 2: High Availability Database

**Question:** "A company requires a MySQL database with automatic failover and minimal downtime. Which solution provides this?"

**Analysis:**
- MySQL = RDS
- Automatic failover = Multi-AZ
- Minimal downtime = Built-in HA

**Answer:** **RDS MySQL Multi-AZ**

**Why wrong answers fail:**
- RDS MySQL Single-AZ + Read Replica: Manual failover
- EC2 with MySQL: No automatic failover
- Aurora MySQL: Correct but more expensive (RDS Multi-AZ sufficient)

---

### Example 3: Lambda + RDS Connection Errors

**Question:** "Lambda functions are experiencing 'too many connections' errors when connecting to RDS during traffic spikes. What is the solution?"

**Analysis:**
- Lambda + RDS = Connection pooling issue
- Traffic spikes = Need connection management

**Answer:** **RDS Proxy**

**Why wrong answers fail:**
- Increase RDS max connections: Doesn't solve Lambda scaling issue
- Use Aurora Serverless: Solves it but unnecessary migration
- Add Read Replicas: Doesn't solve connection pooling

---

## Summary Checklist

**Master these for 90%+ score:**

✅ **Storage:** S3 classes, EFS vs FSx, EBS types, Instance Store
✅ **Databases:** RDS vs DynamoDB vs Aurora, Multi-AZ vs Read Replicas
✅ **Compute:** Lambda vs EC2 vs Fargate, Spot vs Reserved vs On-Demand
✅ **Networking:** VPC, Subnets, Route Tables, Security Groups vs NACLs
✅ **Load Balancing:** ALB vs NLB, Target Groups, Health Checks
✅ **Security:** IAM (Roles, Policies, AssumeRole), KMS, Secrets Manager
✅ **Monitoring:** CloudWatch (Metrics, Alarms, Logs), CloudTrail, Config
✅ **DR Strategies:** Backup/Restore, Pilot Light, Warm Standby, Multi-Site
✅ **High Availability:** Multi-AZ, Auto Scaling, Route 53 routing policies
✅ **Cost Optimization:** RI/Savings Plans, Spot, S3 Lifecycle, Trusted Advisor

---

**Good luck on your exam! 🚀**

**Remember:**
- Read the ENTIRE question
- Eliminate wrong answers first
- Choose "MOST" cost-effective/available/secure option
- Don't overthink - trust your preparation
