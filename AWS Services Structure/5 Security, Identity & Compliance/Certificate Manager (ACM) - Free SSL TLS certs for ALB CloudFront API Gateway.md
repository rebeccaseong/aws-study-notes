- **What is it? (1-Sentence Pitch):** AWS Certificate Manager (ACM) provisions, manages, and deploys public and private SSL/TLS certificates for use with AWS services, providing free public certificates and automatic renewal.

- **Core Use Cases:**
    - Enabling HTTPS for websites hosted on AWS (ALB, CloudFront, API Gateway)
    - Securing API endpoints with SSL/TLS
    - Encrypting traffic between clients and AWS services
    - Issuing private certificates for internal applications (ACM Private CA)

- **Key Features & Concepts:**
    - **Free Public Certificates:** ACM provides free SSL/TLS certificates for AWS services
    - **Automatic Renewal:** ACM automatically renews certificates before expiration
    - **Domain Validation:** Validate domain ownership via email or DNS
    - **Wildcard Certificates:** Support for wildcard domains (e.g., `*.example.com`)
    - **Managed Service:** AWS handles certificate provisioning, deployment, and renewal
    - **Regional Service:** Certificates are region-specific (except for CloudFront/API Gateway edge)
    - **No Export:** Public ACM certificates **cannot be exported** (stay within AWS)

- **ACM Public vs ACM Private CA:**
    | **Aspect** | **ACM Public** | **ACM Private CA** |
    |---|---|---|
    | **Cost** | **Free** | **Paid** ($400/month per CA) |
    | **Use Case** | Public-facing websites/APIs | Internal applications, VPN |
    | **Certificate Authority** | Trusted public CA | Your own private CA |
    | **Domain** | Public domain names | Any domain (internal included) |
    | **Export** | **Cannot export** | **Can export** |

- **Supported AWS Services:**
    | **Service** | **Supported** | **Region** |
    |---|---|---|
    | **Elastic Load Balancer (ALB/NLB)** | ✅ Yes | Certificate must be in **same region** as ELB |
    | **CloudFront** | ✅ Yes | Certificate must be in **us-east-1** |
    | **API Gateway** | ✅ Yes | Certificate must be in **same region** (or us-east-1 for edge) |
    | **Elastic Beanstalk** | ✅ Yes | Via ALB integration |
    | **EC2** | ❌ No | Cannot install ACM certs on EC2 directly |

- **How ACM Works:**
    1. **Request:** Request a certificate in ACM (specify domain name)
    2. **Validate:** Validate domain ownership via email or DNS (add CNAME record)
    3. **Deploy:** Attach certificate to ALB, CloudFront, or API Gateway
    4. **Renew:** ACM automatically renews certificate before expiration (no manual action)

- **ACM vs Let's Encrypt:**
    | **Aspect** | **ACM** | **Let's Encrypt** |
    |---|---|---|
    | **Cost** | Free | Free |
    | **Renewal** | **Automatic** (no action needed) | Manual (or automate with scripts) |
    | **AWS Integration** | Native (ALB, CloudFront, etc.) | Manual installation required |
    | **Export** | **Cannot export** | **Can export** (use anywhere) |
    | **Use Case** | AWS-hosted applications | Self-hosted or non-AWS environments |

- **Integration:**
    - **ALB/NLB:** Attach ACM certificate for HTTPS listeners
    - **CloudFront:** Use ACM certificate for custom domain HTTPS (must be in us-east-1)
    - **API Gateway:** Attach ACM certificate for custom domain APIs
    - **Route 53:** Validate domain ownership via DNS (add CNAME record)
    - **CloudTrail:** Audit certificate requests and renewals

- **Exam Tips / What to Remember:**
    - **ACM provides free SSL/TLS certificates** for AWS services
    - **Automatic renewal** (no need to manually renew before expiration)
    - Exam scenario: **"Enable HTTPS for Application Load Balancer"** = ACM
    - **Cannot export public ACM certificates** (they stay within AWS)
    - For **CloudFront**, certificate must be in **us-east-1** region
    - For **ALB/NLB/API Gateway**, certificate must be in the **same region**
    - **Cannot install ACM certificates directly on EC2** (use Let's Encrypt or third-party CA)
    - Supports **wildcard certificates** (`*.example.com`)
    - **Domain validation** via email or DNS (DNS is preferred for automation)
    - **ACM Private CA** is for private/internal certificates (costs $400/month per CA)
