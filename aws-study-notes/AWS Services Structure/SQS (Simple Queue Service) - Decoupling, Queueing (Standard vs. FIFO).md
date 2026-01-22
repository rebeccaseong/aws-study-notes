- **What is it? (1-Sentence Pitch):** A fully managed message queuing service that decouples application components, enabling them to scale independently and ensuring messages are not lost if a consumer fails.
- **Standard vs. FIFO: The Critical Choice**
    
    
    | **Characteristic** | **Standard Queue (Default)** | **FIFO (First-In, First-Out) Queue** |
    | --- | --- | --- |
    | **Ordering** | Best-Effort (no guarantee) | Strict (First-In, First-Out) |
    | **Delivery** | **At-Least-Once** | **Exactly-Once** Processing |
    | **Throughput** | Nearly Unlimited | High, but lower than Standard |
    | **Scenario** | High-throughput tasks where order doesn't matter (e.g., image thumbnail generation, logging).
    Requires **idempotent** consumers. (**Idempotent:** A process is idempotent if running it multiple times with the same input has no negative effect. This is **required** for Standard Queue consumers.) | Order is critical (e.g., financial transactions, user command processing). |
    - **FIFO Queue Throughput: With and Without Batching**
        - **Without Batching:** By default, a FIFO queue can process up to **300 messages per second**. This limit applies to send, receive, and delete operations (300 sends/sec, 300 receives/sec, etc.).
        - **With Batching:** To achieve higher throughput, you must batch messages. A single API call (e.g., SendMessageBatch) can include up to 10 messages.
            - **API Call Limit:** FIFO queues are limited to **300 API calls per second** for batched operations.
            - **Maximum Throughput:** By batching 10 messages per API call, you can achieve a throughput of up to **3,000 messages per second** (300 calls/second * 10 messages/call).
            - **High Throughput Mode:** For workloads exceeding 3,000 messages per second, you can enable a high-throughput mode, which can scale up to 30,000 messages per second by relaxing ordering within message groups.
        - **Calculation Example:** If a system needs to process 1,000 messages per second in strict order:
            - A batch size of 2 would require 500 API calls/sec (1000 / 2), which exceeds the 300 calls/sec limit.
            - A batch size of 4 would require 250 API calls/sec (1000 / 4), which is within the 300 calls/sec limit.
- **Key Configuration Parameters (For Resilience & Performance)**
    
    
    | **Visibility Timeout** | The time a message is "invisible" after a consumer reads it. | If your processing takes 3 minutes, set the timeout to > 3 minutes. If the timeout is too short (e.g., 30 seconds), another consumer will pick up the same message while the first is still working on it, causing duplicate processing. |
    | --- | --- | --- |
    | **Dead-Letter Queue (DLQ)** | A separate queue to capture messages that fail to be processed after a set number of retries (maxReceiveCount). | A "poison pill" message is causing your consumer to crash. To prevent it from blocking the entire queue and to allow for debugging, configure a DLQ. This is a best practice for **resilience**. |
    | **Long Polling** | A consumer can wait (up to 20 seconds) for a message to arrive instead of constantly making empty requests. | To reduce cost and increase the efficiency of your SQS consumers, enable Long Polling (ReceiveMessageWaitTimeSeconds > 0). |
- **The Most Common Architectural Patterns**
    
    
    |  | **How it Works** | **Why** |
    | --- | --- | --- |
    | **Serverless Processing (SQS + Lambda)** | Messages arrive in an SQS queue. AWS automatically triggers a Lambda function (via an Event Source Mapping) with a batch of messages. | The most common serverless pattern for reliable, scalable, decoupled task processing. You don't manage any servers. |
    | **Fan-Out (SNS → SQS)** | A single message is published to an SNS Topic. The topic pushes the message to multiple subscribed SQS queues. | • Use this when you need the **same event** to trigger multiple, different, independent processing workflows. |
- **SQS vs. The World: When to Choose What**
    
    
    | **Service** | **Primary Job** | **Model** | **Key Exam Scenario** |
    | --- | --- | --- | --- |
    | **SQS** | **Decouple Tasks** | Pull | "A web application needs to send tasks to a backend service. The service must be able to handle spikes in traffic without being overwhelmed." |
    | **SNS** | **Fan-out Notifications** | Push | "A new user signs up. You need to send a welcome email, notify the analytics team, and update the CRM system all at once." |
    | **Kinesis Data Streams** | **Real-time Data Stream** | Pull | "Ingest and process a continuous, ordered stream of data from thousands of IoT devices or application log files in real-time." |
    | **EventBridge** | **Event Routing / Bus** | Push | "Route events from various AWS services (or custom apps) to different targets based on the event's content. Orchestrate complex workflows." |
    - Pull & Push Model
        
        
        | **Aspect** | **Pull Model** | **Push Model** |
        | --- | --- | --- |
        | **Who is in control?** | **The Receiver**. It decides when to ask for data. | **The Sender**. It decides when to send data. |
        | **Analogy** | Checking your P.O. Box. | Getting mail delivered to your house. |
        | **Primary AWS Service** | **SQS** (You poll the queue)
        Your application (the "consumer") has to continuously ask the SQS queue, "Are there any new messages for me?" This is called "polling.” | **SNS** (It pushes to subscribers)
        You "subscribe" your service (like a Lambda function) to an SNS topic. When a message is published to the topic, SNS immediately **pushes** that message to your service. |