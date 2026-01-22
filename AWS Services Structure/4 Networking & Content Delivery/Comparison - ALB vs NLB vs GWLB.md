# Comparison: ALB vs NLB vs GWLB

Quick reference for choosing the right load balancer type.

---

## High-Level Decision Tree

```
What layer are you load balancing?

├─ HTTP/HTTPS (Layer 7 - Application)
│  └─ ALB (Application Load Balancer)
│
├─ TCP/UDP (Layer 4 - Transport)
│  ├─ Need extreme performance, static IP, or TCP passthrough
│  │  └─ NLB (Network Load Balancer)
│  └─ Standard use case
│     └─ ALB can also handle TCP (via HTTP/HTTPS)
│
└─ Need to inspect/filter traffic through third-party appliances
   └─ GWLB (Gateway Load Balancer)
```

---

## Comparison Table

| Aspect | ALB | NLB | GWLB |
|--------|-----|-----|------|
| **Layer** | **Layer 7** (Application) | **Layer 4** (Transport) | **Layer 3** (Network) |
| **Protocols** | HTTP, HTTPS, gRPC, WebSocket | TCP, UDP, TLS | IP (all traffic) |
| **Use Case** | **Web applications** | **High performance**, non-HTTP | **Security appliances** (firewalls, IDS/IPS) |
| **Routing** | Path, host, header, query string | IP address, port only | Transparent (no routing) |
| **Static IP** | No (DNS name only) | **Yes** (Elastic IP per AZ) | No |
| **Target Types** | EC2, IP, Lambda, ALB | EC2, IP, ALB | Security appliances (EC2, virtual appliances) |
| **Performance** | Good | **Extreme** (millions of requests/sec) | Moderate |
| **SSL Termination** | **Yes** | **Yes** (optional) | No |
| **Sticky Sessions** | **Yes** (cookie-based) | **Yes** (IP-based) | N/A |
| **Health Checks** | HTTP/HTTPS | TCP, HTTP, HTTPS | TCP, HTTP, HTTPS |
| **Cost** | $$ | $$ | $$$ |
| **Latency** | ~10ms | **<1ms** (ultra-low latency) | Varies |

---

## When to Use Each Load Balancer

### **Use ALB (Application Load Balancer) If:**

✅ Load balancing **HTTP/HTTPS traffic** (web applications)
✅ Need **advanced routing** (path-based, host-based, header-based)
✅ Need to route to **Lambda functions** (serverless)
✅ Need **SSL termination** (offload SSL from backends)
✅ Need **authentication** (Cognito, OIDC, SAML)
✅ Building **microservices** (route /api → service A, /checkout → service B)
✅ Need **HTTP/2** or **WebSocket** support

**Common Use Cases:**
- Web applications and APIs
- Microservices routing
- Container-based applications (ECS, EKS)
- Serverless applications (Lambda targets)

---

### **Use NLB (Network Load Balancer) If:**

✅ Need **extreme performance** (millions of requests per second)
✅ Need **static IP addresses** (Elastic IP per AZ)
✅ Load balancing **TCP** or **UDP** traffic (not just HTTP)
✅ Need **ultra-low latency** (<1ms)
✅ Need to **preserve source IP** (client IP visible to backend)
✅ **TCP passthrough** (no SSL termination, TLS handled by backend)

**Common Use Cases:**
- Gaming servers (UDP traffic)
- IoT applications (MQTT, non-HTTP protocols)
- Financial trading platforms (low latency)
- VoIP applications
- Applications requiring static IPs for whitelisting

---

### **Use GWLB (Gateway Load Balancer) If:**

✅ Need to **inspect all traffic** through security appliances
✅ Deploying **third-party firewalls** (Palo Alto, Fortinet, Check Point)
✅ Need **intrusion detection/prevention systems** (IDS/IPS)
✅ Want **transparent traffic forwarding** to security appliances
✅ Need to **scale security appliances** horizontally

**Common Use Cases:**
- Deploying third-party firewall appliances
- Intrusion detection systems (IDS/IPS)
- Deep packet inspection (DPI)
- Network security inspection at scale

---

## Detailed Feature Comparison

### **ALB (Application Load Balancer)**

| Feature | Details |
|---------|---------|
| **Layer** | Layer 7 (Application) |
| **Protocols** | HTTP, HTTPS, HTTP/2, gRPC, WebSocket |
| **Routing** | Path (`/api/*`), Host (`app.example.com`), Headers, Query Strings, HTTP Methods |
| **Targets** | EC2, IP addresses, Lambda, ALB (yes, can chain ALBs) |
| **SSL Offloading** | Yes (integrate with ACM) |
| **Authentication** | Cognito, OIDC, SAML |
| **Sticky Sessions** | Cookie-based (AWSALB cookie) |
| **WAF Integration** | **Yes** (can attach WAF rules) |
| **Fixed Response** | Can return fixed HTTP responses (maintenance pages) |

**Advanced Routing Example:**
- `/api/*` → API microservice
- `/images/*` → Image processing service
- `app.example.com` → App server
- `admin.example.com` → Admin panel

---

### **NLB (Network Load Balancer)**

| Feature | Details |
|---------|---------|
| **Layer** | Layer 4 (Transport) |
| **Protocols** | TCP, UDP, TLS |
| **Routing** | IP address and port only (no content-based routing) |
| **Targets** | EC2, IP addresses, ALB |
| **Static IP** | **Yes** (1 Elastic IP per AZ) |
| **Performance** | **Millions of requests/sec**, <1ms latency |
| **SSL Offloading** | Yes (TLS termination, but typically used for passthrough) |
| **Preserve Source IP** | **Yes** (client IP visible to backend) |
| **Connection Draining** | Yes (deregistration delay) |

**Key Advantages:**
- **Extreme performance** and low latency
- **Static IP per AZ** (whitelisting, firewalls)
- **Source IP preservation** (know client IP)

---

### **GWLB (Gateway Load Balancer)**

| Feature | Details |
|---------|---------|
| **Layer** | Layer 3 (Network) + Layer 4 |
| **Protocols** | All IP traffic (transparent) |
| **Routing** | None (transparent forwarding to appliances) |
| **Targets** | EC2 instances running security appliances |
| **Use Case** | **Traffic inspection** by third-party appliances |
| **GENEVE Protocol** | Uses GENEVE encapsulation (port 6081) |
| **Health Checks** | TCP, HTTP, HTTPS |

**How GWLB Works:**
1. Traffic is sent to GWLB
2. GWLB distributes to security appliances (firewall, IDS/IPS)
3. Appliance inspects/processes traffic
4. Traffic is returned to GWLB
5. GWLB forwards to destination

---

## Exam Scenarios

| Scenario | Load Balancer |
|----------|---------------|
| Load balance HTTP/HTTPS web app | **ALB** |
| Route `/api` to one service, `/checkout` to another | **ALB** (path-based routing) |
| Need static IP for firewall whitelisting | **NLB** |
| Gaming server (UDP traffic) | **NLB** |
| Millions of requests per second, low latency | **NLB** |
| Invoke Lambda function from load balancer | **ALB** (Lambda target) |
| Integrate with WAF for security | **ALB** (WAF integration) |
| Deploy third-party firewall appliance | **GWLB** |
| Inspect all VPC traffic through IDS/IPS | **GWLB** |
| Need to preserve client IP address | **NLB** |
| SSL termination for web app | **ALB** |

---

## Key Exam Tips

1. **ALB = Layer 7 (HTTP/HTTPS)** with advanced routing
2. **NLB = Layer 4 (TCP/UDP)** with extreme performance and static IP
3. **GWLB = Traffic inspection** through security appliances
4. **Static IP needed?** → NLB (Elastic IP per AZ)
5. **Path-based routing?** → ALB
6. **Lambda targets?** → ALB only
7. **WAF integration?** → ALB
8. **Ultra-low latency?** → NLB
9. **Third-party security appliances?** → GWLB
10. **Preserve source IP?** → NLB

---

## Summary

| Load Balancer | Layer | Best For | Key Feature |
|---------------|-------|----------|-------------|
| **ALB** | Layer 7 | Web apps, microservices | Advanced routing, Lambda targets |
| **NLB** | Layer 4 | High performance, non-HTTP | Static IP, ultra-low latency |
| **GWLB** | Layer 3/4 | Security appliances | Transparent traffic inspection |
