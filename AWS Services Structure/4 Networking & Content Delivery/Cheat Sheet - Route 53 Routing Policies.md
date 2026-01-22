# Cheat Sheet: Route 53 Routing Policies

Quick reference for Route 53 routing policy selection.

---

## All Routing Policies Overview

| Policy | Use Case | Health Checks | Returns |
|--------|----------|---------------|---------|
| **Simple** | Single resource, no health checks | ❌ No | All values (client chooses randomly) |
| **Weighted** | A/B testing, gradual rollout | ✅ Yes | One record based on weight |
| **Latency** | Minimize latency for users | ✅ Yes | Lowest latency endpoint |
| **Failover** | Active-passive disaster recovery | ✅ **Required** | Primary (or secondary if unhealthy) |
| **Geolocation** | Route based on user's location | ✅ Yes | Record for user's location |
| **Geoproximity** | Route based on resources and bias | ✅ Yes | Nearest resource (with bias) |
| **Multi-Value Answer** | Simple load balancing with health checks | ✅ Yes | Up to 8 healthy records |

---

## Decision Tree

```
What's your requirement?

├─ Simple single-resource routing (no health checks)
│  └─ Simple
│
├─ Active-passive disaster recovery
│  └─ Failover (requires health checks)
│
├─ A/B testing or gradual rollout
│  └─ Weighted (70% old, 30% new)
│
├─ Minimize latency for global users
│  └─ Latency-based
│
├─ Route based on user's geographic location
│  ├─ Compliance/content restrictions
│  │  └─ Geolocation (by country/continent)
│  └─ Route to nearest resource
│     └─ Geoproximity (+ bias to shift traffic)
│
└─ Simple load balancing with health checks
   └─ Multi-Value Answer
```

---

## Detailed Policy Comparison

### **1. Simple Routing**

**Use Case:** Route to a single resource (no health checks, no advanced logic)

| Feature | Details |
|---------|---------|
| **Health Checks** | ❌ **Not supported** |
| **Multiple Values** | Can return multiple IPs (client picks randomly) |
| **Use Case** | Single web server, simple DNS resolution |

**Example:**
- `example.com` → `192.0.2.1` (single IP)
- `example.com` → `192.0.2.1`, `192.0.2.2` (client picks one randomly)

**Limitation:** No health checks - if resource is down, Route 53 still returns it

---

### **2. Weighted Routing**

**Use Case:** A/B testing, gradual rollout, load balancing with weights

| Feature | Details |
|---------|---------|
| **Health Checks** | ✅ Supported (skip unhealthy endpoints) |
| **How It Works** | Assign weight to each record (0-255), Route 53 returns based on weight |
| **Use Case** | 70% to old version, 30% to new version (blue-green deployment) |

**Example:**
- Record A: `us-east-1.example.com`, weight = 70
- Record B: `us-west-2.example.com`, weight = 30
- Result: 70% of traffic → us-east-1, 30% → us-west-2

**Weight Calculation:** Weight / Sum of all weights = Traffic percentage

---

### **3. Latency-Based Routing**

**Use Case:** Minimize latency by routing users to the closest AWS region

| Feature | Details |
|---------|---------|
| **Health Checks** | ✅ Supported |
| **How It Works** | Route 53 measures latency from user to each region, returns lowest latency |
| **Use Case** | Global application, minimize response time |

**Example:**
- User in Tokyo → Routes to `ap-northeast-1` (lowest latency)
- User in London → Routes to `eu-west-2` (lowest latency)

**Note:** Based on actual AWS latency measurements (not geographic distance)

---

### **4. Failover Routing**

**Use Case:** Active-passive disaster recovery (primary + secondary)

| Feature | Details |
|---------|---------|
| **Health Checks** | ✅ **Required** (must have health check on primary) |
| **How It Works** | If primary is healthy → return primary, if unhealthy → return secondary |
| **Use Case** | DR setup (primary in us-east-1, secondary in us-west-2) |

**Example:**
- Primary: `primary.example.com` (us-east-1) - healthy → use this
- Secondary: `secondary.example.com` (us-west-2) - used only if primary fails

**Critical:** Primary **must** have a health check configured

---

### **5. Geolocation Routing**

**Use Case:** Route based on user's geographic location (continent, country, state)

| Feature | Details |
|---------|---------|
| **Health Checks** | ✅ Supported |
| **How It Works** | Route based on where the query originates (user's location) |
| **Use Case** | Content localization, compliance (GDPR, data residency) |

**Example:**
- Users from **Europe** → `eu.example.com` (GDPR-compliant server)
- Users from **US** → `us.example.com` (US-based server)
- Users from **Asia** → `asia.example.com`
- **Default** record for all other locations

**Critical:** Always configure a **default** record (for locations not explicitly defined)

---

### **6. Geoproximity Routing**

**Use Case:** Route based on geographic location + bias (shift traffic towards/away from resources)

| Feature | Details |
|---------|---------|
| **Health Checks** | ✅ Supported |
| **How It Works** | Route to nearest resource, but can adjust with **bias** (+/- 99) |
| **Use Case** | Fine-tune traffic distribution, shift load away from overloaded region |

**Bias:**
- **Positive bias (+):** Expand geographic area (attract more traffic)
- **Negative bias (-):** Shrink geographic area (send less traffic)

**Example:**
- Resource in **us-east-1**, bias = +50 → Attracts more traffic from surrounding areas
- Resource in **us-west-1**, bias = -20 → Sends less traffic (redirect to other regions)

---

### **7. Multi-Value Answer Routing**

**Use Case:** Simple load balancing with health checks (return up to 8 healthy records)

| Feature | Details |
|---------|---------|
| **Health Checks** | ✅ Supported (returns only healthy records) |
| **How It Works** | Returns up to 8 healthy records, client picks one |
| **Use Case** | Simple load balancing across multiple web servers |

**Example:**
- `example.com` → `192.0.2.1`, `192.0.2.2`, `192.0.2.3` (all healthy)
- If one server is unhealthy, Route 53 excludes it from response

**Difference from Simple:**
- Simple: No health checks, returns all values
- Multi-Value: Returns only healthy values (up to 8)

---

## Exam Scenarios

| Scenario | Routing Policy |
|----------|----------------|
| Single website, one IP | **Simple** |
| Active-passive DR (primary + backup) | **Failover** |
| A/B test (90% old, 10% new) | **Weighted** |
| Route EU users to EU servers (compliance) | **Geolocation** |
| Minimize latency for global users | **Latency-based** |
| Gradually shift traffic to new region | **Geoproximity** (with bias) |
| Load balance with health checks | **Multi-Value Answer** |
| Blue-green deployment (50/50 split) | **Weighted** |
| Route based on user's country | **Geolocation** |

---

## Key Exam Tips

1. **Failover = Active-passive** (requires health checks)
2. **Weighted = A/B testing** (gradual rollout, percentage-based)
3. **Latency = Fastest response** (minimize latency for users)
4. **Geolocation = User's location** (continent, country, state)
5. **Geoproximity = Nearest resource + bias** (fine-tune with +/-)
6. **Multi-Value = Load balancing** (up to 8 healthy records)
7. **Simple = No health checks** (basic routing)
8. Only **Simple** and **Multi-Value** can return **multiple values** in one response
9. **Failover requires health check on primary** (exam favorite!)
10. Always create **default** record for **Geolocation** (for unmapped locations)

---

## Summary

| Policy | Best For | Key Feature |
|--------|----------|-------------|
| **Simple** | Single resource | No health checks |
| **Weighted** | A/B testing | Percentage-based traffic split |
| **Latency** | Global apps | Route to lowest latency region |
| **Failover** | DR | Active-passive with health checks |
| **Geolocation** | Compliance | Route by user's location |
| **Geoproximity** | Fine-tuning | Nearest resource + bias |
| **Multi-Value** | Load balancing | Up to 8 healthy records |
