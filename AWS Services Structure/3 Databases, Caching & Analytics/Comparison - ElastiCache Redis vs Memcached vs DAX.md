# Comparison: ElastiCache Redis vs Memcached vs DAX

Quick reference for choosing the right caching service.

---

## High-Level Decision Tree

```
What do you need to cache?

├─ DynamoDB data only
│  └─ DAX (DynamoDB Accelerator)
│
├─ Any database or application data
   ├─ Need advanced data structures, persistence, pub/sub
   │  └─ ElastiCache for Redis
   │
   └─ Need simple, multi-threaded, distributed cache
      └─ ElastiCache for Memcached
```

---

## Comparison Table

| Aspect | Redis | Memcached | DAX |
|--------|-------|-----------|-----|
| **Purpose** | Advanced caching + data structures | Simple distributed cache | DynamoDB-specific cache |
| **Data Structures** | **Strings, Lists, Sets, Hashes, Sorted Sets** | Key-Value only | Key-Value (DynamoDB items) |
| **Persistence** | **Yes** (snapshots, AOF) | **No** (in-memory only) | No |
| **Replication** | **Yes** (Multi-AZ, read replicas) | **No** | Multi-AZ (automatic) |
| **Multi-Threading** | **No** (single-threaded) | **Yes** (multi-threaded) | N/A |
| **Pub/Sub** | **Yes** | No | No |
| **Transactions** | **Yes** (ACID-like) | No | No |
| **Backup/Restore** | **Yes** (snapshots) | No | No |
| **Sorted Sets** | **Yes** (leaderboards, rankings) | No | No |
| **Geospatial** | **Yes** (location-based queries) | No | No |
| **Use Case** | **Complex caching, session store, leaderboards** | Simple cache, horizontal scaling | **DynamoDB read acceleration** |
| **API** | Redis API | Memcached API | **DynamoDB-compatible API** |
| **Complexity** | Higher (more features) | Lower (simpler) | Lowest (automatic) |
| **Cost** | $$ | $ | $$ |

---

## When to Use Each Service

### **Use ElastiCache for Redis If:**

✅ Need **advanced data structures** (lists, sets, sorted sets, hashes)
✅ Need **data persistence** (survive node failures)
✅ Need **high availability** (Multi-AZ, automatic failover)
✅ Building **leaderboards** or **real-time analytics** (sorted sets)
✅ Need **Pub/Sub messaging** (real-time notifications)
✅ Want **read replicas** for scaling reads
✅ Need **backup and restore**
✅ **Session store** for web applications (with persistence)

**Common Use Cases:**
- Gaming leaderboards (sorted sets)
- Real-time analytics
- Session management (with persistence)
- Pub/Sub messaging
- Caching complex queries

---

### **Use ElastiCache for Memcached If:**

✅ Need **simple key-value caching**
✅ Need **multi-threaded performance** (utilize multi-core CPUs)
✅ Want **horizontal scaling** (add/remove nodes easily)
✅ **Don't need persistence** or high availability
✅ **Don't need advanced data structures**
✅ Want **lowest complexity**

**Common Use Cases:**
- Simple database query caching
- Object caching (API responses, HTML fragments)
- Session store (non-critical, can lose data)

❌ **Don't use if:** Need persistence, Multi-AZ, or complex data structures

---

### **Use DAX If:**

✅ Using **DynamoDB** and need faster reads
✅ Need **microsecond latency** (vs DynamoDB's millisecond latency)
✅ Want **zero code changes** (DynamoDB-compatible API)
✅ Want **automatic cache management** (no manual invalidation)
✅ Read-heavy DynamoDB workload

**Common Use Cases:**
- Accelerating DynamoDB reads
- Gaming applications with DynamoDB backend
- Real-time bidding platforms
- E-commerce product catalogs

❌ **Don't use if:** Not using DynamoDB, or need to cache other data sources

---

## Detailed Feature Comparison

### **ElastiCache for Redis**

| Feature | Details |
|---------|---------|
| **Data Types** | Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLogs, Geospatial |
| **Persistence** | RDB snapshots, AOF (Append-Only File) |
| **Replication** | Primary + up to 5 read replicas |
| **Multi-AZ** | Yes (automatic failover) |
| **Cluster Mode** | Yes (partitioning for horizontal scaling) |
| **Max Node Size** | 6.1 TB memory (r6gd.16xlarge) |
| **Auth** | Redis AUTH, IAM authentication |

**Advanced Features:**
- **Pub/Sub:** Real-time messaging between applications
- **Lua Scripting:** Server-side scripts for complex operations
- **Transactions:** ACID-like transactions (MULTI/EXEC)
- **Geospatial:** Location-based queries (GEOADD, GEORADIUS)

---

### **ElastiCache for Memcached**

| Feature | Details |
|---------|---------|
| **Data Types** | Key-Value (strings only) |
| **Persistence** | None (in-memory only) |
| **Replication** | None |
| **Multi-AZ** | No |
| **Cluster Mode** | Yes (auto-discovery) |
| **Max Node Size** | 1.5 TB memory (r6g.16xlarge) |
| **Auth** | SASL authentication |

**Key Characteristics:**
- **Multi-threaded:** Better CPU utilization
- **Horizontal scaling:** Easy to add/remove nodes
- **Simpler architecture:** No replication, no persistence

---

### **DynamoDB Accelerator (DAX)**

| Feature | Details |
|---------|---------|
| **Data Types** | DynamoDB items (key-value, documents) |
| **Persistence** | No (cache only, data in DynamoDB) |
| **Replication** | Multi-AZ (automatic) |
| **Multi-AZ** | Yes (3 nodes minimum for production) |
| **Cluster Mode** | Yes (partitioned cache) |
| **API** | **DynamoDB-compatible** (no code changes) |
| **Cache Invalidation** | **Automatic** (write-through cache) |

**Key Advantages:**
- **Zero code changes:** Point app to DAX endpoint instead of DynamoDB
- **Automatic cache management:** No manual invalidation needed
- **Microsecond latency:** Up to 10x faster than DynamoDB

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Cache database queries (MySQL, PostgreSQL) | **ElastiCache (Redis or Memcached)** |
| Gaming leaderboard (sorted rankings) | **ElastiCache for Redis** (sorted sets) |
| Simple session store, can lose data | **Memcached** |
| Session store, must persist data | **Redis** |
| Accelerate DynamoDB reads | **DAX** |
| Need pub/sub messaging | **Redis** |
| Need to cache DynamoDB + RDS | **Redis** (DAX only works with DynamoDB) |
| Multi-threaded caching, simple use case | **Memcached** |
| Need high availability, automatic failover | **Redis** or **DAX** |

---

## Key Exam Tips

1. **Redis = Advanced features** (data structures, persistence, replication, pub/sub)
2. **Memcached = Simple caching** (key-value, multi-threaded, no persistence)
3. **DAX = DynamoDB-only** (microsecond latency, automatic cache management)
4. **Sorted sets (leaderboards)** = Redis only
5. **Pub/Sub messaging** = Redis only
6. **Multi-threaded** = Memcached only
7. **DynamoDB acceleration** = DAX
8. **Persistence required** = Redis (not Memcached)

---

## Summary

| Service | Best For | Key Feature |
|---------|----------|-------------|
| **Redis** | Complex caching, session store, leaderboards | Advanced data structures, persistence |
| **Memcached** | Simple distributed cache | Multi-threaded, horizontal scaling |
| **DAX** | DynamoDB read acceleration | DynamoDB-compatible, automatic caching |
