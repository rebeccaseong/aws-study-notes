# EFS (Elastic File System) — Linux: : Shared Linux (NFS), cross-AZ.

- **What is it? (1-Sentence Pitch):** A managed, scalable, and fully elastic file system (using NFS protocol) that can be shared across thousands of EC2 instances and on-premises servers.
- **Core Use Cases:** A common data source for a fleet of web servers, content management systems, big data analytics, and any application where multiple compute instances need to access the same file data simultaneously.
- **Key Features & Concepts:**
    - **Storage Tiers:** EFS Standard and EFS Infrequent Access (EFS-IA), with lifecycle management to automatically move files.
    - **Resilience:** Stores data across multiple AZs by default.
    - **Performance Modes:** General Purpose (default) and Max I/O (for highly parallelized workloads).
    - **Use Case Distinction:** Choose EFS for **Linux-based** workloads that need a shared file system.