# Comparison: RDS vs Aurora vs DynamoDB

Quick reference for choosing the right database service.

---

## High-Level Decision Tree

```
What type of data model?

├─ Relational (SQL) - tables, joins, ACID transactions
│  ├─ Need maximum performance & AWS-native features → Aurora
│  ├─ Standard relational database → RDS
│  └─ Migrating from on-prem (Oracle, SQL Server) → RDS
│
└─ Non-Relational (NoSQL) - key-value, document
   └─ Need single-digit millisecond latency, massive scale → DynamoDB
```

---

## Comparison Table

| Aspect | RDS | Aurora | DynamoDB |
|--------|-----|--------|----------|
| **Database Type** | Relational (SQL) | Relational (SQL) | NoSQL (Key-Value, Document) |
| **Engines** | PostgreSQL, MySQL, MariaDB, Oracle, SQL Server | PostgreSQL, MySQL | DynamoDB (proprietary) |
| **Performance** | Good | **5x MySQL, 3x PostgreSQL** | Single-digit millisecond latency |
| **Scaling** | Vertical (change instance size) | Vertical + Horizontal (read replicas) | Horizontal (automatic, unlimited) |
| **Multi-AZ** | Yes (standby replica) | **Yes (6 copies across 3 AZs)** | Yes (automatic, across 3 AZs) |
| **Read Replicas** | Up to 15 | Up to 15 | N/A (DynamoDB handles internally) |
| **Backups** | Automated + manual snapshots | Automated + manual snapshots | Point-in-time recovery (PITR) |
| **Cost** | $ - $$$ (instance-based) | $$$ (more than RDS) | $ - $$ (pay per request or provisioned) |
| **Use Case** | Traditional apps, migrations | **High-performance apps**, cloud-native | **Serverless apps**, massive scale |
| **Management** | AWS manages (patching, backups) | AWS manages (less ops than RDS) | **Fully managed** (zero admin) |

---

## When to Use Each Service

### **Use RDS If:**

✅ You need a standard relational database (PostgreSQL, MySQL, Oracle, SQL Server)
✅ Migrating from on-premises databases
✅ Existing application uses SQL
✅ Need ACID transactions, complex queries, joins
✅ Cost-conscious (cheaper than Aurora for small workloads)

❌ **Don't use if:** Need extreme scalability or NoSQL

---

### **Use Aurora If:**

✅ Need **high performance** (5x faster than standard MySQL)
✅ Building **cloud-native applications** on AWS
✅ Need **high availability** (6 copies across 3 AZs, automatic failover <30 sec)
✅ Want **MySQL or PostgreSQL compatibility** with AWS enhancements
✅ Need **auto-scaling storage** (up to 128 TB)
✅ Global applications (**Aurora Global Database** - cross-region replication)

❌ **Don't use if:** Small workload (RDS is cheaper), need Oracle/SQL Server

---

### **Use DynamoDB If:**

✅ Need **single-digit millisecond latency** at any scale
✅ Building **serverless applications** (integrates with Lambda)
✅ Need **massive scalability** (petabytes of data, millions of requests/sec)
✅ Data model fits **key-value or document store**
✅ Want **zero database administration**
✅ Global applications (**DynamoDB Global Tables** - multi-region, multi-active)

❌ **Don't use if:** Need complex SQL queries, joins, or relational data model

---

## Detailed Comparison

### **RDS (Relational Database Service)**

| Feature | Details |
|---------|---------|
| **Engines** | PostgreSQL, MySQL, MariaDB, Oracle, SQL Server |
| **Max Storage** | 64 TB (SQL Server: 16 TB) |
| **Multi-AZ** | Synchronous standby replica in another AZ |
| **Read Replicas** | Up to 15 (asynchronous replication) |
| **Backups** | Automated (1-35 day retention), manual snapshots |
| **Scaling** | Vertical (change instance size), read replicas for reads |
| **Pricing** | Instance hours + storage + I/O + backups |

**Key Features:**
- Automated patching and backups
- Point-in-time recovery
- Encryption at rest (KMS) and in transit (SSL)
- Performance Insights for monitoring

---

### **Aurora**

| Feature | Details |
|---------|---------|
| **Engines** | MySQL-compatible, PostgreSQL-compatible |
| **Max Storage** | 128 TB (auto-scaling, no need to provision) |
| **Multi-AZ** | **6 copies across 3 AZs** (built-in, no extra cost) |
| **Read Replicas** | Up to 15 Aurora Replicas (low latency) |
| **Backups** | Continuous backup to S3, point-in-time recovery |
| **Scaling** | Auto-scaling storage, Aurora Serverless v2 for compute |
| **Pricing** | Higher than RDS (pay for I/O operations) |

**Key Features:**
- **Aurora Serverless** (auto-scales compute, pay per second)
- **Aurora Global Database** (cross-region replication, <1 sec lag)
- **Faster failover** (<30 seconds) vs RDS (1-2 minutes)
- **Backtrack** (rewind database to previous point in time without restore)

---

### **DynamoDB**

| Feature | Details |
|---------|---------|
| **Data Model** | Key-Value, Document (JSON-like) |
| **Max Item Size** | 400 KB per item |
| **Multi-AZ** | Automatic, across 3 AZs |
| **Scaling** | **Automatic** (horizontal scaling, no limits) |
| **Backups** | On-demand backups, Point-in-time recovery (PITR) |
| **Consistency** | Eventual (default) or Strongly consistent reads |
| **Pricing** | Provisioned capacity (RCU/WCU) or On-Demand |

**Key Features:**
- **DynamoDB Streams** (capture changes, trigger Lambda)
- **Global Tables** (multi-region, multi-active, <1 sec replication)
- **DAX** (in-memory cache for microsecond latency)
- **Transactions** (ACID transactions across items/tables)

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Migrate Oracle database from on-prem | **RDS (Oracle)** |
| High-performance MySQL for cloud-native app | **Aurora MySQL** |
| Serverless app needs key-value store | **DynamoDB** |
| Need complex SQL joins and analytics | **RDS** or **Aurora** |
| Gaming leaderboard, millions of requests/sec | **DynamoDB** |
| Multi-region database with <1 sec replication | **Aurora Global Database** or **DynamoDB Global Tables** |
| E-commerce app with variable traffic | **DynamoDB (On-Demand)** or **Aurora Serverless** |
| Need SQL + 5x performance boost | **Aurora** |

---

## Key Exam Tips

1. **RDS = Standard relational databases** (PostgreSQL, MySQL, Oracle, SQL Server)
2. **Aurora = High-performance, cloud-native MySQL/PostgreSQL** (5x faster, auto-scaling storage)
3. **DynamoDB = NoSQL, serverless, massive scale** (key-value, single-digit ms latency)
4. **Aurora is more expensive than RDS** but offers better performance and availability
5. **DynamoDB is fully managed NoSQL** - no servers, no patching, auto-scales infinitely
6. For **global applications**: Aurora Global Database (SQL) or DynamoDB Global Tables (NoSQL)
7. For **serverless**: Aurora Serverless (SQL) or DynamoDB (NoSQL)

---

## Summary

- **RDS:** Traditional SQL databases, migrations, cost-effective
- **Aurora:** High-performance SQL, cloud-native, auto-scaling
- **DynamoDB:** NoSQL, serverless, massive scale, low latency
