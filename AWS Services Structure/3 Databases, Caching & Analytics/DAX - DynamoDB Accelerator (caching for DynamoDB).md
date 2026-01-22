- **What is it? (1-Sentence Pitch):** Amazon DynamoDB Accelerator (DAX) is a fully managed, in-memory cache specifically designed for DynamoDB that delivers up to 10x performance improvement (microsecond latency) for read-heavy workloads.

- **Core Use Cases:**
    - Read-heavy applications requiring microsecond response times
    - Gaming leaderboards and real-time bidding
    - Shopping carts and session stores
    - Applications with repeated reads of the same data
    - Reducing DynamoDB read costs by caching frequently accessed items

- **Key Features & Concepts:**
    - **In-Memory Caching:** Provides microsecond latency for cached reads
    - **Fully Managed:** No cache invalidation logic needed - DAX handles it automatically
    - **DynamoDB-Compatible API:** Minimal code changes (just point your app to DAX cluster endpoint)
    - **Write-Through Cache:** Writes go through DAX to DynamoDB (eventual consistency)
    - **High Availability:** Multi-AZ deployment with automatic failover
    - **Scalable:** Can scale horizontally by adding nodes to the cluster

- **How DAX Works:**
    1. Application makes a read request to DAX
    2. **Cache Hit:** DAX returns data from cache (microsecond latency)
    3. **Cache Miss:** DAX queries DynamoDB, caches the result, and returns it to the application
    4. Writes are written to DynamoDB immediately and then cached in DAX

- **DAX vs ElastiCache:**
    | **Aspect** | **DAX** | **ElastiCache** |
    |---|---|---|
    | **Purpose** | Caching for **DynamoDB only** | General-purpose caching (any database) |
    | **API** | DynamoDB-compatible API | Redis or Memcached API |
    | **Integration** | Seamless with DynamoDB | Requires code changes for caching logic |
    | **Best For** | DynamoDB read-heavy workloads | Complex caching scenarios, session stores |
    | **Cache Management** | Automatic (no invalidation logic) | Manual (you handle cache invalidation) |

- **Integration:**
    - **DynamoDB:** DAX sits between your application and DynamoDB
    - **VPC:** DAX clusters run in your VPC
    - **CloudWatch:** Monitors DAX cluster metrics

- **Exam Tips / What to Remember:**
    - **DAX is exclusively for DynamoDB** (not for RDS or other databases)
    - Provides **microsecond latency** (vs. DynamoDB's single-digit millisecond latency)
    - **No code changes required** beyond updating the endpoint
    - DAX is **NOT suitable for write-heavy applications** - it's optimized for reads
    - Exam scenario: **"DynamoDB read performance is slow"** = DAX
    - **ElastiCache is for general caching** (any database), **DAX is DynamoDB-specific**
    - DAX automatically handles **cache invalidation** when items are updated in DynamoDB
    - **Eventually consistent reads** from DAX (not strongly consistent)
