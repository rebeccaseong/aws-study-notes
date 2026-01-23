# Step Functions - Visual workflow orchestration (State Machine)

- **What is it? (1-Sentence Pitch):** A serverless function orchestrator that makes it easy to sequence multiple AWS services into visual workflows.
- **Core Use Cases:** For complex, multi-step business processes that require state, retries, and branching logic (e.g., an e-commerce order processing workflow).
- **Integration:** Orchestrates services like Lambda, ECS, DynamoDB, and SNS into a reliable, stateful workflow.

---

## What Step Functions Does

Step Functions lets you build **workflows** (state machines) that coordinate multiple AWS services:

**Key Capabilities:**
- **Visual workflow designer** (drag-and-drop in console)
- **Declarative definition** (JSON state machine)
- **Built-in error handling** (retry, catch, fallback)
- **Native integrations** with 200+ AWS services
- **Execution history** (full audit trail)
- **State management** (no code to track workflow state)

**Use Case:** Instead of writing complex orchestration code in Lambda (with manual retry logic, state tracking, error handling), define your workflow declaratively in Step Functions.

---

## Step Functions vs SWF

| Aspect | Step Functions | SWF |
|--------|----------------|-----|
| **Recommended** | ✅ **Yes** (modern, preferred) | ❌ **No** (legacy service) |
| **Visual Workflow** | ✅ Yes (visual designer) | ❌ No (code-based) |
| **Orchestration** | Fully AWS-managed | You manage workflow logic |
| **Programming Model** | **Declarative** (JSON) | **Imperative** (write code) |
| **Error Handling** | Built-in | You implement |
| **Complexity** | **Low** (easy to use) | **High** (write lots of code) |

**Bottom Line:** Always use **Step Functions** for new workflows. SWF is legacy and not recommended.

---

## Key Concepts

### **1. State Machine**

A **state machine** is a workflow definition written in **Amazon States Language (JSON)**.

**Example State Machine:**
```json
{
  "Comment": "Order processing workflow",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ValidateOrder",
      "Next": "ChargeCard"
    },
    "ChargeCard": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ChargeCard",
      "Next": "FulfillOrder",
      "Retry": [
        {
          "ErrorEquals": ["PaymentGatewayTimeout"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["PaymentDeclined"],
          "Next": "RefundCustomer"
        }
      ]
    },
    "FulfillOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:Fulfill",
      "End": true
    },
    "RefundCustomer": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:Refund",
      "End": true
    }
  }
}
```

**Visual Representation:**
```
┌────────────────┐
│ ValidateOrder  │
└────────┬───────┘
         ↓
┌────────────────┐
│  ChargeCard    │ (retry 3x on timeout)
└────┬───────┬───┘
     ↓       ↓ (on PaymentDeclined)
┌──────┐  ┌──────────────┐
│Fulfill│  │RefundCustomer│
└──────┘  └──────────────┘
```

---

### **2. States**

States are the building blocks of a workflow. Step Functions supports 8 state types:

| State Type | Purpose | Example |
|------------|---------|---------|
| **Task** | Execute work (invoke Lambda, run ECS task, etc.) | Call Lambda function to validate order |
| **Choice** | Conditional branching (if/else logic) | If order total > $100, apply discount |
| **Parallel** | Execute branches in parallel | Process payment AND reserve inventory simultaneously |
| **Wait** | Delay execution (seconds, timestamp) | Wait 24 hours before sending reminder email |
| **Succeed** | End execution successfully | Order completed successfully |
| **Fail** | End execution with failure | Order validation failed |
| **Pass** | Pass input to output (no-op, useful for testing) | Pass order data to next state |
| **Map** | Iterate over array (process each item) | Process each item in shopping cart |

---

### **3. Task State (Most Important)**

The **Task** state executes work by invoking AWS services:

**Supported Services (200+):**
- **Lambda:** Invoke function
- **ECS/Fargate:** Run container task
- **Batch:** Run batch job
- **DynamoDB:** Put/Get/Update item
- **SNS:** Publish message
- **SQS:** Send message
- **Glue:** Start job
- **SageMaker:** Create training job
- **Step Functions:** Start nested workflow

**Task Integration Types:**

| Integration Type | How It Works | Use Case |
|------------------|--------------|----------|
| **Request-Response** | Wait for service to complete, return result | Lambda function (synchronous) |
| **Run a Job (.sync)** | Wait for job to complete, poll status | ECS task, Glue job, SageMaker training |
| **Callback (.waitForTaskToken)** | Wait for external callback (manual approval) | Human approval workflow |

---

### **4. Choice State (Branching Logic)**

**Choice** state adds conditional logic (if/else):

```json
{
  "Type": "Choice",
  "Choices": [
    {
      "Variable": "$.orderTotal",
      "NumericGreaterThan": 100,
      "Next": "ApplyDiscount"
    },
    {
      "Variable": "$.orderTotal",
      "NumericLessThanEquals": 100,
      "Next": "ChargeFullPrice"
    }
  ]
}
```

**Visual:**
```
┌─────────┐
│ Choice  │
└───┬─┬───┘
    │ │
    ↓ ↓ (orderTotal > 100)
ApplyDiscount
    │
    ↓ (orderTotal <= 100)
ChargeFullPrice
```

---

### **5. Parallel State (Execute in Parallel)**

**Parallel** state executes multiple branches simultaneously:

```json
{
  "Type": "Parallel",
  "Branches": [
    {
      "StartAt": "ProcessPayment",
      "States": { "ProcessPayment": { "Type": "Task", ... } }
    },
    {
      "StartAt": "ReserveInventory",
      "States": { "ReserveInventory": { "Type": "Task", ... } }
    }
  ],
  "Next": "FulfillOrder"
}
```

**Visual:**
```
    ┌──────────────┐
    │   Parallel   │
    └──┬───────┬───┘
       ↓       ↓
ProcessPayment  ReserveInventory
       ↓       ↓
    └──────────────┘
          ↓
    FulfillOrder
```

---

### **6. Map State (Iterate Over Array)**

**Map** state processes each item in an array:

```json
{
  "Type": "Map",
  "ItemsPath": "$.items",
  "Iterator": {
    "StartAt": "ProcessItem",
    "States": {
      "ProcessItem": {
        "Type": "Task",
        "Resource": "arn:aws:lambda:...:function:ProcessCartItem",
        "End": true
      }
    }
  },
  "Next": "CalculateTotal"
}
```

**Use Case:** Process each item in a shopping cart, resize each image in a batch

---

### **7. Wait State (Delay Execution)**

**Wait** state pauses execution:

```json
{
  "Type": "Wait",
  "Seconds": 86400,
  "Next": "SendReminderEmail"
}
```

**Use Cases:**
- Wait 24 hours before sending reminder
- Delay until specific timestamp (scheduled execution)
- Rate limiting (wait between API calls)

---

## Error Handling

Step Functions has **built-in error handling** (retry, catch, fallback):

### **Retry (Automatic Retries)**

```json
{
  "Type": "Task",
  "Resource": "arn:aws:lambda:...:function:ChargeCard",
  "Retry": [
    {
      "ErrorEquals": ["PaymentGatewayTimeout"],
      "IntervalSeconds": 2,
      "MaxAttempts": 3,
      "BackoffRate": 2.0
    }
  ],
  "Next": "FulfillOrder"
}
```

**How It Works:**
- **ErrorEquals:** Match specific errors (or `States.ALL` for all errors)
- **IntervalSeconds:** Initial wait time (2 seconds)
- **MaxAttempts:** Retry up to 3 times
- **BackoffRate:** Exponential backoff (2x each retry: 2s → 4s → 8s)

---

### **Catch (Error Handling)**

```json
{
  "Type": "Task",
  "Resource": "arn:aws:lambda:...:function:ChargeCard",
  "Catch": [
    {
      "ErrorEquals": ["PaymentDeclined"],
      "Next": "RefundCustomer"
    },
    {
      "ErrorEquals": ["States.ALL"],
      "Next": "HandleError"
    }
  ],
  "Next": "FulfillOrder"
}
```

**How It Works:**
- If error matches, transition to fallback state
- Can catch specific errors or all errors (`States.ALL`)

---

## Workflow Types

Step Functions offers two workflow types:

| Feature | Standard Workflows | Express Workflows |
|---------|-------------------|-------------------|
| **Max Duration** | **1 year** | **5 minutes** |
| **Execution Semantics** | Exactly-once | At-least-once |
| **Execution Rate** | 2,000/sec | 100,000/sec |
| **Execution History** | Full history retained | Sent to CloudWatch Logs (not retained by Step Functions) |
| **Pricing** | $0.025 per 1,000 state transitions | $1.00 per 1M executions |
| **Use Case** | **Long-running** workflows (hours to months) | **High-volume**, short-duration (IoT, streaming, microservices) |

### **Standard Workflows**

**Best For:**
- Long-running workflows (order processing, approval workflows)
- Need full execution history
- Exactly-once execution required

**Example:** E-commerce order processing (validate → charge → fulfill → notify)

---

### **Express Workflows**

**Best For:**
- High-throughput, short-duration workflows (<5 min)
- IoT data processing, streaming data pipelines
- API orchestration, microservices coordination

**Express Workflow Types:**
- **Synchronous:** Wait for result, return response (API Gateway integration)
- **Asynchronous:** Fire-and-forget (EventBridge integration)

**Example:** API Gateway → Step Functions Express (synchronous) → Lambda → DynamoDB → Return response

---

## Integration Patterns

### **Pattern 1: Lambda Orchestration**

**Use Case:** Coordinate multiple Lambda functions

```
ValidateLambda → TransformLambda → LoadLambda
```

**Benefits:**
- No code to orchestrate (Step Functions handles it)
- Built-in retry and error handling
- Visual workflow representation

---

### **Pattern 2: Human Approval Workflow**

**Use Case:** Wait for manual approval before proceeding

```
SubmitRequest → WaitForApproval (.waitForTaskToken) → ExecuteRequest
```

**How It Works:**
1. Step Functions pauses at `WaitForApproval`
2. Sends task token to external system (email, Slack, etc.)
3. Approver clicks "Approve" (calls Step Functions API with task token)
4. Workflow resumes

---

### **Pattern 3: ECS Task Orchestration**

**Use Case:** Run containerized batch jobs

```
StartECSTask (.sync) → ProcessResults → NotifyComplete
```

**How It Works:**
- Step Functions starts ECS task
- Waits for task to complete (`.sync` integration)
- Proceeds to next state

---

### **Pattern 4: Saga Pattern (Distributed Transactions)**

**Use Case:** Implement compensating transactions

```
ReserveInventory → ChargeCard → ShipOrder
     ↓ (fail)         ↓ (fail)      ↓ (fail)
ReleaseInventory ← RefundCard ← CancelShipment
```

**How It Works:**
- Each step has a compensating action
- If any step fails, execute rollback chain

---

## Common Use Cases

| Use Case | Why Step Functions |
|----------|-------------------|
| **ETL Pipeline** | Orchestrate extract → transform → load steps with retry logic |
| **Order Processing** | Validate → charge → fulfill → notify (with error handling) |
| **Machine Learning Pipeline** | Preprocess data → train model → evaluate → deploy |
| **Human Approval Workflow** | Submit request → wait for approval → execute (callback integration) |
| **Batch Processing** | Iterate over large dataset, process each item (Map state) |
| **Microservices Orchestration** | Coordinate multiple microservices in sequence or parallel |

---

## Integration with Other Services

### **Step Functions + Lambda**
- Most common pattern (orchestrate Lambda functions)
- Step Functions handles retry, state, branching

### **Step Functions + ECS/Fargate**
- Run long-running containerized tasks
- `.sync` integration waits for task completion

### **Step Functions + EventBridge**
- Trigger workflows from events (S3 upload, CloudWatch alarm, etc.)
- Step Functions publishes events to EventBridge (workflow started, completed, failed)

### **Step Functions + API Gateway**
- Synchronous Express Workflows (API responds with workflow result)
- Use Case: Multi-step API orchestration

### **Step Functions + DynamoDB**
- Direct integration (no Lambda required)
- Read/write data as part of workflow

---

## Monitoring and Debugging

### **Execution History**

Step Functions provides **full execution history**:
- Visual timeline of each state
- Input/output of each state
- Errors and retries
- Execution duration

**Use Case:** Debug failed workflows (see exactly where and why it failed)

---

### **CloudWatch Integration**

- **Metrics:** Execution count, success rate, duration
- **Logs:** Express Workflows send logs to CloudWatch
- **Alarms:** Alert on failed executions

---

### **X-Ray Integration**

- Trace Step Functions executions
- Visualize service calls (Lambda, DynamoDB, etc.)
- Identify bottlenecks

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Orchestrate Lambda functions in ETL pipeline | **Step Functions Standard** (long-running) |
| Multi-step order workflow (new application) | **Step Functions Standard** |
| Visual workflow designer required | **Step Functions** (not SWF) |
| Human approval workflow | **Step Functions** with `.waitForTaskToken` |
| High-throughput, short-duration orchestration (<5 min) | **Step Functions Express** |
| Need built-in retry and error handling | **Step Functions** |
| Orchestrate ECS tasks in multi-step workflow | **Step Functions** with `.sync` integration |
| Iterate over array of items | **Step Functions Map** state |
| Wait 24 hours before next step | **Step Functions Wait** state |
| Conditional branching (if/else logic) | **Step Functions Choice** state |

---

## Key Exam Tips

1. **Step Functions = Modern workflow orchestration** (use for all new workflows)
2. **SWF = Legacy** (don't use for new applications)
3. **Visual workflow designer** available in console
4. **Built-in error handling** (retry, catch, fallback)
5. **200+ AWS service integrations** (Lambda, ECS, DynamoDB, SNS, etc.)
6. **Standard Workflows:** Long-running (up to 1 year), exactly-once
7. **Express Workflows:** High-volume (<5 min), at-least-once
8. **Task state:** Execute work (invoke Lambda, run ECS task, etc.)
9. **Choice state:** Conditional branching (if/else)
10. **Parallel state:** Execute branches simultaneously
11. **Map state:** Iterate over array
12. **Wait state:** Delay execution
13. **`.sync` integration:** Wait for job to complete (ECS, Glue, SageMaker)
14. **`.waitForTaskToken`:** Wait for external callback (human approval)
15. **Amazon States Language:** JSON-based workflow definition

---

## Best Practices

### **1. Use Standard Workflows for Long-Running Processes**
- Order processing, approval workflows, batch jobs

### **2. Use Express Workflows for High-Volume, Short Tasks**
- IoT data processing, API orchestration, streaming pipelines

### **3. Leverage Built-in Integrations**
- Direct integrations (DynamoDB, SNS, SQS) instead of Lambda wrappers

### **4. Implement Proper Error Handling**
- Add `Retry` for transient errors (timeouts, rate limits)
- Add `Catch` for business logic errors (payment declined)

### **5. Use Map State for Batch Processing**
- Process large datasets efficiently
- Parallel execution for better performance

### **6. Monitor Execution History**
- Review failed executions
- Identify bottlenecks and optimize

### **7. Use Nested Workflows for Reusability**
- Break complex workflows into smaller, reusable state machines

---

## Costs

| Item | Cost |
|------|------|
| **Standard Workflows** | $0.025 per 1,000 state transitions |
| **Express Workflows (Synchronous)** | $1.00 per 1M requests + $0.00001667 per GB-second |
| **Express Workflows (Asynchronous)** | $1.00 per 1M requests + $0.00001667 per GB-second |

**Cost Optimization:**
- Use Express Workflows for high-volume, short tasks
- Minimize state transitions in Standard Workflows
- Use direct service integrations (avoid Lambda wrappers)

**Example:**
- 1M Standard Workflow executions, 10 states each = 10M state transitions = $250
- 1M Express Workflow executions (5 min, 512MB) = $1 + $6.25 = $7.25

---

## Summary

**Step Functions** is AWS's **modern workflow orchestration service**:

| Feature | Description |
|---------|-------------|
| **Purpose** | Coordinate multiple AWS services into visual workflows |
| **Definition** | Declarative JSON state machine (Amazon States Language) |
| **Visual Designer** | Drag-and-drop workflow builder in console |
| **Error Handling** | Built-in retry, catch, fallback |
| **Integrations** | 200+ AWS services (Lambda, ECS, DynamoDB, SNS, SQS, etc.) |
| **Workflow Types** | Standard (long-running, 1 year) vs Express (high-volume, 5 min) |
| **State Types** | Task, Choice, Parallel, Map, Wait, Succeed, Fail, Pass |
| **Use Cases** | ETL pipelines, order processing, ML pipelines, human approvals |

**Bottom Line:** Step Functions is the **recommended service** for all new workflow orchestration needs. It's serverless, visual, and has built-in integrations with AWS services.
