- **What is it? (1-Sentence Pitch):** Amazon Managed Streaming for Apache Kafka (MSK) is a fully managed service that makes it easy to build and run applications that use Apache Kafka to process streaming data.

- **Core Use Cases:**
    - Real-time data streaming and processing
    - Event-driven architectures
    - Migrating existing Kafka workloads to AWS
    - Log aggregation and analysis
    - IoT data ingestion
    - Change Data Capture (CDC) from databases

- **Key Features & Concepts:**
    - **Fully Managed Kafka:** AWS handles provisioning, configuration, patching, and failure recovery
    - **Apache Kafka Compatibility:** Use existing Kafka tools and applications without modification
    - **MSK Serverless:** Automatically scales capacity (no cluster sizing needed)
    - **MSK Connect:** Easily stream data to/from external systems (connectors for S3, DynamoDB, etc.)
    - **Multi-AZ Deployment:** Kafka brokers deployed across multiple Availability Zones
    - **Encryption:** In-transit and at-rest encryption, integrates with KMS
    - **Monitoring:** Integration with CloudWatch, Prometheus, and other monitoring tools

- **MSK vs Kinesis Data Streams:**
    | **Aspect** | **Amazon MSK** | **Kinesis Data Streams** |
    |---|---|---|
    | **Technology** | Apache Kafka (open-source) | AWS proprietary |
    | **Use Case** | **Migrating existing Kafka apps**, Kafka ecosystem tools | AWS-native, simpler managed streaming |
    | **Compatibility** | **Kafka-compatible** (use Kafka APIs) | Kinesis API only |
    | **Retention** | Configurable (default 7 days, unlimited with tiered storage) | 1-365 days |
    | **Consumers** | Unlimited consumers | Max 5 consumers per shard (or use enhanced fan-out) |
    | **Management** | More control over Kafka configuration | Fully managed, less control |
    | **Message Size** | Up to 1 MB (default), configurable | Up to 1 MB |

- **When to Use MSK vs Kinesis:**
    - **Use MSK if:**
        - You already use Kafka and want to migrate to AWS
        - You need Kafka-specific features (exactly-once semantics, Kafka Connect, Kafka Streams)
        - You want to use existing Kafka tools and ecosystem
    - **Use Kinesis if:**
        - You're building a new AWS-native application
        - You want simpler management (no Kafka expertise needed)
        - You need tight integration with other AWS services (Lambda, Firehose)

- **Integration:**
    - **MSK Connect:** Stream data to/from S3, DynamoDB, RDS, etc.
    - **Lambda:** Process Kafka messages with Lambda functions
    - **Kinesis Data Analytics:** Analyze Kafka data with SQL or Apache Flink
    - **Glue Schema Registry:** Manage schemas for Kafka messages
    - **IAM:** Fine-grained access control for Kafka topics

- **Exam Tips / What to Remember:**
    - **MSK is AWS-managed Apache Kafka** (use when scenario mentions "Kafka")
    - Exam keyword: **"Migrate existing Kafka workloads"** or **"Kafka-compatible"** = MSK
    - **Alternative to Kinesis Data Streams** for streaming data
    - **MSK Serverless** automatically scales capacity (no manual provisioning)
    - **MSK Connect** simplifies integration with external systems
    - **Kinesis is simpler and more AWS-native**, **MSK is for Kafka users**
    - Can have **unlimited consumers** (unlike Kinesis' 5 consumers per shard limit)
    - Supports **exactly-once semantics** (Kafka feature)
