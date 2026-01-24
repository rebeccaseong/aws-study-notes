- **What is it? (1-Sentence Pitch):** An Infrastructure as Code (IaC) service that lets you model and provision your entire AWS infrastructure in a simple text file (YAML or JSON).
- **Core Use Cases:**
    - Creating a reproducible and automated way to deploy your environments (dev, test, prod).
    - Tracking and managing changes to your infrastructure as code.
    - Ensuring your infrastructure architecture is standardized and compliant.
- **Key Features & Concepts:**
    - **Template:** The YAML/JSON file that defines your resources.
    - **Stack:** The set of AWS resources created from a single template.

---

## CloudFormation Core Concepts

### **Template**

**What It Is:** A YAML or JSON text file that declares the AWS resources you want to create.

**Template Structure:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'My application infrastructure'

Parameters:          # Input values (instance type, environment name, etc.)
Mappings:            # Key-value lookup tables (AMI IDs by region, etc.)
Conditions:          # Conditional resource creation (create DB only in prod)
Resources:           # AWS resources to create (EC2, RDS, S3, etc.) **REQUIRED**
Outputs:             # Values to export (load balancer DNS, bucket name, etc.)
```

**Only Required Section:** `Resources` (all others are optional)

---

#### **Parameters**

**What They Are:** Input values provided when creating/updating a stack

**Use Cases:**
- Make templates reusable (e.g., `InstanceType`, `Environment`)
- Avoid hardcoding values (e.g., `DBPassword`)

**Example:**
```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    Description: EC2 instance type

  Environment:
    Type: String
    AllowedValues:
      - dev
      - prod
```

**Parameter Types:**
- `String`, `Number`, `CommaDelimitedList`
- `AWS::EC2::KeyPair::KeyName` (validates existing key pair)
- `AWS::EC2::VPC::Id` (validates existing VPC)
- `AWS::EC2::Image::Id` (validates AMI ID)

---

#### **Mappings**

**What They Are:** Fixed lookup tables (like a dictionary/map)

**Use Cases:**
- Different values per region (e.g., AMI IDs)
- Different values per environment (e.g., instance sizes)

**Example:**
```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    eu-west-1:
      AMI: ami-0d71ea30463e0ff8d

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref AWS::Region, AMI]
```

**Exam Tip:** Use **Mappings** for fixed values, **Parameters** for user input

---

#### **Conditions**

**What They Are:** Control whether resources are created based on conditions

**Use Cases:**
- Create resources only in prod (e.g., Multi-AZ RDS)
- Create resources based on region (e.g., specific features only in us-east-1)

**Example:**
```yaml
Conditions:
  CreateProdResources: !Equals [!Ref Environment, prod]
  CreateDevResources: !Equals [!Ref Environment, dev]

Resources:
  ProdDatabase:
    Type: AWS::RDS::DBInstance
    Condition: CreateProdResources
    Properties:
      MultiAZ: true

  DevDatabase:
    Type: AWS::RDS::DBInstance
    Condition: CreateDevResources
    Properties:
      MultiAZ: false
```

---

#### **Resources (REQUIRED)**

**What They Are:** AWS resources to create (EC2, S3, RDS, etc.)

**Example:**
```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-unique-bucket-name
      VersioningConfiguration:
        Status: Enabled

  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0c55b159cbfafe1f0
```

**Key Point:** Resource logical names (e.g., `MyBucket`) are used to reference resources within the template

---

#### **Outputs**

**What They Are:** Values to export from the stack (for use by other stacks or for display)

**Use Cases:**
- Export VPC ID for use by other stacks
- Display load balancer DNS name
- Cross-stack references

**Example:**
```yaml
Outputs:
  BucketName:
    Description: Name of the S3 bucket
    Value: !Ref MyBucket
    Export:
      Name: !Sub ${AWS::StackName}-BucketName

  LoadBalancerDNS:
    Description: DNS name of load balancer
    Value: !GetAtt MyLoadBalancer.DNSName
```

**Cross-Stack Reference:**
```yaml
# In another stack, import the bucket name:
BucketName: !ImportValue my-stack-BucketName
```

---

### **Intrinsic Functions**

**Common Functions:**

| Function | Purpose | Example |
|----------|---------|---------|
| `!Ref` | Reference a parameter or resource | `!Ref MyBucket` |
| `!GetAtt` | Get attribute of a resource | `!GetAtt MyInstance.PublicIp` |
| `!FindInMap` | Lookup value in mapping | `!FindInMap [RegionMap, us-east-1, AMI]` |
| `!Sub` | Substitute variables in string | `!Sub 'Bucket: ${BucketName}'` |
| `!Join` | Join strings | `!Join ['-', [app, !Ref Environment]]` |
| `!If` | Conditional value | `!If [CreateProd, t3.large, t3.micro]` |
| `!Equals` | Compare values | `!Equals [!Ref Environment, prod]` |
| `!ImportValue` | Import exported value from another stack | `!ImportValue vpc-stack-VpcId` |

---

## Stack Lifecycle

### **Stack States**

| State | Meaning |
|-------|---------|
| `CREATE_IN_PROGRESS` | Stack is being created |
| `CREATE_COMPLETE` | Stack created successfully |
| `CREATE_FAILED` | Stack creation failed (stack rolled back) |
| `UPDATE_IN_PROGRESS` | Stack is being updated |
| `UPDATE_COMPLETE` | Stack updated successfully |
| `UPDATE_ROLLBACK_COMPLETE` | Update failed, rolled back to previous state |
| `DELETE_IN_PROGRESS` | Stack is being deleted |
| `DELETE_COMPLETE` | Stack deleted successfully |
| `DELETE_FAILED` | Stack deletion failed (some resources couldn't be deleted) |

---

### **Stack Operations**

#### **Create Stack**

**What It Does:** Creates all resources defined in the template

**Behavior:**
- Resources created in dependency order
- If any resource fails, entire stack **rolls back** (all resources deleted)
- Stack enters `CREATE_FAILED` state

**Exam Tip:** Automatic rollback on failure (no partial stacks left behind)

---

#### **Update Stack**

**What It Does:** Modifies existing stack by applying template changes

**Update Behaviors:**

| Change Type | Behavior |
|-------------|----------|
| **No Interruption** | Resource updated without disruption (e.g., add tag) |
| **Some Interruption** | Resource briefly unavailable (e.g., reboot EC2) |
| **Replacement** | Resource deleted and recreated (e.g., change instance type) |

**Update Rollback:**
- If update fails, CloudFormation **automatically rolls back** to previous state
- Stack enters `UPDATE_ROLLBACK_COMPLETE`

**Exam Tip:** Update rollback is automatic (restores previous working state)

---

#### **Delete Stack**

**What It Does:** Deletes all resources in the stack

**Behavior:**
- Resources deleted in reverse dependency order
- **DeletionPolicy** controls what happens to specific resources
- If deletion fails (e.g., S3 bucket not empty), stack enters `DELETE_FAILED`

---

### **DeletionPolicy**

**What It Is:** Controls what happens to a resource when the stack is deleted

**Options:**

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `Delete` (default) | Resource is deleted | Most resources |
| `Retain` | Resource is NOT deleted (orphaned) | Preserve data (S3, RDS) |
| `Snapshot` | Create snapshot before deleting | Backup before deletion (RDS, EBS) |

**Example:**
```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot    # Create snapshot before deleting
    Properties:
      DBInstanceClass: db.t3.micro

  MyBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain      # Keep bucket when stack is deleted
```

**Exam Tip:** Use `Snapshot` for databases, `Retain` for S3 buckets with important data

---

### **UpdateReplacePolicy**

**What It Is:** Controls what happens to a resource when it's **replaced during an update**

**Use Case:** Prevent data loss when CloudFormation replaces a resource (e.g., changing RDS instance class)

**Example:**
```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    UpdateReplacePolicy: Snapshot    # Snapshot before replacing
    DeletionPolicy: Snapshot          # Snapshot before stack deletion
```

---

## Change Sets

**What It Is:** Preview changes before applying a stack update

**How It Works:**
1. Create Change Set (preview)
2. Review what will be added/modified/removed
3. Execute Change Set (apply changes) OR delete it (cancel)

**Benefits:**
- **No surprises** (see exactly what will change)
- **Prevent accidental deletion** (see if resources will be replaced)
- **Approval workflow** (review before applying)

**Change Set Actions:**

| Action | Meaning |
|--------|---------|
| `Add` | New resource will be created |
| `Modify` | Existing resource will be updated |
| `Remove` | Resource will be deleted |
| `Dynamic` | Replacement depends on other changes |

**Example Workflow:**
```bash
# 1. Create Change Set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --change-set-name my-changes

# 2. View Change Set
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# 3. Execute Change Set (apply changes)
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

**Exam Tip:** Use Change Sets to **preview updates before applying** (prevent breaking changes)

---

## Drift Detection

**What It Is:** Detects when stack resources have been **manually changed** outside of CloudFormation

**Why It Matters:**
- Someone manually modified EC2 instance via console
- Configuration drift (actual state ≠ template)
- CloudFormation unaware of manual changes

**How It Works:**
1. Run Drift Detection on stack
2. CloudFormation compares actual resource state with template
3. Reports which resources have drifted

**Drift Statuses:**

| Status | Meaning |
|--------|---------|
| `IN_SYNC` | Resource matches template (no drift) |
| `MODIFIED` | Resource manually changed (drifted) |
| `DELETED` | Resource was manually deleted |
| `NOT_CHECKED` | Drift detection not supported for this resource type |

**Example:**
```bash
# Start drift detection
aws cloudformation detect-stack-drift --stack-name my-stack

# View drift results
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id <detection-id>
```

**Exam Tip:** Drift Detection **does not fix drift** (only detects it). You must manually reconcile or update the stack.

---

## Nested Stacks

**What It Is:** A CloudFormation stack created from within another stack (modular templates)

**Why Use Nested Stacks:**
- **Break large templates into smaller, reusable components**
- **Exceed template size limit** (1 MB for S3-uploaded templates)
- **Reuse common patterns** (VPC, security groups, etc.)

**How It Works:**
- Parent stack creates child stacks
- Child stacks are full CloudFormation stacks (can be updated independently)
- Outputs from child stacks can be referenced by parent

**Example:**
```yaml
# Parent Stack
Resources:
  VPCStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/vpc-template.yaml
      Parameters:
        VpcCIDR: 10.0.0.0/16

  AppStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/app-template.yaml
      Parameters:
        VpcId: !GetAtt VPCStack.Outputs.VpcId
```

**Nested Stack vs. Cross-Stack Reference:**

| Feature | Nested Stacks | Cross-Stack Reference |
|---------|---------------|----------------------|
| **Relationship** | Parent-child (hierarchy) | Independent stacks |
| **Lifecycle** | Child deleted when parent deleted | Independent lifecycles |
| **Use Case** | Modular components (VPC, App, DB) | Shared resources (VPC used by multiple apps) |

**Exam Tip:** Nested Stacks = **modular components**, Cross-Stack = **shared resources**

---

## StackSets

**What It Is:** Deploy a single CloudFormation template across **multiple AWS accounts and regions**

**Use Cases:**
- **Multi-account deployments** (deploy same infrastructure to 50 accounts)
- **Multi-region deployments** (deploy to all regions at once)
- **Organizational standards** (enforce security baseline across all accounts)

**How It Works:**
1. Create StackSet with template
2. Specify **target accounts** and **target regions**
3. CloudFormation deploys stacks to all specified accounts/regions
4. Update StackSet → all stacks updated automatically

**StackSet Concepts:**

| Concept | Description |
|---------|-------------|
| **StackSet** | The template and configuration |
| **Stack Instances** | Individual stacks created in each account/region |
| **Administrator Account** | Account that manages the StackSet |
| **Target Accounts** | Accounts where stacks are deployed |

**Example:**
- StackSet: Security baseline (GuardDuty, CloudTrail, Config)
- Target Accounts: All 50 AWS accounts in organization
- Target Regions: us-east-1, eu-west-1
- **Result:** 100 stack instances (50 accounts × 2 regions)

**Permissions:**

| Permission Model | Description |
|-----------------|-------------|
| **Self-Managed** | Manual IAM role setup (AWSCloudFormationStackSetAdministrationRole, AWSCloudFormationStackSetExecutionRole) |
| **Service-Managed** | Automatic permissions via AWS Organizations (recommended) |

**Exam Tip:** StackSets = **multi-account/multi-region** deployments (single template everywhere)

---

## Stack Policies

**What It Is:** JSON document that defines which resources can be updated/deleted

**Use Cases:**
- **Prevent accidental deletion** of critical resources (production database)
- **Prevent updates** to specific resources (immutable infrastructure)

**Default:** If no stack policy, all resources can be updated

**Example:**
```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:Delete",
      "Resource": "LogicalResourceId/ProductionDatabase"
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    }
  ]
}
```

**Behavior:**
- **Deny takes precedence** over Allow
- Once set, stack policy **cannot be deleted** (only modified)

**Exam Tip:** Stack Policies **prevent accidental updates/deletions** (protection layer)

---

## Rollback Configuration

**What It Is:** Automatically roll back stack on CloudWatch alarm trigger

**Use Cases:**
- Deploy new version, but roll back if error rate increases
- Deploy infrastructure, but roll back if health checks fail

**Example:**
```yaml
RollbackConfiguration:
  RollbackTriggers:
    - Arn: arn:aws:cloudwatch:us-east-1:123456789012:alarm:HighErrorRate
      Type: AWS::CloudWatch::Alarm
  MonitoringTimeInMinutes: 5
```

**Exam Tip:** Rollback Configuration = **automatic rollback on alarm** (deployment safety)

---

## CloudFormation Best Practices

| Best Practice | Why |
|---------------|-----|
| **Use Change Sets** | Preview changes before applying |
| **Use Parameters** | Make templates reusable (avoid hardcoding) |
| **Use Outputs** | Enable cross-stack references |
| **Use DeletionPolicy: Retain** | Prevent accidental data loss (S3, RDS) |
| **Use Nested Stacks** | Break large templates into modules |
| **Use StackSets** | Deploy to multiple accounts/regions |
| **Use Drift Detection** | Find manual changes |
| **Use Stack Policies** | Protect critical resources from updates |
| **Version Templates** | Store in S3 or Git for version control |
| **Test in Dev First** | Validate templates before prod deployment |

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| "Deploy same infrastructure to 50 AWS accounts" | **StackSets** (multi-account deployment) |
| "Preview changes before updating production stack" | **Change Sets** (preview before apply) |
| "Find resources that were manually modified" | **Drift Detection** |
| "Prevent accidental deletion of production database" | **DeletionPolicy: Retain** or **Stack Policy** |
| "Break 2 MB template into smaller pieces" | **Nested Stacks** (modular templates) |
| "Reference VPC ID from another stack" | **Outputs** + **ImportValue** (cross-stack reference) |
| "Deploy to us-east-1 and eu-west-1 simultaneously" | **StackSets** (multi-region deployment) |
| "Create snapshot of RDS before stack deletion" | **DeletionPolicy: Snapshot** |
| "Rollback deployment if error rate increases" | **Rollback Configuration** with CloudWatch Alarm |
| "Prevent updates to critical resources" | **Stack Policy** (deny updates) |
| "Make template reusable for dev/prod environments" | **Parameters** (Environment, InstanceType, etc.) |
| "Use different AMI IDs per region" | **Mappings** (RegionMap with AMI IDs) |
| "Create Multi-AZ RDS only in production" | **Conditions** (based on Environment parameter) |

---

## Key Exam Tips

1. **Template = YAML/JSON file**, Stack = **resources created from template**
2. **Only required section:** `Resources` (all others optional)
3. **Parameters:** User input (instance type, environment)
4. **Mappings:** Fixed lookup tables (AMI IDs by region)
5. **Conditions:** Conditional resource creation (prod vs dev)
6. **Outputs:** Export values for cross-stack references
7. **Change Sets:** Preview updates before applying (no surprises)
8. **Drift Detection:** Find manual changes (does NOT fix them)
9. **Nested Stacks:** Modular templates (parent-child relationship)
10. **StackSets:** Multi-account/multi-region deployments
11. **DeletionPolicy:** Control resource deletion (`Delete`, `Retain`, `Snapshot`)
12. **UpdateReplacePolicy:** Control resource replacement during updates
13. **Stack Policy:** Prevent accidental updates/deletions
14. **Rollback:** Automatic on create/update failure
15. **Intrinsic Functions:** `!Ref`, `!GetAtt`, `!Sub`, `!FindInMap`, `!ImportValue`

---

## Template Example (Complete)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web application infrastructure'

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, prod]
    Default: dev

  InstanceType:
    Type: String
    Default: t3.micro

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    eu-west-1:
      AMI: ami-0d71ea30463e0ff8d

Conditions:
  CreateProdResources: !Equals [!Ref Environment, prod]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      BucketName: !Sub '${AWS::StackName}-bucket'

  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]

  MyDatabase:
    Type: AWS::RDS::DBInstance
    Condition: CreateProdResources
    DeletionPolicy: Snapshot
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MultiAZ: true

Outputs:
  BucketName:
    Description: S3 bucket name
    Value: !Ref MyBucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'

  InstanceId:
    Description: EC2 instance ID
    Value: !Ref MyInstance
```
