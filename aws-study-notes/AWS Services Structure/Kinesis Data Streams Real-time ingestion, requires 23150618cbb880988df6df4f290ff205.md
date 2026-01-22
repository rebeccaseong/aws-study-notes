# Kinesis Data Streams: Real-time ingestion, requires shard management ("The Pipe").

- **What is it? (1-Sentence Pitch):** Kinesis Data Streams is a massively scalable and durable service for ingesting and processing large streams of data records in **real-time**, with guaranteed ordering for records that share the same partition key.
- **Core Kinesis Components (The Building Blocks)**
    - **Data Stream:** The main resource, which is a collection of one or more shards (e.g., iot-sensor-data-stream).
    - **Shard:** The fundamental unit of capacity in a stream. You must provision enough shards to handle your workload.
        - **Ingest Capacity:** Each shard provides **1 MB/s** or **1,000 records/s** of write capacity.
        - **Read Capacity:** Each shard provides **2 MB/s** of read capacity (shared by standard consumers).
    - **Data Record:** A single piece of data put into the stream (up to 1 MB). It contains the data itself, a partition key, and a sequence number.
    - **Partition Key:** The most critical concept for ordering. Kinesis uses this key to group data into shards. **All records with the same partition key will go to the same shard**, which guarantees they are processed in order.
    - **Data Retention Period:** The length of time data is stored in the stream and is available for re-reading. The default is **24 hours**, but it can be configured up to **365 days**.
- **Key Use Cases (When to Choose Kinesis)**
    - **Real-time Analytics:** Processing application logs, clickstreams, or social media feeds as they happen.
    - **IoT Data Ingestion:** Collecting and processing high-volume data from thousands or millions of IoT devices (sensors, smart cars, etc.).
    - **Log and Event Data Collection:** Aggregating logs from many sources (e.g., EC2 instances, containers) into a central stream for processing and analysis.
    - **Real-time Leaderboards:** Processing game events to update player scores in real-time.
- **The Big Comparison: Kinesis vs. SQS**
    
    **Exam Rule of Thumb:** If the scenario involves **real-time processing**, **multiple applications needing the same data**, or **guaranteed order**, choose **Kinesis**. If it involves **decoupling**, **buffering tasks**, or **worker queues**, choose **SQS**.
    
    | **Feature** | **Kinesis Data Streams** | **Amazon SQS (Standard Queue)** |
    | --- | --- | --- |
    | **Primary Job** | **Ordered, real-time data streaming** | **Decoupling application tasks** |
    | **Data Model** | A continuous, replayable stream of records | A temporary queue of discrete messages |
    | **Consumers** | **Multiple consumers** can read the **same data** | **One consumer** processes a message, then it's deleted |
    | **Ordering** | **Guaranteed ordering** within a shard | Best-effort ordering |
    | **Data Persistence** | **Replayable** (24 hours to 1 year) | Deleted after processing (max 14 days) |
- **Capacity & Scaling Management**
    - **Capacity Modes:**
        - **Provisioned Mode:** You manually set the shard count. Best for **predictable workloads** where you can optimize costs. You scale by splitting or merging shards.
        - **On-Demand Mode:** Kinesis automatically manages shards for you. Best for **unpredictable or spiky workloads**. Easier to manage, but can be more expensive.
    - **The "Hot Shard" Problem:**
        - **What it is:** When a poor choice of **partition key** (e.g., one with low cardinality like "status_active") sends too much data to a single shard, exceeding its 1 MB/s limit.
        - **Symptom:** You will get WriteProvisionedThroughputExceeded errors even if the stream's total capacity is not reached.
        - **Solution:** Use a **high-cardinality partition key** (like user_id, device_id, or a UUID) to ensure data is distributed evenly across all shards.
- **Producers & Consumers (Getting Data In & Out)**
    - **Producers (Sending data to Kinesis):**
        - **Kinesis Producer Library (KPL):** A Java library that is the best-practice for high-performance producers. It handles batching, aggregation, and retries automatically.
        - **Kinesis Agent:** A standalone agent installed on servers to automatically send files (like logs) to Kinesis.
        - **AWS SDK:** The standard PutRecord and PutRecords API calls.
    - **Consumers (Reading data from Kinesis):**
        - **Standard Consumer:** All consumers reading from a shard share its 2 MB/s read limit. If you add more consumers, performance for all of them can degrade.
        - **Enhanced Fan-Out (EFO) Consumer:** Each EFO consumer gets its own **dedicated 2 MB/s per shard** read pipe. This is the solution when you need multiple applications to read from the same stream in real-time **without impacting each other**.
- **Operations: Monitoring & Alerting**
    - **Key CloudWatch Metrics:**
        - GetRecords.IteratorAgeMilliseconds: **THE MOST IMPORTANT METRIC.** This shows how far behind the latest data your consumer is. **ALERT IF THIS IS RISING!** It means your consumers can't keep up.
        - WriteProvisionedThroughputExceeded: Your producers are being throttled. This signals a hot shard or an under-provisioned stream.
        - ReadProvisionedThroughputExceeded: Your standard consumers are being throttled. Add more shards or switch to Enhanced Fan-Out.
- **Security & Networking**
    - **Encryption:**
        - **In Transit:** Enforced by using HTTPS endpoints (default in SDKs).
        - **At Rest:** Enable Server-Side Encryption (SSE) using AWS KMS to encrypt the data stored in the shards. **This should always be enabled in production.**
    - **IAM:** Use granular IAM policies to control who can read from (kinesis:GetRecords), write to (kinesis:PutRecord), and manage (kinesis:UpdateShardCount) the stream.
    - **VPC Integration:** Use a **VPC Interface Endpoint** for Kinesis. This allows resources inside your private VPC (like Lambda or EC2) to access the Kinesis stream without traversing the public internet, improving security and performance.