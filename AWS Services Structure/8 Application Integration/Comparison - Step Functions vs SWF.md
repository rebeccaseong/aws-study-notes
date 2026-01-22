# Comparison: Step Functions vs SWF

Quick reference for choosing between AWS orchestration services.

---

## Quick Decision

**For New Applications:** Use **Step Functions** (modern, easier, serverless)
**For Legacy Applications:** Use **SWF** only if already using it (deprecated for new apps)

---

## Comparison Table

| Aspect | Step Functions | SWF |
|--------|----------------|-----|
| **Recommended** | ✅ **Yes** (modern, preferred) | ❌ **No** (legacy service) |
| **Visual Workflow** | ✅ **Yes** (visual designer in console) | ❌ No (code-based) |
| **Orchestration** | Fully AWS-managed | **You manage workflow logic** |
| **Max Duration** | **1 year** (Standard), 5 minutes (Express) | **1 year** |
| **Execution History** | Automatic (visual timeline) | **You track manually** |
| **Integration** | **Native** with AWS services (Lambda, ECS, SNS, etc.) | Manual (via code) |
| **Programming Model** | **Declarative** (JSON state machine) | **Imperative** (write code) |
| **Error Handling** | Built-in (retry, catch, fallback) | **You implement** |
| **Human Tasks** | ✅ Yes (wait for callback) | ✅ Yes (activity workers) |
| **Complexity** | **Low** (easy to use) | **High** (write lots of code) |
| **Use Case** | **Modern serverless workflows** | **Legacy apps**, human-driven processes |
| **Cost** | $ (pay per state transition) | $ (pay per workflow execution) |

---

## When to Use Each Service

### **Use Step Functions If:**

✅ Building **new workflows** (recommended for all new applications)
✅ Need **visual workflow** (easy to design and understand)
✅ Want **serverless orchestration** (no code to manage workflow logic)
✅ Orchestrating **AWS services** (Lambda, ECS, SNS, SQS, DynamoDB, etc.)
✅ Need **built-in error handling** (retry, catch, fallback)
✅ Want **automatic execution history** and **visual monitoring**

**Common Use Cases:**
- Orchestrate Lambda functions (ETL pipeline: extract → transform → load)
- Order processing workflow (validate → charge → fulfill → notify)
- Machine learning pipelines (preprocess → train → evaluate → deploy)
- Human approval workflows (submit request → wait for approval → execute)
- ECS/Fargate task orchestration

---

### **Use SWF If:**

✅ **Already using SWF** (maintain existing workflows)
✅ Need **external signals** to direct workflow (not event-driven)
✅ Need **long-running workflows with human tasks** (multi-day/multi-week)
✅ Workflow involves **non-AWS systems** (on-prem, third-party)

❌ **Don't use for new applications** - Step Functions is the modern replacement

**Common Use Cases (Legacy):**
- Order fulfillment with human intervention (warehouse workers)
- Multi-step business processes spanning days/weeks
- Workflows requiring external decision-making

---

## Detailed Comparison

### **Step Functions**

| Feature | Details |
|---------|---------|
| **Workflow Definition** | JSON state machine (declarative) |
| **Visual Designer** | **Yes** (drag-and-drop in console) |
| **Execution Types** | Standard (1 year max), Express (5 min max) |
| **Built-in Integrations** | **200+ AWS services** (Lambda, ECS, SNS, SQS, DynamoDB, etc.) |
| **Error Handling** | Automatic retry, catch, fallback states |
| **Execution History** | Automatic (visual timeline in console) |
| **Pricing** | $0.025 per 1,000 state transitions |

**State Types:**
- **Task:** Execute work (invoke Lambda, run ECS task)
- **Choice:** Conditional branching (if/else logic)
- **Parallel:** Execute branches in parallel
- **Wait:** Delay execution (seconds, timestamp)
- **Succeed/Fail:** End execution with success or failure

**Example State Machine:**
```json
{
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ValidateOrder",
      "Next": "ChargeCard"
    },
    "ChargeCard": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ChargeCard",
      "Next": "FulfillOrder"
    },
    "FulfillOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:Fulfill",
      "End": true
    }
  }
}
```

---

### **SWF (Simple Workflow Service)**

| Feature | Details |
|---------|---------|
| **Workflow Definition** | Code-based (you write workflow logic) |
| **Visual Designer** | **No** (code only) |
| **Execution Types** | Workflow executions (up to 1 year) |
| **Built-in Integrations** | **None** (you write code to integrate) |
| **Error Handling** | **You implement** (write code) |
| **Execution History** | **You track** (SWF provides task list) |
| **Pricing** | $0.00001 per workflow execution |

**Architecture:**
- **Decider:** Code that implements workflow logic
- **Activity Workers:** Code that executes tasks
- **You manage:** Poll for tasks, track state, implement retries

**Complexity:**
- Requires writing **decider** and **activity worker** code
- More control, but **much more work**

---

## Step Functions Workflow Types

### **Standard Workflows**

| Feature | Details |
|---------|---------|
| **Max Duration** | **1 year** |
| **Execution Semantics** | Exactly-once execution |
| **Execution History** | Full history retained |
| **Pricing** | $0.025 per 1,000 state transitions |
| **Use Case** | Long-running workflows (hours to months) |

**Example:** Order processing, multi-day approval workflows

---

### **Express Workflows**

| Feature | Details |
|---------|---------|
| **Max Duration** | **5 minutes** |
| **Execution Semantics** | At-least-once execution |
| **Execution History** | Sent to CloudWatch Logs (not retained by Step Functions) |
| **Pricing** | $1.00 per 1M executions |
| **Use Case** | High-volume, short-duration (IoT, streaming data, microservices) |

**Example:** Real-time data processing, API orchestration

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Orchestrate Lambda functions in ETL pipeline | **Step Functions** |
| Multi-step order workflow (new application) | **Step Functions** |
| Visual workflow designer required | **Step Functions** |
| Human approval workflow (wait for manager approval) | **Step Functions** (Standard) or **SWF** (legacy) |
| High-throughput, short-duration orchestration (<5 min) | **Step Functions Express** |
| Legacy SWF-based order system (already deployed) | **SWF** (maintain existing) |
| Orchestrate ECS tasks in multi-step workflow | **Step Functions** |
| Need built-in retry and error handling | **Step Functions** |

---

## Key Exam Tips

1. **Step Functions = Modern, recommended** for all new workflows
2. **SWF = Legacy service** (only for existing applications)
3. **Visual workflow?** → Step Functions (not SWF)
4. **Built-in AWS integrations?** → Step Functions
5. **Automatic error handling?** → Step Functions
6. **Step Functions Standard:** Long-running (up to 1 year)
7. **Step Functions Express:** High-volume, short-duration (<5 min)
8. **SWF requires writing code** (decider + activity workers)
9. **Step Functions is declarative** (JSON state machine)
10. Exam favorite: **"New workflow"** = Step Functions (not SWF)

---

## Migration from SWF to Step Functions

If you have existing SWF workflows, AWS recommends migrating to Step Functions:

**Benefits of Migration:**
- Visual workflow designer (easier to understand)
- Built-in AWS service integrations (no code to integrate)
- Automatic error handling (retry, catch)
- Lower operational overhead (no decider/worker code to maintain)

---

## Summary

| Service | Recommended | Best For |
|---------|-------------|----------|
| **Step Functions** | ✅ **Yes** (modern) | New workflows, serverless orchestration |
| **SWF** | ❌ **No** (legacy) | Existing workflows only (migrate to Step Functions) |

**Bottom Line:** Always use **Step Functions** for new applications. SWF is deprecated for new use cases.
