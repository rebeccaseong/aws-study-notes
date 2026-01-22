# Comparison: VPC Peering vs Transit Gateway vs PrivateLink

Quick reference for choosing the right VPC connectivity solution.

---

## High-Level Decision Tree

```
What connectivity do you need?

├─ Connect 2-3 VPCs (simple, direct)
│  └─ VPC Peering
│
├─ Connect many VPCs (10+) or on-premises
│  └─ Transit Gateway (hub-and-spoke)
│
└─ Expose a specific service privately (not entire VPC)
   └─ PrivateLink (service-level access)
```

---

## Comparison Table

| Aspect | VPC Peering | Transit Gateway | PrivateLink |
|--------|-------------|-----------------|-------------|
| **Connectivity Model** | **One-to-one** (pair of VPCs) | **Hub-and-spoke** (centralized) | **Service-level** (expose specific service) |
| **Transitive** | **No** | **Yes** | N/A |
| **Scalability** | Poor (N*(N-1)/2 connections) | Excellent (attach thousands of VPCs) | Excellent (consumer-provider model) |
| **IP Overlap** | **Not allowed** | **Not allowed** | **Allowed** (uses ENI) |
| **On-Prem Connectivity** | No | **Yes** (via VPN/Direct Connect) | No |
| **Use Case** | **Simple VPC-to-VPC** | **Many VPCs** or **hybrid cloud** | **Expose services privately** |
| **Complexity** | Low | Medium | Low-Medium |
| **Cost** | Data transfer only | **Hourly** + data transfer | **Hourly** + data transfer |
| **Cross-Region** | **Yes** | **Yes** (TGW peering) | **Yes** |
| **Cross-Account** | **Yes** | **Yes** (via RAM) | **Yes** |

---

## When to Use Each Solution

### **Use VPC Peering If:**

✅ Connecting **2-3 VPCs** (simple, direct connection)
✅ **Low complexity** (don't want central management)
✅ **No IP overlap** between VPCs
✅ Both VPCs can be in **different regions** or **different accounts**
✅ **Cost-conscious** (no hourly charges, only data transfer)

**Common Use Cases:**
- Connect Dev and Prod VPCs
- Connect VPC in Account A to VPC in Account B
- Share resources between a few VPCs

❌ **Don't use if:** Need to connect many VPCs (creates full mesh), or need transitive routing

---

### **Use Transit Gateway If:**

✅ Connecting **many VPCs** (10+)
✅ Need **transitive routing** (VPC A → VPC B → VPC C)
✅ **Hybrid cloud** (connect VPCs to on-premises networks)
✅ Need **centralized network management**
✅ Want to avoid **full mesh** of VPC Peering connections
✅ Multi-region connectivity (**Transit Gateway peering**)

**Common Use Cases:**
- Enterprise with dozens of VPCs
- Hub-and-spoke network architecture
- Connect VPCs + on-premises via single VPN/Direct Connect
- Multi-account organizations
- Scaling network architecture (avoid VPC Peering mesh)

❌ **Don't use if:** Only need to connect 2-3 VPCs (VPC Peering is simpler and cheaper)

---

### **Use PrivateLink If:**

✅ **Expose a specific service** (not entire VPC) privately
✅ **Consumer-provider model** (provider exposes service, consumers access it)
✅ Want to **avoid routing entire VPC traffic**
✅ **IP overlap is acceptable** (uses ENI, not routing)
✅ Third-party SaaS integration (provider creates endpoint service)

**Common Use Cases:**
- Expose internal API or microservice to other VPCs
- Access third-party SaaS privately (without internet)
- Share database or application with partners (without exposing entire VPC)
- Multi-tenant architecture (provider serves multiple consumers)

❌ **Don't use if:** Need full VPC connectivity (use VPC Peering or Transit Gateway)

---

## Detailed Comparison

### **VPC Peering**

| Feature | Details |
|---------|---------|
| **Connection Type** | One-to-one (pair of VPCs) |
| **Transitive** | **No** (A-B + B-C ≠ A-C connectivity) |
| **IP Overlap** | **Not allowed** (CIDRs must not overlap) |
| **Routing** | Update route tables on **both sides** |
| **Pricing** | Data transfer charges only (no hourly fee) |
| **Max Connections** | 125 peering connections per VPC |

**Limitations:**
- **Not transitive** (requires full mesh for multiple VPCs)
- **No edge-to-edge routing** (cannot use one VPC's IGW from another)
- Full mesh = N*(N-1)/2 connections (e.g., 10 VPCs = 45 connections!)

**Example:**
- 3 VPCs = 3 peering connections (A-B, B-C, A-C)
- 10 VPCs = 45 peering connections (unmanageable!)

---

### **Transit Gateway**

| Feature | Details |
|---------|---------|
| **Connection Type** | Hub-and-spoke (centralized hub) |
| **Transitive** | **Yes** (all attached networks can communicate) |
| **IP Overlap** | **Not allowed** |
| **Routing** | Transit Gateway route tables (centralized) |
| **Pricing** | **$0.05/hour per attachment** + data transfer |
| **Max Attachments** | 5,000 VPCs per TGW |

**Key Features:**
- **Centralized routing** (one route table to rule them all)
- **Supports VPN and Direct Connect** (hybrid cloud)
- **Inter-region peering** (TGW in us-east-1 ↔ TGW in eu-west-1)
- **Multicast support** (broadcast to multiple targets)

**Cost Example:**
- 10 VPCs = 10 attachments = $0.50/hour = ~$365/month (just attachments, excluding data transfer)

---

### **PrivateLink (VPC Endpoint Service)**

| Feature | Details |
|---------|---------|
| **Connection Type** | Service-level (provider → consumer) |
| **IP Overlap** | **Allowed** (uses ENI, no routing conflicts) |
| **Routing** | Not required (uses DNS and ENI) |
| **Pricing** | **$0.01/hour per endpoint** + data transfer |
| **Access Control** | Principal-based (allow specific AWS accounts) |

**How It Works:**
1. **Provider** creates VPC Endpoint Service (attached to NLB)
2. **Consumer** creates Interface VPC Endpoint (ENI in their VPC)
3. Consumer accesses service via **private IP** (DNS name)
4. Traffic stays on AWS private network

**Key Advantages:**
- **No routing conflicts** (works even with overlapping IPs)
- **Fine-grained access control** (expose only specific service, not entire VPC)
- **Scalable** (provider serves many consumers without complex routing)

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Connect 2 VPCs (Dev and Prod) | **VPC Peering** |
| Connect 20 VPCs in hub-and-spoke architecture | **Transit Gateway** |
| Expose internal API to 50 customer VPCs | **PrivateLink** |
| Connect VPCs + on-premises network | **Transit Gateway** |
| VPCs have overlapping CIDR blocks | **PrivateLink** (only option) |
| Need transitive routing (A → B → C) | **Transit Gateway** |
| Share a specific service (not entire VPC) | **PrivateLink** |
| Cost-optimize 3 VPC connectivity | **VPC Peering** (cheapest) |
| Connect VPCs across multiple regions | **VPC Peering** (regional) or **Transit Gateway** (inter-region peering) |
| Third-party SaaS private access | **PrivateLink** |

---

## Key Exam Tips

1. **VPC Peering = One-to-one, simple, no transitive routing**
2. **Transit Gateway = Hub-and-spoke, transitive, many VPCs**
3. **PrivateLink = Service-level access, IP overlap allowed**
4. **Transitive routing needed?** → Transit Gateway (not VPC Peering)
5. **IP overlap between VPCs?** → PrivateLink (only option)
6. **2-3 VPCs?** → VPC Peering (simpler, cheaper)
7. **10+ VPCs?** → Transit Gateway (avoid full mesh)
8. **Hybrid cloud (VPC + on-prem)?** → Transit Gateway
9. **Expose specific service?** → PrivateLink
10. **Cost matters?** → VPC Peering (cheapest for few VPCs)

---

## Summary

| Solution | Best For | Key Feature |
|----------|----------|-------------|
| **VPC Peering** | 2-3 VPCs, simple connectivity | Direct connection, no hourly fee |
| **Transit Gateway** | Many VPCs, hybrid cloud | Hub-and-spoke, transitive routing |
| **PrivateLink** | Service-level access | Expose specific services, IP overlap OK |
