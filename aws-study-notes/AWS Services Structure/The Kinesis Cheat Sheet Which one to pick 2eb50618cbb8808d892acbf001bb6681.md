# The Kinesis Cheat Sheet: Which one to pick?

1. **Kinesis Data Firehose (Amazon Data Firehose)**
    - **Best for:** "Load data into S3 / Redshift / OpenSearch / Splunk."
    - **Key Phrase:** "Least administrative overhead," "Easiest way to load streaming data."
    - **Transformation:** Can use **Lambda** to transform (e.g., convert JSON to CSV) before writing to S3.
    - **Management:** Fully Serverless (No shards to manage).
2. **Kinesis Data Streams (KDS)**
    - **Best for:** "Real-time (<1 second) processing," "Custom consumer applications," "Replaying data."
    - **Management:** You must provision **Shards** (or use On-Demand mode).
    - **Difference:** It stores data (retention period) so multiple apps can read the same stream independently. Firehose does not store; it delivers.
3. **Kinesis Data Analytics (Managed Service for Apache Flink)**
    - **Best for:** "Running SQL queries on streaming data," "Real-time anomaly detection," "Time-series analytics."
    - **Input:** Reads from Streams or Firehose.
    - **Output:** Sends results to Streams or Firehose.