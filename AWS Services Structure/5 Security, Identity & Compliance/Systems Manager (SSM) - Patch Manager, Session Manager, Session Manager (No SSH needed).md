- **What is it? (1-Sentence Pitch):** AWS Systems Manager (SSM) is a unified operational toolkit that allows you to securely manage, automate, and gain insights into your fleet of servers (both EC2 and on-premises) without needing to use bastion hosts or direct SSH/RDP access.
- **How it Works (The Prerequisites)**
    - For SSM to manage an instance (called a "Managed Instance"), three things are required. This is a common source of troubleshooting.
        1. **The SSM Agent:** The agent must be installed and running on the instance. It is pre-installed on Amazon Linux 2 and some Windows AMIs, but may need to be manually installed on others.
        2. **IAM Instance Profile:** The EC2 instance must have an IAM Role attached with the necessary permissions. The standard managed policy is **AmazonSSMManagedInstanceCore**.
        3. **Network Connectivity:** The instance must be able to communicate with the SSM service endpoints. This can be via the internet (e.g., in a public subnet or through a NAT Gateway) or, more securely, via **VPC Endpoints**.
- **Key SSM Capabilities (The Components)**
    | **Capability** | **What it is** | **Key Use Case & Exam Keywords** |
    | --- | --- | --- |
    | **Session Manager** | Secure, browser-based shell or CLI access to your instances. | **Replacing Bastion Hosts.** "Securely access a private EC2 instance without opening SSH/RDP ports." All sessions can be logged to S3 or CloudWatch for auditing. |
    | **Run Command** | **Ad-hoc** execution of commands or scripts on a fleet of instances. | "Install an application or restart a service on 100 servers at once." Used for one-time tasks. |
    | **Patch Manager** | Automated OS and application patching based on rules and schedules. | "Automate the patching of all 'Production' instances for critical security updates every Tuesday at 2 AM." Uses "Patch Baselines" to define approved patches. |
    | **Parameter Store** | A secure, centralized store for configuration data and secrets. | "Store a database connection string, license key, or API endpoint." Can store values as Plaintext or SecureString (encrypted by KMS). |
    | **State Manager** | **Desired State Configuration.** Ensures instances are always in a defined state. | "Ensure an antivirus agent is *always* installed and running on all servers." It re-applies the configuration if it ever deviates. |
    | **Inventory** | Automatically collects metadata from your managed instances. | "Get a list of all instances running a specific version of NGINX" or "Find all servers missing a specific Windows patch." |
    | **Maintenance Windows** | Defines a scheduled window of time to perform potentially disruptive actions. | "Define a 4-hour window on Saturday nights where it is safe for Patch Manager and Run Command to execute." |
- **Parameter Store vs. Secrets Manager**
    | **Feature** | **SSM Parameter Store** | **AWS Secrets Manager** |
    | --- | --- | --- |
    | **Cost** | **Standard tier is free.** Higher tier has a cost. | **Costs per secret per month** + per API call. |
    | **Primary Use** | Configuration data (endpoints, AMI IDs) and basic secrets. | Primarily for secrets, especially database credentials. |
    | **Key Rotation** | No built-in rotation. You must build it with custom Lambda functions. | **Automatic, built-in secret rotation** with Lambda. |
    | **Cross-Account** | Possible, but more complex to set up. | Simpler cross-account access via resource-based policies. |
- **Security & Networking Deep Dive**
    - **Replacing Bastion Hosts:** The primary benefit of **Session Manager** is security. By eliminating bastion hosts, you eliminate a server you need to patch and secure, and you completely close off inbound SSH (port 22) and RDP (port 3389) ports in your security groups, dramatically reducing your attack surface.
    - **VPC Endpoints for SSM:** For instances in a private subnet with no internet access, you must create VPC Endpoints to allow them to communicate with the SSM APIs privately. This is a critical security best practice. You need three endpoints:
        - com.amazonaws.<region>.ssm (for the main SSM service)
        - com.amazonaws.<region>.ssmmessages (for Session Manager)
        - com.amazonaws.<region>.ec2messages (core communication)
- **Hybrid Cloud Management**
    - SSM is not just for EC2. It can manage **on-premises servers** and virtual machines in other clouds (e.g., Azure, GCP).
    - **Process:** You generate an **Activation Code** in SSM. You then install the SSM Agent on the on-prem server and run a command with the activation code. The server registers with SSM and becomes a "Managed Instance," just like an EC2 instance.
- Exam Cheat Sheet
    | **If the question asks for...** | **The answer is almost always...** | **Key Concept** |
    | --- | --- | --- |
    | Securely accessing private instances **without SSH/RDP** | **SSM Session Manager** | Eliminates the need for bastion hosts and open inbound ports (22, 3389). |
    | Executing a **one-time command** or script on many servers | **SSM Run Command** | Ad-hoc, remote execution for tasks like restarting a service or installing an app. |
    | **Automated OS patching** on a schedule | **SSM Patch Manager** | Uses "Patch Baselines" and "Maintenance Windows" to keep fleets of servers up-to-date. |
    | **Storing configuration data** or basic secrets for free | **SSM Parameter Store** | Centralized management of strings like database URLs, license keys, or AMI IDs. |
    | **Ensuring a consistent state** or desired configuration | **SSM State Manager** | "Desired State Configuration." Automatically re-applies configurations if they drift. |
    | Getting an **inventory of installed software** or configurations | **SSM Inventory** | Collects metadata from managed instances to answer questions like "which servers have this application?" |
    | An instance **cannot be managed** by SSM | **Check the prerequisites:** 
    1. **SSM Agent** (installed & running) 
    2. **IAM Role** (with AmazonSSMManagedInstanceCore) 3. **Network Connectivity** (to SSM endpoints) |  |