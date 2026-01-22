- **What is it? (1-Sentence Pitch):** AWS Firewall Manager is a security management service that enables you to centrally configure and manage firewall rules (WAF, Shield, Security Groups, Network Firewall) across all accounts in your AWS Organization.

- **Core Use Cases:**
    - Centrally managing WAF rules across multiple AWS accounts
    - Enforcing Shield Advanced protection organization-wide
    - Applying consistent security group rules across all VPCs
    - Managing AWS Network Firewall rules at scale
    - Ensuring compliance with security policies across accounts

- **Key Features & Concepts:**
    - **Centralized Management:** Manage security policies from a single pane of glass
    - **Multi-Account Support:** Works with AWS Organizations to manage multiple accounts
    - **Automatic Enforcement:** Automatically applies security policies to new resources
    - **Supported Services:**
        - **AWS WAF:** Web application firewall rules
        - **AWS Shield Advanced:** DDoS protection
        - **VPC Security Groups:** EC2, ENI, ALB security groups
        - **AWS Network Firewall:** Traffic filtering rules
        - **Route 53 Resolver DNS Firewall:** DNS-level filtering
    - **Compliance Reporting:** Dashboard shows non-compliant resources
    - **Auto-Remediation:** Automatically remediate non-compliant resources

- **How Firewall Manager Works:**
    1. **Define Security Policy:** Create a Firewall Manager policy (e.g., WAF rule group)
    2. **Apply to Organization:** Specify which accounts/OUs (organizational units) to apply policy to
    3. **Automatic Deployment:** Firewall Manager deploys policies to all specified accounts
    4. **Continuous Monitoring:** Monitors resources for compliance
    5. **Auto-Remediation:** Fixes non-compliant resources automatically (if enabled)

- **Firewall Manager vs Individual Services:**
    | **Aspect** | **Firewall Manager** | **Direct Service Management** |
    |---|---|---|
    | **Management** | **Centralized** (one place for all accounts) | Decentralized (manage per account) |
    | **Scope** | **Multi-account** (AWS Organizations) | Single account |
    | **Use Case** | **Large organizations** with many accounts | Small setups, single account |
    | **Auto-Enforcement** | **Yes** (new resources auto-protected) | Manual (must configure each resource) |
    | **Compliance Reporting** | **Yes** (dashboard shows violations) | Manual auditing |

- **Supported Policies:**
    1. **WAF Policy:** Apply WAF rules to ALB, API Gateway, CloudFront
    2. **Shield Advanced Policy:** Enable Shield Advanced protection on resources
    3. **Security Group Policy:** Enforce security group rules across VPCs
    4. **Network Firewall Policy:** Deploy Network Firewall rules to VPCs
    5. **DNS Firewall Policy:** Apply Route 53 Resolver DNS Firewall rules

- **Integration:**
    - **AWS Organizations:** Required for multi-account management
    - **WAF:** Centrally manage WAF rule groups
    - **Shield Advanced:** Centrally enable DDoS protection
    - **Security Groups:** Enforce security group policies across VPCs
    - **Network Firewall:** Centrally manage network traffic filtering
    - **Route 53 Resolver:** Manage DNS firewall rules
    - **CloudWatch:** Monitor Firewall Manager metrics
    - **SNS:** Send notifications for policy violations

- **Exam Tips / What to Remember:**
    - **Firewall Manager = Centralized security management across multiple AWS accounts**
    - Exam scenario: **"Apply WAF rules to all accounts in the organization"** = Firewall Manager
    - **Requires AWS Organizations** (cannot use in standalone accounts)
    - **Automatic enforcement:** New resources automatically comply with policies
    - Supports **WAF, Shield, Security Groups, Network Firewall, DNS Firewall**
    - **WAF alone** is for single account, **Firewall Manager** is for multi-account WAF management
    - **Compliance dashboard** shows which resources are non-compliant
    - **Auto-remediation** fixes non-compliant resources automatically
    - Ideal for **large organizations** with centralized security teams
