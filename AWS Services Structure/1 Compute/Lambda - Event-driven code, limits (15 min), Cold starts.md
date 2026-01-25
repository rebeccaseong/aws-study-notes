- **What is it? (1-Sentence Pitch):** A serverless compute service that runs your code in response to events without you needing to manage any servers.
- **Core Use Cases:**
    - Event-driven data processing (e.g., process an image when it's uploaded to S3).
    - Building backend APIs for web/mobile apps (via API Gateway).
    - Scheduled tasks (via EventBridge/CloudWatch Events).
- **Key Features & Concepts:**
    - **Event-Driven:** A function is triggered by an event source (S3, DynamoDB, API Gateway, etc.).
    - **Serverless:** No servers to provision or manage. AWS handles scaling, patching, and availability.
    - **Configuration:** You only configure memory, which also allocates proportional CPU. Max timeout is 15 minutes.
    - **Concurrency:** The number of executions of your function that can run simultaneously.
- **Integration:**
    - The "glue" of AWS; integrates with over 200 AWS services.
    - Often triggered by **S3, SQS, SNS, DynamoDB Streams, and API Gateway**.
    - Writes logs to **CloudWatch Logs**.
    - Can be orchestrated by **Step Functions**.

---

## Lambda Layers

**What is it?** A ZIP archive containing libraries, custom runtimes, or other dependencies that can be shared across multiple Lambda functions.

**Key Point:** Layers allow you to **separate function code from dependencies**, reducing deployment package size and enabling code reuse.

---

### **Why Use Layers?**

| Problem | Without Layers | With Layers |
|---------|---------------|-------------|
| **Deployment Size** | Each function packages its own dependencies (large package) | Dependencies packaged once in layer (small function package) |
| **Code Duplication** | 10 functions = 10 copies of same library | 10 functions share 1 layer |
| **Update Dependencies** | Update 10 functions individually | Update 1 layer, all functions benefit |
| **Deployment Speed** | Upload full package every time | Upload small function code only |

**Example Scenario:**
- You have 50 Lambda functions that all use `requests` library (Python)
- Without layers: Each function packages `requests` (50 copies, slow deployments)
- With layers: Create 1 layer with `requests`, all 50 functions reference it

---

### **Layer Structure**

**Layer Contents:**
```
layer.zip
└── python/          (for Python runtime)
    └── lib/
        └── python3.9/
            └── site-packages/
                └── requests/

OR

└── nodejs/          (for Node.js runtime)
    └── node_modules/
        └── lodash/
```

**Key Point:** Layer must follow specific directory structure based on runtime.

| Runtime | Required Directory Structure |
|---------|----------------------------|
| **Python** | `python/` or `python/lib/python3.x/site-packages/` |
| **Node.js** | `nodejs/node_modules/` |
| **Java** | `java/lib/` |
| **Ruby** | `ruby/gems/` |
| **.NET** | No specific structure (assemblies in root) |

---

### **Layer Limits**

| Limit | Value |
|-------|-------|
| **Max layers per function** | 5 layers |
| **Max layer size (unzipped)** | 250 MB (combined with function code) |
| **Max layer size (zipped)** | 50 MB (direct upload) or 250 MB (via S3) |
| **Layer versions** | Unlimited (each publish creates new version) |

**Exam Tip:** If function + layers exceed 250 MB unzipped → Use container images instead (up to 10 GB)

---

### **Creating a Layer**

**Step 1: Package Dependencies**
```bash
# Python example
mkdir -p layer/python
pip install requests -t layer/python/
cd layer
zip -r requests-layer.zip python/
```

**Step 2: Publish Layer**
```bash
aws lambda publish-layer-version \
  --layer-name requests-layer \
  --description "Python requests library" \
  --zip-file fileb://requests-layer.zip \
  --compatible-runtimes python3.9 python3.10
```

**Step 3: Attach Layer to Function**
```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:123456789012:layer:requests-layer:1
```

---

### **Layer Versioning**

**Key Point:** Each time you publish a layer, AWS creates a **new immutable version**.

```
Layer: requests-layer
├─ Version 1 (requests 2.25.0)
├─ Version 2 (requests 2.26.0)
└─ Version 3 (requests 2.28.0)
```

**Function References:**
- Function A uses `requests-layer:1` (2.25.0)
- Function B uses `requests-layer:3` (2.28.0)
- Both coexist independently

**Exam Tip:** Old layer versions remain available until explicitly deleted

---

### **Sharing Layers**

#### **Within Your Account**
- All functions in your account can access your layers automatically

#### **Cross-Account Sharing**
```bash
aws lambda add-layer-version-permission \
  --layer-name my-layer \
  --version-number 1 \
  --statement-id public-access \
  --action lambda:GetLayerVersion \
  --principal "*"
```

**Use Cases:**
- Share internal libraries across multiple AWS accounts
- Publish public layers (e.g., monitoring, observability)

---

### **AWS-Provided Layers**

**Common AWS Layers:**

| Layer | Purpose | ARN Pattern |
|-------|---------|-------------|
| **AWS SDK** | Latest AWS SDK for Python/Node.js | `arn:aws:lambda:<region>:aws:layer:AWSSDKPandas-Python39` |
| **AWS Parameters and Secrets** | Access SSM Parameters/Secrets Manager | `arn:aws:lambda:<region>:aws:layer:AWS-Parameters-and-Secrets-Lambda-Extension` |
| **AWS Lambda Powertools** | Logging, tracing, metrics utilities | `arn:aws:lambda:<region>:aws:layer:AWSLambdaPowertoolsPython` |
| **AWS X-Ray** | X-Ray daemon for tracing | Built into runtime (no layer needed) |

**Exam Tip:** AWS-provided layers are pre-optimized and updated by AWS

---

### **Common Use Cases**

#### **1. Shared Libraries**
**Scenario:** 10 functions all use Pandas library (100 MB)
**Solution:** Create layer with Pandas, all functions reference it
**Benefit:** Single upload, faster deployments, consistent version

---

#### **2. Custom Runtime**
**Scenario:** Need to use unsupported language (e.g., Rust, PHP)
**Solution:** Create layer with custom runtime
**Benefit:** Run any language on Lambda

---

#### **3. Configuration Files**
**Scenario:** Functions need shared config files (e.g., ML model, certificates)
**Solution:** Package config in layer
**Benefit:** Update config once, all functions updated

---

#### **4. Organization-Wide Utilities**
**Scenario:** Common logging/monitoring utilities across teams
**Solution:** Publish shared layer, teams reference it
**Benefit:** Centralized management, consistent behavior

---

### **Layers vs. Environment Variables vs. Parameter Store**

| Use Case | Best Choice |
|----------|-------------|
| **Shared libraries/dependencies** | **Layers** |
| **Small config values (<4 KB)** | Environment Variables |
| **Secrets** | Parameter Store (SSM) or Secrets Manager |
| **Large config files (>4 KB)** | **Layers** or S3 |
| **Frequently changing config** | Parameter Store or S3 |
| **Infrequently changing code** | **Layers** |

---

## Lambda Versions

**What is it?** An **immutable snapshot** of your Lambda function code and configuration at a specific point in time.

**Key Point:** Once published, a version **cannot be changed**. This ensures stability and enables rollback.

---

### **How Versions Work**

```
Lambda Function: my-function
├─ $LATEST (mutable, always points to latest code)
├─ Version 1 (immutable snapshot)
├─ Version 2 (immutable snapshot)
└─ Version 3 (immutable snapshot)
```

**`$LATEST`:**
- Special version that always points to the **latest unpublished code**
- Mutable (can be edited)
- Used during development
- Cannot be used with weighted aliases

**Published Versions:**
- Immutable (cannot be edited)
- Numbered sequentially (1, 2, 3, ...)
- Include code + configuration (memory, timeout, env vars, layers)
- Can be invoked directly or via aliases

---

### **Why Use Versions?**

| Benefit | Description |
|---------|-------------|
| **Immutability** | Published version never changes (guaranteed stability) |
| **Rollback** | If v3 has bug, switch back to v2 instantly |
| **Testing** | Test v3 in staging while v2 runs in production |
| **Audit Trail** | Track which version deployed when |
| **Blue/Green Deployments** | Traffic shifting between versions via aliases |

---

### **Publishing a Version**

**AWS Console:**
1. Lambda → Functions → my-function
2. Actions → Publish new version
3. Enter description (e.g., "Added authentication")
4. Click Publish

**AWS CLI:**
```bash
aws lambda publish-version \
  --function-name my-function \
  --description "Added authentication feature"
```

**Response:**
```json
{
  "Version": "3",
  "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-function:3",
  "CodeSha256": "abc123...",
  "Description": "Added authentication feature"
}
```

---

### **Invoking Specific Versions**

**Invoke `$LATEST` (default):**
```bash
aws lambda invoke \
  --function-name my-function \
  response.json
```

**Invoke Version 2:**
```bash
aws lambda invoke \
  --function-name my-function:2 \
  response.json
```

**ARN Format:**
```
arn:aws:lambda:us-east-1:123456789012:function:my-function:2
                                                           ↑
                                                      version number
```

---

### **Version Management**

#### **Listing Versions**
```bash
aws lambda list-versions-by-function --function-name my-function
```

#### **Deleting Versions**
```bash
aws lambda delete-function --function-name my-function:2
```

**Key Point:** Cannot delete `$LATEST` (it's the function itself)

---

### **Version Best Practices**

| Best Practice | Why |
|---------------|-----|
| **Always publish before production** | Ensures immutability, enables rollback |
| **Tag versions** | Use description field for change notes |
| **Don't invoke `$LATEST` in production** | Use aliases pointing to versions instead |
| **Clean up old versions** | Reduce clutter, stay organized |
| **Use versions with aliases** | Enable traffic shifting and blue/green deployments |

---

## Lambda Aliases

**What is it?** A **pointer to a specific Lambda version** that can be updated to point to different versions.

**Key Point:** Aliases enable **traffic shifting between versions** for blue/green deployments and canary releases.

---

### **Why Use Aliases?**

| Problem | Without Aliases | With Aliases |
|---------|----------------|--------------|
| **Hard-coded version** | API Gateway points to `my-function:3` | API Gateway points to `prod` alias |
| **Deployment change** | Update all integrations to `my-function:4` | Update alias to point to version 4 |
| **Gradual rollout** | All-or-nothing deployment | Route 10% to v4, 90% to v3 (canary) |
| **Rollback** | Redeploy previous version | Update alias back to v3 |
| **Environment separation** | Manage separate functions for dev/prod | Use `dev`, `staging`, `prod` aliases |

**Exam Trigger:** "Need to gradually shift traffic to new version" → **Aliases with weighted routing**

---

### **Alias Structure**

```
Lambda Function: my-function
├─ $LATEST
├─ Version 1
├─ Version 2
├─ Version 3
│
└─ Aliases:
   ├─ dev → $LATEST
   ├─ staging → Version 2
   └─ prod → Version 3 (90%) + Version 4 (10%)  ← Weighted routing
```

**Key Point:** An alias can point to:
- Single version (100% traffic)
- Two versions with weighted routing (e.g., 90% v3, 10% v4)

---

### **Creating an Alias**

**AWS Console:**
1. Lambda → Functions → my-function
2. Aliases → Create alias
3. Name: `prod`
4. Version: `3`
5. Click Create

**AWS CLI:**
```bash
aws lambda create-alias \
  --function-name my-function \
  --name prod \
  --function-version 3 \
  --description "Production environment"
```

---

### **Updating an Alias**

**Scenario:** Deploy new version to production

**Step 1: Publish new version**
```bash
aws lambda publish-version --function-name my-function
# Returns: Version 4
```

**Step 2: Update alias to point to v4**
```bash
aws lambda update-alias \
  --function-name my-function \
  --name prod \
  --function-version 4
```

**Result:** All traffic instantly switches from v3 to v4

---

### **Weighted Routing (Traffic Shifting)**

**What is it?** Route percentage of traffic to two different versions for **canary deployments** or **A/B testing**.

**Use Case:** Deploy v4 to 10% of users, monitor errors, then gradually increase to 100%

---

#### **Canary Deployment Example**

**Step 1: Deploy 10% to new version**
```bash
aws lambda update-alias \
  --function-name my-function \
  --name prod \
  --function-version 4 \
  --routing-config '{"AdditionalVersionWeights": {"3": 0.9}}'
```

**Traffic Split:**
- Version 4: 10% (new version)
- Version 3: 90% (old version)

**Step 2: Monitor metrics (errors, latency)**
- If v4 performs well → increase traffic
- If v4 has issues → rollback by updating alias to 100% v3

**Step 3: Gradually increase traffic**
```bash
# 50% to v4
aws lambda update-alias \
  --function-name my-function \
  --name prod \
  --function-version 4 \
  --routing-config '{"AdditionalVersionWeights": {"3": 0.5}}'

# 100% to v4 (remove weight)
aws lambda update-alias \
  --function-name my-function \
  --name prod \
  --function-version 4 \
  --routing-config '{}'
```

---

### **Weighted Routing Limits**

| Limit | Value |
|-------|-------|
| **Max versions in routing** | 2 (primary + 1 additional) |
| **Weight range** | 0.0 - 1.0 (0% - 100%) |
| **Weight precision** | Up to 3 decimal places (e.g., 0.123 = 12.3%) |

**Exam Tip:** Can only split traffic between **2 versions** (not 3 or more)

---

### **Invoking via Alias**

**Invoke production alias:**
```bash
aws lambda invoke \
  --function-name my-function:prod \
  response.json
```

**ARN Format:**
```
arn:aws:lambda:us-east-1:123456789012:function:my-function:prod
                                                           ↑
                                                      alias name
```

**API Gateway Integration:**
```
API Gateway → Lambda: my-function:prod
```

**Benefit:** Update alias without changing API Gateway integration

---

### **Alias Environment Variables**

**Key Point:** Aliases can have **different environment variables** than the function version.

**Use Case:** Same code, different config per environment

**Example:**
```
Version 3 code:
  DATABASE_URL = os.environ.get('DATABASE_URL')

Alias: dev
  DATABASE_URL = dev-database.example.com

Alias: prod
  DATABASE_URL = prod-database.example.com
```

**Setting Alias Environment Variables:**
```bash
aws lambda update-alias \
  --function-name my-function \
  --name prod \
  --function-version 3 \
  --environment-variables '{"DATABASE_URL": "prod-database.example.com"}'
```

**Exam Tip:** Alias env vars **override** function version env vars

---

### **Alias Permissions**

**Key Point:** Aliases can have **separate IAM policies** from the function.

**Use Case:** Grant external service access to production alias only (not dev/staging)

**Example:**
```bash
aws lambda add-permission \
  --function-name my-function:prod \
  --statement-id api-gateway-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn arn:aws:execute-api:us-east-1:123456789012:api-id/*
```

**Result:** API Gateway can invoke `prod` alias only

---

### **Common Alias Strategies**

#### **1. Environment-Based Aliases**

```
my-function
├─ dev → $LATEST (always latest code)
├─ staging → Version 5 (tested in staging)
└─ prod → Version 4 (stable production)
```

**Deployment Flow:**
1. Developer commits code → deployed to `$LATEST`
2. `dev` alias points to `$LATEST` (automatic testing)
3. Publish version 6
4. Update `staging` → Version 6 (QA testing)
5. Update `prod` → Version 6 (production release)

---

#### **2. Blue/Green Deployment**

```
my-function:prod
├─ Blue (Version 3): 100% traffic
└─ Green (Version 4): 0% traffic

Step 1: Deploy Green
├─ Blue (Version 3): 90%
└─ Green (Version 4): 10%

Step 2: Increase Green
├─ Blue (Version 3): 50%
└─ Green (Version 4): 50%

Step 3: Full Green
├─ Blue (Version 3): 0%
└─ Green (Version 4): 100%
```

---

#### **3. Canary Release**

```
prod alias
├─ Stable (Version 3): 95%
└─ Canary (Version 4): 5%

Monitor metrics for 24 hours:
- Errors < 1%? → Increase canary to 10%
- Errors > 1%? → Rollback (set canary to 0%)
```

---

## Common Exam Scenarios

### Scenario 1: Gradual Traffic Shift

**Question:** "You need to deploy a new version of your Lambda function but want to monitor errors before fully switching traffic. How do you achieve this?"

**Answer:** Use **Lambda Aliases with weighted routing**

**Implementation:**
1. Publish new version (e.g., v4)
2. Update `prod` alias to route 10% to v4, 90% to v3
3. Monitor CloudWatch metrics
4. Gradually increase traffic to v4
5. Eventually route 100% to v4

---

### Scenario 2: Separate Dev/Prod Environments

**Question:** "Your team needs separate Lambda environments for development and production. How do you manage this efficiently?"

**Answer:** Use **Lambda Aliases** (`dev`, `prod`) pointing to different versions

**Implementation:**
- `dev` alias → `$LATEST` (always latest code)
- `prod` alias → Version 5 (stable, tested)
- API Gateway integrates with alias, not version
- Update alias to deploy to production

---

### Scenario 3: Shared Dependencies

**Question:** "You have 20 Lambda functions that all use the same 50 MB library. Deployments are slow and packages are large. How do you optimize?"

**Answer:** Create **Lambda Layer** with shared library

**Implementation:**
1. Package library in layer
2. Publish layer
3. Attach layer to all 20 functions
4. Each function package now only contains function code (small, fast)

---

### Scenario 4: Instant Rollback

**Question:** "You deployed a new Lambda version to production but discovered a critical bug. How do you rollback instantly?"

**Answer:** Update **Lambda Alias** to point to previous version

**Implementation:**
```bash
# Current: prod → Version 4 (buggy)
aws lambda update-alias \
  --function-name my-function \
  --name prod \
  --function-version 3
# Now: prod → Version 3 (previous stable version)
```

**Result:** Instant rollback, no redeployment needed

---

### Scenario 5: A/B Testing

**Question:** "You want to test two different implementations of a Lambda function to see which performs better. How do you route traffic to both versions?"

**Answer:** Use **weighted alias routing**

**Implementation:**
- Version 3: Implementation A
- Version 4: Implementation B
- `prod` alias: 50% v3, 50% v4
- Monitor metrics, choose winner
- Update alias to 100% winning version

---

## Key Exam Tips

### Layers
1. **Max 5 layers per function**, 250 MB total (unzipped)
2. Use layers for **shared dependencies** and **custom runtimes**
3. Layers are **versioned** (immutable once published)
4. Can **share layers across accounts**
5. Each runtime requires **specific directory structure** (e.g., `python/`, `nodejs/node_modules/`)

### Versions
6. **`$LATEST`** = mutable, always latest code
7. **Published versions** = immutable snapshots
8. Use versions for **stability** and **rollback**
9. **Cannot edit** published version (must publish new)
10. Versions include code + configuration + layers

### Aliases
11. **Alias = pointer** to version (can be updated)
12. **Weighted routing** = split traffic between 2 versions (canary, A/B testing)
13. Aliases enable **blue/green deployments**
14. Alias can have **separate env vars** and **permissions**
15. **Cannot weight `$LATEST`** (only published versions)

### Deployment Patterns
16. **Canary:** 10% new version, 90% old → gradually increase
17. **Blue/Green:** Switch all traffic from old to new version
18. **Environment separation:** `dev`, `staging`, `prod` aliases
19. **Rollback:** Update alias to previous version (instant)
20. **A/B testing:** 50/50 split between two versions

---

## Summary

**Lambda Layers:**
- **Share dependencies** across functions
- Reduce deployment package size
- Centralize library management
- Support custom runtimes

**Lambda Versions:**
- **Immutable snapshots** of function code
- Enable rollback and audit trail
- Numbered sequentially (1, 2, 3...)
- `$LATEST` is always mutable

**Lambda Aliases:**
- **Pointers to versions** (can be updated)
- Enable **weighted traffic routing** (canary, blue/green)
- Separate environment variables and permissions
- Simplify deployment and rollback

**Bottom Line:** Use **Layers** for shared code, **Versions** for immutability, and **Aliases** for deployment strategies and traffic management.
