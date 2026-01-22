# Route 53: DNS records (A, CNAME, Alias), Routing Policies (Failover, Latency, Geolocation).

- **What is it? (1-Sentence Pitch):** A highly available and scalable cloud Domain Name System (DNS) web service.
- **Core Use Cases:**
    - Registering domain names (e.g., mycoolapp.com).
    - Routing end-user traffic to your application's endpoint (EC2, ELB, CloudFront).
    - Configuring DNS health checks and failover.
- **Key Features & Concepts (Routing Policies are CRITICAL):**
    - **Simple:** Route traffic to a single resource.
    - **Weighted:** Route traffic to multiple resources in proportions you specify (e.g., 80% to v1, 20% to v2 for testing).
    - **Latency:** Route users to the AWS region that provides the lowest latency.
    - **Failover:** An active-passive setup. Redirects traffic to a standby resource if the primary becomes unhealthy.
    - **Geolocation:** Route users based on their geographic location (e.g., users from Europe see the German version of a site).