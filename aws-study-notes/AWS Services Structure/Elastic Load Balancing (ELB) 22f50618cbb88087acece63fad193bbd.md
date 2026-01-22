# Elastic Load Balancing (ELB)

- **What is it? (1-Sentence Pitch):** A service that automatically distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses, to increase fault tolerance and availability.
- **Core Use Cases:**
    - Preventing a single EC2 instance from being a point of failure.
    - Distributing workload to handle high traffic.
    - Providing a single, stable DNS endpoint for your application.
- **Key Features & Concepts (CRITICAL TO KNOW THE DIFFERENCES):**
    - **Application Load Balancer (ALB):** Layer 7 (HTTP/S). Best for flexible application routing based on path (/users, /images) or hostname. Integrates with WAF.
        - Four Main Routing Types of an ALB
            - Path-based Routing
                - **What it inspects:** The path of the URL (the part after the .com).
                - **Example:** It sees https://www.example.com/orders and sends it to the "Orders" microservice. It sees https://www.example.com/products and sends it to the "Products" microservice.
            - Host-based Routing
                - **What it inspects:** The hostname (the domain name part of the URL).
                - **Example:** It sees a request for orders.example.com and sends it to the "Orders" microservice. It sees a request for products.example.com and sends it to the "Products" microservice. This is useful for routing traffic for different subdomains.
            - HTTP Header-based Routing
                - **What it inspects:** The headers of the HTTP request, like the User-Agent.
                - **Example:** You could route users based on the browser they are using. IF User-Agent contains "Mobile", send them to the target group with servers optimized for mobile. IF User-Agent contains "Chrome", send them to another target group.
            - Query String Parameter-based Routing
                - **What it inspects:** The query string (the part of the URL after a ?).
                - **Example:** A request for www.example.com/search?user=Alice could be routed to one server, while a request for www.example.com/search?user=Bob could be routed to another. Or you could route .../video?format=mp4 and .../video?format=mov to different processing servers.
                - **Why it's incorrect:** The question's routing logic was based on the path, not query strings.
        - Target Groups and Health Checks
            - **Target Groups:** A collection of targets (EC2 instances, IP addresses, Lambda functions) that receive traffic.
            - **Health Checks:** The ALB constantly pings a specific path on your instances (e.g., /health). If an instance fails to respond correctly multiple times (e.g., returns a 404 error or times out), the ALB marks it as "unhealthy" and **stops sending traffic to it**. This is fundamental to high availability.
        - SSL/TLS Termination
            - SSL/TLS Termination: The process where the Load Balancer decrypts incoming traffic before sending it to the backend servers.
                - Think of the ALB as the central, highly secure **mailroom** for a giant company (your application). The individual offices in the company are your EC2 instances.
                    1. **You send your secure envelope (HTTPS request)** addressed to the company. It arrives at the secure mailroom (the ALB). The journey from you to the mailroom is fully encrypted and safe.
                    2. The mailroom's job is to receive and sort the mail. The mailroom clerk (the ALB) has the special key to open the secure envelope. The act of the ALB opening this encrypted envelope is called **"SSL/TLS Termination."**
                    3. The "termination" means the encrypted part of the journey **ends** at the ALB.
                    4. Now the mailroom clerk has the open, readable message (an unencrypted HTTP request). They read the address inside and say, "This is for the billing department."
                    5. The clerk then sends the **open postcard** down the private, secure company hallway to the billing department (the EC2 instance). This is safe because no outsiders can get into the company's private hallways.
                - What are the benefits of the "mailroom" (ALB) opening the mail?
                    - **Less Work for the Offices (EC2 Instances):** The individual offices don't need to have their own special keys or learn how to open secure envelopes. They can focus on their real job. This offloads the heavy processing work of encryption/decryption from your backend servers, letting them run faster.
                    - **Centralized Security:** You manage security at one single entry point—the mailroom—instead of at hundreds of different office doors. This is much easier and safer.
            - This is a critical feature. The ALB can handle the heavy work of decrypting HTTPS traffic from users.
            - The connection from the **User to the ALB is encrypted (HTTPS)**.
            - The connection from the **ALB to the backend EC2 instances can be unencrypted (HTTP)**, which simplifies your backend server configuration.
            - It integrates seamlessly with **AWS Certificate Manager (ACM)** for free SSL/TLS certificates.
        - Sticky Sessions (Session Affinity)
            - **Use Case:** For applications that store user session information locally on a server (stateful applications).
            - **What it does:** The ALB generates a cookie. For all subsequent requests from the same user (with that cookie), the ALB will send the request to the **exact same backend instance**. This ensures the user doesn't lose their session information.
            - **Purpose:** To maintain **session state** for a specific user.
            - **What is stored?** User-specific, temporary information (e.g., items in a shopping cart, login status, form data).
            - **Where is it stored?** On the local memory/disk of **one specific server**. It is not shared with other servers.
            - **Function:** It is a **ROUTING** feature of the Load Balancer. It doesn't store data itself; it just directs you to the server that *does* have your data.
            - Differences with Caching
                - **True Caching (like CloudFront or ElastiCache)** is about storing a *copy of data* in a faster location to avoid asking for it from a slow source.
                - **Sticky Sessions** is about making sure a user always talks to the *same server* because that server is holding onto that user's unique information. It's a **routing rule**, not a shared data store.
                
                | **Feature** | **Sticky Sessions (Session Affinity)** | **True Caching (CloudFront, ElastiCache)** |
                | --- | --- | --- |
                | **Main Purpose** | Ensure a user consistently connects to the **same server**. | Store a **copy of data** in a faster location to speed up access. |
                | **Primary Function** | **Routing** Strategy | **Data Storage** Strategy |
                | **Data Scope** | **User-specific**, private session data (e.g., shopping cart). | **Shared data** (public files, popular query results). |
                | **Where Data is Stored** | On the local disk/memory of **one specific EC2 instance**. | In a **separate, shared service** (Edge Location or ElastiCache cluster). |
                | **Who Manages It?** | It's a feature of the **Application Load Balancer**. | It is a **dedicated caching service** (CloudFront, ElastiCache). |
                | **Analogy** | Being sent to your personal banker who has your paperwork. | Reading a public notice board or using a shared, fast-access vault. |
                | **Why use it?** | Your application is **stateful** (stores session data on the server). | Your application is slow due to network latency or database load. |
        - Integration with AWS WAF & Shield
            - You can attach an **AWS WAF (Web Application Firewall)** directly to an ALB to protect your application from common web attacks like SQL injection and cross-site scripting.
            - ALBs are protected by **AWS Shield Standard** for free against common DDoS attacks.
        - Other Rule Actions (Besides "Forward")
            - **Redirect:** You can create a rule to redirect traffic. The most common use is redirecting HTTP requests to HTTPS, or example.com to www.example.com.
            - **Fixed Response:** You can have the ALB return a simple, plain-text response directly, without forwarding to a target. Useful for maintenance pages.
            - **Authentication:** You can configure a rule to require users to authenticate via **Amazon Cognito** or another identity provider *before* their request is forwarded to a target.
        - Cross-Zone Load Balancing
            
            • This is **enabled by default** on an ALB. It means the ALB distributes traffic evenly across all registered instances in **all enabled Availability Zones**. If you have 2 instances in AZ-A and 8 in AZ-B, it will still try to send 50% of traffic to each AZ. This is a key high-availability feature.
            
    - **Network Load Balancer (NLB):** Layer 4 (TCP/UDP). Best for extreme performance, high throughput, and when you need a static IP address.
    - **Gateway Load Balancer (GWLB):** Layer 3/4. Used to deploy, scale, and manage third-party virtual appliances like firewalls and intrusion detection systems.