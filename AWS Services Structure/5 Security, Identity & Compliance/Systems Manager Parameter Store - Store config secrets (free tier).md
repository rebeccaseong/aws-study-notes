- **What is it? (1-Sentence Pitch):** AWS Systems Manager Parameter Store provides secure, hierarchical storage for configuration data and secrets (passwords, database strings, license codes) that can be referenced by AWS services and applications.

- **Core Use Cases:**
    - Storing application configuration parameters (database connection strings, API endpoints)
    - Storing secrets (passwords, API keys) with encryption
    - Managing parameters across environments (dev, staging, prod)
    - Centralized configuration management for Lambda functions, EC2 instances, etc.
    - Storing AMI IDs, license codes, and other reference data

- **Key Features & Concepts:**
    - **Hierarchical Storage:** Organize parameters in a tree structure (e.g., `/myapp/dev/db-url`, `/myapp/prod/db-url`)
    - **Three Types of Parameters:**
        1. **String:** Plain text values
        2. **StringList:** Comma-separated values
        3. **SecureString:** Encrypted using KMS (for secrets like passwords)
    - **Versioning:** Track changes to parameter values over time
    - **Free Tier:** First 10,000 parameters are free (standard parameters)
    - **Advanced Parameters:** Higher throughput, more storage, parameter policies (additional cost)
    - **Parameter Policies:** Expiration notifications, TTL, etc. (advanced parameters only)
    - **Integration with AWS Services:** Lambda, ECS, EC2, CloudFormation, CodeBuild, etc.

- **Standard vs Advanced Parameters:**
    | **Aspect** | **Standard** | **Advanced** |
    |---|---|---|
    | **Cost** | **Free** (up to 10,000) | Charges apply |
    | **Max Value Size** | 4 KB | 8 KB |
    | **Max Parameters** | 10,000 | 100,000 |
    | **Parameter Policies** | No | **Yes** (expiration, notifications) |
    | **Use Case** | Most use cases | High-throughput, large configs |

- **Parameter Store vs Secrets Manager:**
    | **Aspect** | **Parameter Store** | **Secrets Manager** |
    |---|---|---|
    | **Cost** | **Free** (standard tier) | **Paid** ($0.40 per secret/month) |
    | **Use Case** | Configuration + simple secrets | **Database credentials**, API keys |
    | **Rotation** | Manual | **Automatic rotation** (Lambda) |
    | **Encryption** | KMS (optional) | KMS (mandatory) |
    | **Integration** | All AWS services | RDS, Redshift, DocumentDB |
    | **Complexity** | Simple | More features, higher cost |

- **How Parameter Store Works:**
    1. **Store:** Create parameter in Parameter Store (`/myapp/db-password`, type: SecureString, KMS-encrypted)
    2. **Reference:** Application or AWS service retrieves parameter via API or SDK
    3. **Update:** Change parameter value (new version created)
    4. **Audit:** CloudTrail logs all parameter access and changes

- **Integration:**
    - **Lambda:** Retrieve parameters at function invocation (environment variables or SDK)
    - **EC2:** Use Systems Manager Agent to retrieve parameters
    - **ECS/Fargate:** Reference parameters in task definitions
    - **CloudFormation:** Use dynamic references to retrieve parameters
    - **CodeBuild/CodePipeline:** Retrieve parameters during build/deploy
    - **KMS:** Encrypt SecureString parameters
    - **CloudTrail:** Audit parameter access and modifications

- **Exam Tips / What to Remember:**
    - **Parameter Store is free** (standard tier, up to 10,000 parameters)
    - Use for **configuration management** and **simple secrets**
    - **SecureString type uses KMS encryption** for sensitive data
    - **Hierarchical naming** enables organized parameter management (`/app/env/key`)
    - Exam scenario: **"Store database password cheaply"** = Parameter Store (SecureString)
    - **Secrets Manager is better for automatic rotation** of secrets (e.g., RDS passwords)
    - **Parameter Store is simpler and cheaper**, **Secrets Manager has more features**
    - Widely integrated with AWS services (Lambda, EC2, ECS, CloudFormation, etc.)
    - **Versioning** allows tracking parameter changes over time
