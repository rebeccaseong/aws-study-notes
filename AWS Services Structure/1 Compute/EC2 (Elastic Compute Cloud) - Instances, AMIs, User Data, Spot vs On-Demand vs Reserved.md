- **What is it? (1-Sentence Pitch):**EC2 provides secure, resizable **virtual servers** (called instances) in the cloud, allowing you to scale your computing capacity up or down to meet changing demands with minimal friction.
- **Core EC2 Components (The Building Blocks)**
    - **Amazon Machine Image (AMI):** The software template used to create an instance. It includes the Operating System (e.g., Linux, Windows), any pre-installed software, and configurations. You can use AWS-provided AMIs, community AMIs, or create your own custom AMIs.
    - **Instance Type:** The hardware profile of your instance. This defines the CPU, memory, storage, and networking capacity (e.g., t3.micro, m6g.large).
    - **Security Group:** A stateful, virtual firewall at the **instance level**. It controls what inbound and outbound traffic is allowed. By default, all inbound traffic is denied, and all outbound traffic is allowed.
    - **Key Pair:** A set of public-private keys used to securely connect to your Linux instances via SSH. AWS stores the public key, and you are responsible for securely storing the private key.
- **Instance Families (CRITICAL KNOWLEDGE)**


    | **Family** | **Mnemonic** | **Use Case / Optimized For** | **Example** |
    | --- | --- | --- | --- |
    | **General Purpose** | **M**y **T**all | A balance of compute, memory, and networking. Good for web servers, dev/test environments. | M series, T series (burstable) |
    | **Compute Optimized** | **C**omputer | High-performance processors. Good for CPU-intensive tasks like batch processing, media transcoding, HPC. | C series |
    | **Memory Optimized** | **R**eally **M**emorable | Large amounts of RAM. Good for in-memory databases (like Redis), real-time big data analytics. | R series, X series |
    | **Storage Optimized** | **I**'m **D**ense | High-performance, low-latency local storage (NVMe SSDs). Good for databases, data warehousing, distributed file systems. | I series, D series |
    | **Accelerated Computing** | **G**raphics **P**ower | Hardware accelerators (GPUs, FPGAs). Good for machine learning, graphics rendering, scientific computing. | P series (ML/HPC), G series (Graphics) |
- **EC2 Pricing Models (How You Pay)**


    | **Model** | **Commitment** | **Best For / Use Case** | **Cost Savings** |
    | --- | --- | --- | --- |
    | **On-Demand** | **None**. Pay by the second. | Spiky, unpredictable workloads or short-term dev/test. | None (Baseline) |
    | **Savings Plans** | **1 or 3-year** commitment to a certain $/hour spend. | Stable, predictable workloads. **The modern, flexible choice.** | High (up to 72%) |
    | **Reserved Instances** | **1 or 3-year** commitment to a specific instance type/region. | Stable workloads. Less flexible than Savings Plans. | High (up to 72%) |
    | **Spot Instances** | **None**. Bid on spare AWS capacity. **Can be terminated with a 2-minute warning.** | Fault-tolerant, stateless workloads that can be interrupted (e.g., batch jobs, big data processing). | Highest (up to 90%) |
    | **Dedicated Hosts** | **Physical server** dedicated to you. | Meeting strict compliance requirements or for software with complex licensing (BYOL). | Lowest (Most Expensive) |
- **Storage Options for EC2**
    - **Amazon FSx Family (Third-Party File Systems)**
        - *Use this when the scenario asks for a specific "industry standard" or "operating system" file system.*
        - **FSx for Windows File Server**
            - **Protocol:** SMB.
            - **Exam Trigger:** "Shared storage for **Windows** applications," "Active Directory integration," "Lift-and-shift Windows app."
            - **Key Feature:** Fully managed Windows file system. Supports DFS Namespaces (grouping shares).
        - **FSx for Lustre**
            - **Protocol:** Parallel distributed file system.
            - **Exam Trigger:** "**High Performance Computing (HPC)**," "Machine Learning," "Video Processing," "Millions of IOPS."
            - **Key Feature:** Can **integrate natively with S3**. It can "hydrate" (read) data directly from an S3 bucket for processing and write results back to S3.
            - **Deployment:** "Scratch" (temp/fast) or "Persistent" (long-term).
        - **FSx for NetApp ONTAP**
            - **Protocol:** NFS, SMB, and **iSCSI** (Multi-protocol).
            - **Exam Trigger:** "**NetApp** migration," "Deduplication & Compression," "Snapshot replication (SnapMirror)."
            - **Key Feature:** Highly efficient storage; supports tiering cold data to S3 automatically.
        - **FSx for OpenZFS**
            - **Protocol:** NFS (v3, v4).
            - **Exam Trigger:** "Migrating **ZFS** or Linux file servers to AWS," "High-performance NFS without managing servers."
    - **AWS Storage Gateway (Hybrid Cloud Storage)**
        - *Use this when the scenario involves **On-Premises** servers that need access to AWS storage.*
        - **S3 File Gateway**
            - **Interface:** NFS or SMB (File interface).
            - **Backing:** Stores files as **Objects in S3**.
            - **Use Case:** Backing up on-prem files to the cloud, creating a "bottomless" file server for an office.
        - **Volume Gateway** (Block Storage / iSCSI)
            - **Interface:** iSCSI (Block interface).
            - **Backing:** Stores data as **EBS Snapshots** in the cloud.
            - **Two Modes (Classic Exam Question):**
                1. **Stored Volumes:** All data is kept **locally** (on-prem) for speed; asynchronous backups are sent to AWS. *(Choose if: "Low latency access to **entire** dataset is required").*
                2. **Cached Volumes:** Primary data is in **AWS (S3)**; only frequently accessed data is cached locally. *(Choose if: "Extend on-prem storage," "Local disk space is limited").*
        - **Tape Gateway**
            - **Interface:** VTL (Virtual Tape Library).
            - **Use Case:** Replacing physical tape backup infrastructure (e.g., Veeam, Veritas) with cloud storage (Glacier).
    - **Elastic Block Store (EBS):** A persistent, network-attached block storage volume. It survives instance termination if configured to do so.
        - General Purpose SSD (gp2/gp3): The default. Best for boot volumes, dev/test. gp3 is the newer, cost-effective standard.
        - Provisioned IOPS SSD (io1/io2): For mission-critical, I/O-intensive workloads (e.g., Relational DBs). Expensive due to paying for storage + provisioned IOPS.
        - Throughput Optimized HDD (st1): Low-cost magnetic storage for streaming/big data.
        - Cold HDD (sc1): Cheapest option for infrequent access.
    - **Instance Store:** Ephemeral (temporary) storage that is physically attached to the host machine.
        - Performance: Offers the highest possible Random I/O and lowest latency because it avoids the network hop required by EBS.
        - Cost: Most cost-effective for high performance (storage is included in the EC2 hourly cost; no extra fee for IOPS).
        - Persistence: All data is lost on stop/termination.
        - Ideal Use Case:
            - Temporary caches / Buffers.
            - Distributed/Replicated Applications: If the application architecture handles data replication (e.g., NoSQL clusters, Hadoop, specialized fleets), use Instance Store to save money and maximize speed. You do not need to pay for EBS persistence if the app is already resilient.
    - **Amazon EFS (Elastic File System)**
        - Type: Managed Network File System (NFS).
        - Key Feature: Shared storage. Can be mounted by hundreds of EC2 instances across multiple Availability Zones (AZs) simultaneously.
        - OS Compatibility: Linux only (Windows uses Amazon FSx).
        - Performance: Designed for throughput (big file transfers). It has higher latency than EBS or Instance Store because it is a network-shared protocol. It is not suitable for high random I/O tasks.
        - Cost: generally more expensive than EBS (gp3) and Instance Store.
    - **Amazon S3 (Simple Storage Service)**
        - Type: Object Storage (API-based), NOT Block Storage.
        - Usage with EC2: Instances access S3 via API/SDK or specialized mount tools (like Mountpoint for S3), but it does not behave like a standard hard drive.
        - Performance: Infinite scalability and high throughput, but very high latency compared to EBS/Instance Store.
        - Suitability:
            - Good for: Storing static assets (images/videos), backups, logs, and data lakes.
            - Bad for: Running operating systems, databases, or high-performance random I/O applications.
- **Networking & Placement**
    - **Elastic IP (EIP):** A static, public IPv4 address that you own in your account. You can attach it to an EC2 instance to give it a fixed public IP that persists across stops/starts.
    - **Placement Groups:** Control the physical placement of your instances.
        - **Cluster:** Packs instances close together in one AZ for **low latency and high throughput** (for HPC).
        - **Spread:** Places each instance on distinct hardware to reduce correlated failures (for critical applications).
        - **Partition:** Spreads instances across logical partitions (racks) for fault tolerance (for big data apps like Cassandra/HDFS).
- **High Availability & Scalability**
    - **Elastic Load Balancing (ELB):** Distributes incoming traffic across multiple EC2 instances in different Availability Zones.
    - **Auto Scaling Group (ASG):** Automatically adds or removes EC2 instances based on demand (metrics like CPU utilization) or on a schedule. This is the core of building elastic and self-healing applications. It uses a **Launch Template** or **Launch Configuration** to define the instances it creates.
- **Instance Lifecycle & Metadata**
    - **Stop vs. Terminate:**
        - **Stop:** Shuts down the instance. The EBS root volume is preserved. You are not billed for instance usage, only for the EBS storage.
        - **Terminate:** Permanently deletes the instance. The EBS root volume is deleted by default.
    - **Instance Metadata Service (IMDS):** A service accessible from *within* an EC2 instance at the special IP address 169.254.169.254. It provides information about the instance itself, such as its instance ID, IAM role, and IP address. **IMDSv2** is the newer, more secure version that uses session-based authentication.

## EC2 Hibernate (Preserve RAM State)

**What is it?** EC2 Hibernate allows you to **save the instance's RAM state** to the EBS root volume when you stop the instance, then **resume from exactly where it left off** when you start it again.

---

### **How Hibernate Works**

```
Normal Stop:
Instance Running → Stop → Shutdown → RAM cleared → Start → Boot OS → Load apps

Hibernate:
Instance Running → Hibernate → Save RAM to EBS → Stop → Start → Restore RAM → Resume
```

**Key Point:** With Hibernate, the instance **does not go through a full boot process**. It resumes with all processes, applications, and memory state intact.

---

### **Stop vs. Hibernate vs. Terminate**

| Action | RAM State | Boot Process | EBS Root Volume | Billing | Use Case |
|--------|-----------|--------------|-----------------|---------|----------|
| **Stop** | **Lost** (cleared) | Full reboot required | Preserved | ✅ No instance charge (only EBS storage) | Normal shutdown |
| **Hibernate** | **Saved** to EBS | Resume instantly | Preserved | ✅ No instance charge (only EBS storage) | Long-running processes, quick resume |
| **Terminate** | Lost | N/A (instance deleted) | Deleted (by default) | ❌ No charges | Permanent deletion |

**Exam Tip:** If the question mentions "**preserve in-memory state**" or "**resume quickly without rebooting**" → **Hibernate**

---

### **Use Cases**

| Scenario | Why Hibernate? |
|----------|---------------|
| **Long-running processes** | Applications with state that takes hours to initialize (e.g., loading large datasets into memory) |
| **Save costs during off-hours** | Stop instances overnight/weekends without losing state (dev/test environments) |
| **Faster startup** | Skip boot process and application initialization (resume in ~60 seconds vs. minutes) |
| **Preserve session state** | Keep user sessions, cached data, and application state intact |

**Example:** A machine learning model that takes 2 hours to load training data into RAM. Without Hibernate, stopping the instance means reloading the data every time. With Hibernate, the instance resumes with data already in memory.

---

### **Requirements and Limitations**

#### **Requirements**

| Requirement | Details |
|-------------|---------|
| **EBS Root Volume** | Must use EBS (not Instance Store) |
| **Encryption** | Root volume **must be encrypted** |
| **RAM Size** | Instance RAM must be **<150 GB** |
| **Supported Families** | C3, C4, C5, M3, M4, M5, R3, R4, R5 (General Purpose, Compute, Memory optimized) |
| **Supported OS** | Amazon Linux 2, Amazon Linux AMI, Ubuntu, Windows Server |
| **Root Volume Size** | Must be large enough to store RAM contents |

**Exam Trigger:** If the question mentions Hibernate but the instance has **>150 GB RAM** → Hibernate is **not supported**

---

#### **Limitations**

| Limitation | Details |
|------------|---------|
| **Maximum Hibernate Duration** | **60 days** (after 60 days, instance must be started) |
| **Not Supported** | Spot Instances, Bare Metal instances, instances with Instance Store volumes |
| **Reserved Instances** | Billing continues for Reserved Instance term (but not On-Demand charges) |
| **Elastic IP** | Remains attached during hibernation |

---

### **How to Enable Hibernate**

**Step 1:** Enable Hibernate when launching the instance
- **AWS Console:** Check "Enable hibernation" in Advanced Details
- **AWS CLI:** Use `--hibernation-options Configured=true`

**Step 2:** Ensure root volume is encrypted

**Step 3:** Hibernate the instance
```bash
aws ec2 stop-instances --instance-ids i-1234567890abcdef0 --hibernate
```

**Key Point:** You **cannot enable Hibernate on an existing instance**. It must be configured at launch time.

---

### **Exam Scenarios**

| Scenario | Solution |
|----------|----------|
| "Application takes hours to load data into memory, needs to pause overnight" | **EC2 Hibernate** (saves RAM state) |
| "Resume instance quickly without full boot" | **EC2 Hibernate** (skip boot process) |
| "Instance has 200 GB of RAM, needs to preserve state" | ❌ **Hibernate not supported** (>150 GB limit), use Stop + Reload |
| "Spot Instance needs to preserve state when interrupted" | ❌ **Hibernate not supported** for Spot, use checkpointing to S3 |
| "Dev environment used only during business hours" | **EC2 Hibernate** (stop overnight, resume quickly) |
| "Root volume is not encrypted" | ❌ **Cannot use Hibernate** (encryption required) |

---

### **Hibernate vs. Snapshots**

| Feature | Hibernate | EBS Snapshot |
|---------|-----------|--------------|
| **What's Saved** | RAM state (in-memory data) | Disk state (EBS volume data) |
| **Resume Time** | Fast (~60 seconds) | Slower (need to create volume, attach, boot) |
| **Use Case** | Quick pause/resume of running instance | Backup, disaster recovery, AMI creation |
| **Encryption Requirement** | ✅ Required | Optional |

**Exam Tip:** Hibernate saves **RAM state** (what's in memory), Snapshots save **disk state** (what's on EBS). They serve different purposes.

---

### **Cost Considerations**

| Cost Component | Details |
|----------------|---------|
| **Instance Charges** | ✅ **No charges** while hibernated (same as Stop) |
| **EBS Storage** | ✅ **Charged** for root volume (stores RAM + disk data) |
| **Data Transfer** | ✅ **Charged** for writing RAM to EBS (minimal, one-time) |
| **Reserved Instance** | ⚠️ **Continues billing** for RI term (but not On-Demand) |

**Savings:** Hibernate during off-hours = 50-70% cost savings (if used 8 hours/day, 5 days/week)

---

### **Key Exam Tips**

1. **Hibernate = Save RAM state** (in-memory data preserved)
2. **Requirements:** EBS root, encrypted, <150 GB RAM
3. **Not supported:** Spot Instances, >150 GB RAM, Instance Store root
4. **Maximum duration:** 60 days hibernation
5. **Enable at launch:** Cannot enable on existing instances
6. **Use case triggers:** "Long-running processes," "Quick resume," "Preserve memory state"
7. **Billing:** No instance charges (same as Stop), but EBS storage charges apply
8. **Boot time:** Hibernate resumes in ~60 seconds (faster than Stop → Start)

---

### **Quick Decision Tree**

```
Need to preserve state?
├─ Preserve disk state (EBS data)
│  └─ Use: EBS Snapshot
└─ Preserve RAM state (in-memory data)
   ├─ RAM <150 GB, root volume encrypted?
   │  └─ Yes → Use: EC2 Hibernate
   └─ No → Use: Stop + Reload state from storage (S3/EFS)
```

- **Exam Cheat Sheet**


    | **If the question asks for...** | **The answer likely involves...** |
    | --- | --- |
    | Highest performance for **HPC / tightly coupled** apps | **Cluster Placement Group** |
    | Highest availability for a **small number of critical** instances | **Spread Placement Group** |
    | The cheapest possible compute for **fault-tolerant** workloads | **Spot Instances** |
    | **Predictable workloads** needing cost savings and flexibility | **Savings Plans** |
    | **Persistent** storage for a database or boot volume | **EBS Volume** |
    | **Temporary, high-performance** scratch space | **Instance Store** |
    | A **static public IP** address for a server | **Elastic IP (EIP)** |
    | **Automatic scaling** and self-healing of instances | **Auto Scaling Group (ASG)** |
    | **Distributing traffic** across multiple instances | **Elastic Load Balancer (ELB)** |
    | **Restricting traffic** to an instance | **Security Group** |
