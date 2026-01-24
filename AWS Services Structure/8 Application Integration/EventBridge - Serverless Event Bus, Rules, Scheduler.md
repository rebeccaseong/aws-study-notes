# EventBridge - Serverless Event Bus, Rules, Scheduler

- **What is it? (1-Sentence Pitch):** A serverless event bus that makes it easy to connect applications together using data from your own apps, third-party SaaS applications, and AWS services.
- **Core Use Cases:** Building event-driven architectures where services react to events from many different sources, routing events to multiple targets, scheduling tasks, integrating SaaS applications.
- **Key Distinction:** An evolution of CloudWatch Events. It uses the same API but adds advanced features like schema registry and integrations with SaaS partners. Think of it as the central nervous system for event-driven applications.

---

## What EventBridge Does

EventBridge enables event-driven architectures by:
- **Routing events** from sources to targets based on rules
- **Filtering events** with pattern matching
- **Scheduling** tasks (cron-like functionality)
- **Integrating** with SaaS partners (Shopify, Zendesk, Datadog, etc.)
- **Transforming** event data before delivery
- **Archiving & replaying** events

**Key Point:** EventBridge is the **successor to CloudWatch Events** with additional features.

---

## EventBridge vs CloudWatch Events

| Feature | CloudWatch Events | EventBridge |
|---------|-------------------|-------------|
| **API** | Same API | Same API (backward compatible) |
| **Event Bus** | Default event bus only | Default + Custom + Partner event buses |
| **SaaS Integration** | ❌ No | ✅ Yes (Shopify, Zendesk, Datadog, etc.) |
| **Schema Registry** | ❌ No | ✅ Yes (discover, version, generate code bindings) |
| **Archive & Replay** | ❌ No | ✅ Yes |
| **Cross-Account Events** | Limited | Enhanced (event bus policies) |
| **Recommendation** | Legacy | ✅ **Use EventBridge for all new applications** |

**Bottom Line:** EventBridge is the modern replacement for CloudWatch Events. Use EventBridge for new projects.

---

## Core Concepts

```
┌─────────────────────────────────────────────────────┐
│                  EventBridge                        │
├──────────────┬──────────────┬──────────────────────┤
│ Event Source │  Event Bus   │  Rules → Targets    │
├──────────────┼──────────────┼──────────────────────┤
│ AWS Services │   Default    │  Lambda              │
│ SaaS Partners│   Custom     │  Step Functions      │
│ Custom Apps  │   Partner    │  SQS, SNS, Kinesis   │
└──────────────┴──────────────┴──────────────────────┘
```

---

## 1. Event Buses

**What They Are:** Channels that receive events from sources and route them to targets based on rules.

### **Event Bus Types**

| Type | Description | Use Case |
|------|-------------|----------|
| **Default Event Bus** | Automatically receives events from AWS services | AWS service events (EC2 state change, S3 object created) |
| **Custom Event Bus** | Receives events from your own applications | Application-to-application communication within your account |
| **Partner Event Bus** | Receives events from SaaS partners (Shopify, Zendesk, etc.) | Integrate third-party SaaS into your workflows |

---

### **Default Event Bus**

- Automatically exists in every AWS account
- Receives events from **all AWS services**
- Cannot be deleted

**Example Sources:**
- EC2: Instance state changes, spot instance interruptions
- S3: Object created, deleted
- CodePipeline: Pipeline execution state changes
- Auto Scaling: Scaling events
- CloudTrail: API calls

---

### **Custom Event Bus**

- Create your own event bus for custom applications
- Isolate events by application or team
- Cross-account access via resource-based policies

**Use Cases:**
- Multi-tenant applications (one event bus per tenant)
- Microservices communication
- Cross-account event routing

**How to Create:**
```bash
aws events create-event-bus --name my-custom-bus
```

---

### **Partner Event Bus**

- Automatically created when you connect a SaaS partner
- Receives events from third-party applications

**Supported Partners:**
- **Shopify:** Order created, product updated
- **Zendesk:** Ticket created, ticket updated
- **Datadog:** Alerts triggered
- **Auth0:** User login, registration
- **PagerDuty:** Incident created
- **MongoDB:** Database changes

**Use Case:** Trigger Lambda when Shopify receives a new order

---

## 2. Events

**What They Are:** JSON objects that represent a change in state.

### **Event Structure**

```json
{
  "version": "0",
  "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
  "detail-type": "EC2 Instance State-change Notification",
  "source": "aws.ec2",
  "account": "123456789012",
  "time": "2025-01-24T10:00:00Z",
  "region": "us-east-1",
  "resources": [
    "arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0"
  ],
  "detail": {
    "instance-id": "i-1234567890abcdef0",
    "state": "terminated"
  }
}
```

**Key Fields:**
- **source:** Where the event came from (e.g., `aws.ec2`, `my-app`)
- **detail-type:** Type of event (e.g., `EC2 Instance State-change Notification`)
- **detail:** Event-specific data (flexible JSON object)
- **resources:** ARNs of affected resources
- **time:** When the event occurred

---

### **Publishing Custom Events**

You can publish your own events to EventBridge:

```python
import boto3
import json
from datetime import datetime

eventbridge = boto3.client('events')

response = eventbridge.put_events(
    Entries=[
        {
            'Source': 'my-app',
            'DetailType': 'Order Placed',
            'Detail': json.dumps({
                'orderId': '12345',
                'customerId': '67890',
                'amount': 99.99
            }),
            'EventBusName': 'my-custom-bus'
        }
    ]
)
```

**Use Cases:**
- Application events (user registered, order placed)
- Microservice communication
- Custom workflows

---

## 3. Rules

**What They Are:** Match incoming events and route them to targets based on event patterns.

### **Rule Components**

| Component | Description |
|-----------|-------------|
| **Event Pattern** | Filter that determines which events match the rule |
| **Targets** | Where matched events are sent (Lambda, SQS, SNS, Step Functions, etc.) |
| **Event Bus** | Which event bus the rule monitors (default, custom, partner) |
| **State** | Enabled or disabled |

---

### **Event Patterns**

Event patterns define which events trigger the rule using JSON matching.

**Example 1: Match EC2 instance terminations**
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

**Example 2: Match all S3 object creations**
```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"]
}
```

**Example 3: Match custom application events**
```json
{
  "source": ["my-app"],
  "detail-type": ["Order Placed"],
  "detail": {
    "amount": [{"numeric": [">", 100]}]
  }
}
```

**Pattern Matching Features:**
- **Exact match:** `"state": ["terminated"]`
- **Prefix match:** `"detail-type": [{"prefix": "AWS API Call"}]`
- **Numeric comparison:** `{"numeric": [">", 100]}`
- **Exists:** `"error": [{"exists": true}]`
- **Multiple values (OR):** `"state": ["pending", "running"]`

---

### **Targets**

When an event matches a rule, EventBridge can send it to multiple targets (up to 5 per rule).

**Supported Targets:**

| Target | Use Case |
|--------|----------|
| **Lambda** | Execute function to process event |
| **Step Functions** | Start workflow |
| **SQS** | Queue event for processing |
| **SNS** | Fan out to multiple subscribers |
| **Kinesis Data Streams** | Stream events for real-time analytics |
| **Kinesis Data Firehose** | Deliver to S3, Redshift, OpenSearch |
| **ECS Task** | Run containerized task |
| **Batch Job** | Run batch processing job |
| **Systems Manager** | Run automation document |
| **EC2 Action** | Reboot, terminate, stop instance |
| **API Gateway** | Invoke REST API |
| **Another Event Bus** | Cross-account or cross-region routing |

**Example Rule with Multiple Targets:**
```
Event: EC2 instance terminated
Targets:
  1. Lambda (send notification to Slack)
  2. SQS (queue for audit processing)
  3. SNS (email ops team)
```

---

### **Input Transformation**

Transform event data before sending to target.

**Use Cases:**
- Extract specific fields from event
- Reformat data for target
- Add static values

**Example:** Extract instance ID and state from EC2 event
```json
{
  "InputPathsMap": {
    "instance": "$.detail.instance-id",
    "state": "$.detail.state"
  },
  "InputTemplate": "{\"instance\": \"<instance>\", \"state\": \"<state>\"}"
}
```

**Result sent to target:**
```json
{
  "instance": "i-1234567890abcdef0",
  "state": "terminated"
}
```

---

## 4. EventBridge Scheduler

**What It Is:** Cron-like scheduling for AWS services (successor to CloudWatch Events scheduled rules).

### **Schedule Types**

| Type | Description | Use Case |
|------|-------------|----------|
| **Rate-based** | Run every X minutes/hours/days | Run Lambda every 5 minutes |
| **Cron-based** | Run at specific times (cron expression) | Run backup at 2 AM daily |
| **One-time** | Run once at a specific time | Schedule future task |

---

### **Rate Expressions**

**Syntax:** `rate(value unit)`

**Examples:**
- `rate(5 minutes)` - Every 5 minutes
- `rate(1 hour)` - Every hour
- `rate(7 days)` - Every 7 days

**Use Case:** Health check API endpoint every 5 minutes

---

### **Cron Expressions**

**Syntax:** `cron(minutes hours day-of-month month day-of-week year)`

**Examples:**
- `cron(0 2 * * ? *)` - Daily at 2:00 AM UTC
- `cron(0 12 * * MON-FRI *)` - Weekdays at noon
- `cron(0 0 1 * ? *)` - First day of month at midnight
- `cron(0/5 * * * ? *)` - Every 5 minutes

**Use Case:** Daily backup at 2 AM, weekly report on Mondays

**Important:** EventBridge uses **UTC** timezone (no timezone conversion)

---

### **Flexible Time Windows**

EventBridge Scheduler allows flexible execution within a time window:

**Example:**
- Schedule: Daily at 2 AM
- Flexible window: 15 minutes
- **Result:** Task runs sometime between 2:00 AM - 2:15 AM (exact time varies)

**Use Case:** Distribute load, avoid spikes

---

## 5. Common Integration Patterns

### **Pattern 1: AWS Service Event → Lambda**

**Use Case:** Automatically respond to AWS service events

```
S3 Object Created → EventBridge → Lambda (process file)
EC2 Terminated → EventBridge → Lambda (clean up resources)
```

**Example Rule:**
```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["my-uploads-bucket"]
    }
  }
}
```
**Target:** Lambda function to resize image

---

### **Pattern 2: Custom App Event → Multiple Targets**

**Use Case:** Fan out application events to multiple consumers

```
Order Placed (custom event)
  → Lambda (send confirmation email)
  → SQS (queue for fulfillment)
  → SNS (notify warehouse)
  → Step Functions (start order workflow)
```

---

### **Pattern 3: SaaS Partner → Workflow**

**Use Case:** Integrate third-party SaaS into your workflows

```
Shopify: New Order → EventBridge → Step Functions
  → Lambda (validate order)
  → Lambda (process payment)
  → Lambda (update inventory)
  → SNS (notify customer)
```

---

### **Pattern 4: Scheduled Tasks**

**Use Case:** Cron-like scheduling

```
Schedule: Every day at 2 AM (cron(0 2 * * ? *))
  → Lambda (backup database)
  → Lambda (generate reports)
  → Step Functions (cleanup old data)
```

---

### **Pattern 5: Cross-Account Event Routing**

**Use Case:** Send events from one account to another

```
Production Account (Event Source)
  → EventBridge Rule → Custom Event Bus in Audit Account
    → Lambda in Audit Account (log event)
```

**Setup:**
1. Create custom event bus in Audit Account
2. Add resource policy to allow Production Account to send events
3. Create rule in Production Account targeting Audit Account event bus

---

## 6. Schema Registry

**What It Is:** Discover, create, and manage schemas for events.

### **Features**

- **Schema Discovery:** Automatically infer schemas from events
- **Schema Versioning:** Track schema changes over time
- **Code Bindings:** Generate code for Java, Python, TypeScript

### **Use Cases**

- Understand event structure
- Generate code to work with events
- Validate events against schema
- Track schema evolution

**Example:**
1. EventBridge discovers `Order Placed` event schema
2. Generate Python code to parse event
3. Import generated class in Lambda function

---

## 7. Archive & Replay

**What It Is:** Store events and replay them later for testing or disaster recovery.

### **Archive**

**Features:**
- Store all events or filtered events
- Set retention period (unlimited or time-limited)
- Encrypted at rest

**Use Cases:**
- Compliance (retain events for auditing)
- Debugging (replay events to reproduce issues)
- Testing (replay events against new targets)

---

### **Replay**

**How It Works:**
1. Select time range from archive
2. Choose event bus to replay to
3. EventBridge resends events

**Use Cases:**
- Test new Lambda function with real production events
- Recover from processing failures
- Reprocess events after bug fix

**Exam Tip:** Archive & Replay is unique to EventBridge (not available in CloudWatch Events)

---

## 8. EventBridge Pipes

**What It Is:** Point-to-point integration between event sources and targets with built-in filtering and enrichment.

**Difference from EventBridge (Event Bus):**
- **Event Bus:** Many sources → Event Bus → Many targets (pub/sub)
- **Pipes:** One source → Filter/Transform → One target (point-to-point)

**Use Cases:**
- Stream DynamoDB changes to EventBridge
- Filter Kinesis stream and send to Step Functions
- Enrich SQS messages before sending to Lambda

**Exam Tip:** Pipes are for **point-to-point** integrations, Event Bus is for **pub/sub**

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| React to EC2 instance termination | **EventBridge rule** (match EC2 state change) → Lambda |
| Run Lambda every 5 minutes | **EventBridge Scheduler** (`rate(5 minutes)`) → Lambda |
| Daily backup at 2 AM | **EventBridge Scheduler** (`cron(0 2 * * ? *)`) → Lambda |
| Integrate Shopify orders into AWS | **EventBridge Partner Event Bus** (Shopify) → Lambda/Step Functions |
| Fan out S3 events to multiple targets | **EventBridge rule** → Lambda + SQS + SNS |
| Send events to another AWS account | **EventBridge rule** → Custom event bus in target account |
| Archive events for compliance | **EventBridge Archive** (retain all events) |
| Replay events for testing | **EventBridge Replay** (select time range) |
| Discover event schema for code generation | **EventBridge Schema Registry** |
| Filter events before processing | **Event Pattern** (match specific fields) |
| Transform event data before sending to target | **Input Transformation** |
| Point-to-point integration (DynamoDB → Lambda) | **EventBridge Pipes** |

---

## EventBridge vs SNS vs SQS

| Feature | EventBridge | SNS | SQS |
|---------|-------------|-----|-----|
| **Pattern** | Event bus (routing + filtering) | Pub/Sub (fanout) | Queue (point-to-point) |
| **Filtering** | ✅ Advanced (event patterns) | Basic (subscription filters) | ❌ No filtering |
| **Routing** | ✅ Rule-based | No routing (all subscribers) | No routing |
| **Targets** | 18+ AWS services | SQS, Lambda, HTTP, Email, SMS | Consumer polls queue |
| **Scheduling** | ✅ Yes (cron, rate) | ❌ No | ❌ No |
| **SaaS Integration** | ✅ Yes (partner event buses) | ❌ No | ❌ No |
| **Schema Registry** | ✅ Yes | ❌ No | ❌ No |
| **Archive & Replay** | ✅ Yes | ❌ No | ❌ No (retention only) |
| **Use Case** | Event-driven architectures | Simple fanout | Decouple components |

**Bottom Line:**
- **EventBridge:** Event-driven architectures with routing, filtering, scheduling
- **SNS:** Simple pub/sub fanout
- **SQS:** Work queues, decouple components

---

## Key Exam Tips

1. **EventBridge = Successor to CloudWatch Events** (same API, more features)
2. **Event Buses:** Default (AWS services), Custom (your apps), Partner (SaaS)
3. **Rules:** Match events with patterns, route to targets (up to 5 per rule)
4. **Event Patterns:** JSON-based filtering (exact match, prefix, numeric, exists)
5. **Targets:** Lambda, Step Functions, SQS, SNS, Kinesis, ECS, Batch, etc.
6. **Scheduling:** Cron (`cron(0 2 * * ? *)`) or Rate (`rate(5 minutes)`)
7. **Input Transformation:** Extract, reformat event data before sending to target
8. **Schema Registry:** Discover schemas, generate code bindings
9. **Archive & Replay:** Store events, replay for testing/recovery
10. **Pipes:** Point-to-point integrations (source → filter → target)
11. **Cross-Account:** Send events to another account's custom event bus
12. **SaaS Partners:** Shopify, Zendesk, Datadog, Auth0, PagerDuty, etc.
13. **Cron Timezone:** Always UTC (no automatic timezone conversion)
14. **Fanout:** Use EventBridge or SNS (not SQS)
15. **"Event-driven architecture"** → EventBridge (not SNS or SQS)

---

## Best Practices

### **1. Use Event Patterns for Filtering**
- Filter events in EventBridge (not in Lambda)
- Reduces invocations, lowers cost

### **2. Limit Targets per Rule**
- Max 5 targets per rule
- Create multiple rules if needed

### **3. Use Dead-Letter Queues**
- Configure DLQ for failed target invocations
- Capture events that couldn't be delivered

### **4. Enable Archive for Critical Events**
- Archive events for compliance, auditing
- Enable replay for disaster recovery

### **5. Use Input Transformation**
- Send only necessary data to targets
- Reduce payload size, improve performance

### **6. Monitor with CloudWatch**
- Track failed invocations
- Set alarms for errors

---

## Cost Optimization

**Pricing:**
- **Events published:** $1.00 per million events (first 1M free per month)
- **Targets invoked:** Free (you pay for target service, e.g., Lambda invocation)
- **Schema Registry:** $0.10 per schema per month
- **Archive:** $0.10 per GB per month
- **Replay:** $0.10 per GB replayed

**Optimization Tips:**
- Use event patterns to filter (avoid unnecessary target invocations)
- Batch events when possible
- Use SQS as target for buffering (cheaper than direct Lambda invocations)

---

## Summary

**EventBridge** is AWS's **serverless event bus** for building event-driven architectures:

| Component | Purpose |
|-----------|---------|
| **Event Buses** | Receive events from AWS, SaaS partners, custom apps |
| **Rules** | Match events with patterns, route to targets |
| **Targets** | Lambda, Step Functions, SQS, SNS, Kinesis, ECS, etc. |
| **Scheduler** | Cron/rate-based task scheduling |
| **Schema Registry** | Discover schemas, generate code |
| **Archive & Replay** | Store and replay events |
| **Pipes** | Point-to-point integrations |

**Bottom Line:** EventBridge is the **modern replacement for CloudWatch Events**. Use it for event-driven architectures, scheduling, SaaS integrations, and routing events to multiple targets.
