- **What is it? (1-Sentence Pitch):** AWS Snow Family consists of physical devices (Snowcone, Snowball Edge, Snowmobile) that help you transfer massive amounts of data (terabytes to exabytes) to and from AWS when network transfer is impractical or cost-prohibitive.

- **Core Use Cases:**
    - Migrating large datasets to AWS (petabytes of data)
    - Data center shutdowns and migrations
    - Disaster recovery (ship data offsite for backup)
    - Edge computing and processing data in remote locations (oil rigs, ships, military bases)
    - Content distribution (ship large media files to remote locations)

---

## Snow Family Devices:

### **1. AWS Snowcone**
- **Capacity:** 8 TB (HDD) or 14 TB (SSD)
- **Size:** Small, portable, rugged (2.1 kg / 4.5 lbs)
- **Use Case:** Edge computing, small data transfers, IoT, remote locations
- **Features:** Battery-powered option, can run EC2 instances and Lambda functions
- **Transfer Method:** Ship back to AWS **or** online transfer via AWS DataSync

### **2. AWS Snowball Edge**
- **Capacity:** 80 TB (Storage Optimized) or 42 TB (Compute Optimized) or 210 TB (Storage Optimized with EC2)
- **Size:** Suitcase-sized (22 kg / 49.7 lbs)
- **Use Case:** Large data migrations, edge computing, local storage/compute
- **Features:** Can run EC2 instances, Lambda functions, and use S3-compatible storage locally
- **Variants:**
    - **Storage Optimized:** Maximize storage capacity (80-210 TB)
    - **Compute Optimized:** More compute power (vCPUs, GPUs for ML)

### **3. AWS Snowmobile**
- **Capacity:** **100 PB** (petabytes!) per Snowmobile
- **Size:** 45-foot shipping container (pulled by semi-trailer truck)
- **Use Case:** Exabyte-scale data center migration (10 PB+ data)
- **Security:** GPS tracking, 24/7 video surveillance, escort security vehicle
- **Transfer Method:** Ship back to AWS only (no online option)

---

## Snow Family Comparison:

| **Device** | **Capacity** | **Size** | **Use Case** |
|---|---|---|---|
| **Snowcone** | 8-14 TB | Small, portable | Edge computing, small migrations |
| **Snowball Edge** | 42-210 TB | Suitcase | Large migrations, edge compute |
| **Snowmobile** | **100 PB** | Shipping container (truck) | **Exabyte-scale** migrations |

---

## When to Use Snow Family vs Online Transfer:

| **Scenario** | **Recommendation** |
|---|---|
| < 10 TB | **Online transfer** (Direct Connect, VPN, internet) |
| 10-100 TB | **Snowball Edge** |
| 100 TB - 10 PB | **Multiple Snowball Edge** devices |
| 10 PB+ | **Snowmobile** |
| Limited bandwidth | **Snow devices** (faster than network transfer) |

---

## Snow Family Workflow:

1. **Order Device:** Request Snowcone/Snowball/Snowmobile via AWS Console
2. **Receive Device:** AWS ships device to you
3. **Load Data:** Connect device to your network, copy data to device
4. **Ship Back:** Return device to AWS (Snowmobile is picked up)
5. **Data Import:** AWS loads your data into S3 (or Glacier)
6. **Verification:** AWS verifies data integrity and securely erases device

---

## Integration:

- **S3:** Primary destination for data transferred via Snow devices
- **Glacier:** Can import directly to Glacier for archival
- **DataSync:** Snowcone supports online transfer via DataSync (alternative to shipping back)
- **EC2/Lambda:** Snowball Edge and Snowcone can run compute workloads locally
- **Storage Gateway:** Can export Storage Gateway data to Snowball

---

## Exam Tips / What to Remember:

- **Snow Family = Physical data transfer** when network transfer is too slow/expensive
- **Snowcone (small)** → **Snowball Edge (medium)** → **Snowmobile (massive)**
- Exam scenario: **"Transfer 50 TB to AWS, limited bandwidth"** = Snowball Edge
- Exam scenario: **"Migrate 50 PB data center to AWS"** = Snowmobile
- **Snowcone can use AWS DataSync** for online transfer (doesn't need to be shipped back)
- **Snowball Edge can run EC2 and Lambda** at the edge (edge computing)
- **Snowmobile for exabyte-scale migrations** (10 PB+)
- All devices are **encrypted** (256-bit encryption, KMS integration)
- **Rule of thumb:** If it takes more than a week to transfer over network, use Snow device
- **Use case overlap:** DataSync (online agent-based) vs Snow Family (offline physical transfer)
