- _Use this when the scenario involves **On-Premises** servers that need access to AWS storage._
- **S3 File Gateway**
    - **Interface:** NFS or SMB (File interface).
    - **Backing:** Stores files as **Objects in S3**.
    - **Use Case:** Backing up on-prem files to the cloud, creating a "bottomless" file server for an office.
- **Volume Gateway** (Block Storage / iSCSI)
    - **Interface:** iSCSI (Block interface).
    - **Backing:** Stores data as **EBS Snapshots** in the cloud.
    - **Two Modes (Classic Exam Question):**
        1. **Stored Volumes:** All data is kept **locally** (on-prem) for speed; asynchronous backups are sent to AWS. _(Choose if: "Low latency access to **entire** dataset is required")._
        2. **Cached Volumes:** Primary data is in **AWS (S3)**; only frequently accessed data is cached locally. _(Choose if: "Extend on-prem storage," "Local disk space is limited")._
- **Tape Gateway**
    - **Interface:** VTL (Virtual Tape Library).
    - **Use Case:** Replacing physical tape backup infrastructure (e.g., Veeam, Veritas) with cloud storage (Glacier).