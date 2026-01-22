# Comparison: EBS Volume Types

Quick reference for choosing the right EBS volume type.

---

## Volume Types Overview

| Volume Type | Category | Use Case | Max IOPS | Max Throughput | Price |
|-------------|----------|----------|----------|----------------|-------|
| **gp3** | General Purpose SSD | Most workloads | 16,000 | 1,000 MB/s | $ |
| **gp2** | General Purpose SSD | Legacy (use gp3 instead) | 16,000 | 250 MB/s | $$ |
| **io2 Block Express** | Provisioned IOPS SSD | Mission-critical, highest performance | 256,000 | 4,000 MB/s | $$$$ |
| **io2** | Provisioned IOPS SSD | Mission-critical databases | 64,000 | 1,000 MB/s | $$$ |
| **io1** | Provisioned IOPS SSD | Legacy (use io2 instead) | 64,000 | 1,000 MB/s | $$$ |
| **st1** | Throughput Optimized HDD | Big data, data warehouses | 500 | 500 MB/s | $ |
| **sc1** | Cold HDD | Infrequently accessed data | 250 | 250 MB/s | $ |

---

## Decision Tree

```
What's your primary requirement?

├─ Low latency, transactional workloads (databases, boot volumes)
│  ├─ Standard performance → gp3 (default choice)
│  └─ Extreme performance, mission-critical → io2 or io2 Block Express
│
├─ High throughput, sequential access (big data, logs)
│  ├─ Frequently accessed → st1 (Throughput Optimized HDD)
│  └─ Infrequently accessed → sc1 (Cold HDD)
│
└─ Cost optimization
   └─ Use gp3 (cheaper than gp2, better performance control)
```

---

## Detailed Comparison

### **SSD Volumes (for transactional workloads)**

#### **gp3 (General Purpose SSD) - RECOMMENDED**

| Attribute | Value |
|-----------|-------|
| **IOPS** | 3,000 baseline, up to 16,000 |
| **Throughput** | 125 MB/s baseline, up to 1,000 MB/s |
| **Size** | 1 GB - 16 TB |
| **Use Case** | Boot volumes, dev/test, most applications |
| **Cost** | ~$0.08/GB-month |
| **Key Advantage** | **Independently configure IOPS and throughput** |

**Why gp3 over gp2:**
- 20% cheaper
- Predictable performance (doesn't depend on volume size)
- Can increase IOPS/throughput without increasing volume size

---

#### **gp2 (General Purpose SSD) - LEGACY**

| Attribute | Value |
|-----------|-------|
| **IOPS** | 3 IOPS per GB (min 100, max 16,000) |
| **Throughput** | Up to 250 MB/s |
| **Size** | 1 GB - 16 TB |
| **Use Case** | Legacy (migrate to gp3) |
| **Cost** | ~$0.10/GB-month |
| **Key Limitation** | IOPS tied to volume size (need large volume for high IOPS) |

**Exam Tip:** gp2 is legacy. Always choose **gp3** for new workloads.

---

#### **io2 Block Express (Provisioned IOPS SSD) - EXTREME PERFORMANCE**

| Attribute | Value |
|-----------|-------|
| **IOPS** | Up to 256,000 |
| **Throughput** | Up to 4,000 MB/s |
| **Size** | 4 GB - 64 TB |
| **Use Case** | Largest, most critical databases |
| **Cost** | $$$$$ (very expensive) |
| **Key Advantage** | Highest performance available |

---

#### **io2 (Provisioned IOPS SSD)**

| Attribute | Value |
|-----------|-------|
| **IOPS** | Up to 64,000 (can specify exact IOPS) |
| **Throughput** | Up to 1,000 MB/s |
| **Size** | 4 GB - 16 TB |
| **Use Case** | Mission-critical databases (Oracle, SQL Server, MongoDB) |
| **Cost** | $$$ (expensive: pay for GB + IOPS) |
| **Key Advantage** | 99.999% durability (vs 99.8-99.9% for gp3) |

**When to use io2:**
- Need consistent, guaranteed IOPS (>16,000)
- Mission-critical production databases
- Sub-millisecond latency required

---

### **HDD Volumes (for throughput-intensive workloads)**

#### **st1 (Throughput Optimized HDD)**

| Attribute | Value |
|-----------|-------|
| **IOPS** | Up to 500 |
| **Throughput** | Up to 500 MB/s |
| **Size** | 125 GB - 16 TB |
| **Use Case** | Big data, data warehouses, log processing, Kafka |
| **Cost** | ~$0.045/GB-month (cheapest for high throughput) |
| **Key Advantage** | High throughput for sequential access |

**Cannot be a boot volume** (only SSD can be boot volumes)

---

#### **sc1 (Cold HDD)**

| Attribute | Value |
|-----------|-------|
| **IOPS** | Up to 250 |
| **Throughput** | Up to 250 MB/s |
| **Size** | 125 GB - 16 TB |
| **Use Case** | Infrequently accessed data, archives, backups |
| **Cost** | ~$0.015/GB-month (cheapest EBS option) |
| **Key Advantage** | Lowest cost |

**Cannot be a boot volume**

---

## Exam Scenarios

| Scenario | Volume Type |
|----------|-------------|
| Boot volume for EC2 | **gp3** |
| General application, balanced performance | **gp3** |
| Mission-critical database, need 30,000 IOPS | **io2** |
| NoSQL database, need 100,000 IOPS | **io2 Block Express** |
| Big data processing (Hadoop, Kafka) | **st1** |
| Infrequent backups, archives | **sc1** |
| "Cost-optimized general purpose" | **gp3** (not gp2!) |

---

## Key Exam Tips

1. **gp3 is the default choice** (replaces gp2)
2. **io2 for mission-critical** databases requiring >16,000 IOPS
3. **st1 for big data** with high sequential throughput
4. **sc1 for cold storage** (cheapest)
5. **Only SSD (gp3, io2) can be boot volumes** - HDD (st1, sc1) cannot
6. **gp2 and io1 are legacy** - use gp3 and io2 instead

---

## Cost Optimization Tips

- **Migrate gp2 → gp3** (20% cost savings)
- **Right-size volumes** (don't over-provision)
- **Use st1 for throughput-heavy workloads** (cheaper than SSD for sequential access)
- **Archive to sc1** instead of keeping on gp3
- **Delete unused snapshots** (they accumulate cost)
