# DynamoDB: Key-value, single-digit millisecond latency, DAX, Streams, Global Tables.

- **What is it? (1-Sentence Pitch):** A fully managed, serverless, key-value and document NoSQL database designed for single-digit millisecond performance at any scale.
- **Core Use Cases:** Web and mobile apps, gaming leaderboards, IoT applications, and any application needing extremely low latency and massive scalability.??
- **Key Features & Concepts:**
    - **Capacity Modes:** Provisioned (you specify read/write capacity) and On-Demand (pay-per-request).
    - **DAX (DynamoDB Accelerator):** A fully managed, in-memory cache for DynamoDB that delivers microsecond latency for read-heavy workloads.
    - **Global Tables:** Creates a multi-region, multi-active database for globally distributed applications.
    - **DynamoDB Streams:** A time-ordered sequence of item-level changes. Perfect for triggering Lambda functions for event-driven processing.