- **What is it? (1-Sentence Pitch):** Amazon S3 is a secure, durable, and infinitely scalable **object storage** service, designed to store and retrieve any amount of data, for any use case, from anywhere on the internet.
- **Core S3 Concepts (The Object Model)**
    - **Bucket:** A container for objects. Bucket names are **globally unique**. Think of it as a top-level folder.
    - **Object:** The fundamental entity stored in S3. It consists of:
        - **Key:** The unique name/path of the object within the bucket (e.g., images/puppy.jpg).
        - **Value:** The data itself (the file content).
        - **Metadata:** Information about the object (e.g., content-type, last-modified).
    - **Region:** You choose an AWS Region when you create a bucket, and the data is stored within that region.
- **S3 Storage Classes (CRITICAL TO KNOW):**


    | **Storage Class** | **Use Case / Retrieval Pattern** | **Retrieval Time** | **Cost Profile** | **Minimum Billing Duration** |
    | --- | --- | --- | --- | --- |
    | **S3 Standard** | **Frequently accessed data**; default class. | Milliseconds | Highest storage cost, lowest access cost. | **None** |
    | **S3 Intelligent-Tiering** | **Data with unknown or changing access patterns.** | Milliseconds | Small monthly monitoring fee; automatically moves data to save you money. | **30 days** |
    | **S3 Standard-IA** (Infrequent Access) | **Long-lived, but less frequently accessed data** that needs rapid access when requested (e.g., long-term backups). | Milliseconds | Lower storage cost than Standard, but has a **per-GB retrieval fee**. | **30 days** |
    | **S3 One Zone-IA** | Same as Standard-IA, but data is stored in **only one Availability Zone**. | Milliseconds | Lower storage cost than Standard-IA. Good for **reproducible data**. | **30 days** |
    | **S3 Glacier Instant Retrieval** | **Archive data** that needs immediate, millisecond access (e.g., medical images, news media assets). | Milliseconds | Very low storage cost, higher retrieval fee than IA classes. | **90 days** |
    | **S3 Glacier Flexible Retrieval** | **Long-term archives** where retrieval time is flexible (minutes to hours). | Minutes (expedited), Hours (standard) | Extremely low storage cost. | **90 days** |
    | **S3 Glacier Deep Archive** | **Deepest, cheapest archival** for data accessed once or twice a year (e.g., compliance/regulatory data). | Hours (12-48) | The absolute lowest storage cost on AWS. | **180 days** |
    - **Exam Rule of Thumb:** For data with unknown access patterns, **Intelligent-Tiering** is often the correct answer. For reproducible data, **One Zone-IA** is a cost-effective choice.
- **Security & Access Control**
    - **IAM Policies & Bucket Policies:** The primary way to control access. Bucket Policies are attached to the bucket; IAM Policies are attached to users/roles.
        - **Best Practice:** Use IAM policies for user-specific permissions and Bucket Policies for broad, bucket-level permissions (e.g., forcing encryption, granting access to another AWS service).
    - **S3 Block Public Access:** A set of controls at the account or bucket level that acts as a safety net to **prevent accidental public exposure**. **This should be enabled by default.**
    - **Access Control Lists (ACLs):** A legacy mechanism. Avoid using them unless you have a specific need to grant access to other AWS accounts on a per-object basis. **IAM/Bucket Policies are preferred.**
    - **Pre-signed URLs:** The way to grant temporary, time-limited access to a private object in your bucket. The URL is signed with your security credentials.
        - **Use Case:** Allowing a user to download a specific private file for 10 minutes.
    - **Encryption:** Server-Side Encryption (SSE-S3, SSE-KMS, SSE-C) and Client-Side Encryption.
        - **Encryption in Transit:** Achieved by using SSL/TLS (HTTPS). All modern S3 endpoints support this.
        - **Encryption at Rest (Server-Side Encryption - SSE):**
            - **SSE-S3 (Managed by S3):** S3 manages the encryption keys. Easiest to set up.
            - **SSE-KMS (Managed by KMS):** You manage the keys via AWS Key Management Service (KMS). Provides an audit trail (via CloudTrail) of who used the key. Offers more control than SSE-S3.
            - **SSE-C (Customer-Provided Keys):** You provide your own encryption key with every request. AWS does not store the key.
        - **Client-Side Encryption:** You encrypt the data *before* you upload it to S3. You are fully responsible for key management.
        - **Comparison Table**


            | **Feature** | **SSE-S3 (Bank Lock)** | **SSE-KMS (Audited Lock)** | **SSE-C (Your Own Lock)** | **Client-Side Encryption (Lock Before You Go)** |
            | --- | --- | --- | --- | --- |
            | **Where Encryption Happens** | On AWS servers | On AWS servers | On AWS servers | **On your own machine (the "client")** |
            | **Who Manages the Key?** | AWS | AWS KMS (but controlled by you) | You, the customer | **You, the customer** |
            | **Key Sent to AWS?** | Never | No, just a reference to it | **Yes, with every request** (but never stored) | **Never** |
            | **Audit Trail (CloudTrail)** | No | **Yes, this is a key benefit** | No (auditing is your responsibility) | No (auditing is your responsibility) |
            | **Control over Key** | None | High (access policies, rotation, enable/disable) | Absolute (you manage everything outside of AWS) | **Absolute** (you manage everything) |
            | **Responsibility** | Low | Medium | **Very High** | **Maximum** |
            | **How to Use** | Check a box in the S3 console. | Select SSE-KMS and choose a key. | Provide the key in the header of every API request. | Use an AWS SDK or your own library to encrypt locally. |
            | **Best For** | Basic, effortless encryption. | **Compliance and regulatory needs.** | Strict policies requiring you to manage your own keys. | **Maximum security;** data must be encrypted before transit. |
- **Data Management & Automation**
    - **Versioning:** Automatically keeps a history of all object versions. This is your primary protection against **accidental deletions and overwrites**. Once enabled, it cannot be disabled, only suspended.
        - **MFA Delete:** An extra layer of security that requires a Multi-Factor Authentication code to permanently delete an object version.
    - **Lifecycle Policies:** Automates moving objects between storage classes or deleting them.
        - **Transition Action:** Move objects to a cheaper storage class (e.g., S3 Standard → S3-IA after 30 days → Glacier after 90 days).
        - **Expiration Action:** Delete old objects or old versions of objects.
    - **Replication (CRR & SRR):** Automatically copies objects to another bucket. **Requires versioning to be enabled on both source and destination buckets.**
        - **Cross-Region Replication (CRR):** Copies to a bucket in a *different* region. Used for disaster recovery and to reduce latency for users in other parts of the world.
        - **Same-Region Replication (SRR):** Copies to a bucket in the *same* region. Used for log aggregation or to create a separate copy for dev/test accounts.

---

## Deep Dive: S3 Replication (CRR & SRR)

S3 Replication automatically copies objects from a source bucket to one or more destination buckets. This is a powerful feature for disaster recovery, compliance, latency reduction, and data aggregation.

### **Key Concepts**

| Feature | Details |
|---------|---------|
| **Prerequisites** | **Versioning must be enabled** on both source and destination buckets |
| **Replication Types** | **CRR** (Cross-Region Replication), **SRR** (Same-Region Replication) |
| **What Replicates** | New objects created after replication is enabled (existing objects require S3 Batch Replication) |
| **Permissions** | Source bucket needs IAM role with permission to replicate to destination |
| **Encryption** | Supports replication of encrypted objects (SSE-S3, SSE-KMS, SSE-C) |
| **Delete Markers** | Optional - can replicate delete markers (default: no) |

---

### **Cross-Region Replication (CRR)**

**What It Does:** Copies objects to a bucket in a **different AWS region**

**Common Use Cases:**

| Use Case | Why CRR |
|----------|---------|
| **Disaster Recovery** | Protect against region-wide failures by maintaining a copy in another region |
| **Compliance** | Meet regulatory requirements to store data in specific geographic locations |
| **Latency Reduction** | Serve users in different geographic locations from nearby replicas |
| **Data Sovereignty** | Keep copies in regions required by data residency laws |

**Example:**
- Primary bucket: `us-east-1`
- Replica bucket: `eu-west-1`
- **Use Case:** Disaster recovery (if `us-east-1` fails, serve from `eu-west-1`)

---

### **Same-Region Replication (SRR)**

**What It Does:** Copies objects to a bucket in the **same AWS region**

**Common Use Cases:**

| Use Case | Why SRR |
|----------|---------|
| **Log Aggregation** | Combine logs from multiple buckets into a single bucket for analysis |
| **Dev/Test Environments** | Create separate copies for development and testing without affecting production |
| **Account Separation** | Replicate data across different AWS accounts (e.g., production → audit account) |
| **Data Resilience** | Maintain multiple copies within the same region for additional data protection |

**Example:**
- Source bucket: `production-data` (Account A)
- Replica bucket: `audit-data` (Account B)
- Both in `us-east-1`
- **Use Case:** Audit/compliance team needs read-only copy of production data

---

### **How S3 Replication Works**

**Step-by-Step:**

1. **Enable Versioning** on both source and destination buckets
2. **Create IAM Role** with permission to replicate objects
3. **Configure Replication Rule** on source bucket:
   - Specify destination bucket (can be in same or different region)
   - Choose what to replicate (all objects or filtered by prefix/tags)
   - Configure storage class for replica (can be different from source)
   - Enable/disable delete marker replication
4. **New Objects Replicate Automatically** after rule is created
5. **Existing Objects** require S3 Batch Replication to replicate

**Important:** Replication is **asynchronous** (objects replicate within minutes, but not instantly)

---

### **What Gets Replicated**

**Replicated by Default:**
- ✅ New objects created after replication is enabled
- ✅ Object metadata
- ✅ Object tags
- ✅ Object ACLs
- ✅ SSE-S3 encrypted objects (automatically)

**NOT Replicated by Default (Require Configuration):**
- ❌ Existing objects (use S3 Batch Replication)
- ❌ Delete markers (must enable "Delete Marker Replication")
- ❌ Objects encrypted with SSE-KMS (must grant KMS permissions)
- ❌ Objects encrypted with SSE-C (not supported for replication)
- ❌ Objects in Glacier or Glacier Deep Archive (must restore first)
- ❌ System-generated metadata (e.g., object creation date)

---

### **Replication Configuration Options**

| Option | Details |
|--------|---------|
| **Replication Time Control (RTC)** | Guarantees replication within **15 minutes** (SLA with monitoring) |
| **Replica Storage Class** | Choose different storage class for replica (e.g., source: Standard, replica: Glacier) |
| **Replication Ownership** | Control who owns the replica (source account or destination account) |
| **Delete Marker Replication** | Choose whether to replicate delete markers (default: no) |
| **Filter by Prefix/Tags** | Replicate only objects matching specific prefix (e.g., `/logs/*`) or tags |

---

### **Delete Marker Replication**

**What Is a Delete Marker?**
- When you delete an object in a versioned bucket, S3 doesn't actually delete it
- Instead, S3 adds a **delete marker** (a placeholder indicating the object is "deleted")
- The object still exists in previous versions

**Delete Marker Replication Options:**

| Setting | Behavior |
|---------|----------|
| **Disabled (default)** | Delete markers are NOT replicated to destination |
| **Enabled** | Delete markers ARE replicated to destination |

**Why Disable?**
- Prevent accidental deletions from affecting replica
- Replica remains available even if source is "deleted"

**Why Enable?**
- Keep source and replica in sync (if deleted in source, also "delete" in replica)

---

### **Replicating Encrypted Objects**

| Encryption Type | Replication Support |
|----------------|---------------------|
| **SSE-S3** | ✅ Supported (automatic) |
| **SSE-KMS** | ✅ Supported (requires KMS key permissions in destination region) |
| **SSE-C** | ❌ NOT supported (customer-provided keys cannot be replicated) |

**SSE-KMS Replication Requirements:**
1. Grant source bucket IAM role permission to use KMS key in destination region
2. Specify destination KMS key in replication configuration
3. Ensure KMS key policy allows replication role to use the key

---

### **Replication Time Control (RTC)**

**What It Is:** A feature that guarantees **99.99% of objects replicate within 15 minutes**

**Benefits:**
- **SLA-backed replication** (predictable replication time)
- **Monitoring metrics** (track replication progress)
- **Replication status per object** (see which objects have replicated)

**Use Case:** Compliance requirements that mandate timely replication

**Cost:** Additional charge per GB replicated (on top of standard replication cost)

---

### **Replicating Existing Objects (S3 Batch Replication)**

**Problem:** Replication rules only apply to **new objects** created after the rule is configured

**Solution:** Use **S3 Batch Replication** to replicate existing objects

**How It Works:**
1. Create replication rule (as usual)
2. Enable **Batch Replication** for the rule
3. S3 creates a batch job to replicate existing objects
4. Monitor progress via S3 Batch Operations console

**Use Case:** You have 1 million existing objects and want to replicate them to a new region

---

### **Chaining Replication (Multi-Region Replication)**

**Can You Chain Replication?**
- **No** by default (objects replicated from Bucket A to Bucket B are NOT automatically replicated from Bucket B to Bucket C)

**Workaround:**
- Enable **Replica Modification Sync** (replicates metadata/tag changes on replicas)
- Configure separate replication rules for each bucket pair

**Example:**
- Bucket A (us-east-1) → Bucket B (eu-west-1)
- Bucket B (eu-west-1) → Bucket C (ap-south-1)
- **Result:** Objects in A are replicated to B, then B replicates to C (requires two separate replication rules)

---

### **Cross-Account Replication**

**Use Case:** Replicate data from one AWS account to another (e.g., production → audit account)

**Requirements:**
1. **Destination bucket policy** must grant source account permission to replicate
2. **Source bucket IAM role** must have permission to write to destination bucket
3. **Replica ownership** must be configured (who owns the replica?)

**Replica Ownership Options:**

| Setting | Who Owns Replica |
|---------|------------------|
| **Source Account (default)** | Source account retains ownership (destination account cannot access) |
| **Destination Account** | Destination account owns replica (recommended for cross-account replication) |

**Best Practice:** Use "Destination Account Ownership" for cross-account replication

---

### **Replication Metrics and Monitoring**

**CloudWatch Metrics:**
- `BytesPendingReplication`: Bytes waiting to be replicated
- `OperationsPendingReplication`: Number of operations pending
- `ReplicationLatency`: Time taken to replicate (with RTC enabled)

**S3 Replication Status:**
Each object has a replication status:
- `PENDING`: Replication in progress
- `COMPLETED`: Successfully replicated
- `FAILED`: Replication failed (check permissions, encryption, etc.)
- `REPLICA`: This object is a replica (created by replication)

**How to Check:**
- Via S3 Console: Object properties → Replication status
- Via AWS CLI: `aws s3api head-object --bucket my-bucket --key my-object`

---

### **Exam Scenarios**

| Scenario | Solution |
|----------|----------|
| Need disaster recovery for S3 data in another region | **CRR** (Cross-Region Replication) |
| Aggregate logs from multiple buckets in same region | **SRR** (Same-Region Replication) |
| Compliance requires data copy in specific region | **CRR** to that region |
| Replicate existing objects (not just new ones) | **S3 Batch Replication** |
| Need guaranteed replication within 15 minutes | **Replication Time Control (RTC)** |
| Delete object in source, but keep replica intact | Disable "Delete Marker Replication" |
| Replicate encrypted objects (SSE-KMS) | Grant KMS key permissions + specify destination KMS key |
| Replicate to another AWS account | **Cross-account replication** with destination account ownership |
| Reduce latency for users in multiple regions | **CRR** to regions closer to users |
| Create separate copy for dev/test | **SRR** to different bucket in same region |

---

### **Key Exam Tips**

1. **Versioning required** on both source and destination buckets
2. **CRR = Cross-Region** (disaster recovery, compliance, latency)
3. **SRR = Same-Region** (log aggregation, dev/test, account separation)
4. **Only new objects replicate** by default (use S3 Batch Replication for existing)
5. **Delete markers NOT replicated** by default (must enable)
6. **SSE-KMS replication** requires KMS key permissions in destination region
7. **SSE-C NOT supported** for replication
8. **Replication Time Control (RTC):** 15-minute SLA
9. **Replication is asynchronous** (not instant)
10. **Cross-account replication:** Use "Destination Account Ownership"
11. **Chaining NOT automatic** (requires separate replication rules)
12. **Monitor via CloudWatch metrics:** BytesPendingReplication, ReplicationLatency

---

### **Replication Use Case Examples**

#### **Example 1: Disaster Recovery**
- **Setup:** CRR from `us-east-1` to `us-west-2`
- **Goal:** Protect against region-wide failure
- **Configuration:**
  - Enable versioning on both buckets
  - Create CRR rule: replicate all objects
  - Enable Delete Marker Replication (keep source and replica in sync)
  - Enable RTC (guarantee 15-minute replication)

#### **Example 2: Compliance (Data Sovereignty)**
- **Setup:** CRR from `us-east-1` to `eu-central-1`
- **Goal:** Meet GDPR requirement to store EU customer data in EU region
- **Configuration:**
  - Filter replication by tag: `region=eu`
  - Replicate only objects tagged with `region=eu` to EU bucket

#### **Example 3: Log Aggregation**
- **Setup:** SRR from multiple application buckets → central logging bucket
- **Goal:** Centralize logs for analysis
- **Configuration:**
  - App Bucket 1 → Logging Bucket (prefix: `/app1/`)
  - App Bucket 2 → Logging Bucket (prefix: `/app2/`)
  - All in same region (`us-east-1`)

#### **Example 4: Cross-Account Audit**
- **Setup:** SRR from production account → audit account (same region)
- **Goal:** Audit team needs read-only access to production data
- **Configuration:**
  - Enable "Destination Account Ownership"
  - Destination bucket policy grants read-only access to audit team
  - Disable Delete Marker Replication (prevent production deletions from affecting audit copy)

---

### **Cost Considerations**

**Replication Costs:**
1. **Data Transfer (CRR only):** Cross-region data transfer fees
2. **Replication Requests:** PUT/COPY requests to destination bucket
3. **Storage:** Storage costs for replica (can use cheaper storage class)
4. **RTC (if enabled):** Additional charge per GB replicated

**Cost Optimization:**
- Use **SRR** instead of CRR when possible (no cross-region transfer fees)
- Set **replica storage class** to cheaper class (e.g., Standard-IA or Glacier)
- Use **prefix filters** to replicate only necessary objects
- Disable **RTC** if 15-minute SLA not required

---


---

## Deep Dive: S3 Object Lock (WORM - Write-Once-Read-Many)

**What is it?** S3 Object Lock prevents objects from being deleted or overwritten for a fixed amount of time or indefinitely, implementing a **Write-Once-Read-Many (WORM)** model for regulatory compliance.

---

### **Key Concepts**

| Feature | Details |
|---------|---------|
| **Purpose** | Prevent object deletion/modification for compliance (SEC, FINRA, HIPAA, etc.) |
| **WORM Model** | Write-Once-Read-Many (immutable storage) |
| **Requirement** | **Versioning must be enabled** (Object Lock works at version level) |
| **Two Components** | **Retention Mode** + **Legal Hold** (can be used together or separately) |
| **Enable Time** | Must be enabled at **bucket creation** (cannot enable on existing bucket) |

**Exam Tip:** If the question mentions "**compliance**," "**immutable storage**," or "**prevent deletion**" → Object Lock

---

### **Retention Modes: Governance vs. Compliance**

**This is the most tested distinction on the exam.**

| Aspect | Governance Mode | Compliance Mode |
|--------|----------------|-----------------|
| **Protection Level** | Moderate | **Maximum** |
| **Can Delete?** | ✅ Yes (with special permissions) | ❌ **NO** (nobody, not even root) |
| **Can Shorten Retention?** | ✅ Yes (with special permissions) | ❌ **NO** |
| **Can Extend Retention?** | ✅ Yes | ✅ Yes (anyone can extend) |
| **Override Permission** | `s3:BypassGovernanceRetention` | ❌ **Cannot override** |
| **Use Case** | Protect from accidental deletion, internal audits | **Strict regulatory compliance** (SEC, HIPAA) |
| **Analogy** | Manager lock (supervisor can override) | Bank vault (nobody can open until timer expires) |

**Exam Trigger Phrases:**

| Phrase in Question | Answer |
|-------------------|---------|
| "Regulatory compliance (SEC, FINRA)" | **Compliance Mode** |
| "Protect from accidental deletion by administrators" | **Governance Mode** |
| "Absolutely cannot be deleted by anyone" | **Compliance Mode** |
| "Can be overridden with special permissions" | **Governance Mode** |
| "Internal testing before production compliance" | **Governance Mode** (test first, then Compliance) |

---

### **How Governance Mode Works**

**What It Does:**
- Prevents deletion/overwriting by most users
- Allows users with `s3:BypassGovernanceRetention` permission to override

**Use Cases:**
- Protect production data from accidental deletion
- Internal audit requirements (not external regulatory)
- Testing Object Lock before moving to Compliance mode

**How to Override (with Permission):**
```bash
aws s3api delete-object \
  --bucket my-bucket \
  --key my-object \
  --version-id version-id \
  --bypass-governance-retention
```

**Key Point:** Only users with `s3:BypassGovernanceRetention` permission can override

---

### **How Compliance Mode Works**

**What It Does:**
- **Absolutely prevents** deletion/overwriting by **anyone** (including root user)
- Retention period **cannot be shortened** (can only be extended)
- **No one** can bypass (not even AWS Support)

**Use Cases:**
- **Regulatory compliance:** SEC Rule 17a-4, FINRA, HIPAA, GDPR
- **Legal preservation:** Court-ordered data retention
- **Audit logs:** Immutable audit trails

**Key Point:** Once an object version is locked in Compliance mode, it **cannot be deleted** until the retention period expires

**Example:**
- Object locked in Compliance mode for 7 years
- Even if you delete the bucket, the object remains until 7 years expire
- **No exceptions**

---

### **Retention Period**

**What It Is:** The length of time an object version is protected from deletion/modification

**Configuration:**
- Set at **bucket level** (default retention for all new objects)
- Set at **object level** (per-version retention)
- Can specify retention until a specific **date** or for a **number of days**

**Retention Period Options:**

| Setting | Details |
|---------|---------|
| **Fixed Date** | Retain until specific date (e.g., 2030-12-31) |
| **Number of Days** | Retain for X days from object creation (e.g., 2,555 days = 7 years) |
| **No Retention** | Object not locked (can still use Legal Hold) |

**Key Behaviors:**
- ✅ Retention period **can be extended** (both Governance and Compliance)
- ✅ Governance mode: Can be shortened (with `BypassGovernanceRetention` permission)
- ❌ Compliance mode: **Cannot be shortened** (immutable)

---

### **Legal Hold**

**What It Is:** An **on/off switch** that prevents deletion/modification **indefinitely** (no expiration date)

**Key Differences from Retention:**

| Feature | Retention Period | Legal Hold |
|---------|------------------|------------|
| **Duration** | Fixed time period | **Indefinite** (until manually removed) |
| **Expiration** | Automatic (after period ends) | Manual (must be removed explicitly) |
| **Permission to Remove** | N/A | `s3:PutObjectLegalHold` |
| **Use Case** | Compliance (known duration) | **Litigation, investigations** (unknown duration) |

**Use Cases:**
- **Litigation hold:** Preserve evidence during legal investigation
- **Pending investigation:** Freeze data until cleared
- **Emergency freeze:** Immediately prevent deletion (add Legal Hold, then configure retention)

**How to Set Legal Hold:**
```bash
aws s3api put-object-legal-hold \
  --bucket my-bucket \
  --key my-object \
  --legal-hold Status=ON
```

**How to Remove Legal Hold:**
- Requires `s3:PutObjectLegalHold` permission
- Simply set `Status=OFF`

**Key Point:** Legal Hold is **independent** of retention mode (can use both together)

---

### **Retention vs. Legal Hold (Side-by-Side)**

| Aspect | Retention (Governance/Compliance) | Legal Hold |
|--------|-----------------------------------|------------|
| **Duration** | Fixed period (days/date) | Indefinite |
| **Expiration** | Automatic | Manual removal required |
| **Use Case** | Compliance (SEC, HIPAA) | Litigation, investigations |
| **Can Extend?** | ✅ Yes | N/A (already indefinite) |
| **Can Remove?** | Governance: Yes (with permission)<br>Compliance: No | ✅ Yes (with `PutObjectLegalHold` permission) |

**Exam Tip:** Retention = **compliance/regulatory** (fixed period). Legal Hold = **litigation/investigation** (indefinite).

---

### **Bucket-Level vs. Object-Level Lock**

**Bucket-Level (Default Retention):**
- Set default retention for **all new objects** uploaded to bucket
- Each new object automatically gets this retention
- Objects can have **longer** retention (overriding default)

**Object-Level (Per-Version Retention):**
- Set retention on **specific object version**
- Overrides bucket-level default
- More granular control

**Best Practice:**
- Set bucket-level default for consistent compliance
- Override at object-level when specific objects need longer/shorter retention

---

### **How to Enable S3 Object Lock**

**Step 1:** Enable at bucket creation (cannot enable on existing bucket)
```bash
aws s3api create-bucket \
  --bucket my-compliant-bucket \
  --object-lock-enabled-for-bucket
```

**Step 2:** Enable versioning (automatic when Object Lock is enabled)

**Step 3:** Set default retention (optional)
```bash
aws s3api put-object-lock-configuration \
  --bucket my-compliant-bucket \
  --object-lock-configuration \
    'Mode=COMPLIANCE,Days=2555'
```

**Step 4:** Upload objects (they inherit default retention)

**Key Point:** Once Object Lock is enabled on a bucket, it **cannot be disabled** (but you can stop setting retention on new objects)

---

### **Exam Scenarios**

| Scenario | Solution |
|----------|----------|
| "SEC 17a-4 compliance (7-year retention)" | **Compliance Mode** with 2,555-day retention |
| "Protect from accidental deletion by admins" | **Governance Mode** |
| "Litigation hold (unknown duration)" | **Legal Hold** |
| "Prevent deletion by anyone, including root" | **Compliance Mode** |
| "Test retention policy before production" | **Governance Mode** (test, then switch to Compliance) |
| "Audit logs must be immutable" | **Compliance Mode** |
| "Need to extend retention period" | ✅ Both Governance and Compliance (can extend) |
| "Need to shorten retention period" | **Governance Mode** only (with BypassGovernanceRetention permission) |
| "Preserve evidence during investigation" | **Legal Hold** |
| "7-year retention + litigation hold" | **Compliance Mode retention** + **Legal Hold** (use both together) |

---

### **Object Lifecycle with Object Lock**

**What Happens When Retention Expires?**
1. Object becomes **unlocked** (can be deleted/overwritten)
2. Object is **not automatically deleted** (remains in bucket)
3. Lifecycle policy can delete it **after retention expires**

**Example:**
- Compliance Mode: 7-year retention
- Lifecycle Policy: Delete after 7 years + 1 day
- **Result:** Object is locked for 7 years, then automatically deleted

**Key Point:** Object Lock **prevents deletion** during retention. Lifecycle policies **can delete** after retention expires.

---

### **Cost Considerations**

| Cost Component | Details |
|----------------|---------|
| **Object Lock** | ❌ **No additional cost** (free feature) |
| **Versioning** | ✅ **Storage cost** for all versions (required for Object Lock) |
| **Retention** | ✅ **Storage cost** for objects during retention (cannot delete to save cost) |
| **Legal Hold** | ✅ **Storage cost** while Legal Hold is active |

**Cost Optimization:**
- Use **Governance Mode** for non-critical data (can delete with permission if needed)
- Combine with **Lifecycle Policies** to delete objects **after retention expires**
- Use **Intelligent-Tiering** or **Glacier** for long-term compliance data (reduces storage cost)

---

### **Key Exam Tips**

1. **Versioning required** (Object Lock works at version level)
2. **Enable at bucket creation only** (cannot enable on existing bucket)
3. **Governance Mode:** Can override with `s3:BypassGovernanceRetention` permission
4. **Compliance Mode:** **Cannot override** (not even root user)
5. **Retention period can be extended** (both modes)
6. **Retention period can be shortened:** Governance only (with permission)
7. **Legal Hold = indefinite** (no expiration, manual removal)
8. **Retention = fixed period** (automatic expiration)
9. **Use both together:** Compliance Mode + Legal Hold (maximum protection)
10. **Regulatory compliance (SEC, FINRA, HIPAA):** Compliance Mode
11. **Litigation/investigation:** Legal Hold
12. **Protect from accidental deletion:** Governance Mode
13. **Lifecycle policies can delete** objects **after retention expires**
14. **No additional cost** for Object Lock (pay only for storage of versions)
15. **WORM model:** Write-Once-Read-Many (immutable storage)

---

### **Governance vs. Compliance Decision Tree**

```
Need to prevent object deletion?
├─ Regulatory compliance (SEC, FINRA, HIPAA)?
│  └─ YES → Compliance Mode (cannot override)
├─ Accidental deletion protection?
│  └─ YES → Governance Mode (can override with permission)
├─ Litigation or investigation?
│  └─ YES → Legal Hold (indefinite, manual removal)
└─ Test retention before production?
   └─ YES → Governance Mode (test), then switch to Compliance Mode
```

---

### **Common Compliance Requirements**

| Regulation | Requirement | S3 Object Lock Solution |
|-----------|-------------|-------------------------|
| **SEC Rule 17a-4** | 7-year immutable retention | Compliance Mode, 2,555 days |
| **FINRA Rule 4511** | 6-year retention | Compliance Mode, 2,190 days |
| **HIPAA** | 6-year medical records | Compliance Mode, 2,190 days |
| **GDPR** | Right to erasure (delete on request) | ❌ **Do NOT use Object Lock** (conflicts with "right to be forgotten") |
| **Sarbanes-Oxley** | Audit log retention | Compliance Mode with appropriate retention |

**Exam Tip:** GDPR's "right to erasure" conflicts with Object Lock (Compliance Mode prevents deletion)

---

- **Performance Optimization**
    - **S3 Transfer Acceleration (S3TA):** For fast, secure uploads over long geographical distances. It uses AWS Edge Locations to route your traffic over the optimized AWS network backbone.
        - **Use Case:** A user in Europe needs to upload a large video file to a bucket in the US.
    - **Multipart Upload:** The recommended way to upload large files (>100 MB). It breaks the file into smaller parts that can be uploaded in parallel, increasing throughput and resilience. The AWS CLI and SDKs do this automatically for large files.
- **Data Processing & Integrations**
    - **S3 Event Notifications:** Trigger an action when an S3 event occurs (e.g., an object is created or deleted).
        - **Targets:** Can send events to **SNS**, **SQS**, or directly invoke an **AWS Lambda** function.
        - **Use Case:** Automatically create an image thumbnail (using Lambda) every time a new image is uploaded to S3.
    - **Static Website Hosting:** S3 can be configured to host a static website (HTML, CSS, JS). For a production website, you would put **Amazon CloudFront** in front of the S3 bucket to provide a custom domain (via HTTPS), caching, and lower latency.

[S3 Transfer Acceleration (S3TA)](S3%20Transfer%20Acceleration%20(S3TA).md)
