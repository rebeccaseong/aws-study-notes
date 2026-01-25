# AWS Study Notes - Remaining Tasks (PROPOSED-STRUCTURE)

**Date**: 2026-01-25 (Updated)
**Status**: Content expansion completed. Structural reorganization and new service files remain.

---

## ✅ COMPLETED (Content Expansion - Week 2)

The following content expansions are **DONE**:
- ✅ AWS Config expanded (Rules, Remediation, Aggregators, Conformance Packs)
- ✅ Organizations & Control Tower expanded (OUs, SCPs, Guardrails)
- ✅ Trusted Advisor expanded (5 categories, support tiers)
- ✅ RDS Proxy section added
- ✅ Lambda Layers/Versions/Aliases added
- ✅ EC2 Hibernate fixes (instance families updated)
- ✅ S3 Object Lock fixes (existing buckets support)
- ✅ Exam Day Cheat Sheet created (comprehensive)

---

## 🔴 REMAINING: Structural Reorganization

### Phase 1: File Organization & Cleanup

**Move Files:**
1. Move `Aurora.md` from "2 Storage" → "3 Databases, Caching & Analytics"
2. Move `API Gateway.md` from "5 Security" → "8 Application Integration"
3. Move `Availability Zone and Region/image.png` up one level (out of subfolder)

**Delete Duplicates:**
4. Delete `FSx for Lustre.md` (duplicate - keep consolidated FSx file)
5. Delete `FSx for Windows File Server.md` (duplicate - keep consolidated FSx file)
6. Delete `Storage Gateway.md` (keep the one with full title)
7. Delete truncated folder: `VPC (Virtual Private Cloud) Fundamentals CIDR, Sub/`

**Fix Typo:**
8. Rename: `Comparation - Global Accelerator vs CloudFront.md` → `Comparison - Global Accelerator vs CloudFront.md`

**Merge Files:**
9. Merge `Snowball.md` into comprehensive `Snow Family.md` overview

---

## 🟢 REMAINING: New Service Notes (24 files)

### 1 Compute (4 new):
- [ ] Auto Scaling Groups (ASG) - Scaling policies, Launch Templates.md
- [ ] AWS Fargate - Serverless compute for containers.md
- [ ] Lambda@Edge - Run Lambda at CloudFront edge locations.md *(file exists but may need expansion)*
- [ ] AWS Batch - Batch computing (HPC) using Spot instances.md

### 2 Storage (1 new):
- [ ] Instance Store - Ephemeral, High Random IO, locally attached.md

### 3 Databases (2 new):
- [ ] DAX - DynamoDB Accelerator (caching for DynamoDB).md
- [ ] OpenSearch - Log analytics, search, dashboards (formerly Elasticsearch).md
- [ ] Amazon MSK - Managed Kafka (alternative to Kinesis Data Streams).md

### 4 Networking (4 new):
- [ ] NAT Gateway - Allows private subnets to talk to the internet.md
- [ ] VPC Endpoints - PrivateLink (Interface) vs Gateway (S3 DynamoDB).md
- [ ] VPC Peering - Connect two VPCs.md
- [ ] Transit Gateway - Hub-and-spoke topology for many VPCs.md

### 5 Security (5 new):
- [ ] IAM Identity Center (SSO) - Centralized login for organizations.md
- [ ] STS (Security Token Service) - Temporary credentials, AssumeRole.md
- [ ] Systems Manager Parameter Store - Store config secrets (free tier).md
- [ ] AWS Certificate Manager (ACM) - Free SSL TLS certs for ALB CloudFront API Gateway.md
- [ ] Inspector - EC2 vulnerability scanning.md
- [ ] AWS Firewall Manager - Centrally manage WAF Shield across accounts.md

### 6 Monitoring (3 new):
- [ ] X-Ray - Tracing and debugging distributed apps Lambda.md
- [ ] Cost Explorer & Compute Optimizer - Saving money.md
- [ ] AWS Budgets - Set alerts when costs exceed thresholds.md

### 7 Migration (2 new):
- [ ] Schema Conversion Tool (SCT) - Convert Oracle to Aurora.md
- [ ] Snow Family - Physical devices (Snowcone, Snowball Edge, Snowmobile) for massive data.md

### 8 Application Integration (1 new):
- [ ] Amazon MQ - Broker for industry standards (MQTT, AMQP) lift and shift legacy apps.md

---

## 📊 REMAINING: Comparison & Cheat Sheet Files (15 new)

### 1 Compute (2 new):
- [ ] Comparison - EC2 vs Lambda vs Fargate vs ECS vs EKS.md
- [ ] Cheat Sheet - EC2 Instance Types (Families & Use Cases).md

### 2 Storage (1 new):
- [ ] Comparison - EBS Volume Types (gp2, gp3, io1, io2, st1, sc1).md

### 3 Databases (3 new):
- [ ] Comparison - RDS vs Aurora vs DynamoDB.md
- [ ] Comparison - ElastiCache Redis vs Memcached vs DAX.md
- [ ] Comparison - Athena vs Redshift vs EMR vs OpenSearch.md

### 4 Networking (3 new):
- [ ] Comparison - ALB vs NLB vs GWLB.md
- [ ] Comparison - VPC Peering vs Transit Gateway vs PrivateLink.md
- [ ] Cheat Sheet - Route 53 Routing Policies.md

### 5 Security (2 new):
- [ ] Comparison - KMS vs CloudHSM vs Secrets Manager vs Parameter Store.md
- [ ] Cheat Sheet - IAM Best Practices.md

### 6 Monitoring (1 new):
- [ ] Comparison - CloudWatch vs CloudTrail vs Config vs X-Ray.md

### 7 Migration (1 new):
- [ ] Comparison - DataSync vs Transfer Family vs Snow Family vs Storage Gateway.md

### 8 Application Integration (2 new):
- [ ] Comparison - SQS vs SNS vs EventBridge vs Amazon MQ.md
- [ ] Comparison - Step Functions vs SWF.md

### 9 Developer Tools (1 new):
- [ ] Cheat Sheet - CI CD Pipeline (Commit → Build → Deploy → Pipeline).md

---

## 📝 REMAINING: Meta Files (2 new)

- [ ] README.md - Vault overview and study guide
- [ ] STUDY-PROGRESS-TRACKER.md - Track learning progress

---

## 📊 Summary

**Total Remaining Tasks:**
- 9 structural cleanup/reorganization tasks
- 24 new service note files
- 15 new comparison/cheat sheet files
- 2 meta files

**Total: 50 remaining tasks**

---

## ❓ Questions to Resolve

1. **Systems Manager (SSM)** - Currently in Security folder. Should it:
   - Stay in Security only
   - Move to Monitoring/Management only
   - Remain in both (cross-referenced)

2. **Lambda@Edge** - File exists but is stub (15 lines). Should I expand it or leave as-is?

3. **Priority** - Which phase should be tackled first?
   - Phase 1: Structural cleanup (quickest)
   - Phase 2: New service notes (most comprehensive)
   - Phase 3: Comparison/cheat sheets (most useful for exam)

---

**Note:** The "EXAM DAY CHEAT SHEET - SAA-C03.md" I created is a comprehensive exam reference that differs from the individual cheat sheets proposed here. Those individual cheat sheets would be category-specific (e.g., just Route 53 routing policies, just EC2 instance types), while the Exam Day Cheat Sheet covers the entire exam.
