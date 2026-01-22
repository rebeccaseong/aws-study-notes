# AWS Study Notes - Proposed Reorganization Structure

**Date**: 2026-01-23
**Purpose**: Clean up current structure, fix issues, add missing services, and add cheat sheets/comparisons

---

## 📋 Summary of Changes

### 🔴 Issues to Fix
1. **Move misplaced files** (Aurora, API Gateway)
2. **Delete truncated folder**: `VPC (Virtual Private Cloud) Fundamentals CIDR, Sub/`
3. **Fix typo**: Rename "Comparation" → "Comparison"
4. **Delete duplicate files**: FSx and Storage Gateway duplicates
5. **Move image to parent folder**: `Availability Zone and Region/image.png`

### 🟢 Services to Add (from your outline)
- Auto Scaling Groups (ASG)
- AWS Fargate
- Lambda@Edge
- AWS Batch
- Instance Store
- DAX (DynamoDB Accelerator)
- OpenSearch
- Amazon MSK (Managed Kafka)
- NAT Gateway
- VPC Endpoints
- VPC Peering
- Transit Gateway
- IAM Identity Center (SSO)
- STS (Security Token Service)
- Systems Manager Parameter Store
- AWS Certificate Manager (ACM)
- Inspector
- AWS Firewall Manager
- X-Ray
- Cost Explorer & Compute Optimizer
- AWS Budgets
- Schema Conversion Tool (SCT)
- Snow Family (overview)
- Amazon MQ

### 📊 Cheat Sheets & Comparisons to Add
Each folder gets relevant comparison/cheat sheet files directly inside

---

## 📂 Complete Proposed Structure

```
AWS Study Notes/
├── README.md (NEW - vault overview & study guide)
├── AWS Services Structure.md (existing - keep as index)
├── STUDY-PROGRESS-TRACKER.md (NEW - track learning)
│
└── AWS Services Structure/
    │
    ├── 0 Concept/
    │   ├── Availability Zone and Region.md
    │   └── image.png (MOVE from subfolder)
    │
    ├── 1 Compute/
    │   ├── EC2 (Elastic Compute Cloud) - Instances, AMIs, User Data, Spot vs On-Demand vs Reserved.md
    │   ├── Auto Scaling Groups (ASG) - Scaling policies, Launch Templates.md (NEW)
    │   ├── ECS (Elastic Container Service) - Docker management.md
    │   ├── EKS (Elastic Kubernetes Service) - Kubernetes management.md
    │   ├── AWS Fargate - Serverless compute for containers.md (NEW)
    │   ├── Lambda - Event-driven code, limits (15 min), Cold starts.md
    │   ├── Lambda@Edge - Run Lambda at CloudFront edge locations.md (NEW)
    │   ├── Elastic Beanstalk - PaaS (Platform as a Service), easy deployment.md
    │   ├── AWS Batch - Batch computing (HPC) using Spot instances.md (NEW)
    │   ├── Comparison - EC2 vs Lambda vs Fargate vs ECS vs EKS.md (NEW)
    │   └── Cheat Sheet - EC2 Instance Types (Families & Use Cases).md (NEW)
    │
    ├── 2 Storage & File Systems/
    │   ├── S3 (Simple Storage Service) - Standard, Intelligent-Tiering, Glacier, Lifecycle, Versioning.md
    │   ├── S3 Transfer Acceleration (S3TA).md
    │   ├── EBS (Elastic Block Store) - Persistent, Network-attached (gp3, io2).md
    │   ├── Instance Store - Ephemeral, High Random IO, locally attached.md (NEW)
    │   ├── EFS (Elastic File System) - Shared Linux (NFS), cross-AZ.md
    │   ├── FSx (File System for X) - Windows (SMB), Lustre (HPC), NetApp ONTAP.md
    │   ├── FSx for Lustre.md (DELETE - duplicate)
    │   ├── FSx for Windows File Server.md (DELETE - duplicate)
    │   ├── Storage Gateway - File (S3), Volume (iSCSI), Tape (VTL).md
    │   ├── Storage Gateway.md (DELETE - duplicate)
    │   ├── Cheat Sheet - Storage (When to use S3 vs EBS vs EFS vs FSx vs Instance Store).md (EXISTING)
    │   └── Comparison - EBS Volume Types (gp2, gp3, io1, io2, st1, sc1).md (NEW)
    │
    ├── 3 Databases, Caching & Analytics/
    │   ├── RDS (Relational Database Service) - Managed Postgres MySQL etc., Multi-AZ vs Read Replicas.md
    │   ├── Aurora - Cloud native, high performance, Serverless v2, Global Database.md (MOVE from Storage folder)
    │   ├── DynamoDB - Key-value, single-digit millisecond latency, DAX, Streams, Global Tables.md
    │   ├── DAX - DynamoDB Accelerator (caching for DynamoDB).md (NEW)
    │   ├── ElastiCache - Redis (complex data structures) vs Memcached (simple).md
    │   ├── Redshift - Data Warehouse, Columnar storage (OLAP).md
    │   ├── EMR (Elastic MapReduce) - Hadoop Spark clusters, high maintenance.md
    │   ├── Glue - Serverless ETL, Data Catalog.md
    │   ├── Athena - Serverless SQL on S3 files.md
    │   ├── OpenSearch - Log analytics, search, dashboards (formerly Elasticsearch).md (NEW)
    │   ├── Kinesis Data Streams - Real-time ingestion, requires shard management (The Pipe).md
    │   ├── Kinesis Data Firehose - Near real-time delivery to S3 Redshift, zero admin (The Delivery Truck).md
    │   ├── Kinesis Data Analytics - SQL Flink analysis inside the stream.md
    │   ├── Amazon MSK - Managed Kafka (alternative to Kinesis Data Streams).md (NEW)
    │   ├── Cheat Sheet - Kinesis - Which one to pick.md (EXISTING)
    │   ├── Comparison - RDS vs Aurora vs DynamoDB.md (NEW)
    │   ├── Comparison - ElastiCache Redis vs Memcached vs DAX.md (NEW)
    │   └── Comparison - Athena vs Redshift vs EMR vs OpenSearch.md (NEW)
    │
    ├── 4 Networking & Content Delivery/
    │   ├── VPC (Virtual Private Cloud) - Fundamentals CIDR, Subnets, Route Tables, IGW.md
    │   ├── VPC (Virtual Private Cloud) Fundamentals CIDR, Sub/ (DELETE - truncated folder)
    │   ├── NAT Gateway - Allows private subnets to talk to the internet.md (NEW)
    │   ├── VPC Endpoints - PrivateLink (Interface) vs Gateway (S3 DynamoDB).md (NEW)
    │   ├── VPC Peering - Connect two VPCs.md (NEW)
    │   ├── Transit Gateway - Hub-and-spoke topology for many VPCs.md (NEW)
    │   ├── VPN vs Direct Connect - Public internet encrypted tunnel vs Private physical fiber.md
    │   ├── Route 53 - DNS records (A, CNAME, Alias), Routing Policies (Failover, Latency, Geolocation).md
    │   ├── CloudFront - CDN (Content Delivery Network), caching at Edge.md
    │   ├── Global Accelerator - Static IP, improves performance via AWS backbone (not caching).md
    │   ├── Elastic Load Balancing (ELB) - ALB (Layer 7 HTTP), NLB (Layer 4 TCP UDP), GWLB.md
    │   ├── Comparison - Security Groups vs NACLs.md (EXISTING)
    │   ├── Comparison - Global Accelerator vs CloudFront.md (EXISTING - fix typo "Comparation")
    │   ├── Comparison - ALB vs NLB vs GWLB.md (NEW)
    │   ├── Comparison - VPC Peering vs Transit Gateway vs PrivateLink.md (NEW)
    │   └── Cheat Sheet - Route 53 Routing Policies.md (NEW)
    │
    ├── 5 Security, Identity & Compliance/
    │   ├── IAM (Identity and Access Management) - Users, Roles, Policies, MFA.md
    │   ├── Cognito - Identity for Mobile Web Apps (User Pools = Auth, Identity Pools = AWS Access).md
    │   ├── IAM Identity Center (SSO) - Centralized login for organizations.md (NEW)
    │   ├── STS (Security Token Service) - Temporary credentials, AssumeRole.md (NEW)
    │   ├── KMS (Key Management Service) - Encryption keys (managed).md
    │   ├── CloudHSM (Hardware Security Module) - Dedicated hardware (compliance).md
    │   ├── Secrets Manager - Rotate DB credentials automatically.md
    │   ├── Systems Manager Parameter Store - Store config secrets (free tier).md (NEW)
    │   ├── AWS Certificate Manager (ACM) - Free SSL TLS certs for ALB CloudFront API Gateway.md (NEW)
    │   ├── Shield - DDoS protection (Standard vs Advanced).md
    │   ├── WAF (Web Application Firewall) - Block SQL injection XSS.md
    │   ├── GuardDuty - Intelligent threat detection (logs analysis).md
    │   ├── Macie - PII Sensitive data discovery in S3.md
    │   ├── Inspector - EC2 vulnerability scanning.md (NEW)
    │   ├── AWS Firewall Manager - Centrally manage WAF Shield across accounts.md (NEW)
    │   ├── Systems Manager (SSM) - Patch Manager, Session Manager (No SSH needed).md
    │   ├── API Gateway - REST HTTP APIs, Throttling, API Keys.md (MOVE from this folder to folder 8)
    │   ├── Comparison - KMS vs CloudHSM vs Secrets Manager vs Parameter Store.md (NEW)
    │   └── Cheat Sheet - IAM Best Practices.md (NEW)
    │
    ├── 6 Monitoring, Management & Governance/
    │   ├── CloudWatch - Metrics, Alarms, Logs (Performance).md
    │   ├── X-Ray - Tracing and debugging distributed apps Lambda.md (NEW)
    │   ├── CloudTrail - Who did what (API Auditing).md
    │   ├── Config - What does my infrastructure look like (Compliance Rules).md
    │   ├── Trusted Advisor - Best practice checklist.md
    │   ├── CloudFormation - Infrastructure as Code (JSON YAML).md
    │   ├── Systems Manager (SSM) - Patch Manager, Session Manager (No SSH needed).md (MOVE to Security folder OR keep here - it's in both categories)
    │   ├── Organizations & Control Tower - SCPs (Service Control Policies), Consolidated Billing.md
    │   ├── Cost Explorer & Compute Optimizer - Saving money.md (NEW)
    │   ├── AWS Budgets - Set alerts when costs exceed thresholds.md (NEW)
    │   └── Comparison - CloudWatch vs CloudTrail vs Config vs X-Ray.md (NEW)
    │
    ├── 7 Migration & Transfer/
    │   ├── DMS (Database Migration Service) - Move DBs while keeping source live.md
    │   ├── MGN (Application Migration Service) - Lift-and-shift servers.md
    │   ├── Schema Conversion Tool (SCT) - Convert Oracle to Aurora.md (NEW)
    │   ├── Snow Family - Physical devices (Snowcone, Snowball Edge, Snowmobile) for massive data.md (NEW)
    │   ├── Snowball.md (MERGE into Snow Family overview)
    │   ├── DataSync - Automated data transfer (NAS to S3).md
    │   ├── Transfer Family - FTP SFTP to S3.md
    │   └── Comparison - DataSync vs Transfer Family vs Snow Family vs Storage Gateway.md (NEW)
    │
    ├── 8 Application Integration/
    │   ├── SQS (Simple Queue Service) - Decoupling, Queueing (Standard vs FIFO).md
    │   ├── SNS (Simple Notification Service) - Pub Sub, Notifications to Email SMS Lambda.md
    │   ├── Amazon MQ - Broker for industry standards (MQTT, AMQP) lift and shift legacy apps.md (NEW)
    │   ├── EventBridge - Serverless Event Bus, Rules, Scheduler.md
    │   ├── Step Functions - Visual workflow orchestration (State Machine).md
    │   ├── SWF (Simple Workflow Service).md
    │   ├── API Gateway - REST HTTP APIs, Throttling, API Keys.md (MOVE from folder 5)
    │   ├── Comparison - SQS vs SNS vs EventBridge vs Amazon MQ.md (NEW)
    │   └── Comparison - Step Functions vs SWF.md (NEW)
    │
    └── 9 Developer Tools/
        ├── CodeCommit - Git repo.md
        ├── CodeBuild - Build and test.md
        ├── CodeDeploy - Deploy to EC2, Lambda.md
        ├── CodePipeline - Orchestrate the flow.md
        └── Cheat Sheet - CI CD Pipeline (Commit → Build → Deploy → Pipeline).md (NEW)
```

---

## 📊 New Cheat Sheets & Comparisons Summary

### By Category:

**1 Compute** (2 new):
- Comparison - EC2 vs Lambda vs Fargate vs ECS vs EKS
- Cheat Sheet - EC2 Instance Types (Families & Use Cases)

**2 Storage** (1 new):
- Comparison - EBS Volume Types (gp2, gp3, io1, io2, st1, sc1)

**3 Databases** (3 new):
- Comparison - RDS vs Aurora vs DynamoDB
- Comparison - ElastiCache Redis vs Memcached vs DAX
- Comparison - Athena vs Redshift vs EMR vs OpenSearch

**4 Networking** (3 new):
- Comparison - ALB vs NLB vs GWLB
- Comparison - VPC Peering vs Transit Gateway vs PrivateLink
- Cheat Sheet - Route 53 Routing Policies

**5 Security** (2 new):
- Comparison - KMS vs CloudHSM vs Secrets Manager vs Parameter Store
- Cheat Sheet - IAM Best Practices

**6 Monitoring** (1 new):
- Comparison - CloudWatch vs CloudTrail vs Config vs X-Ray

**7 Migration** (1 new):
- Comparison - DataSync vs Transfer Family vs Snow Family vs Storage Gateway

**8 Application Integration** (2 new):
- Comparison - SQS vs SNS vs EventBridge vs Amazon MQ
- Comparison - Step Functions vs SWF

**9 Developer Tools** (1 new):
- Cheat Sheet - CI CD Pipeline

**Total: 16 new cheat sheets/comparisons**

---

## 🔧 Action Items Summary

### Phase 1: Clean Up (Fix Issues)
1. Move `Aurora.md` from folder 2 → folder 3
2. Move `API Gateway.md` from folder 5 → folder 8
3. Move `Availability Zone and Region/image.png` up one level
4. Delete truncated folder: `VPC (Virtual Private Cloud) Fundamentals CIDR, Sub/`
5. Rename: `Comparation - Global Accelerator vs CloudFront.md` → `Comparison - Global Accelerator vs CloudFront.md`
6. Delete duplicate: `FSx for Lustre.md` and `FSx for Windows File Server.md`
7. Delete duplicate: `Storage Gateway.md` (keep the one with full title)

### Phase 2: Add Missing Services (24 new files)
Create service notes for all services listed in your outline that don't have files yet

### Phase 3: Add Cheat Sheets & Comparisons (16 new files)
Create comparison and cheat sheet files directly in relevant category folders

### Phase 4: Create Meta Files (3 new files)
- `README.md` - Vault overview and study guide
- `STUDY-PROGRESS-TRACKER.md` - Track learning progress
- Update `AWS Services Structure.md` with new links

---

## ❓ Questions for You

1. **Systems Manager (SSM)** appears in both "5 Security" and "6 Monitoring/Management" - which category should it stay in? Or keep in both?
2. **Snowball** - Should I merge it into a comprehensive "Snow Family" overview, or keep separate files for each device?
3. Do you want me to proceed with all phases, or just specific ones?

---

**Next Steps**: Review this structure and let me know if you approve. I'll then execute the reorganization systematically.
