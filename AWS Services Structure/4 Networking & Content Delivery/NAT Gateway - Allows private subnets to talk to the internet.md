- **What is it? (1-Sentence Pitch):** NAT Gateway is a managed Network Address Translation (NAT) service that allows instances in private subnets to initiate outbound connections to the internet while preventing inbound connections from the internet.

- **Core Use Cases:**
    - Enabling EC2 instances in private subnets to download software updates
    - Allowing private instances to access external APIs or services
    - Egress-only internet access (no inbound traffic allowed)
    - Meeting security requirements by keeping instances in private subnets

- **Key Features & Concepts:**
    - **Managed Service:** Highly available (within a single AZ), AWS handles maintenance
    - **Deployed in Public Subnet:** NAT Gateway must be in a public subnet with an Elastic IP
    - **Private Subnet Route Table:** Route `0.0.0.0/0` traffic to NAT Gateway
    - **Outbound Only:** Allows private instances to initiate outbound connections, blocks inbound
    - **Per-AZ Resource:** Deploy one NAT Gateway per AZ for high availability
    - **Bandwidth:** Automatically scales up to 100 Gbps
    - **No Security Groups:** NAT Gateways don't have security groups (controlled by route tables)

- **NAT Gateway vs NAT Instance vs Internet Gateway:**
    | **Aspect** | **NAT Gateway** | **NAT Instance** | **Internet Gateway (IGW)** |
    |---|---|---|---|
    | **Managed** | Fully managed by AWS | You manage EC2 instance | Fully managed by AWS |
    | **Availability** | Highly available within AZ | Single point of failure (unless using ASG) | Highly available across AZs |
    | **Bandwidth** | Scales to 100 Gbps | Depends on instance type | No limit |
    | **Cost** | Hourly charge + data processed | EC2 instance cost | Free |
    | **Use Case** | **Private subnet outbound** | Legacy/custom NAT | **Public subnet bi-directional** |
    | **Maintenance** | None (AWS handles) | You patch/update | None |

- **How NAT Gateway Works:**
    1. Instance in **private subnet** wants to reach the internet
    2. Route table sends traffic (`0.0.0.0/0`) to **NAT Gateway** in public subnet
    3. NAT Gateway translates private IP to its Elastic IP
    4. Traffic goes through **Internet Gateway** to the internet
    5. Response comes back, NAT Gateway forwards to the private instance

- **Integration:**
    - **VPC:** Deployed within a VPC's public subnet
    - **Elastic IP:** Requires an Elastic IP address (static public IP)
    - **Route Tables:** Private subnet route tables point to NAT Gateway
    - **Internet Gateway:** NAT Gateway uses IGW to reach the internet
    - **CloudWatch:** Monitor NAT Gateway metrics (bytes processed, packets, connection count)

- **Exam Tips / What to Remember:**
    - **NAT Gateway allows outbound internet access from private subnets**
    - Must be deployed in a **public subnet** and requires an **Elastic IP**
    - **Deploy one NAT Gateway per AZ** for high availability (costs more, but resilient)
    - **Internet Gateway (IGW)** is for **public subnets** (bi-directional), **NAT Gateway** is for **private subnets** (outbound only)
    - Exam scenario: **"Private instances need to download updates from the internet"** = NAT Gateway
    - **NAT Gateway is preferred over NAT Instance** (fully managed, more reliable)
    - NAT Gateway charges: **hourly** + **per GB of data processed**
    - Cannot be used as a bastion host (use bastion host in public subnet for SSH access)
