# EBS (Elastic Block Store): Persistent, Network-attached (gp3, io2).

- **What is it? (1-Sentence Pitch):** A high-performance, persistent block storage volume (like a "virtual hard drive") for use with a single EC2 instance.
- **Core Use Cases:** Boot volumes for EC2 instances, storing data for transactional or NoSQL databases running on EC2, throughput-intensive workloads.
- **Key Features & Concepts:**
    - **Snapshots:** Point-in-time backups of your EBS volumes stored in S3. The foundation of EBS backup and disaster recovery.
    - **Encryption:** EBS volumes can be encrypted at rest, including the boot volume.
    - **Tied to an AZ:** An EBS volume lives in a specific Availability Zone and can only be attached to an EC2 instance in that same AZ.
- **EBS Volume Types (Exam Essentials):**
    - **General Purpose SSD (gp3/gp2):** The default. A great balance of price and performance for a wide variety of workloads like boot volumes and web servers. **Choose gp3 over gp2.**
    - **Provisioned IOPS SSD (io2 Block Express/io1):** Highest performance SSD for mission-critical, I/O-intensive workloads like large relational databases (e.g., RDS, Oracle on EC2).
    - **Throughput Optimized HDD (st1):** Low-cost HDD designed for frequently accessed, throughput-intensive workloads like big data and data warehousing.
    - **Cold HDD (sc1):** Lowest cost HDD for less frequently accessed workloads.

[Availability Zone and Region](EBS%20(Elastic%20Block%20Store)%20Persistent,%20Network-atta/Availability%20Zone%20and%20Region%202eb50618cbb88009aa96c0a9700ec535.md)