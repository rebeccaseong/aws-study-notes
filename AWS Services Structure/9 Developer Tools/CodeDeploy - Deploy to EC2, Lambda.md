# CodeDeploy - Deploy to EC2, Lambda

- **What is it? (1-Sentence Pitch):** A fully managed deployment service that automates software deployments to compute services like EC2, Fargate, Lambda, and on-premises servers.
- **Core Use Cases:** Automate deployments with zero downtime using blue/green deployments, canary deployments, or rolling updates with automatic rollback on failure.
- **Key Distinction:** CodeDeploy **deploys** your application (vs CodeBuild which **builds** it, vs CodePipeline which **orchestrates** the entire workflow).

---

## What CodeDeploy Does

CodeDeploy automates application deployments to:
- **EC2 instances** (in-place or blue/green)
- **AWS Lambda** (canary, linear, all-at-once)
- **ECS** (blue/green)
- **On-premises servers** (in-place)

**Key Features:**
- Multiple deployment strategies (blue/green, canary, rolling)
- Automatic rollback on failure
- Deployment monitoring and health checks
- Gradual traffic shifting for Lambda and ECS
- Integration with CodePipeline

---

## Core Concepts

```
┌──────────────────────────────────────────────────┐
│              CodeDeploy Components               │
├─────────────┬─────────────┬──────────────────────┤
│ Application │ Deployment  │ Deployment           │
│             │ Group       │ Configuration        │
├─────────────┼─────────────┼──────────────────────┤
│ AppSpec     │ Revision    │ Deployment           │
│ File        │             │ Strategy             │
└─────────────┴─────────────┴──────────────────────┘
```

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Application** | Container for deployment groups, revisions, and configurations |
| **Deployment Group** | Set of EC2 instances, Lambda functions, or ECS services to deploy to |
| **Revision** | The version of your application (code + AppSpec file) |
| **AppSpec File** | Configuration file (`appspec.yml`) defining deployment steps and lifecycle hooks |
| **Deployment Configuration** | Rules for deployment (how fast, how many at once) |
| **Deployment Type** | In-place or Blue/Green |

---

## Deployment Types

### **1. In-Place Deployment**

**What It Is:** Update instances in place by stopping the application, deploying new code, and restarting.

**How It Works:**
```
┌─────────────┐
│ EC2 Instance│  (Application v1 running)
└──────┬──────┘
       ↓
  Stop Application
       ↓
  Deploy v2
       ↓
  Start Application
       ↓
┌─────────────┐
│ EC2 Instance│  (Application v2 running)
└─────────────┘
```

**Characteristics:**

| Aspect | Details |
|--------|---------|
| **Downtime** | Yes (during deployment) |
| **Capacity** | Reduced during deployment (instances taken out of service) |
| **Rollback** | Redeploy previous version (slower) |
| **Cost** | Lower (no additional resources) |
| **Use Case** | Dev/test environments, cost-sensitive deployments |

**Supported Platforms:** EC2, on-premises servers

---

### **2. Blue/Green Deployment**

**What It Is:** Deploy new version to new instances (green), test, then switch traffic from old (blue) to new (green).

**How It Works:**
```
┌────────────┐                    ┌────────────┐
│ Blue Env   │  (v1 running)      │ Green Env  │  (v2 deployed)
│ EC2/ECS/λ  │                    │ EC2/ECS/λ  │
└──────┬─────┘                    └──────┬─────┘
       │                                 │
       │  Traffic (100%)                 │  (waiting)
       │                                 │
       └─────────────┐     ┌─────────────┘
                     ↓     ↓
              ┌──────────────┐
              │    ELB/λ     │
              └──────────────┘
                     ↓
           Switch traffic to Green
                     ↓
       ┌─────────────────────────────────┐
       │  Green now receives 100% traffic │
       │  Blue kept for rollback          │
       └─────────────────────────────────┘
```

**Characteristics:**

| Aspect | Details |
|--------|---------|
| **Downtime** | **Zero** (traffic switches instantly) |
| **Capacity** | Full capacity maintained (both environments running) |
| **Rollback** | **Instant** (switch traffic back to blue) |
| **Cost** | Higher (double resources during deployment) |
| **Use Case** | **Production deployments**, zero downtime required |

**Supported Platforms:** EC2 (via ELB), ECS, Lambda

**EC2 Blue/Green:**
- Requires **Elastic Load Balancer** (ALB or NLB)
- CodeDeploy provisions new instances, deploys code, registers with ELB
- Traffic shifts from old instances to new instances
- Old instances can be terminated or kept for rollback

**Lambda Blue/Green:**
- Create new Lambda version
- Shift traffic gradually using **alias** and **traffic shifting**

**ECS Blue/Green:**
- Create new ECS task set with new version
- Shift traffic from old task set to new task set

---

## Deployment Strategies

### **1. All-at-Once**

**What It Is:** Deploy to all instances/functions simultaneously.

**Use Case:** Dev/test, fastest deployment, acceptable downtime

**Risk:** High (all instances affected if deployment fails)

---

### **2. Half-at-a-Time**

**What It Is:** Deploy to 50% of instances, then the remaining 50%.

**Use Case:** Balance between speed and safety

**Risk:** Medium (50% of fleet affected if deployment fails)

---

### **3. One-at-a-Time**

**What It Is:** Deploy to one instance at a time.

**Use Case:** Safest rolling deployment, minimal risk

**Risk:** Lowest (only 1 instance affected at a time)

**Downside:** Slowest deployment

---

### **4. Custom**

**What It Is:** Define your own deployment configuration (e.g., 25% at a time, minimum healthy hosts).

**Example:** Deploy to 25% of instances, wait for health checks, then proceed to next 25%.

---

## Lambda Deployment Strategies

Lambda deployments support **traffic shifting** with automatic rollback:

### **1. Canary**

**What It Is:** Shift a small percentage of traffic to new version first, then shift remaining traffic.

**How It Works:**
```
┌──────────────────────────────────────┐
│  Time 0:   100% traffic → v1         │
│  Time 5m:   10% traffic → v2 (canary)│
│             90% traffic → v1         │
│  (Wait, monitor metrics/alarms)      │
│  Time 10m: 100% traffic → v2         │
└──────────────────────────────────────┘
```

**Configurations:**
- **Canary10Percent5Minutes:** 10% for 5 minutes, then 100%
- **Canary10Percent10Minutes:** 10% for 10 minutes, then 100%
- **Canary10Percent15Minutes:** 10% for 15 minutes, then 100%
- **Canary10Percent30Minutes:** 10% for 30 minutes, then 100%

**Use Case:** Test new version with small traffic sample before full rollout

**Automatic Rollback:** If CloudWatch alarms trigger during canary period, rollback to v1

---

### **2. Linear**

**What It Is:** Shift traffic gradually in equal increments.

**How It Works:**
```
┌──────────────────────────────────────┐
│  Time 0:    100% → v1                │
│  Time 1m:    10% → v2, 90% → v1      │
│  Time 2m:    20% → v2, 80% → v1      │
│  Time 3m:    30% → v2, 70% → v1      │
│  ...                                 │
│  Time 10m:  100% → v2                │
└──────────────────────────────────────┘
```

**Configurations:**
- **Linear10PercentEvery1Minute:** +10% every 1 minute (complete in 10 minutes)
- **Linear10PercentEvery2Minutes:** +10% every 2 minutes (complete in 20 minutes)
- **Linear10PercentEvery3Minutes:** +10% every 3 minutes (complete in 30 minutes)
- **Linear10PercentEvery10Minutes:** +10% every 10 minutes (complete in 100 minutes)

**Use Case:** Gradual, controlled rollout with monitoring at each step

---

### **3. All-at-Once**

**What It Is:** Shift 100% of traffic immediately to new version.

**Use Case:** Dev/test, non-critical functions

**Risk:** High (all traffic affected if deployment fails)

---

## AppSpec File

**What It Is:** Configuration file (`appspec.yml` for EC2/on-premises, `appspec.yaml` or `appspec.json` for Lambda/ECS) that defines deployment steps.

### **AppSpec for EC2/On-Premises**

**Structure:**
```yaml
version: 0.0
os: linux  # or windows
files:
  - source: /
    destination: /var/www/html
permissions:
  - object: /var/www/html
    owner: apache
    group: apache
    mode: 755
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
  AfterInstall:
    - location: scripts/configure_app.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
  ValidateService:
    - location: scripts/validate.sh
      timeout: 300
```

**Key Sections:**

| Section | Purpose |
|---------|---------|
| **version** | AppSpec version (always `0.0` for EC2) |
| **os** | Operating system (`linux` or `windows`) |
| **files** | Where to copy files on the instance |
| **permissions** | File/directory permissions |
| **hooks** | Lifecycle event scripts (when to run scripts) |

---

### **Lifecycle Hooks (EC2)**

Lifecycle events run in this order:

```
ApplicationStop        (stop current version)
  ↓
DownloadBundle        (download new revision)
  ↓
BeforeInstall         (pre-install tasks: backup, dependencies)
  ↓
Install               (copy files to destination)
  ↓
AfterInstall          (post-install tasks: configuration)
  ↓
ApplicationStart      (start new version)
  ↓
ValidateService       (health checks, smoke tests)
```

**Common Hook Scripts:**

| Hook | Purpose | Example |
|------|---------|---------|
| **ApplicationStop** | Stop running application | `systemctl stop myapp` |
| **BeforeInstall** | Install dependencies, backup | `yum install -y httpd`, `backup.sh` |
| **AfterInstall** | Configure application | `configure.sh`, `chmod 755 /var/www/html` |
| **ApplicationStart** | Start application | `systemctl start myapp` |
| **ValidateService** | Health checks | `curl localhost:80`, `check_health.sh` |

---

### **AppSpec for Lambda**

**Structure (YAML):**
```yaml
version: 0.0
Resources:
  - MyFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: my-lambda-function
        Alias: live
        CurrentVersion: 1
        TargetVersion: 2
Hooks:
  BeforeAllowTraffic:
    - location: validate_lambda.js
      timeout: 300
  AfterAllowTraffic:
    - location: smoke_test.js
      timeout: 300
```

**Lambda Lifecycle Hooks:**

| Hook | When It Runs | Use Case |
|------|-------------|----------|
| **BeforeAllowTraffic** | Before traffic shifts to new version | Validation, pre-checks |
| **AfterAllowTraffic** | After traffic shifts to new version | Smoke tests, integration tests |

**Example:** Run integration tests before allowing traffic to new Lambda version

---

## Deployment Monitoring

### **Health Checks**

CodeDeploy monitors instance/function health during deployment:

**EC2:**
- Elastic Load Balancer health checks
- EC2 instance status checks
- Custom health check scripts (ValidateService hook)

**Lambda:**
- CloudWatch alarms (errors, duration, throttles)
- Custom validation Lambda functions

**ECS:**
- ECS service health checks
- ALB target group health checks

---

### **Automatic Rollback**

CodeDeploy can automatically rollback on failure:

**Rollback Triggers:**
- Deployment failure (instances/functions fail health checks)
- CloudWatch alarms (e.g., error rate > threshold)
- Manual rollback

**Rollback Behavior:**
- **In-Place:** Redeploy previous revision (slower)
- **Blue/Green:** Switch traffic back to blue environment (instant)

**Configuration:**
- Enable automatic rollback in deployment group settings
- Specify CloudWatch alarms to monitor

---

## Integration with CodePipeline

CodeDeploy is typically used as the **Deploy stage** in CodePipeline:

```
┌────────────┐    ┌────────────┐    ┌────────────┐
│ CodeCommit │ →  │ CodeBuild  │ →  │ CodeDeploy │
│  (Source)  │    │  (Build)   │    │  (Deploy)  │
└────────────┘    └────────────┘    └────────────┘
```

**Workflow:**
1. Developer commits code to CodeCommit
2. CodePipeline triggers CodeBuild (compile, test, package)
3. CodeBuild uploads build artifacts to S3
4. CodePipeline triggers CodeDeploy
5. CodeDeploy deploys artifacts to EC2/Lambda/ECS

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Deploy to EC2 with zero downtime | **CodeDeploy Blue/Green** (with ELB) |
| Deploy to Lambda with gradual traffic shift | **CodeDeploy Canary** or **Linear** |
| Test new Lambda version with 10% traffic first | **CodeDeploy Canary10Percent5Minutes** |
| Automatically rollback on errors | **CodeDeploy** with CloudWatch alarms |
| Deploy to EC2, acceptable downtime | **CodeDeploy In-Place** |
| Deploy to on-premises servers | **CodeDeploy In-Place** (install CodeDeploy agent) |
| Most cost-effective deployment to EC2 | **CodeDeploy In-Place** (no additional instances) |
| Production deployment, zero downtime, instant rollback | **CodeDeploy Blue/Green** |
| Define deployment steps (stop, install, start) | **AppSpec file** with lifecycle hooks |
| Deploy to half of instances at a time | **CodeDeploy Half-at-a-Time** deployment configuration |

---

## Key Exam Tips

1. **CodeDeploy = Deploy** (CodeBuild = Build, CodePipeline = Orchestrate)
2. **In-Place:** Downtime, slower rollback, lower cost (dev/test)
3. **Blue/Green:** Zero downtime, instant rollback, higher cost (production)
4. **Canary:** Small % first, monitor, then 100% (Lambda/ECS)
5. **Linear:** Gradual increments (10% every 1 min) (Lambda/ECS)
6. **AppSpec file:** Required for all deployments (defines what and how to deploy)
7. **Lifecycle hooks:** Scripts run at specific points (BeforeInstall, AfterInstall, ApplicationStart, ValidateService)
8. **Automatic rollback:** Trigger on deployment failure or CloudWatch alarms
9. **Deployment Group:** Set of instances/functions to deploy to (can be tagged EC2 instances)
10. **Targets:** EC2, Lambda, ECS, on-premises
11. **Agent required:** EC2 and on-premises instances need CodeDeploy agent installed
12. **Blue/Green requires ELB** for EC2 deployments (ALB or NLB)
13. **Lambda deployments:** Use aliases and traffic shifting
14. **Free for EC2/Lambda:** No additional charge (only for ECS task updates)
15. **Monitoring:** CloudWatch metrics, CloudWatch alarms for automatic rollback

---

## Common Deployment Configurations

### **EC2 Deployment Configurations**

| Configuration | Description | Use Case |
|---------------|-------------|----------|
| **AllAtOnce** | Deploy to all instances simultaneously | Dev/test, fastest |
| **HalfAtATime** | Deploy to 50% of instances, then remaining 50% | Balanced |
| **OneAtATime** | Deploy to one instance at a time | Safest, slowest |
| **Custom** | Define minimum healthy hosts percentage | Custom requirements |

---

### **Lambda Deployment Configurations**

| Configuration | Description | Timeline |
|---------------|-------------|----------|
| **AllAtOnce** | 100% traffic shift immediately | Instant |
| **Canary10Percent5Minutes** | 10% for 5 min, then 100% | 5 minutes |
| **Canary10Percent10Minutes** | 10% for 10 min, then 100% | 10 minutes |
| **Linear10PercentEvery1Minute** | +10% every 1 min | 10 minutes |
| **Linear10PercentEvery2Minutes** | +10% every 2 min | 20 minutes |
| **Linear10PercentEvery10Minutes** | +10% every 10 min | 100 minutes |

---

## Best Practices

### **1. Use Blue/Green for Production**
- Zero downtime
- Instant rollback
- Full testing before traffic shift

### **2. Enable Automatic Rollback**
- Configure CloudWatch alarms (error rate, latency)
- Rollback on deployment failure

### **3. Test in Staging First**
- Deploy to staging environment before production
- Use same deployment configuration

### **4. Use Lifecycle Hooks for Validation**
- ValidateService hook for health checks
- Fail deployment if validation fails

### **5. Monitor Deployments**
- CloudWatch metrics (deployment success rate)
- Set alarms for failures

### **6. Tag Instances for Deployment Groups**
- Use EC2 tags to define deployment groups
- Dynamic fleet management (auto-scaling)

---

## Cost

**Pricing:**
- **EC2/On-Premises:** **Free** (no CodeDeploy charges)
- **Lambda:** **Free** (no CodeDeploy charges)
- **ECS:** $0.02 per ECS task update (active tasks)

**Cost Drivers:**
- ECS deployments only (per task update)
- Underlying compute costs (EC2 instances, Lambda invocations)
- Blue/Green deployments (double EC2 instances during deployment)

**Optimization:**
- Use In-Place for dev/test (save on EC2 instance costs)
- Terminate blue environment after successful Blue/Green deployment
- Use Lambda deployments (free)

---

## Summary

**CodeDeploy** automates application deployments with multiple strategies:

| Deployment Type | Downtime | Rollback Speed | Cost | Use Case |
|----------------|----------|----------------|------|----------|
| **In-Place** | Yes | Slow (redeploy) | Lower | Dev/test |
| **Blue/Green** | No | Instant | Higher | Production |

| Lambda Strategy | Description | Timeline |
|----------------|-------------|----------|
| **Canary** | Small % first, then 100% | 5-30 minutes |
| **Linear** | Gradual increments | 10-100 minutes |
| **AllAtOnce** | 100% immediately | Instant |

**Bottom Line:** CodeDeploy deploys your application to EC2, Lambda, ECS, or on-premises with automatic rollback and multiple deployment strategies for zero-downtime deployments.
