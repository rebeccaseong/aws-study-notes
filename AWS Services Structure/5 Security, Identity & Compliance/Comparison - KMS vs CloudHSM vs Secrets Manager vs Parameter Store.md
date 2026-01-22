# Comparison: KMS vs CloudHSM vs Secrets Manager vs Parameter Store

Quick reference for choosing the right secrets and encryption management service.

---

## High-Level Decision Tree

```
What do you need to manage?

├─ Encryption keys
│  ├─ Standard encryption needs (most use cases)
│  │  └─ KMS (Key Management Service)
│  └─ Regulatory compliance (FIPS 140-2 Level 3, dedicated hardware)
│     └─ CloudHSM
│
└─ Secrets/configuration
   ├─ Passwords, API keys (need automatic rotation)
   │  └─ Secrets Manager
   └─ Configuration parameters (simple secrets, cost-effective)
      └─ Parameter Store
```

---

## Comparison Table

| Aspect | KMS | CloudHSM | Secrets Manager | Parameter Store |
|--------|-----|----------|-----------------|-----------------|
| **Purpose** | Encryption key management | Dedicated hardware encryption | Secret rotation (passwords, API keys) | Config + simple secrets |
| **Key Storage** | AWS-managed, multi-tenant | **Dedicated HSM** (single-tenant) | AWS-managed | AWS-managed |
| **FIPS 140-2** | Level 2 (software) | **Level 3** (hardware) | N/A | N/A |
| **Key Control** | AWS has access to keys | **You control keys** | AWS-managed | AWS-managed |
| **Rotation** | Manual or automatic (AWS managed keys) | **You manage rotation** | **Automatic** (for RDS, Redshift, DocumentDB) | **Manual** |
| **Multi-Region** | Yes (multi-region keys) | No | Yes (replica secrets) | No |
| **Pricing** | $ (pay per key + API calls) | $$$$ (~$1.60/hour per HSM) | $$ ($0.40/secret/month) | **Free** (standard tier) |
| **Use Case** | **General encryption** | **Strict compliance**, custom crypto | **Database passwords**, API keys | **App config**, simple secrets |
| **Complexity** | Low | **High** (HSM expertise) | Low | **Lowest** |

---

## When to Use Each Service

### **Use KMS If:**

✅ Need to **encrypt data at rest** (S3, EBS, RDS, etc.)
✅ **Most encryption use cases** (general-purpose)
✅ Want **AWS-managed encryption** (easy, integrated)
✅ Need **multi-region keys** (encrypt in us-east-1, decrypt in eu-west-1)
✅ Don't need dedicated hardware (FIPS 140-2 Level 2 is sufficient)

**Common Use Cases:**
- Encrypt S3 buckets, EBS volumes, RDS databases
- Envelope encryption for application data
- Signing and verification
- Integration with AWS services (seamless encryption)

---

### **Use CloudHSM If:**

✅ Need **regulatory compliance** (FIPS 140-2 **Level 3**)
✅ Need **dedicated hardware** (single-tenant HSM)
✅ Want **full control over encryption keys** (AWS has no access)
✅ Need to use **custom encryption algorithms**
✅ Need **symmetric and asymmetric** encryption

**Common Use Cases:**
- Financial services (strict compliance)
- Government/healthcare (HIPAA, PCI-DSS)
- Custom cryptographic operations
- SSL/TLS offloading (web servers)

❌ **Don't use if:** Standard KMS is sufficient (CloudHSM is expensive and complex)

---

### **Use Secrets Manager If:**

✅ Need to **store database credentials** (RDS, Redshift, DocumentDB)
✅ Need **automatic secret rotation** (change passwords regularly)
✅ Storing **API keys, OAuth tokens**, sensitive credentials
✅ Want **audit logging** (CloudTrail integration)
✅ Need **multi-region replication** (global applications)

**Common Use Cases:**
- RDS database passwords (automatic rotation)
- Third-party API keys
- OAuth tokens
- Application secrets with automatic rotation

---

### **Use Parameter Store If:**

✅ Need to store **application configuration** (non-sensitive or simple secrets)
✅ Want **cost-effective solution** (free tier: 10,000 parameters)
✅ Storing **feature flags, database connection strings**, license codes
✅ Don't need automatic rotation (manual rotation is OK)

**Common Use Cases:**
- Application configuration (database URLs, feature flags)
- Simple secrets (passwords you rotate manually)
- AMI IDs, license keys
- Environment-specific settings (dev/staging/prod)

---

## Detailed Comparison

### **KMS (Key Management Service)**

| Feature | Details |
|---------|---------|
| **Key Types** | Symmetric (AES-256), Asymmetric (RSA, ECC) |
| **Key Storage** | AWS-managed, multi-tenant |
| **FIPS 140-2** | **Level 2** (software-based) |
| **Pricing** | $1/key/month + $0.03 per 10,000 API calls |
| **Rotation** | Automatic (AWS managed keys: yearly), Manual (customer managed keys) |
| **Integration** | **Native** with all AWS services (S3, EBS, RDS, etc.) |

**Key Management:**
- **AWS Managed Keys:** Free, automatic rotation (yearly)
- **Customer Managed Keys:** $1/month, manual rotation (recommended: enable auto-rotation)
- **AWS Owned Keys:** Free, used by AWS services (you don't manage)

---

### **CloudHSM**

| Feature | Details |
|---------|---------|
| **Key Types** | Symmetric, Asymmetric, custom algorithms |
| **Key Storage** | **Dedicated HSM** (single-tenant hardware) |
| **FIPS 140-2** | **Level 3** (hardware-based, tamper-resistant) |
| **Pricing** | ~$1.60/hour per HSM (~$1,200/month) |
| **Rotation** | **You manage** (manual rotation) |
| **Integration** | **Manual** (no native AWS service integration) |

**Cluster Deployment:**
- Minimum 2 HSMs (for high availability across AZs)
- You manage HSMs using CloudHSM client software

**Key Control:**
- **You own and manage keys** (AWS has no access)
- Keys never leave the HSM in plaintext

---

### **Secrets Manager**

| Feature | Details |
|---------|---------|
| **Secret Types** | Database credentials, API keys, OAuth tokens |
| **Rotation** | **Automatic** (for RDS, Redshift, DocumentDB) |
| **Pricing** | $0.40 per secret/month + $0.05 per 10,000 API calls |
| **Encryption** | Encrypted with KMS |
| **Replication** | Multi-region (replicate secrets across regions) |

**Automatic Rotation:**
- **RDS, Redshift, DocumentDB:** Lambda function rotates passwords automatically
- **Custom secrets:** Write custom Lambda function for rotation

---

### **Parameter Store**

| Feature | Details |
|---------|---------|
| **Parameter Types** | String, StringList, SecureString (encrypted with KMS) |
| **Rotation** | **Manual** (you handle rotation) |
| **Pricing** | **Free** (standard tier: up to 10,000 parameters) |
| **Encryption** | SecureString uses KMS |
| **Versioning** | Track changes to parameter values |

**Tiers:**
- **Standard:** Free, 4 KB max value, 10,000 parameters
- **Advanced:** $0.05 per parameter/month, 8 KB max value, 100,000 parameters, parameter policies

---

## Exam Scenarios

| Scenario | Service |
|----------|---------|
| Encrypt EBS volume | **KMS** |
| Store RDS password with automatic rotation | **Secrets Manager** |
| Store application config (database URL) | **Parameter Store** |
| FIPS 140-2 Level 3 compliance (banking) | **CloudHSM** |
| Encrypt S3 bucket at rest | **KMS** |
| Store API keys (no rotation needed) | **Parameter Store** (SecureString) |
| Automatic password rotation for RDS | **Secrets Manager** |
| Custom encryption algorithm | **CloudHSM** |
| Cost-effective secret storage | **Parameter Store** (free) |
| Dedicated hardware for encryption | **CloudHSM** |

---

## Key Exam Tips

1. **KMS = General encryption** (S3, EBS, RDS, most use cases)
2. **CloudHSM = Compliance** (FIPS 140-2 Level 3, dedicated hardware)
3. **Secrets Manager = Automatic rotation** (RDS passwords, API keys)
4. **Parameter Store = Config + simple secrets** (free, manual rotation)
5. **CloudHSM is expensive** (~$1,200/month per HSM)
6. **Parameter Store is free** (standard tier)
7. **Secrets Manager costs $0.40/secret/month** (not free like Parameter Store)
8. **KMS is multi-tenant**, **CloudHSM is single-tenant** (dedicated)
9. **Automatic rotation?** → Secrets Manager
10. **You must control keys?** → CloudHSM (AWS cannot access keys)

---

## Summary

| Service | Best For | Key Feature |
|---------|----------|-------------|
| **KMS** | General encryption | AWS-managed, integrated with all services |
| **CloudHSM** | Compliance, custom crypto | Dedicated hardware, FIPS 140-2 Level 3 |
| **Secrets Manager** | Database passwords, API keys | Automatic rotation |
| **Parameter Store** | App config, simple secrets | Free, manual rotation |
