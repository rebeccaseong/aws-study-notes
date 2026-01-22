# Comparison: SQS vs SNS vs EventBridge vs MQ

Quick reference for choosing the right messaging and event service.

---

## High-Level Decision Tree

```
What's your messaging pattern?

├─ Decouple producers and consumers (queue-based)
│  └─ SQS (Simple Queue Service)
│
├─ Pub/Sub (one message to many subscribers)
│  ├─ Simple fanout (no routing logic)
│  │  └─ SNS (Simple Notification Service)
│  └─ Event-driven with routing rules
│     └─ EventBridge
│
└─ Migrate from on-prem message broker (JMS, AMQP)
   └─ Amazon MQ
```

---

## Comparison Table

| Aspect | SQS | SNS | EventBridge | Amazon MQ |
|--------|-----|-----|-------------|-----------|
| **Pattern** | **Queue** (point-to-point) | **Pub/Sub** (fanout) | **Event bus** (routing) | **Broker** (JMS, AMQP) |
| **Messaging** | Producer → Queue → Consumer | Publisher → Topic → Subscribers | Event source → Bus → Targets | Producer → Broker → Consumer |
| **Routing** | No routing | No routing (all subscribers get message) | **Yes** (rule-based filtering) | Yes (broker logic) |
| **Subscribers** | **One consumer** processes message | **Multiple** subscribers | **Multiple** targets | Multiple consumers |
| **Delivery** | Pull (consumer polls) | **Push** (sends to endpoints) | **Push** (invokes targets) | Pull or Push |
| **Retention** | 1-14 days (default 4 days) | None (fire-and-forget) | Stored in event archive (optional) | Depends on broker config |
| **Use Case** | **Decouple** components, work queues | **Fanout** to multiple endpoints | **Event-driven** architectures | **Lift-and-shift** from on-prem |
| **Protocols** | SQS API | SNS API | EventBridge API | JMS, AMQP, MQTT, STOMP |
| **Cost** | $ (pay per request) | $ (pay per request) | $ (pay per event) | $$ (instance hours) |

---

## When to Use Each Service

### **Use SQS If:**

✅ Need **message queue** (point-to-point, one consumer)
✅ **Decouple** application components
✅ Want **guaranteed delivery** (message stays until processed)
✅ Need **dead-letter queue** (handle failed messages)
✅ Building **work queue** (distribute tasks to workers)

**Common Use Cases:**
- Order processing queue (Order Service → Queue → Fulfillment Service)
- Background job processing (resize images, send emails)
- Buffer between microservices (handle traffic spikes)
- Distributed task queue

**Queue Types:**
- **Standard:** At-least-once delivery, best-effort ordering, unlimited throughput
- **FIFO:** Exactly-once processing, strict ordering, 300 TPS (3000 with batching)

---

### **Use SNS If:**

✅ Need **pub/sub** pattern (one-to-many fanout)
✅ Send same message to **multiple subscribers**
✅ Want **push-based** delivery (not polling)
✅ Building **notification system** (email, SMS, mobile push)
✅ Simple fanout (no complex routing)

**Common Use Cases:**
- Application alerts (send to email + SMS + Lambda)
- Fanout to multiple SQS queues
- Mobile push notifications
- Webhook delivery
- Trigger Lambda functions on events

**Subscriber Types:**
- SQS, Lambda, HTTP/HTTPS endpoints, Email, SMS, Mobile push

---

### **Use EventBridge If:**

✅ Need **event-driven architecture**
✅ Want **event routing** based on content (filter rules)
✅ Integrating **third-party SaaS** (Shopify, Zendesk, etc.)
✅ Building **serverless workflows** (event → multiple targets)
✅ Need **event replay** or **event archive**

**Common Use Cases:**
- React to AWS service events (EC2 state change → Lambda)
- Schedule tasks (cron-like scheduling)
- Integrate SaaS applications (Shopify order → Lambda)
- Event-driven microservices (filter events, route to multiple targets)
- Complex event processing with filtering

**Key Features:**
- **Event Patterns:** Filter events based on content
- **Targets:** Lambda, Step Functions, SQS, SNS, Kinesis, etc.
- **Schema Registry:** Discover event schemas
- **Archive & Replay:** Store events, replay later

---

### **Use Amazon MQ If:**

✅ **Migrating from on-prem** message brokers (ActiveMQ, RabbitMQ)
✅ Need **industry-standard protocols** (JMS, AMQP, MQTT, STOMP)
✅ **Lift-and-shift** applications (no code changes)
✅ Need **advanced messaging features** (routing, transactions, priority queues)

**Common Use Cases:**
- Migrate existing ActiveMQ/RabbitMQ to AWS
- Legacy applications using JMS
- Applications requiring AMQP, MQTT protocols

❌ **Don't use if:** Building new AWS-native apps (use SQS/SNS instead - simpler, cheaper, fully managed)

---

## Detailed Comparison

### **SQS (Simple Queue Service)**

| Feature | Details |
|---------|---------|
| **Pattern** | Queue (point-to-point) |
| **Delivery** | At-least-once (Standard) or Exactly-once (FIFO) |
| **Ordering** | Best-effort (Standard) or Strict (FIFO) |
| **Throughput** | Unlimited (Standard), 300 TPS (FIFO) |
| **Retention** | 1-14 days (default 4 days) |
| **Message Size** | Up to 256 KB |
| **Visibility Timeout** | Hide message while being processed (30 sec default) |

**Key Concepts:**
- **Long Polling:** Reduce empty responses (wait up to 20 sec for message)
- **Dead-Letter Queue:** Send failed messages to separate queue after max retries
- **Delay Queue:** Delay delivery of messages (up to 15 min)

---

### **SNS (Simple Notification Service)**

| Feature | Details |
|---------|---------|
| **Pattern** | Pub/Sub (one-to-many fanout) |
| **Delivery** | Push (fire-and-forget) |
| **Retention** | None (doesn't store messages) |
| **Message Size** | Up to 256 KB |
| **Filtering** | Basic (subscription filter policies) |
| **FIFO** | Yes (FIFO topics for ordering) |

**Key Concepts:**
- **Fanout:** Publish once, deliver to multiple subscribers
- **Message Filtering:** Subscribers receive only messages matching filter policy
- **SNS → SQS:** Common pattern (fanout to multiple queues)

---

### **EventBridge**

| Feature | Details |
|---------|---------|
| **Pattern** | Event bus (event-driven) |
| **Event Sources** | AWS services, Custom apps, SaaS partners |
| **Routing** | **Rule-based** (filter on event content) |
| **Targets** | Lambda, Step Functions, SQS, SNS, Kinesis, etc. |
| **Archive** | Store events for replay |
| **Scheduling** | Cron-like event scheduling |

**Event Pattern Example:**
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

---

### **Amazon MQ**

| Feature | Details |
|---------|---------|
| **Pattern** | Broker (message broker) |
| **Protocols** | JMS, AMQP, MQTT, STOMP |
| **Brokers** | ActiveMQ, RabbitMQ |
| **Deployment** | Single-instance or Active/Standby |
| **Management** | Managed (but more control than SQS/SNS) |

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Decouple Order Service from Fulfillment Service | **SQS** |
| Send application alert to email + SMS + Lambda | **SNS** |
| React to EC2 state change (start Lambda when instance stops) | **EventBridge** |
| Migrate on-prem ActiveMQ to AWS | **Amazon MQ** |
| Distribute tasks to multiple worker instances | **SQS** |
| Fanout one message to 10 different SQS queues | **SNS** |
| Schedule Lambda to run every hour | **EventBridge** (cron) |
| Filter events and route to different targets | **EventBridge** (event patterns) |
| Need exactly-once message processing | **SQS FIFO** |
| Integrate with Shopify (SaaS event source) | **EventBridge** |

---

## Key Exam Tips

1. **SQS = Queue** (point-to-point, decouple components)
2. **SNS = Pub/Sub** (one-to-many fanout, push-based)
3. **EventBridge = Event-driven** (routing, filtering, scheduling)
4. **Amazon MQ = Lift-and-shift** (JMS, AMQP, migrate from on-prem)
5. **Fanout pattern:** SNS → multiple SQS queues
6. **Event filtering and routing:** EventBridge (not SNS)
7. **Schedule tasks (cron):** EventBridge
8. **Decouple with guaranteed delivery:** SQS
9. **Exactly-once processing:** SQS FIFO
10. **New AWS-native apps:** Use SQS/SNS (not Amazon MQ)

---

## Common Patterns

### **SNS + SQS (Fanout)**
- SNS topic → Multiple SQS queues
- Use case: One order event → Inventory, Billing, Shipping queues

### **SQS + Lambda**
- SQS queue → Lambda polls and processes
- Use case: Background job processing

### **EventBridge + Lambda**
- AWS event → EventBridge → Lambda
- Use case: React to AWS service events (EC2 stop → trigger cleanup)

### **SQS + Dead-Letter Queue**
- Main queue → Failed messages → DLQ
- Use case: Handle poison messages, retry failed processing

---

## Summary

| Service | Pattern | Best For |
|---------|---------|----------|
| **SQS** | Queue (point-to-point) | Decouple components, work queues |
| **SNS** | Pub/Sub (fanout) | Notifications, fanout to multiple endpoints |
| **EventBridge** | Event bus (routing) | Event-driven architectures, filtering |
| **Amazon MQ** | Message broker | Lift-and-shift from on-prem (JMS, AMQP) |
