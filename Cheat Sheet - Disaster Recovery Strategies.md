# Cheat Sheet: Disaster Recovery Strategies

Quick reference for AWS disaster recovery approaches and choosing the right strategy.

---

## High-Level Decision Tree

```
What's your Recovery Time Objective (RTO) and Recovery Point Objective (RPO)?

├─ RTO: Hours-Days, RPO: Hours-Days, Low Cost Priority
│  └─ Backup and Restore
│
├─ RTO: Hours, RPO: Minutes, Balanced Cost
│  └─ Pilot Light
│
├─ RTO: Minutes, RPO: Seconds, Higher Cost
│  └─ Warm Standby
│
└─ RTO: Seconds, RPO: Near-Zero, Highest Cost
   └─ Multi-Site Active-Active
```

---

## Key Concepts

**RTO (Recovery Time Objective):** How long can your application be down?
- Example: "We can tolerate 4 hours of downtime"

**RPO (Recovery Point Objective):** How much data loss is acceptable?
- Example: "We can lose up to 1 hour of data"

**Rule of Thumb:**
- **Lower RTO/RPO = Higher Cost** (more infrastructure running continuously)
- **Higher RTO/RPO = Lower Cost** (less infrastructure, more manual recovery)

---

## The Four DR Strategies

### **1. Backup and Restore** (Lowest Cost, Highest RTO/RPO)

**What It Is:** Regularly backup data to S3/Glacier, restore when disaster occurs

**Architecture:**
```
┌─────────────┐         ┌──────────────┐
│ Production  │ Backup  │  S3/Glacier  │
│   Region    │ ──────> │   (Backup)   │
└─────────────┘         └──────────────┘
                              │
                        Restore (manual)
                              ↓
                        ┌──────────────┐
                        │ New DR Region│
                        └──────────────┘
```

**Characteristics:**

| Aspect | Details |
|--------|---------|
| **RTO** | **Hours to Days** (time to provision infrastructure + restore data) |
| **RPO** | **Hours** (depends on backup frequency) |
| **Cost** | 💰 **Lowest** (only pay for S3 storage) |
| **Complexity** | **Low** (simple to implement) |

**What's Running:**
- ✅ S3/Glacier storing backups
- ❌ NO infrastructure in DR region (provision during disaster)

**How It Works:**
1. **Normal Operations:** Automated backups to S3 (snapshots, database backups, file backups)
2. **Disaster Occurs:** Provision new infrastructure in DR region
3. **Restore Data:** Download backups from S3, restore to new infrastructure
4. **Update DNS:** Point Route 53 to new region

**Use Case:** Non-critical applications where downtime is acceptable (internal tools, dev/test)

**AWS Services:**
- **Backup:** AWS Backup, EBS snapshots, RDS snapshots, S3 versioning
- **Restore:** CloudFormation (provision infrastructure), EC2, RDS

**Exam Trigger:**
- "Most cost-effective DR solution"
- "Acceptable downtime: several hours"
- "Backup only"

---

### **2. Pilot Light** (Low Cost, Medium RTO/RPO)

**What It Is:** Keep core infrastructure running at minimal capacity, scale up during disaster

**Architecture:**
```
┌─────────────────┐         ┌─────────────────┐
│ Production      │ Replicate│ DR Region       │
│                 │ ──────> │                 │
│ • EC2 (full)    │         │ • RDS (standby) │
│ • RDS (primary) │         │ • Data synced   │
│ • Full capacity │         │ • NO EC2 yet    │
└─────────────────┘         └─────────────────┘
                                   │
                            Scale up during DR
                                   ↓
                            ┌─────────────────┐
                            │ • EC2 launched  │
                            │ • Full capacity │
                            └─────────────────┘
```

**Characteristics:**

| Aspect | Details |
|--------|---------|
| **RTO** | **Minutes to Hours** (time to scale up compute resources) |
| **RPO** | **Minutes** (continuous data replication) |
| **Cost** | 💰💰 **Low-Medium** (minimal infrastructure + data replication) |
| **Complexity** | **Medium** |

**What's Running:**
- ✅ Database (standby replica with continuous replication)
- ✅ Data replication (S3 CRR, RDS read replica, DynamoDB Global Tables)
- ❌ Compute resources (EC2, ECS) - launched during disaster
- ❌ Load balancers - created during disaster

**How It Works:**
1. **Normal Operations:**
   - RDS read replica in DR region (continuous replication)
   - S3 Cross-Region Replication (CRR) to DR region
   - Core data always up-to-date
2. **Disaster Occurs:**
   - Promote RDS read replica to primary
   - Launch EC2/ECS instances using pre-configured AMIs
   - Create load balancers
3. **Update DNS:** Route 53 failover to DR region

**Use Case:** Critical applications with moderate RTO/RPO requirements

**AWS Services:**
- **Data Replication:** RDS read replicas (Cross-Region), DynamoDB Global Tables, S3 CRR
- **Compute:** Pre-baked AMIs, Auto Scaling Groups (min=0 normally, scale up during DR)
- **DNS:** Route 53 failover routing

**Exam Trigger:**
- "Critical data must be replicated"
- "RTO: 1-2 hours"
- "Minimize cost while keeping data current"

---

### **3. Warm Standby** (Medium-High Cost, Low RTO/RPO)

**What It Is:** Fully functional DR environment running at reduced capacity, scale up during disaster

**Architecture:**
```
┌─────────────────┐         ┌─────────────────┐
│ Production      │ Replicate│ DR Region       │
│                 │ ──────> │ (Warm Standby)  │
│ • EC2 (100%)    │         │ • EC2 (20%)     │
│ • RDS (primary) │         │ • RDS (standby) │
│ • ALB (full)    │         │ • ALB (minimal) │
│ • Full traffic  │         │ • Health checks │
└─────────────────┘         └─────────────────┘
                                   │
                            Scale up during DR
                                   ↓
                            ┌─────────────────┐
                            │ • EC2 → 100%    │
                            │ • Handle traffic│
                            └─────────────────┘
```

**Characteristics:**

| Aspect | Details |
|--------|---------|
| **RTO** | **Minutes** (time to scale up existing resources) |
| **RPO** | **Seconds** (continuous synchronous/asynchronous replication) |
| **Cost** | 💰💰💰 **Medium-High** (scaled-down infrastructure always running) |
| **Complexity** | **Medium-High** |

**What's Running:**
- ✅ Scaled-down EC2/ECS instances (10-20% of production capacity)
- ✅ Database standby replica (continuous replication)
- ✅ Load balancers (active, performing health checks)
- ✅ All services configured and ready

**How It Works:**
1. **Normal Operations:**
   - DR environment fully functional at reduced capacity
   - Continuous data replication to DR region
   - Load balancers running health checks (not receiving traffic)
2. **Disaster Occurs:**
   - Auto Scaling scales up EC2/ECS to full capacity (minutes)
   - Promote RDS read replica to primary
3. **Update DNS:** Route 53 switches traffic to DR region

**Use Case:** Business-critical applications requiring fast recovery

**AWS Services:**
- **Compute:** Auto Scaling Groups (min capacity running in DR)
- **Database:** RDS Multi-AZ (cross-region read replica), Aurora Global Database
- **DNS:** Route 53 health checks + failover routing
- **Scaling:** Auto Scaling (target tracking to scale up quickly)

**Exam Trigger:**
- "RTO: Minutes"
- "Business-critical application"
- "Minimal downtime acceptable"

---

### **4. Multi-Site Active-Active** (Highest Cost, Lowest RTO/RPO)

**What It Is:** Full production capacity in multiple regions, both actively serving traffic

**Architecture:**
```
┌─────────────────┐         ┌─────────────────┐
│ Region 1        │ Bidirec-│ Region 2        │
│ (Active)        │ tional  │ (Active)        │
│                 │ Sync    │                 │
│ • EC2 (100%)    │<──────>│ • EC2 (100%)    │
│ • RDS (active)  │         │ • RDS (active)  │
│ • 50% traffic   │         │ • 50% traffic   │
└────────┬────────┘         └────────┬────────┘
         │                           │
         └────────┬──────────────────┘
                  ↓
         ┌────────────────┐
         │  Route 53      │
         │  (Geolocation/ │
         │   Latency)     │
         └────────────────┘
```

**Characteristics:**

| Aspect | Details |
|--------|---------|
| **RTO** | **Near-Zero** (failover is instant) |
| **RPO** | **Near-Zero** (continuous bidirectional replication) |
| **Cost** | 💰💰💰💰 **Highest** (full capacity in multiple regions) |
| **Complexity** | **Highest** (complex data synchronization, conflict resolution) |

**What's Running:**
- ✅ Full production capacity in 2+ regions
- ✅ Both regions actively serving traffic
- ✅ Bidirectional data replication
- ✅ Global load balancing

**How It Works:**
1. **Normal Operations:**
   - Both regions serve production traffic (e.g., 50/50 split)
   - Route 53 distributes traffic based on latency or geolocation
   - Continuous bidirectional data sync
2. **Disaster Occurs:**
   - Route 53 automatically detects failure (health checks)
   - All traffic routes to healthy region(s)
   - NO manual intervention required

**Use Case:** Mission-critical applications with zero-downtime requirements (financial services, healthcare)

**AWS Services:**
- **Compute:** Full EC2/ECS capacity in multiple regions
- **Database:** DynamoDB Global Tables, Aurora Global Database (< 1 second replication lag)
- **DNS:** Route 53 health checks + latency/geolocation routing
- **Global Load Balancing:** AWS Global Accelerator

**Exam Trigger:**
- "Zero downtime"
- "RTO: Seconds or near-zero"
- "Mission-critical"
- "Active-Active"

---

## Comparison Table

| Strategy | RTO | RPO | Cost | Complexity | What's Running in DR Region |
|----------|-----|-----|------|------------|----------------------------|
| **Backup & Restore** | Hours-Days | Hours | 💰 Lowest | Low | Nothing (S3 backups only) |
| **Pilot Light** | Minutes-Hours | Minutes | 💰💰 Low-Med | Medium | Database standby + data sync |
| **Warm Standby** | Minutes | Seconds | 💰💰💰 Med-High | Medium-High | Scaled-down full environment (10-20%) |
| **Multi-Site** | Near-Zero | Near-Zero | 💰💰💰💰 Highest | Highest | Full capacity, actively serving traffic |

---

## Key AWS Services for DR

### **Data Replication**

| Service | DR Strategy | How It Helps |
|---------|-------------|--------------|
| **S3 Cross-Region Replication (CRR)** | All strategies | Replicate objects to DR region automatically |
| **RDS Read Replicas (Cross-Region)** | Pilot Light, Warm Standby | Continuous async replication, promote to primary during DR |
| **Aurora Global Database** | Warm Standby, Multi-Site | < 1 second replication lag, fast failover |
| **DynamoDB Global Tables** | Multi-Site Active-Active | Multi-region, multi-active, bidirectional replication |
| **EBS Snapshots (Cross-Region Copy)** | Backup & Restore, Pilot Light | Copy snapshots to DR region for restore |
| **AWS Backup** | Backup & Restore | Centralized backup management, cross-region copy |

---

### **Compute Provisioning**

| Service | DR Strategy | How It Helps |
|---------|-------------|--------------|
| **AMIs (Cross-Region Copy)** | Pilot Light, Warm Standby | Pre-configured machine images for fast launch |
| **Auto Scaling Groups** | Warm Standby, Multi-Site | Automatically scale up capacity during DR |
| **CloudFormation StackSets** | All strategies | Deploy infrastructure across multiple regions |
| **Elastic Beanstalk** | Warm Standby | Managed platform, easy cross-region deployment |

---

### **DNS & Traffic Management**

| Service | How It Helps DR |
|---------|-----------------|
| **Route 53 Health Checks** | Monitor endpoint health, trigger failover |
| **Route 53 Failover Routing** | Automatically route to DR region when primary fails |
| **Route 53 Latency Routing** | Multi-Site: route users to nearest healthy region |
| **Route 53 Geolocation Routing** | Multi-Site: route based on user location |
| **AWS Global Accelerator** | Multi-Site: Anycast IPs, automatic failover, DDoS protection |

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| "Most cost-effective DR, tolerate hours of downtime" | **Backup & Restore** |
| "Critical data must be current, RTO: 2 hours acceptable" | **Pilot Light** |
| "Business-critical app, RTO: 10 minutes" | **Warm Standby** |
| "Financial trading platform, zero downtime required" | **Multi-Site Active-Active** |
| "E-commerce site, peak season, cannot afford downtime" | **Multi-Site Active-Active** |
| "Internal HR system, can be down for a day" | **Backup & Restore** |
| "Customer-facing API, RTO: 30 minutes, RPO: 5 minutes" | **Warm Standby** |
| "Database replication required, minimize cost" | **Pilot Light** (RDS read replica) |
| "Serve users in US and EU with low latency" | **Multi-Site Active-Active** (geolocation routing) |

---

## Choosing the Right Strategy

### Step 1: Determine RTO/RPO Requirements

**Ask:**
- How long can the application be down? (RTO)
- How much data loss is acceptable? (RPO)
- What's the business impact of downtime?

### Step 2: Calculate Acceptable Cost

**Cost Breakdown:**
- **Backup & Restore:** ~5-10% of production cost
- **Pilot Light:** ~20-30% of production cost
- **Warm Standby:** ~50-70% of production cost
- **Multi-Site:** ~200%+ of production cost (2+ full regions)

### Step 3: Match to Strategy

| Your Requirements | Strategy |
|-------------------|----------|
| RTO: Days, RPO: Hours, Low Budget | **Backup & Restore** |
| RTO: Hours, RPO: Minutes, Medium Budget | **Pilot Light** |
| RTO: Minutes, RPO: Seconds, Higher Budget | **Warm Standby** |
| RTO: Near-Zero, RPO: Near-Zero, Maximum Budget | **Multi-Site** |

---

## Implementation Best Practices

### **1. Test Your DR Plan Regularly**
- Conduct DR drills quarterly
- Measure actual RTO/RPO during tests
- Document failures and improve

### **2. Automate Everything**
- Use CloudFormation/Terraform for infrastructure
- Automate failover with Route 53 health checks
- Automate data replication

### **3. Monitor Replication Lag**
- CloudWatch metrics for RDS replication lag
- S3 replication metrics
- DynamoDB Global Tables replication latency

### **4. Document Runbooks**
- Step-by-step DR activation procedures
- Contact lists and escalation paths
- Rollback procedures

### **5. Use AWS Backup for Centralization**
- Centralized backup policies across services
- Cross-region backup copies
- Compliance reporting

---

## Key Exam Tips

1. **Lower RTO/RPO = Higher Cost** (inverse relationship)
2. **Backup & Restore:** Cheapest, slowest (hours-days RTO)
3. **Pilot Light:** Core data replicated, compute provisioned during DR
4. **Warm Standby:** Scaled-down environment always running
5. **Multi-Site:** Full capacity in 2+ regions, active-active
6. **RDS Read Replicas:** Pilot Light + Warm Standby (async replication)
7. **Aurora Global Database:** < 1 second lag (best for Warm Standby + Multi-Site)
8. **DynamoDB Global Tables:** Multi-Site Active-Active (bidirectional sync)
9. **Route 53 Health Checks + Failover:** Automate DR traffic routing
10. **AWS Global Accelerator:** Multi-Site with Anycast IPs, fast failover
11. **S3 CRR:** All strategies (replicate backups, data, logs)
12. **CloudFormation StackSets:** Deploy same infrastructure across regions
13. **"Most cost-effective DR"** → Backup & Restore
14. **"Zero downtime"** → Multi-Site Active-Active
15. **Test, Test, Test:** DR is only as good as your last successful test

---

## Summary

| Strategy | Best For | Key Characteristic |
|----------|----------|-------------------|
| **Backup & Restore** | Non-critical apps | Data in S3, provision during DR |
| **Pilot Light** | Critical apps with moderate RTO | Core data synced, scale up compute |
| **Warm Standby** | Business-critical apps | Scaled-down environment always running |
| **Multi-Site** | Mission-critical apps | Full capacity, active-active, near-zero RTO/RPO |

**Bottom Line:** Choose DR strategy based on RTO/RPO requirements and budget. Most cost-effective = Backup & Restore. Zero downtime = Multi-Site Active-Active.
