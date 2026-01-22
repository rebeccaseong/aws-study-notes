# Cheat Sheet: EC2 Instance Types (Families & Use Cases)

Quick reference for EC2 instance families and their exam-relevant use cases.

---

## Memory Aid: **M-CRAGS-IP**

| **Letter** | **Family** | **Mnemonic** | **Optimized For** |
|---|---|---|---|
| **M** | General Purpose | **M**y **T**all | Balanced (CPU, Memory, Network) |
| **C** | Compute Optimized | **C**omputer | High-performance CPUs |
| **R** | Memory Optimized | **R**eally **M**emorable / **R**AM | Large amounts of memory |
| **A** | ARM-based | **A**RM / **A**mazon (Graviton) | Cost-effective, energy-efficient |
| **G** | GPU/Graphics | **G**raphics **P**ower | Graphics rendering, ML training |
| **S** | Storage Optimized | **I**'m **D**ense | High I/O, local storage |
| **I** | (see Storage) | (see Storage) | (see Storage) |
| **P** | (see Accelerated) | (see Accelerated) | (see Accelerated) |

---

## Instance Family Details

### **1. General Purpose (M, T)**

| **Family** | **Key Features** | **Use Cases** | **Exam Tip** |
|---|---|---|---|
| **M Series** (M5, M6i, M7g) | Balanced CPU, memory, network | Web servers, application servers, dev/test | Default choice for most workloads |
| **T Series** (T3, T4g) | **Burstable** (baseline + burst credits) | Small databases, dev/test, low-traffic web apps | **Burstable** = keyword for T-series |

**Exam Keyword:** "Balanced workload" / "General-purpose" → **M or T series**

---

### **2. Compute Optimized (C)**

| **Family** | **Key Features** | **Use Cases** | **Exam Tip** |
|---|---|---|---|
| **C Series** (C5, C6i, C7g) | High-performance CPUs (3.5+ GHz) | Batch processing, media transcoding, HPC, gaming servers | CPU-intensive tasks |

**Exam Keyword:** "High-performance processor" / "CPU-intensive" / "Batch processing" → **C series**

---

### **3. Memory Optimized (R, X, High Memory, z1d)**

| **Family** | **Key Features** | **Use Cases** | **Exam Tip** |
|---|---|---|---|
| **R Series** (R5, R6i, R7g) | Large amounts of RAM | In-memory databases (Redis, Memcached), real-time big data analytics | Memory-intensive |
| **X Series** (X2iedn, X2iezn) | **Extreme memory** (up to 4 TB RAM per instance) | SAP HANA, in-memory databases | Massive RAM requirements |
| **High Memory** (u-6tb1, u-24tb1) | **Largest memory** (up to 24 TB RAM!) | SAP HANA, very large in-memory databases | Exam rarely asks about this |
| **z1d** | High memory **+ high compute + NVMe storage** | Electronic Design Automation (EDA), gaming | Rare in exams |

**Exam Keyword:** "In-memory database" / "Large RAM" / "Real-time analytics" → **R series**

---

### **4. Accelerated Computing (P, G, Inf, Trn, DL)**

| **Family** | **Key Features** | **Use Cases** | **Exam Tip** |
|---|---|---|---|
| **P Series** (P3, P4, P5) | **GPUs** (NVIDIA Tesla) | ML training, HPC, scientific computing | GPU = P series |
| **G Series** (G4dn, G5) | **GPUs** (NVIDIA) | Graphics rendering, video encoding, game streaming | Graphics = G series |
| **Inf Series** (Inf1, Inf2) | **AWS Inferentia** (ML inference chip) | Low-cost ML inference (predictions) | Inference-specific |
| **Trn Series** (Trn1, Trn1n) | **AWS Trainium** (ML training chip) | Cost-effective ML model training | Training-specific |
| **DL Series** (DL1) | **Gaudi accelerators** | Deep learning training | Rare in exams |

**Exam Keyword:**
- "Machine learning training" → **P or Trn series**
- "Graphics rendering" / "Video encoding" → **G series**
- "ML inference" / "Low-cost predictions" → **Inf series**

---

### **5. Storage Optimized (I, D, H)**

| **Family** | **Key Features** | **Use Cases** | **Exam Tip** |
|---|---|---|---|
| **I Series** (I3, I4i) | **NVMe SSD**, millions of IOPS, high random I/O | NoSQL databases (Cassandra, MongoDB), data warehousing | High I/O = I series |
| **D Series** (D2, D3) | **HDD**, high sequential I/O, dense storage | Distributed file systems (Hadoop, HDFS), MapReduce | High throughput, sequential |
| **H Series** (H1) | **HDD**, high throughput | MapReduce, distributed file systems | Similar to D-series |

**Exam Keyword:**
- "High random I/O" / "NoSQL database" / "Millions of IOPS" → **I series**
- "Hadoop" / "MapReduce" / "Distributed file system" → **D or H series**

---

### **6. ARM-based (Graviton) (A, T4g, M6g, C7g, R7g)**

| **Family** | **Key Features** | **Use Cases** | **Exam Tip** |
|---|---|---|---|
| **Graviton** (M6g, C7g, R7g, etc.) | AWS custom ARM chips (energy-efficient) | **40% better price/performance** vs x86 | Cost optimization |
| **A1** | First-gen Graviton | Scale-out workloads (web servers, microservices) | Legacy, rare in exams |

**Exam Keyword:** "Cost optimization" / "Better price/performance" → **Graviton (g-suffix)**

---

## Exam Scenarios Quick Reference

| **Exam Scenario** | **Instance Type** |
|---|---|
| Web server, balanced workload | **M series** (General Purpose) |
| Dev/test environment, low traffic | **T series** (Burstable) |
| Batch processing, media transcoding | **C series** (Compute Optimized) |
| In-memory database (Redis, Memcached) | **R series** (Memory Optimized) |
| Machine learning training | **P series** (GPU) or **Trn series** (Trainium) |
| Graphics rendering, video encoding | **G series** (GPU) |
| NoSQL database, millions of IOPS | **I series** (Storage Optimized, NVMe) |
| Hadoop, MapReduce, HDFS | **D series** (Storage Optimized, HDD) |
| Cost optimization, better price/performance | **Graviton** (M6g, C7g, R7g, etc.) |

---

## Instance Size Naming Convention

**Example:** `m5.2xlarge`

- **m** = Family (General Purpose)
- **5** = Generation (newer = better performance)
- **2xlarge** = Size (larger = more vCPUs, RAM)

**Sizes:** nano → micro → small → medium → large → xlarge → 2xlarge → 4xlarge → ... → 192xlarge (largest)

---

## Summary

- **General Purpose (M, T):** Balanced, most common use case
- **Compute (C):** CPU-intensive workloads
- **Memory (R, X):** RAM-heavy, in-memory databases
- **Accelerated (P, G):** GPU for ML/graphics
- **Storage (I, D):** High I/O, local storage
- **Graviton (g-suffix):** Cost-optimized ARM instances
