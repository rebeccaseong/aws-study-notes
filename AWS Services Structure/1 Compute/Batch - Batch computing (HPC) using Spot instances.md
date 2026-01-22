- **What is it? (1-Sentence Pitch):** AWS Batch enables you to run batch computing workloads (hundreds of thousands of jobs) efficiently by dynamically provisioning the optimal quantity and type of compute resources based on the volume and requirements of the jobs.

- **Core Use Cases:**
    - Large-scale parallel processing (genomics, financial risk modeling, image/video processing)
    - ETL (Extract, Transform, Load) jobs
    - Scientific simulations and Monte Carlo simulations
    - Machine learning model training (batch inference)
    - Log analysis and data processing

- **Key Features & Concepts:**
    - **Fully Managed:** AWS Batch handles job scheduling, compute provisioning, and scaling
    - **Batch Jobs:** Units of work (containerized applications) submitted to job queues
    - **Job Queues:** Jobs wait here until compute resources are available
    - **Compute Environments:** The compute resources (EC2 or Fargate) where jobs run
        - **Managed:** AWS Batch provisions and scales EC2 instances automatically
        - **Unmanaged:** You manage your own compute resources
    - **Job Definitions:** Specify how jobs are to run (Docker image, vCPUs, memory, IAM role)
    - **Spot Instance Integration:** Can use EC2 Spot Instances for **up to 90% cost savings**
    - **Array Jobs:** Run multiple copies of the same job with different parameters

- **Integration:**
    - **EC2 and Fargate:** Run batch jobs on EC2 instances or Fargate (serverless)
    - **Spot Instances:** Leverage spot instances for cost-effective batch processing
    - **ECS:** Built on top of Amazon ECS (uses ECS task definitions)
    - **CloudWatch Events/EventBridge:** Trigger jobs based on events
    - **Step Functions:** Orchestrate complex workflows involving Batch jobs
    - **S3:** Input/output data typically stored in S3

- **AWS Batch vs Lambda:**
    | **Aspect** | **AWS Batch** | **Lambda** |
    |---|---|---|
    | **Max Runtime** | No time limit | 15 minutes max |
    | **Use Case** | Long-running batch jobs | Short, event-driven tasks |
    | **Compute** | EC2 or Fargate | Serverless (managed by AWS) |
    | **Complexity** | Complex, resource-intensive jobs | Simple, stateless functions |
    | **Dependencies** | Can have complex dependencies | Limited dependencies |

- **Exam Tips / What to Remember:**
    - **AWS Batch is for long-running, batch processing workloads** (think hours, not seconds)
    - **Fully managed** - you don't provision or scale compute resources manually
    - **Best for HPC (High-Performance Computing)** workloads that can tolerate interruptions
    - **Leverages Spot Instances** for massive cost savings (up to 90%)
    - Built on **ECS** (uses containerized applications)
    - Exam scenario: **"Process millions of images/files in parallel"** = AWS Batch
    - **No time limit** (unlike Lambda's 15 minutes)
    - Jobs can have **dependencies** (Job B waits for Job A to complete)
