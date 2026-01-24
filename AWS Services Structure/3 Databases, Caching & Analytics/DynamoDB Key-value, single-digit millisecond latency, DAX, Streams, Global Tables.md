- **What is it? (1-Sentence Pitch):** A fully managed, serverless, key-value and document NoSQL database designed for single-digit millisecond performance at any scale.
- **Core Use Cases:** Web and mobile apps, gaming leaderboards, IoT applications, and any application needing extremely low latency and massive scalability.
- **Key Features & Concepts:**
    - **Capacity Modes:** Provisioned (you specify read/write capacity) and On-Demand (pay-per-request).
    - **DAX (DynamoDB Accelerator):** A fully managed, in-memory cache for DynamoDB that delivers microsecond latency for read-heavy workloads.
    - **Global Tables:** Creates a multi-region, multi-active database for globally distributed applications.
    - **DynamoDB Streams:** A time-ordered sequence of item-level changes. Perfect for triggering Lambda functions for event-driven processing.

---

## DynamoDB Capacity Modes (Provisioned vs. On-Demand)

**Key Decision:** How you pay for and provision throughput capacity for your DynamoDB table.

---

### **Capacity Modes Overview**

| Capacity Mode | How It Works | Best For | Cost Structure |
|---------------|-------------|----------|----------------|
| **Provisioned** | Specify read/write capacity in advance (RCU/WCU) | **Predictable traffic**, cost optimization | Pay for provisioned capacity (whether used or not) |
| **On-Demand** | Pay-per-request (no capacity planning) | **Unpredictable/spiky traffic**, new applications | Pay only for requests made (no minimum) |

**Exam Tip:** Provisioned = **predictable**, On-Demand = **unpredictable/spiky**

---

### **Provisioned Capacity Mode**

**What It Is:** You specify the number of **Read Capacity Units (RCU)** and **Write Capacity Units (WCU)** your table needs.

---

#### **Read Capacity Units (RCU)**

**What 1 RCU Provides:**
- **1 strongly consistent read per second** for items up to **4 KB**
- **2 eventually consistent reads per second** for items up to **4 KB**

**Calculation Examples:**

| Scenario | Calculation | RCUs Needed |
|----------|-------------|-------------|
| 10 strongly consistent reads/sec, 4 KB each | 10 reads × 1 RCU = **10 RCU** | 10 |
| 10 eventually consistent reads/sec, 4 KB each | 10 reads × 0.5 RCU = **5 RCU** | 5 |
| 10 strongly consistent reads/sec, 8 KB each | 10 reads × 2 RCU (8 KB ÷ 4 KB) = **20 RCU** | 20 |
| 10 strongly consistent reads/sec, 6 KB each | 10 reads × 2 RCU (6 KB rounds up to 8 KB) = **20 RCU** | 20 |

**Key Points:**
- Reads are rounded up to the nearest **4 KB** increment
- Eventually consistent reads are **half the cost** of strongly consistent reads

---

#### **Write Capacity Units (WCU)**

**What 1 WCU Provides:**
- **1 write per second** for items up to **1 KB**

**Calculation Examples:**

| Scenario | Calculation | WCUs Needed |
|----------|-------------|-------------|
| 10 writes/sec, 1 KB each | 10 writes × 1 WCU = **10 WCU** | 10 |
| 10 writes/sec, 2 KB each | 10 writes × 2 WCU (2 KB ÷ 1 KB) = **20 WCU** | 20 |
| 10 writes/sec, 0.5 KB each | 10 writes × 1 WCU (rounds up to 1 KB) = **10 WCU** | 10 |
| 10 writes/sec, 1.5 KB each | 10 writes × 2 WCU (1.5 KB rounds up to 2 KB) = **20 WCU** | 20 |

**Key Points:**
- Writes are rounded up to the nearest **1 KB** increment
- All writes are strongly consistent (no "eventually consistent write" option)

---

#### **Exam Calculation Practice**

**Question 1:** You need to perform 50 strongly consistent reads per second, with each item being 3.5 KB. How many RCUs?
- **Answer:** 3.5 KB rounds up to 4 KB → 50 reads × 1 RCU = **50 RCU**

**Question 2:** You need to perform 100 eventually consistent reads per second, with each item being 10 KB. How many RCUs?
- **Answer:** 10 KB ÷ 4 KB = 2.5 → rounds up to 3 × 4 KB = 12 KB → 3 RCU per read
- Eventually consistent = 3 RCU ÷ 2 = 1.5 RCU per read
- 100 reads × 1.5 RCU = **150 RCU**

**Question 3:** You need to write 20 items per second, with each item being 2.5 KB. How many WCUs?
- **Answer:** 2.5 KB rounds up to 3 KB → 3 WCU per write → 20 writes × 3 WCU = **60 WCU**

---

#### **Provisioned Capacity - Auto Scaling**

**What It Is:** Automatically adjusts provisioned capacity (RCU/WCU) in response to actual traffic patterns.

**How It Works:**
1. Set **target utilization** (e.g., 70% of provisioned capacity)
2. Set **minimum and maximum capacity** (e.g., 5-1000 RCU)
3. DynamoDB monitors CloudWatch metrics
4. **Scales up** when utilization exceeds target (within minutes)
5. **Scales down** when utilization drops (within 15+ minutes)

**Example:**
- Target: 70% utilization
- Min: 5 RCU, Max: 1000 RCU
- Current: 100 RCU, utilization 80%
- **Auto Scaling:** Increases to ~115 RCU (to bring utilization down to 70%)

---

**Auto Scaling Characteristics:**

| Aspect | Details |
|--------|---------|
| **Scale Up Speed** | Fast (within 1-5 minutes) |
| **Scale Down Speed** | Slow (15+ minutes, limited to 4 scale-downs per day) |
| **Target Metric** | Target utilization (%) of provisioned capacity |
| **Cost** | ❌ **No additional cost** for Auto Scaling (pay only for provisioned capacity) |
| **Limitations** | Cannot scale down more than **4 times per UTC day** |

**Exam Tip:** Auto Scaling scales **up quickly** but **down slowly** (prevents thrashing)

---

**When Auto Scaling Helps:**
- ✅ **Predictable but variable traffic** (e.g., higher during business hours, lower at night)
- ✅ **Gradual growth** (e.g., user base growing 10% per month)
- ✅ **Cost optimization** (don't pay for unused capacity at night)

**When Auto Scaling Doesn't Help:**
- ❌ **Sudden spikes** (scale-up takes minutes, may hit throttling first)
- ❌ **Extreme spikes** (max capacity limit may be too low)
- ❌ **Completely unpredictable traffic** (On-Demand mode is better)

---

#### **Burst Capacity**

**What It Is:** DynamoDB reserves unused capacity for **burst traffic** (short-term spikes).

**How It Works:**
- If you don't use all your provisioned capacity, DynamoDB **saves up to 5 minutes** of unused capacity
- You can **burst above** your provisioned capacity using this reserve
- **Not guaranteed** (best effort, not for production reliance)

**Example:**
- Provisioned: 100 WCU
- Usage: 50 WCU (average)
- Unused: 50 WCU × 5 minutes = 15,000 write units saved
- **Burst:** Can temporarily write at 200+ WCU using saved capacity

**Exam Tip:** Burst capacity is **not reliable** for production. Use Auto Scaling or On-Demand for predictable spike handling.

---

### **On-Demand Capacity Mode**

**What It Is:** Pay-per-request pricing with **no capacity planning** required.

**How It Works:**
- DynamoDB **automatically scales** to handle any traffic level
- You pay only for **actual reads and writes**
- **No minimum capacity** or upfront commitment
- **Instant scaling** (no warm-up period)

**Pricing (per request):**
- **Read:** $0.25 per million read request units
- **Write:** $1.25 per million write request units
- **Request Unit** = Same as RCU/WCU (4 KB reads, 1 KB writes)

---

**On-Demand Characteristics:**

| Aspect | Details |
|--------|---------|
| **Capacity Planning** | ❌ **None required** |
| **Scaling** | Instant (no delay) |
| **Cost** | Pay-per-request (no minimum) |
| **Cost Predictability** | Lower (depends on actual usage) |
| **Throttling** | Very rare (DynamoDB handles spikes automatically) |
| **Best For** | Unpredictable traffic, new applications, serverless |

---

**When to Use On-Demand:**
- ✅ **New application** (don't know traffic patterns yet)
- ✅ **Unpredictable/spiky traffic** (e.g., viral content, flash sales)
- ✅ **Serverless architecture** (Lambda-triggered workloads)
- ✅ **Dev/test environments** (intermittent usage)
- ✅ **Simplicity preferred** (don't want to manage capacity)

**When NOT to Use On-Demand:**
- ❌ **Predictable, steady traffic** (Provisioned is cheaper)
- ❌ **High, constant traffic** (Provisioned is significantly cheaper)
- ❌ **Cost optimization priority** (Provisioned + Auto Scaling is more cost-effective)

---

### **Provisioned vs. On-Demand: Side-by-Side**

| Aspect | Provisioned | On-Demand |
|--------|-------------|-----------|
| **Capacity Planning** | ✅ Required (specify RCU/WCU) | ❌ Not required |
| **Scaling** | Manual or Auto Scaling (minutes) | **Instant** (automatic) |
| **Cost Model** | Pay for provisioned capacity (even if unused) | Pay only for actual requests |
| **Cost Predictability** | **High** (fixed monthly cost) | Variable (depends on traffic) |
| **Cost for Steady Traffic** | **Lower** (more cost-effective) | Higher (pay-per-request) |
| **Cost for Spiky Traffic** | Higher (must provision for peaks) | **Lower** (pay only for actual usage) |
| **Best For** | Predictable traffic, cost optimization | Unpredictable traffic, serverless, new apps |
| **Throttling Risk** | ✅ Yes (if exceed provisioned capacity) | ❌ Very rare |
| **Auto Scaling** | ✅ Available (optional) | N/A (always scales automatically) |
| **Burst Capacity** | ✅ Available (5 min reserve) | N/A (always available) |

---

### **Cost Comparison Example**

**Scenario:** Table with 1 million reads/day, 100,000 writes/day (steady traffic)

**Provisioned Mode Calculation:**
- Reads: 1,000,000 ÷ 86,400 sec/day = **12 reads/sec** → 12 RCU
- Writes: 100,000 ÷ 86,400 sec/day = **1.2 writes/sec** → 2 WCU
- **Cost:** ~$0.70/month (12 RCU) + $0.35/month (2 WCU) = **$1.05/month**

**On-Demand Mode Calculation:**
- Reads: 1,000,000 ÷ 1,000,000 × $0.25 = **$0.25**
- Writes: 100,000 ÷ 1,000,000 × $1.25 = **$0.125**
- **Cost:** $0.25 + $0.125 = **$0.375/month**

**Result:** On-Demand is **cheaper** for this low-traffic scenario

**Scenario 2:** Table with 10 million reads/day, 1 million writes/day (steady)

**Provisioned Mode:**
- Reads: 116 RCU → ~$7/month
- Writes: 12 WCU → ~$2/month
- **Cost:** **$9/month**

**On-Demand Mode:**
- Reads: $2.50
- Writes: $1.25
- **Cost:** **$3.75/month**

**Result:** On-Demand is still **cheaper** for low-moderate traffic

**Scenario 3:** Table with 100 million reads/day, 10 million writes/day (steady)

**Provisioned Mode:**
- Reads: 1,157 RCU → ~$70/month
- Writes: 116 WCU → ~$20/month
- **Cost:** **$90/month**

**On-Demand Mode:**
- Reads: $25
- Writes: $12.50
- **Cost:** **$37.50/month**

**Result:** On-Demand is **cheaper** even for high steady traffic!

**Key Insight:** On-Demand is often **cheaper than expected**, especially if you don't need Reserved Capacity pricing.

---

### **Switching Between Modes**

**Can You Switch?**
- ✅ **Yes** (can switch between Provisioned and On-Demand)
- ⚠️ **Limitation:** Can only switch **once every 24 hours** per table

**How to Switch:**
```bash
aws dynamodb update-table \
  --table-name my-table \
  --billing-mode PAY_PER_REQUEST  # On-Demand

aws dynamodb update-table \
  --table-name my-table \
  --billing-mode PROVISIONED \
  --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5
```

**Use Case for Switching:**
- Start with **On-Demand** (learn traffic patterns)
- Switch to **Provisioned** after traffic becomes predictable (save costs)
- Switch back to **On-Demand** during special events (flash sales, marketing campaigns)

---

### **Reserved Capacity (Provisioned Only)**

**What It Is:** Commit to provisioned capacity for **1 or 3 years** for significant cost savings (like EC2 Reserved Instances).

**Savings:**
- **1-year commitment:** ~40% discount
- **3-year commitment:** ~75% discount

**Requirements:**
- Only available for **Provisioned mode**
- Minimum: 100 RCU or 100 WCU
- Cannot cancel (pay for full term)

**Use Case:**
- **Long-term, predictable workloads** with steady traffic
- **Cost optimization** (maximize savings)

**Exam Tip:** Reserved Capacity is **only for Provisioned mode** (not On-Demand)

---

### **Exam Scenarios**

| Scenario | Solution |
|----------|----------|
| "New mobile app with unknown traffic patterns" | **On-Demand** (no capacity planning) |
| "Steady, predictable traffic (24/7)" | **Provisioned** (cheaper for steady traffic) |
| "Traffic spikes during business hours (9-5)" | **Provisioned + Auto Scaling** |
| "Viral content app with unpredictable spikes" | **On-Demand** (instant scaling) |
| "Calculate RCUs for 50 reads/sec, 8 KB items" | 50 × 2 RCU = **100 RCU** (strongly consistent) |
| "Calculate WCUs for 100 writes/sec, 1.5 KB items" | 100 × 2 WCU = **200 WCU** (rounds up to 2 KB) |
| "Cost optimization for long-term steady traffic" | **Provisioned + Reserved Capacity** |
| "Serverless application (Lambda-triggered)" | **On-Demand** (pay only when Lambda runs) |
| "Need to handle flash sale (10x normal traffic)" | **On-Demand** OR **Provisioned + Auto Scaling** (with high max) |
| "Traffic is 70% utilization, want to optimize" | **Enable Auto Scaling** (adjust capacity automatically) |

---

### **Key Exam Tips**

1. **Provisioned = predictable**, On-Demand = **unpredictable/spiky**
2. **RCU:** 1 RCU = 1 strongly consistent read/sec (4 KB) OR 2 eventually consistent reads/sec
3. **WCU:** 1 WCU = 1 write/sec (1 KB)
4. **Rounding:** Reads round to 4 KB, Writes round to 1 KB (always round UP)
5. **Auto Scaling:** Scales up **quickly** (minutes), scales down **slowly** (15+ min, max 4/day)
6. **Burst Capacity:** Reserves up to **5 minutes** of unused capacity (not reliable for production)
7. **On-Demand:** No capacity planning, instant scaling, pay-per-request
8. **Provisioned is cheaper:** For **steady, predictable** traffic (especially with Reserved Capacity)
9. **On-Demand is cheaper:** For **low traffic, unpredictable spikes**, or **new applications**
10. **Switch limit:** Can switch modes **once per 24 hours**
11. **Reserved Capacity:** Only for **Provisioned mode** (not On-Demand)
12. **Cost comparison:** On-Demand often cheaper than expected for low-moderate traffic
13. **Flash sales/events:** Use On-Demand OR Provisioned with very high max capacity
14. **Serverless:** Use On-Demand (pay only when Lambda executes)
15. **Throttling:** Provisioned (if exceed capacity), On-Demand (very rare)

---

### **Capacity Mode Decision Tree**

```
Choosing Capacity Mode?
├─ Traffic pattern known/predictable?
│  ├─ YES, steady 24/7
│  │  └─ High traffic? → Provisioned + Reserved Capacity
│  ├─ YES, variable (day/night)
│  │  └─ Provisioned + Auto Scaling
│  └─ NO, unpredictable
│     └─ On-Demand
├─ Serverless (Lambda-triggered)?
│  └─ On-Demand
├─ New application (learning traffic)?
│  └─ Start On-Demand, switch to Provisioned later
└─ Cost optimization priority?
   └─ Analyze traffic, likely Provisioned + Auto Scaling
```
