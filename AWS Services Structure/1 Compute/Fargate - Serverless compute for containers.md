- **What is it? (1-Sentence Pitch):** AWS Fargate is a serverless compute engine for containers that runs your Docker containers without requiring you to manage the underlying EC2 instances.

- **Core Use Cases:**
    - Running microservices without managing servers
    - Batch processing jobs in containers
    - CI/CD pipelines (build agents, test runners)
    - Event-driven container workloads

- **Key Features & Concepts:**
    - **Serverless Containers:** No EC2 instances to manage, provision, or patch
    - **Works with ECS and EKS:** Can use Fargate as the compute engine for both Amazon ECS and Amazon EKS
    - **Task/Pod Definition:** You define CPU and memory requirements, Fargate handles the rest
    - **Pricing:** Pay only for the vCPU and memory resources your containers use (per-second billing)
    - **Isolation:** Each task runs in its own isolated environment
    - **Integration with AWS Services:** Works seamlessly with VPC, IAM, CloudWatch, ALB/NLB

- **Integration:**
    - **ECS:** Fargate is a launch type for ECS tasks
    - **EKS:** Fargate is a compute option for EKS pods
    - **Load Balancers:** Integrates with ALB/NLB for traffic distribution
    - **CloudWatch:** Logs and metrics for monitoring
    - **IAM:** Task execution roles and task roles for permissions

- **Fargate vs EC2 Launch Type (ECS):**
    | **Aspect** | **Fargate** | **EC2** |
    |---|---|---|
    | **Management** | Serverless, no instances to manage | You manage the EC2 instances |
    | **Pricing** | Pay per task (vCPU + memory) | Pay for EC2 instances (even if idle) |
    | **Best For** | Variable workloads, minimal ops | Steady-state, cost optimization with Reserved Instances |
    | **Control** | Less control over infrastructure | Full control over instances |

- **Exam Tips / What to Remember:**
    - Fargate is the **"serverless"** option for containers (no EC2 management)
    - Works with **both ECS and EKS**
    - You define **CPU and memory** in the task definition
    - **More expensive than EC2** on a per-hour basis, but no idle capacity waste
    - Great for **spiky or unpredictable workloads** where you don't want to over-provision
    - Each Fargate task has its **own Elastic Network Interface (ENI)** in your VPC
    - Exam favorite: **"Run containers without managing servers" = Fargate**
