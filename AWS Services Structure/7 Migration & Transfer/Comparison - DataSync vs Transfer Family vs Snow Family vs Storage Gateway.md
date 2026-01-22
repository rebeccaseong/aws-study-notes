# Comparison: DataSync vs Transfer Family vs Snow Family vs Storage Gateway

Quick reference for choosing the right data migration and transfer service.

---

## High-Level Decision Tree

```
What's your data transfer requirement?

├─ One-time large migration (TB/PB)
│  ├─ Good network bandwidth (10 Gbps+)
│  │  └─ DataSync (online agent-based)
│  └─ Limited network bandwidth
│     └─ Snow Family (offline physical devices)
│
├─ Ongoing file transfers (FTP/SFTP)
│  └─ Transfer Family (managed FTP/SFTP/FTPS server)
│
└─ Hybrid cloud storage (on-prem + AWS)
   └─ Storage Gateway (cache frequently accessed data locally)
```

---

## Comparison Table

| Aspect | DataSync | Transfer Family | Snow Family | Storage Gateway |
|--------|----------|-----------------|-------------|-----------------|
| **Purpose** | **Fast data migration** (online) | **Managed FTP/SFTP server** | **Offline data transfer** (physical) | **Hybrid cloud storage** |
| **Method** | Online (agent on source) | FTP/SFTP over network | **Ship physical device** | On-prem gateway + cloud storage |
| **Bandwidth** | Uses network (efficient, automated) | Uses network (standard protocols) | **No network** (physical shipping) | Uses network (cached access) |
| **Use Case** | **Large migrations**, ongoing sync | **Legacy FTP workflows** | **Massive data** (TB/PB), limited bandwidth | **Low-latency hybrid access** |
| **Protocols** | DataSync protocol | **FTP, SFTP, FTPS** | N/A (physical device) | NFS, SMB, iSCSI |
| **Automation** | ✅ Fully automated | ❌ Manual (user uploads via FTP) | ❌ Manual (load data, ship back) | ✅ Automated caching |
| **Speed** | **Up to 10 Gbps** | Depends on network | **Fastest** (no network limit) | Depends on network + caching |
| **Cost** | $ (pay per GB transferred) | $$ (endpoint hours + data transfer) | $$$ (device rental + shipping) | $ (gateway + storage) |
| **Setup** | Install agent on source | Create managed FTP endpoint | Order device from AWS | Deploy gateway appliance |

---

## When to Use Each Service

### **Use DataSync If:**

✅ **Migrating large amounts of data** to AWS (on-prem → S3/EFS/FSx)
✅ Have **good network bandwidth** (1+ Gbps)
✅ Need **automated, scheduled transfers** (ongoing sync)
✅ Want **fast, efficient migration** (accelerated transfer, compression, deduplication)
✅ Migrating **NAS/file server data** to AWS

**Common Use Cases:**
- Migrate file server to EFS or FSx
- Data lake migration (on-prem → S3)
- Active data archiving (ongoing sync to S3 Glacier)
- Disaster recovery (replicate data to AWS)

**Key Features:**
- **Automated:** Schedule transfers, verify data integrity
- **Efficient:** Network optimization, compression, encryption
- **Supports:** NFS, SMB, HDFS, S3, EFS, FSx

---

### **Use Transfer Family If:**

✅ Need **managed FTP/SFTP/FTPS server** in AWS
✅ **Migrating from legacy FTP workflows**
✅ Partners/customers send files via FTP/SFTP
✅ Need **SFTP endpoint** for third-party integrations
✅ Want to **avoid managing FTP servers** (fully managed)

**Common Use Cases:**
- Receive files from partners via SFTP
- Migrate legacy FTP workflows to AWS
- Share files with external parties (SFTP endpoint)
- B2B file transfers

**Key Features:**
- **Managed:** AWS handles server management (no EC2 to patch)
- **Protocols:** FTP, SFTP, FTPS, AS2
- **Storage:** Files stored in S3 or EFS
- **Authentication:** Integrate with Active Directory, custom IdP, or service-managed users

---

### **Use Snow Family If:**

✅ **Massive data transfer** (tens of TB to PB)
✅ **Limited or no network bandwidth** (remote locations)
✅ Network transfer would take **too long** (weeks/months)
✅ **One-time large migration** (data center shutdown)
✅ Need **edge computing** in disconnected environments (oil rigs, ships)

**Common Use Cases:**
- Data center shutdown / migration
- Disaster recovery (ship data offsite)
- Content distribution to remote locations
- Edge computing (process data locally, send to AWS later)

**Devices:**
- **Snowcone:** 8-14 TB (portable, small migrations)
- **Snowball Edge:** 42-210 TB (large migrations, edge computing)
- **Snowmobile:** 100 PB (exabyte-scale migrations)

**Rule of Thumb:** If network transfer > 1 week, use Snow device

---

### **Use Storage Gateway If:**

✅ Need **hybrid cloud storage** (on-prem + AWS)
✅ Want **low-latency access** to frequently used data (cached locally)
✅ Migrating **gradually** to cloud (while keeping on-prem access)
✅ Need **backup to AWS** from on-prem applications
✅ Want **seamless integration** with on-prem apps (NFS, SMB, iSCSI)

**Common Use Cases:**
- Extend on-prem file server to S3 (transparent to users)
- Backup on-prem VMs to AWS
- Disaster recovery (replicate on-prem data to AWS)
- Tiered storage (hot data on-prem, cold data in S3 Glacier)

**Gateway Types:**
- **File Gateway:** NFS/SMB access to S3
- **Volume Gateway:** iSCSI block storage (cached or stored mode)
- **Tape Gateway:** Virtual tape library (VTL) for backup

---

## Detailed Comparison

### **DataSync**

| Feature | Details |
|---------|---------|
| **How It Works** | Install agent on source, agent transfers data to AWS |
| **Speed** | **Up to 10 Gbps**, accelerated transfer, compression |
| **Automation** | Scheduled transfers, data verification, retry logic |
| **Supported Sources** | On-prem (NFS, SMB, HDFS), AWS (S3, EFS, FSx), other cloud |
| **Pricing** | $0.0125 per GB transferred |

**Use Case:** Migrate 100 TB from on-prem file server to S3

---

### **Transfer Family**

| Feature | Details |
|---------|---------|
| **How It Works** | Managed FTP/SFTP/FTPS server, users upload via FTP client |
| **Protocols** | FTP, SFTP, FTPS, AS2 |
| **Storage Backend** | S3 or EFS (files land in S3 bucket or EFS file system) |
| **Authentication** | Service-managed, Active Directory, custom IdP (Lambda) |
| **Pricing** | $0.30/hour per endpoint + data transfer |

**Use Case:** Receive files from 50 partners via SFTP (store in S3)

---

### **Snow Family**

| Feature | Details |
|---------|---------|
| **How It Works** | Order device → Load data → Ship back to AWS → AWS loads data to S3 |
| **Devices** | Snowcone (8-14 TB), Snowball Edge (42-210 TB), Snowmobile (100 PB) |
| **Encryption** | 256-bit encryption, KMS integration |
| **Shipping** | AWS ships device to you, you ship back (or use DataSync for Snowcone) |
| **Pricing** | Device rental + shipping + S3 storage |

**Use Case:** Migrate 500 TB (network transfer would take 6 weeks, Snowball takes 1 week)

---

### **Storage Gateway**

| Feature | Details |
|---------|---------|
| **How It Works** | On-prem gateway caches data, backs up to S3/Glacier |
| **Gateway Types** | File (NFS/SMB), Volume (iSCSI), Tape (VTL) |
| **Caching** | Frequently accessed data cached locally (low latency) |
| **Storage Backend** | S3, S3 Glacier, EBS snapshots |
| **Pricing** | Gateway + storage + data transfer |

**Use Case:** On-prem file server with transparent S3 backup (users access via NFS)

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Migrate 50 TB from on-prem NAS to S3 (good bandwidth) | **DataSync** |
| Partners upload files via SFTP | **Transfer Family** |
| Migrate 500 TB (limited bandwidth, 1 week deadline) | **Snow Family (Snowball Edge)** |
| Hybrid storage (on-prem + S3, low latency) | **Storage Gateway** |
| Migrate entire data center (exabyte-scale) | **Snow Family (Snowmobile)** |
| Ongoing sync (on-prem file server → S3) | **DataSync** (scheduled) |
| Backup on-prem VMs to AWS | **Storage Gateway (Volume Gateway)** |
| Replace legacy FTP server | **Transfer Family** |
| Edge computing in remote location | **Snow Family (Snowball Edge)** |

---

## Key Exam Tips

1. **DataSync = Online migration** (agent-based, fast, automated)
2. **Transfer Family = Managed FTP/SFTP** (for legacy workflows)
3. **Snow Family = Offline migration** (physical devices for massive data)
4. **Storage Gateway = Hybrid storage** (on-prem + cloud, cached access)
5. **Good bandwidth?** → DataSync
6. **Limited bandwidth?** → Snow Family
7. **FTP/SFTP workflow?** → Transfer Family
8. **Need low-latency on-prem access?** → Storage Gateway
9. **"Migrate 100 TB in 1 week"** → Snowball Edge (network would take too long)
10. **"Ongoing sync"** → DataSync (scheduled transfers)

---

## Decision Matrix: DataSync vs Snow Family

| Factor | DataSync | Snow Family |
|--------|----------|-------------|
| **Network Bandwidth** | **1 Gbps+** | **<1 Gbps** or none |
| **Data Size** | TB scale | **TB to PB** scale |
| **Timeline** | Days to weeks | **Faster** (no network bottleneck) |
| **Ongoing?** | **Yes** (scheduled sync) | No (one-time migration) |

**Rule of Thumb:**
- Transfer time over network **< 1 week** → DataSync
- Transfer time over network **> 1 week** → Snow Family

---

## Summary

| Service | Best For | Transfer Method |
|---------|----------|-----------------|
| **DataSync** | Fast online migration, ongoing sync | Agent-based, automated |
| **Transfer Family** | Managed FTP/SFTP server | FTP/SFTP protocol |
| **Snow Family** | Offline massive data transfer | Physical device shipping |
| **Storage Gateway** | Hybrid cloud storage | Cached on-prem access + S3 backup |
