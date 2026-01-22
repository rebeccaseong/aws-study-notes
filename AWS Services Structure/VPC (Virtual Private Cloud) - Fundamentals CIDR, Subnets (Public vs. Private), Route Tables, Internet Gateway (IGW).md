- **What is it? (1-Sentence Pitch):** Your own private, isolated section of the AWS cloud where you can launch resources in a virtual network that you define.
- **Core VPC Components**
    
    
    ![image.png](VPC%20(Virtual%20Private%20Cloud)%20Fundamentals%20CIDR,%20Sub/image.png)
    
    ![image.png](VPC%20(Virtual%20Private%20Cloud)%20Fundamentals%20CIDR,%20Sub/image%201.png)
    
    | **Component** | **What It Is** | **Why It's Important (SAA Focus)** |
    | --- | --- | --- |
    | **VPC (Virtual Private Cloud)** | Your logically isolated, virtual network in the AWS Cloud. | It's the fundamental container for all your resources. The most critical decision is its **CIDR block** (e.g., 10.0.0.0/16). |
    | **Subnets** | A range of IP addresses within your VPC. A subnet must reside entirely within one **Availability Zone (AZ)**. | This is how you achieve **High Availability (HA)**. By placing subnets (and resources) in different AZs, your application can survive an AZ failure. |
    | **Public Subnet** | A subnet whose associated Route Table has a route to an **Internet Gateway (IGW)**. | For resources that need to be directly accessible from the internet (e.g., public-facing web servers, bastion hosts, public Application Load Balancers). |
    | **Private Subnet** | A subnet that does *not* have a direct route to the internet. | For resources that should **not** be exposed to the internet, increasing security (e.g., databases, backend application servers). |
    | **Route Tables** | A set of rules, called routes, that determine where network traffic from your subnet is directed. | The "GPS" of your VPC. A misconfigured route table is a common cause of connectivity issues. Each subnet is associated with exactly one route table. |
    | **Internet Gateway (IGW)** | A horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. | It's the "front door" for all inbound/outbound internet traffic for your public subnets. You attach one IGW to a VPC. |
    | **NAT Gateway (Managed)** | A managed AWS service that allows instances in a **private subnet** to initiate outbound traffic to the internet (e.g., for patching, API calls) but prevents the internet from initiating a connection with those instances. | The modern, HA, and preferred way for private instances to access the internet. It's deployed in a **public subnet** and must have an Elastic IP. |
    | **NAT Instance (Self-Managed)** | An EC2 instance you manage yourself that performs the same function as a NAT Gateway. | The "old way." The exam will test you on the differences. It's a single point of failure unless you script your own HA, and you manage patching/scaling. **Always choose NAT Gateway unless there's a specific reason not to.** |
    | **Security Groups (SGs)** | A stateful firewall for your EC2 instances. Acts at the **instance level**. | **Stateful:** If you allow inbound traffic, outbound traffic for that connection is automatically allowed. You create **allow rules only**. All SGs deny traffic by default. |
    | **Network ACLs (NACLs)** | A stateless firewall for your subnets. Acts at the **subnet level**. | **Stateless:** You must create explicit rules for both inbound and outbound traffic. You can create both **allow and deny rules**. They are processed in numerical order. |
    | **VPC Endpoints** | Privately connect your VPC to supported AWS services without requiring an IGW, NAT Gateway, VPN, or Direct Connect. | **Crucial for security and cost.** It keeps traffic within the AWS network. There are two types: **Gateway Endpoints** (S3, DynamoDB) and **Interface Endpoints** (most other services, uses an ENI and a private IP). |
    | **VPC Peering** | A network connection between two VPCs that enables you to route traffic between them using private IP addresses. | For connecting a small number of VPCs. It's not transitive (if A is peered with B, and B with C, A cannot talk to C). Can work across accounts and regions. |
    | **Transit Gateway (TGW)** | A network transit hub that you can use to interconnect your VPCs and on-premises networks. | The modern solution for connecting many VPCs. It acts as a cloud router, simplifying network management. It's a "hub and spoke" model, solving the non-transitive issue of peering. |
- Classic SAA Scenario: The 3-Tier Web Application
    1. **VPC:** A single VPC (e.g., 10.0.0.0/16).
    2. **Subnets:**
        - **Public Subnets (in 2 AZs):** For an Application Load Balancer (ALB).
        - **Private Subnets (in 2 AZs):** For your EC2 Web/App Servers.
        - **Private/Database Subnets (in 2 AZs):** For your RDS Database.
    3. **Routing:**
        - **Public Route Table:** Has a route 0.0.0.0/0 -> igw-xxxxxxxx. Associated with public subnets.
        - **Private Route Table:** Has a route 0.0.0.0/0 -> nat-xxxxxxxx. Associated with private web/app subnets (so they can download updates).
        - **Database Route Table:** May have no default route at all for maximum security.
    4. **Security:**
        - **ALB Security Group:** Allows Port 80/443 from 0.0.0.0/0.
        - **Web Server Security Group:** Allows Port 80/443 **only from the ALB's Security Group**.
        - **Database Security Group:** Allows Port 3306 (MySQL) **only from the Web Server's Security Group**.
        
        ![image.png](AWS%20Services%20Structure/0%20Concept/Availability%20Zone%20and%20Region/image.png)
        
- **Design & Best Practices**
    1. **CIDR Planning is EVERYTHING:**
        - **Don't Overlap:** Never use overlapping CIDR blocks for VPCs you might ever want to connect (via peering or TGW). Use the RFC 1918 private address spaces (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16).
        - **Think Ahead:** A /16 gives you ~65,000 IPs. This is usually plenty. A /24 gives you ~250. Don't paint yourself into a corner. Plan for growth.
        - **Subnet Sizing:** Use smaller CIDR blocks for subnets (e.g., /24). Remember AWS reserves 5 IP addresses in every subnet.
    2. **Tagging is Your Best Friend:** Tag everything—VPCs, subnets, route tables, IGWs, NAT Gateways. Use tags for cost allocation, automation, and identification (e.g., Environment: Prod, Project: Phoenix, Owner: a.smith).
    3. **High Availability is Default:**
        - Always design with at least two AZs.
        - For NAT Gateways, deploy one **in each AZ** and configure route tables so that resources use the NAT Gateway in their own AZ. This avoids cross-AZ data transfer costs and creates a resilient architecture.
-