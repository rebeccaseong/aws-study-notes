- **What is it? (1-Sentence Pitch):** A web application firewall that helps protect your web applications from common web exploits like SQL injection and cross-site scripting.
    
- **Core Use Cases:** Filtering malicious web traffic before it reaches your application.
    
- **Key Features & Concepts:**
    
    - Operates at Layer 7 (the application layer), meaning it understands and inspects the actual HTTP requests.
    - You define rules to allow or block requests based on IP addresses, HTTP headers, or request strings.(The "How")
    - The Threats You're Looking For (The "What")
        - **Threat:** SQL Injection
        - **Threat:** Cross-Site Scripting (XSS)
        - **Threat:** Bad Bots
        - **Threat:** A specific person you want to block (by their IP address)
        - **Threat:** Anyone from a specific country you want to block (by their geographic location)
    - Integrates with **CloudFront, Application Load Balancer (ALB), API Gateway, and AppSync**.
        1. **Amazon CloudFront:** This is the most common and best practice. You put WAF at the "edge," protecting your application before malicious traffic even gets close to your main region.
        2. **Application Load Balancer (ALB):** As you mentioned, you can attach a WAF directly to your ALB. This protects all the EC2 instances or services behind that load balancer.
        3. **Amazon API Gateway:** You can attach a WAF to protect your REST and HTTP APIs from malicious requests.
        4. **AWS AppSync:** Protects your GraphQL APIs.
- **When Do We Use a Firewall? (WAF vs. Other Firewalls)**

| **Firewall Type**    | **AWS WAF (Web Application Firewall)**                                | **Security Groups & NACLs (Network Firewall)**                          | **AWS Network Firewall**                                                    |
| -------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Analogy**          | **The Bag-Checking Security Guard** at your application's front door. | **The Property Gatekeeper** checking your ID and what door you can use. | **The Central Security Checkpoint** for the entire corporate campus (VPC).  |
| **What it Inspects** | **Layer 7:** The _content_ of the HTTP request (URL, headers, body).  | **Layer 3/4:** The _source/destination IP address_ and _port number_.   | **Layers 3-7:** IP, Port, and can do deep packet inspection of the content. |
| **Typical Rule**     | "Block any request that contains 'SQL injection' patterns."           | "Allow traffic on Port 443 from the ALB's IP address."                  | "Block all traffic going from the Dev VPC to the Prod VPC on any port."     |
| **Where it's Used**  | Attached to CloudFront, ALB, API Gateway to protect web apps.         | Attached to EC2 instances (Security Groups) and subnets (NACLs).        | Deployed inside your VPC to inspect traffic between subnets or VPCs.        |
| **Is it Mandatory?** | Optional, but highly recommended for public web apps.                 | **Security Groups are essential** for every EC2 instance.               | Optional, for advanced enterprise security and compliance needs.            |