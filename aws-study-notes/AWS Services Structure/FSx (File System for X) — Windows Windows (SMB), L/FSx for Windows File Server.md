- Provides a fully managed native Windows file system using the **SMB (Server Message Block) protocol**. **Use when** you need to lift-and-shift a Windows application that relies on a shared SMB file share.
- It is built on Windows Server and directly supports all the features of a Windows file environment, including:
    - **Distributed File System (DFS):** It natively supports both DFS Namespaces and DFS Replication. This allows the organization to extend its existing on-premises DFS namespace into AWS, creating a seamless, unified file system view for users and applications in the hybrid environment.
    - **Active Directory (AD) Integration:** It integrates with AWS Managed Microsoft AD or a self-managed Active Directory, which is essential for managing permissions and authentication in a Windows environment.
    - **High Performance and Scalability:** It is designed for data-intensive workloads and can scale to meet the performance and capacity needs of hundreds of petabytes of data.
- **Must-Know Features**
    - **Active Directory (AD) Integration**
        - It is **mandatory**. FSx for Windows *must* be joined to an Active Directory domain for authentication.
        - **Two Options:**
            1. **AWS Managed Microsoft AD:** The easy, native AWS solution.
            2. **On-Premises AD:** Connect via **VPN** or **Direct Connect**.
    - **High Availability (HA) & Deployment**
        
        
        |  |  | **Use Case** |
        | --- | --- | --- |
        | **Single-AZ** | A single file server in one Availability Zone | Dev/test, non-critical workloads. If it fails, AWS replaces it automatically, but there is downtime. |
        | **Multi-AZ** | A primary server in one AZ and a **standby server in another AZ**. Data is **synchronously replicated** | Production workloads. Failover is **automatic and sub-minute**. This is the key option for high availability. |
    - **Hybrid Cloud & Migration**
        - **DFS Namespaces:** This is the key feature for hybrid. You can use FSx to create a single, logical namespace (\\your-corp\shares) that points to folders on both your on-premises servers and your FSx file system.
        - **AWS DataSync:** This is the **recommended tool** for performing a large-scale, one-time or recurring migration of data from your on-premises file servers *to* FSx for Windows.
    - **Performance & Cost**
        - **Storage Tiers:**
            - **SSD:** For **latency-sensitive** and high-performance workloads (databases, user profiles).
            - **HDD:** For **cost-sensitive**, throughput-focused workloads (backups, media archives).
        - **Throughput Capacity:** You provision this **separately** from storage capacity. If an application is slow, you might need to increase the throughput capacity.
    - **Security**
        - **Encryption:** Encrypted **by default** both **at-rest** (using AWS KMS) and **in-transit** (using SMB 3.0).
        - **Network Access:** Controlled by **VPC Security Groups**, just like an EC2 instance. You must allow traffic on the SMB port (TCP 445).
- **FSx for Windows vs. EFS**
    
    
    | **Feature** | **Amazon FSx for Windows** | **Amazon EFS** |
    | --- | --- | --- |
    | **Primary OS** | **Windows** | **Linux** |
    | **Protocol** | **SMB** | **NFS** |
    | **Permissions** | **Active Directory** / NTFS ACLs | POSIX Permissions / IAM |
    | **Best For** | Windows applications, .NET apps, user shares, hybrid DFS. | Linux-based web servers, CMS, Big Data, container storage. |
- **Summary Cheat Sheet**
    
    
    | **If the question asks for...** | **Your answer is likely...** |
    | --- | --- |
    | Shared storage for a **Windows/.NET** application. | **FSx for Windows** |
    | Shared storage for a **Linux/LAMP** stack application. | **Amazon EFS** |
    | A way to **extend on-premises DFS** to AWS for hybrid file access. | **FSx for Windows** |
    | **Production-grade resilience** and automatic failover for a file share. | **FSx for Windows Multi-AZ** |
    | File access controlled by **Active Directory users and groups**. | **FSx for Windows** |
    | A way to **migrate petabytes** of file data from on-prem to AWS. | **AWS DataSync** (to an FSx or S3 target) |