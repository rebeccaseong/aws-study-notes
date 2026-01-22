# Comparison: EC2 vs Lambda vs Fargate vs ECS vs EKS

Quick reference for choosing the right compute service.

---

## High-Level Decision Tree

```
Need to run code?
├─ Traditional application / full OS control → EC2
├─ Serverless function (event-driven, <15 min) → Lambda
├─ Containers?
   ├─ Don't want to manage servers → Fargate
   ├─ Want control over infrastructure → EC2 + ECS/EKS
   ├─ Need Kubernetes → EKS (+ EC2 or Fargate)
   └─ Don't need Kubernetes → ECS (+ EC2 or Fargate)
```

---

## Detailed Comparison Table

| **Aspect** | **EC2** | **Lambda** | **Fargate** | **ECS** | **EKS** |
|---|---|---|---|---|---|
| **What It Is** | Virtual servers | Serverless functions | Serverless containers | Container orchestration | Managed Kubernetes |
| **Management** | **You manage instances** | AWS manages everything | AWS manages infrastructure | **You manage EC2** (or use Fargate) | **You manage nodes** (or use Fargate) |
| **Pricing** | Instance hours | Invocations + duration | vCPU + memory per task | EC2 instances (or Fargate pricing) | EC2 nodes + EKS control plane ($0.10/hr) |
| **Scaling** | Manual or Auto Scaling | Automatic (instant) | Automatic | Manual or Auto Scaling | Manual or Auto Scaling |
| **Max Runtime** | Unlimited | **15 minutes** | Unlimited | Unlimited | Unlimited |
| **Cold Start** | No | **Yes** (can be slow) | Minimal | No (if EC2-based) | No (if EC2-based) |
| **Use Case** | **Traditional apps**, full control | **Event-driven**, short tasks | **Containers, no infra mgmt** | **Docker containers** | **Kubernetes workloads** |
| **Best For** | Steady-state workloads, legacy apps | Microservices, event processing | Microservices, variable load | Container apps (Docker) | Kubernetes migrations |
| **Complexity** | Medium | **Low** (easiest) | Low-Medium | Medium-High | **High** (Kubernetes complexity) |

---

## When to Choose Each Service

### **Choose EC2 If:**
- Need full control over the operating system and instance configuration
- Running traditional applications (web servers, databases, enterprise apps)
- Steady-state, predictable workloads (use Reserved Instances for cost savings)
- Applications require specific instance types (GPU, high memory, etc.)
- Need to install custom software or drivers

### **Choose Lambda If:**
- Building serverless, event-driven applications
- Functions run for **less than 15 minutes**
- Unpredictable or spiky traffic (Lambda auto-scales)
- Want zero infrastructure management
- Microservices that respond to events (S3, DynamoDB, API Gateway, etc.)

### **Choose Fargate If:**
- Running containers without managing servers
- Want serverless containers (no EC2 provisioning)
- Variable or unpredictable workloads (pay only for what you use)
- Don't need fine-grained control over instance types
- Simplicity over cost optimization (Fargate is more expensive than EC2)

### **Choose ECS If:**
- Running Docker containers on AWS
- Don't need Kubernetes (simpler than EKS)
- Want to choose between EC2 (control) or Fargate (serverless)
- Building microservices with containers
- AWS-native container orchestration

### **Choose EKS If:**
- Already using Kubernetes (on-prem or elsewhere)
- Need Kubernetes-specific features (Helm, Operators, etc.)
- Want portability across clouds (Kubernetes is cloud-agnostic)
- Have Kubernetes expertise in your team
- Running complex, multi-container applications

---

## Exam Scenarios & Keywords

| **Exam Keyword** | **Service** |
|---|---|
| "Full control over OS" / "Install custom software" | **EC2** |
| "Serverless" / "Event-driven" / "No servers to manage" | **Lambda** or **Fargate** |
| "Function runs for 5 minutes" | **Lambda** |
| "Function runs for 20 minutes" | **NOT Lambda** (use Fargate, ECS, or EC2) |
| "Run containers without managing infrastructure" | **Fargate** |
| "Docker containers" / "Microservices" | **ECS** or **EKS** (+ EC2 or Fargate) |
| "Kubernetes" / "Helm charts" / "K8s" | **EKS** |
| "Cost optimization" / "Steady workload" | **EC2** (with Reserved Instances) |
| "Variable workload" / "Unpredictable traffic" | **Lambda** or **Fargate** |

---

## Cost Comparison Example

**Scenario:** Run a web application 24/7

| **Service** | **Cost Estimate** | **Notes** |
|---|---|---|
| **EC2 (t3.medium)** | ~$30/month | Cheapest for 24/7 workloads (Reserved Instances even cheaper) |
| **Lambda** | ~$50-100/month | Only cost-effective for sporadic/event-driven workloads |
| **Fargate (0.5 vCPU, 1 GB)** | ~$40/month | More than EC2, but no instance management |
| **ECS (EC2)** | ~$30/month | Same as EC2 (you pay for EC2 instances) |
| **EKS (EC2)** | ~$100/month | EC2 cost + $73/month for EKS control plane |

**Key Insight:** EC2 is cheapest for steady workloads, Lambda/Fargate for variable workloads.

---

## Summary

- **EC2:** Maximum control, traditional apps, steady workloads
- **Lambda:** Serverless, event-driven, <15 min runtime, zero management
- **Fargate:** Serverless containers, no infrastructure management
- **ECS:** Container orchestration (AWS-native, simpler than Kubernetes)
- **EKS:** Kubernetes on AWS (portability, complexity, advanced features)
