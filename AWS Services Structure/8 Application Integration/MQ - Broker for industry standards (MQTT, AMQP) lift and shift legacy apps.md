- **What is it? (1-Sentence Pitch):** Amazon MQ is a managed message broker service for Apache ActiveMQ and RabbitMQ that makes it easy to migrate existing messaging applications to the cloud without rewriting code.

- **Core Use Cases:**
    - Migrating on-premises message brokers (ActiveMQ, RabbitMQ) to AWS
    - Lift-and-shift migrations of legacy applications that use JMS, AMQP, MQTT, STOMP
    - Decoupling application components using industry-standard messaging protocols
    - Applications that require advanced messaging patterns (routing, priority queues, transactions)

- **Key Features & Concepts:**
    - **Managed Service:** AWS handles provisioning, patching, and maintenance
    - **Broker Types:**
        - **Apache ActiveMQ:** Supports JMS, NMS, AMQP, STOMP, MQTT, WebSocket
        - **RabbitMQ:** Supports AMQP 0-9-1, MQTT, STOMP
    - **Deployment Modes:**
        - **Single-Instance Broker:** One broker in one AZ (dev/test)
        - **Active/Standby Broker:** Two brokers across two AZs (high availability, production)
    - **Persistent Storage:** Messages stored on EFS or EBS (survive broker restarts)
    - **Multi-AZ Failover:** Active/Standby mode provides automatic failover
    - **VPC Deployment:** Brokers run within your VPC (private connectivity)

- **Amazon MQ vs SQS vs SNS:**
    | **Aspect** | **Amazon MQ** | **SQS** | **SNS** |
    |---|---|---|---|
    | **Protocols** | **JMS, AMQP, MQTT, STOMP** | AWS API only | AWS API only |
    | **Use Case** | **Migrating existing apps** (lift-and-shift) | **AWS-native** applications | Pub/Sub notifications |
    | **Management** | Managed broker (more control) | Fully serverless | Fully serverless |
    | **Cost** | **Higher** (instance-based) | **Lower** (pay-per-use) | **Lower** (pay-per-use) |
    | **Features** | Advanced routing, transactions | Simple queuing | Simple pub/sub |
    | **When to Use** | **Migrating from on-prem brokers** | New AWS-native apps | Fan-out notifications |

- **When to Use Amazon MQ:**
    - **Use Amazon MQ if:**
        - You're migrating existing applications that use ActiveMQ or RabbitMQ
        - You need industry-standard protocols (JMS, AMQP, MQTT, STOMP)
        - You require advanced messaging features (message routing, priority queues, transactions)
    - **Use SQS/SNS if:**
        - You're building new AWS-native applications
        - You want fully serverless, scalable messaging (no broker management)
        - You don't need advanced messaging features

- **Integration:**
    - **VPC:** Amazon MQ brokers run in your VPC
    - **EFS/EBS:** Persistent message storage
    - **CloudWatch:** Monitor broker metrics (CPU, memory, message count)
    - **CloudTrail:** Audit broker management actions
    - **IAM:** Control access to broker management
    - **On-Premises:** Connect via VPN or Direct Connect

- **Supported Messaging Patterns:**
    - **Point-to-Point (Queue):** One producer, one consumer (like SQS)
    - **Publish/Subscribe (Topic):** One producer, multiple consumers (like SNS)
    - **Request/Reply:** Synchronous communication pattern
    - **Message Routing:** Route messages based on headers or content

- **Exam Tips / What to Remember:**
    - **Amazon MQ = Managed message broker for ActiveMQ/RabbitMQ**
    - Exam scenario: **"Migrate on-premises ActiveMQ to AWS without code changes"** = Amazon MQ
    - Exam keyword: **"Lift-and-shift"** or **"JMS/AMQP/MQTT"** = Amazon MQ
    - **SQS/SNS are simpler and cheaper** for new AWS-native apps
    - **Amazon MQ is for migrating existing apps** that use industry-standard protocols
    - **Active/Standby deployment** provides high availability across two AZs
    - Amazon MQ is **NOT serverless** (you pay for broker instances)
    - Messages stored on **EFS (multi-AZ) or EBS (single-AZ)**
    - **Use SQS/SNS for new applications**, **use Amazon MQ for migrations**
    - Amazon MQ supports **transactions** and **advanced routing** (unlike SQS/SNS)
