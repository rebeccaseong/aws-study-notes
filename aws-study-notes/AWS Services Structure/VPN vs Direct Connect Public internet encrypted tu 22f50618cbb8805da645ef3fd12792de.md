# VPN vs. Direct Connect: Public internet encrypted tunnel vs. Private physical fiber.

- **What is it? (1-Sentence Pitch):** Services that establish a private, dedicated network connection from your on-premises data center to AWS.
- **Key Distinction (CRITICAL):**
    - **AWS Direct Connect (DX):** A physical, private fiber-optic connection. **Use when** you need a stable, reliable, high-bandwidth connection (e.g., 1/10/100 Gbps). Reduces network costs and increases consistency.
    - **AWS Site-to-Site VPN:** An encrypted tunnel over the public internet. **Use when** you need a quick, cost-effective, and secure connection, but can tolerate the variability of internet performance.