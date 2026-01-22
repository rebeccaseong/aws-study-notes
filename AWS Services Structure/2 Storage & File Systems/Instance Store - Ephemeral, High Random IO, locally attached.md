- **What is it? (1-Sentence Pitch):** Instance Store provides temporary block-level storage that is physically attached to the host computer running your EC2 instance, offering very high I/O performance but losing all data when the instance stops or terminates.

- **Core Use Cases:**
    - Temporary storage for buffers, caches, scratch data
    - High-performance databases that replicate data (e.g., NoSQL databases with built-in replication)
    - Data that is replicated across multiple instances
    - Applications requiring extremely high random I/O performance

- **Key Features & Concepts:**
    - **Ephemeral Storage:** Data is lost when instance **stops** or **terminates** (also lost on hardware failure)
    - **Physically Attached:** Storage is on disks physically attached to the host machine (not network-attached like EBS)
    - **Very High Performance:** Extremely low latency and high IOPS (millions of IOPS possible)
    - **Included in Instance Price:** No additional cost beyond the EC2 instance
    - **Size Varies by Instance Type:** Certain instance types (I3, D3, etc.) come with instance store volumes
    - **Cannot be Detached/Reattached:** Unlike EBS, you cannot detach and attach to another instance

- **Instance Store vs EBS:**
    | **Aspect** | **Instance Store** | **EBS** |
    |---|---|---|
    | **Persistence** | **Ephemeral** (data lost on stop) | **Persistent** (data survives stop) |
    | **Performance** | **Very high** (local disk) | High (network-attached) |
    | **Backup** | No snapshots | EBS snapshots available |
    | **Use Case** | Temporary, high-performance data | Persistent application data |
    | **Cost** | Included in instance price | Charged separately |
    | **Availability** | Specific instance types only | All instance types |

- **Integration:**
    - **EC2 Instances:** Only certain instance types (I3, D3, R5d, M5d, etc.) have instance store volumes
    - Typically used alongside **EBS** (EBS for persistent data, Instance Store for temporary/cache)

- **Exam Tips / What to Remember:**
    - **Ephemeral = Data is lost** when instance stops, terminates, or underlying hardware fails
    - **Use for temporary data only** (buffers, caches, replicated data)
    - **Cannot create snapshots** of instance store volumes
    - **Very high performance** (lower latency than EBS) due to local attachment
    - Exam keyword: **"Ephemeral storage"** or **"High Random I/O"** = Instance Store
    - **Not all EC2 instances have instance store** - only certain types (I, D, R5d, etc.)
    - Instance store **cannot be stopped and started** - only rebooted (reboot preserves data)
    - If you need **persistent high-performance storage**, use **EBS io2** instead
