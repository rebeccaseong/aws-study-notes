# ElastiCache: Redis (complex data structures) vs. Memcached (simple).

- **What is it? (1-Sentence Pitch):** A managed in-memory data store and cache service that makes it easy to deploy, operate, and scale popular open-source compatible in-memory data stores.
- **Core Use Cases:** Caching database query results or user sessions to reduce latency and offload read traffic from your primary database (like RDS or DynamoDB).
- **Key Distinction (Exam Essential):**
    - **Redis:** More advanced feature set. Supports multi-AZ, backups, read replicas, and complex data types (lists, hashes). Use when you need richer functionality or persistence.
    - **Memcached:** Simpler and offers better multi-threaded performance. Use when you need the simplest caching model possible at high scale.