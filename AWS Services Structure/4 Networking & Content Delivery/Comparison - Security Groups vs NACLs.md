# Comparison: Security Groups vs NACLs

Quick reference for choosing between Security Groups and Network ACLs for VPC security.

---

## High-Level Decision Tree

```
What layer of security do you need?

├─ Instance-level firewall (stateful, allow rules only)
│  └─ Security Groups
│
└─ Subnet-level firewall (stateless, allow + deny rules)
   └─ Network ACLs (NACLs)

Best Practice: Use BOTH (defense in depth)
- Security Groups: Primary security control (instance-level)
- NACLs: Additional subnet-level protection (optional but recommended)
```

---

## Comparison Table

| Aspect | Security Groups | Network ACLs (NACLs) |
|--------|-----------------|----------------------|
| **Scope** | **Instance-level** (attached to ENI) | **Subnet-level** (applies to all instances in subnet) |
| **State** | **Stateful** (return traffic auto-allowed) | **Stateless** (must explicitly allow return traffic) |
| **Rules** | **Allow only** (no deny rules) | **Allow and Deny** rules |
| **Rule Evaluation** | All rules evaluated (most permissive wins) | **Rules evaluated in order** (lowest number first) |
| **Default Behavior** | **Deny all inbound**, allow all outbound | **Allow all** inbound and outbound (default NACL) |
| **Rule Target** | Specify instances, security groups, or CIDR | **CIDR blocks only** |
| **Use Case** | **Primary security layer** (instance firewall) | **Subnet-level defense** (block IPs, additional protection) |
| **Applies To** | Specific instances (via ENI) | **All instances in subnet** automatically |
| **Max Rules** | 60 inbound + 60 outbound (per SG) | 20 inbound + 20 outbound (default, can increase) |

---

## When to Use Each

### **Use Security Groups For:**

✅ **Primary instance-level security** (default choice)
✅ **Allow rules** (permit specific traffic)
✅ **Stateful traffic** (automatic return traffic handling)
✅ **Referencing other security groups** (allow traffic from SG-A to SG-B)
✅ **Application-specific security** (web tier, app tier, DB tier)

**Common Use Cases:**
- Web server: Allow HTTP (80) and HTTPS (443) from 0.0.0.0/0
- Database: Allow MySQL (3306) only from app-tier security group
- SSH access: Allow port 22 from bastion host security group
- Microservices: Reference security groups for service-to-service communication

---

### **Use Network ACLs For:**

✅ **Subnet-level protection** (additional defense layer)
✅ **Deny rules** (block specific IPs or ranges)
✅ **Stateless filtering** (explicit inbound + outbound rules)
✅ **Blocking malicious IPs** (DDoS, attackers)
✅ **Compliance requirements** (explicit deny rules)

**Common Use Cases:**
- Block known malicious IP ranges
- Deny traffic from specific countries (using CIDR blocks)
- Additional subnet-level security for compliance
- Explicit deny rules for security policy enforcement

---

## Detailed Comparison

### **Security Groups**

| Feature | Details |
|---------|---------|
| **Attached To** | EC2 instances (via Elastic Network Interface - ENI) |
| **Stateful** | **Yes** - Return traffic automatically allowed (no need for outbound rule) |
| **Rules** | **Allow only** (implicit deny for everything not allowed) |
| **Rule Evaluation** | All rules evaluated, most permissive wins |
| **Default SG** | Deny all inbound, allow all outbound, allow all traffic from same SG |
| **References** | Can reference other security groups (e.g., "allow from SG-12345") |

**Example Security Group Rules:**

**Inbound:**
| Type | Protocol | Port | Source | Description |
|------|----------|------|--------|-------------|
| HTTP | TCP | 80 | 0.0.0.0/0 | Allow web traffic from anywhere |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Allow secure web traffic |
| SSH | TCP | 22 | sg-bastion | Allow SSH from bastion host SG |

**Outbound:**
| Type | Protocol | Port | Destination | Description |
|------|----------|------|-------------|-------------|
| All | All | All | 0.0.0.0/0 | Allow all outbound (default) |

**Key Behavior:**
- If you allow inbound HTTP (80), outbound response traffic is **automatically allowed** (stateful)
- No need to create outbound rule for return traffic

---

### **Network ACLs (NACLs)**

| Feature | Details |
|---------|---------|
| **Attached To** | Subnets (applies to all instances in subnet) |
| **Stateless** | **No** - Must explicitly allow return traffic (inbound + outbound) |
| **Rules** | **Allow and Deny** rules |
| **Rule Evaluation** | Rules evaluated in order (lowest rule number first, stops at first match) |
| **Default NACL** | Allow all inbound and outbound traffic |
| **Custom NACL** | Deny all inbound and outbound (until you add rules) |

**Example NACL Rules:**

**Inbound:**
| Rule # | Type | Protocol | Port | Source | Allow/Deny |
|--------|------|----------|------|--------|------------|
| 100 | HTTP | TCP | 80 | 0.0.0.0/0 | ALLOW |
| 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | ALLOW |
| 120 | SSH | TCP | 22 | 10.0.0.0/16 | ALLOW |
| 130 | Ephemeral | TCP | 1024-65535 | 0.0.0.0/0 | ALLOW |
| 200 | All | All | All | 203.0.113.0/24 | **DENY** |
| * | All | All | All | 0.0.0.0/0 | DENY |

**Outbound:**
| Rule # | Type | Protocol | Port | Destination | Allow/Deny |
|--------|------|----------|------|-------------|------------|
| 100 | HTTP | TCP | 80 | 0.0.0.0/0 | ALLOW |
| 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | ALLOW |
| 120 | Ephemeral | TCP | 1024-65535 | 0.0.0.0/0 | ALLOW |
| * | All | All | All | 0.0.0.0/0 | DENY |

**Key Behavior:**
- Rules evaluated in order (100 → 110 → 120 → ...)
- First matching rule wins (processing stops)
- Must allow **ephemeral ports** (1024-65535) for return traffic
- Rule * is default deny (lowest priority)

**Why Ephemeral Ports?**
- Clients use random high ports (1024-65535) for return connections
- Must explicitly allow these in stateless NACLs
- Security Groups don't need this (stateful)

---

## Stateful vs Stateless

### **Stateful (Security Groups)**

```
Client (Internet) → EC2 Instance

Inbound Rule: Allow HTTP (80) from 0.0.0.0/0
Outbound Rule: (Not needed - automatic)

✅ Request: Client → EC2:80 (allowed by inbound rule)
✅ Response: EC2:80 → Client (automatic, no outbound rule needed)
```

**Key Point:** Return traffic automatically allowed (stateful connection tracking)

---

### **Stateless (NACLs)**

```
Client (Internet) → EC2 Instance

Inbound Rule: Allow HTTP (80) from 0.0.0.0/0
Outbound Rule: Allow Ephemeral (1024-65535) to 0.0.0.0/0

✅ Request: Client:random-port → EC2:80 (allowed by inbound rule 80)
✅ Response: EC2:80 → Client:random-port (allowed by outbound ephemeral rule)
```

**Key Point:** Must explicitly allow both directions (no connection tracking)

---

## Rule Evaluation Order

### **Security Groups**

```
Rule 1: Allow HTTP (80) from 0.0.0.0/0
Rule 2: Allow HTTPS (443) from 0.0.0.0/0
Rule 3: Allow SSH (22) from 10.0.0.0/16

Evaluation: ALL rules evaluated (no ordering)
Result: Traffic allowed if ANY rule matches (most permissive wins)
```

**No explicit deny** - Everything not explicitly allowed is denied (implicit deny)

---

### **Network ACLs**

```
Rule 100: ALLOW HTTP (80) from 0.0.0.0/0
Rule 200: DENY ALL from 203.0.113.0/24
Rule *: DENY ALL

Example 1: HTTP from 1.2.3.4
- Check rule 100: Match! ALLOW (stop evaluation)

Example 2: HTTP from 203.0.113.50
- Check rule 100: Match! ALLOW (stop evaluation)
- Rule 200 never checked (already matched rule 100)

Example 3: HTTPS from 203.0.113.50
- Check rule 100: No match (different port)
- Check rule 200: Match! DENY (stop evaluation)
```

**Order matters!** Lower numbered rules evaluated first (lowest rule # = highest priority)

**Tip:** Put specific deny rules BEFORE general allow rules

---

## Common Patterns

### **Pattern 1: Web Tier Security**

**Security Group (Web-SG):**
- Inbound: Allow 80, 443 from 0.0.0.0/0
- Outbound: Allow all (default)

**NACL (Public Subnet):**
- Inbound: Allow 80, 443, ephemeral ports
- Outbound: Allow 80, 443, ephemeral ports

---

### **Pattern 2: Database Tier Security**

**Security Group (DB-SG):**
- Inbound: Allow 3306 (MySQL) from App-SG only
- Outbound: Allow all (default)

**NACL (Private Subnet):**
- Inbound: Allow 3306 from 10.0.0.0/16 (VPC CIDR)
- Outbound: Allow ephemeral ports

---

### **Pattern 3: Block Malicious IP**

**Security Group:**
- Cannot block specific IPs (no deny rules)

**NACL:**
- Rule 50: DENY all from 203.0.113.0/24 (malicious IP range)
- Rule 100: ALLOW HTTP from 0.0.0.0/0

**Why rule 50 before 100?** Lower number = higher priority (deny wins)

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Allow web traffic to EC2 instance | **Security Group:** Allow 80, 443 inbound |
| Block specific malicious IP address | **NACL:** Add DENY rule for that IP (low rule number) |
| Allow database access only from app tier | **Security Group:** Allow 3306 from app-tier SG |
| Subnet-level protection for compliance | **NACL:** Custom NACL with explicit allow/deny rules |
| Automatically allow return traffic | **Security Group** (stateful) |
| Must explicitly configure return traffic | **NACL** (stateless - need ephemeral ports) |
| Apply security to specific instances | **Security Group** (instance-level) |
| Apply security to all instances in subnet | **NACL** (subnet-level) |
| Reference another security group in rule | **Security Group** (can reference SGs) |
| Need explicit deny rules | **NACL** (SGs have implicit deny only) |

---

## Key Exam Tips

1. **Security Groups = Stateful** (return traffic automatic)
2. **NACLs = Stateless** (must allow ephemeral ports for return traffic)
3. **Security Groups = Allow only** (no explicit deny)
4. **NACLs = Allow and Deny** rules
5. **Security Groups = Instance-level** (attached to ENI)
6. **NACLs = Subnet-level** (all instances in subnet)
7. **Security Groups evaluate ALL rules** (most permissive wins)
8. **NACLs evaluate rules in order** (lowest number first, first match wins)
9. **Block specific IP?** → Use NACL deny rule
10. **Reference another security group?** → Only Security Groups support this
11. **Default Security Group:** Deny all inbound, allow all outbound
12. **Default NACL:** Allow all inbound and outbound
13. **Custom NACL:** Deny all (until you add rules)
14. **Ephemeral ports (1024-65535)** needed for NACL outbound rules
15. **Best practice:** Use BOTH (Security Groups + NACLs for defense in depth)

---

## Defense in Depth Strategy

```
Internet
   ↓
[NACL - Subnet-level protection]
   ↓ (Allow HTTP/HTTPS, Deny malicious IPs)
[Security Group - Instance-level protection]
   ↓ (Allow HTTP/HTTPS from specific sources)
EC2 Instance
```

**Layer 1 (NACL):**
- Broad subnet-level filtering
- Block known malicious IPs
- Compliance requirements

**Layer 2 (Security Group):**
- Precise instance-level control
- Application-specific rules
- Reference other security groups

---

## Common Mistakes

### **Mistake 1: Forgetting Ephemeral Ports in NACLs**

❌ **Wrong:**
```
Inbound: Allow HTTP (80)
Outbound: Allow HTTP (80)
```
Result: Response traffic blocked (client uses ephemeral port, not 80)

✅ **Correct:**
```
Inbound: Allow HTTP (80)
Outbound: Allow Ephemeral (1024-65535)
```

---

### **Mistake 2: NACL Rule Ordering**

❌ **Wrong:**
```
Rule 100: ALLOW ALL from 0.0.0.0/0
Rule 200: DENY ALL from 203.0.113.0/24
```
Result: Malicious IP allowed (rule 100 matches first)

✅ **Correct:**
```
Rule 50: DENY ALL from 203.0.113.0/24
Rule 100: ALLOW ALL from 0.0.0.0/0
```

---

### **Mistake 3: Using Security Groups to Block IPs**

❌ **Wrong:** Try to use Security Group to block specific IP
Result: Not possible (Security Groups have no deny rules)

✅ **Correct:** Use NACL deny rule to block specific IPs

---

## Summary Table

| Feature | Security Groups | NACLs |
|---------|-----------------|-------|
| **Level** | Instance | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow + Deny |
| **Evaluation** | All rules | Ordered (first match wins) |
| **Default** | Deny inbound, allow outbound | Allow all (default NACL) |
| **Best For** | Instance firewall | Subnet protection, IP blocking |

**Bottom Line:** Use **Security Groups** as primary security (instance-level), add **NACLs** for additional subnet-level protection (defense in depth).
