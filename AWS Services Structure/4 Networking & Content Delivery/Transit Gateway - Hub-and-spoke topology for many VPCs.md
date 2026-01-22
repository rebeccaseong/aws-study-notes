- **What is it? (1-Sentence Pitch):** AWS Transit Gateway is a network transit hub that connects your VPCs, on-premises networks, and remote offices through a single gateway, simplifying network architecture and reducing complexity.

- **Core Use Cases:**
    - Connecting multiple VPCs (hub-and-spoke model)
    - Connecting VPCs to on-premises networks (via VPN or Direct Connect)
    - Centralized routing and network management
    - Scaling VPC connectivity (avoid full-mesh VPC peering)
    - Multi-region networking (Transit Gateway peering)

- **Key Features & Concepts:**
    - **Hub-and-Spoke Model:** Transit Gateway acts as the central hub, VPCs and on-prem networks are spokes
    - **Transitive Routing:** Enables connectivity between all attached networks (unlike VPC Peering)
    - **Supports Thousands of VPCs:** Can attach up to 5,000 VPCs to a single Transit Gateway
    - **Cross-Region Peering:** Transit Gateways in different regions can peer with each other
    - **Route Tables:** Transit Gateway has its own route tables to control traffic flow
    - **Multi-Account Support:** Can share Transit Gateway across AWS accounts using RAM (Resource Access Manager)
    - **Integration with VPN and Direct Connect:** Connect on-premises networks to all VPCs through one connection

- **How Transit Gateway Works:**
    1. **Attach VPCs and VPN/Direct Connect** to the Transit Gateway
    2. **Configure Route Tables** in Transit Gateway to control traffic flow
    3. **Update VPC Route Tables** to route traffic to Transit Gateway
    4. Traffic flows through Transit Gateway to reach destination VPCs or on-premises networks

- **Transit Gateway vs VPC Peering:**
    | **Aspect** | **Transit Gateway** | **VPC Peering** |
    |---|---|---|
    | **Connectivity Model** | **Hub-and-spoke** (centralized) | Point-to-point (pair of VPCs) |
    | **Transitive Routing** | **Yes** (all attached networks can communicate) | **No** (must create separate peering) |
    | **Scalability** | Excellent (thousands of VPCs) | Poor (full mesh = N*(N-1)/2 connections) |
    | **Cost** | **Hourly charge** + data processed | Data transfer only |
    | **Use Case** | **Many VPCs**, on-prem connectivity | Simple VPC-to-VPC (2-3 VPCs) |
    | **Complexity** | Centralized management | Decentralized (many connections) |

- **Transit Gateway vs PrivateLink:**
    | **Aspect** | **Transit Gateway** | **PrivateLink (VPC Endpoint)** |
    |---|---|---|
    | **Purpose** | **Network-level connectivity** | **Service-level access** |
    | **Connectivity** | VPC-to-VPC, VPC-to-on-prem | Access specific AWS services privately |
    | **Use Case** | Connect many networks | Private access to services (S3, SQS, etc.) |

- **Integration:**
    - **VPCs:** Attach multiple VPCs to Transit Gateway
    - **VPN:** Connect on-premises networks via Site-to-Site VPN
    - **Direct Connect:** Connect on-premises via dedicated network connection
    - **Transit Gateway Peering:** Connect Transit Gateways across regions
    - **RAM (Resource Access Manager):** Share Transit Gateway across AWS accounts
    - **Route 53 Resolver:** DNS resolution across attached networks

- **Exam Tips / What to Remember:**
    - **Transit Gateway = Hub for connecting many VPCs and on-prem networks**
    - Exam scenario: **"Connect 10+ VPCs"** or **"Simplify network architecture"** = Transit Gateway
    - **Transitive routing** is supported (unlike VPC Peering)
    - **Hub-and-spoke topology** reduces complexity (vs. full-mesh VPC Peering)
    - **Charges:** Hourly attachment fee + data processing fee
    - Use **VPC Peering for 2-3 VPCs**, use **Transit Gateway for many VPCs**
    - Can connect **on-premises networks** to all VPCs through one VPN/Direct Connect
    - **Transit Gateway Peering** enables cross-region connectivity
    - Integrates with **VPN** and **Direct Connect** for hybrid cloud architectures
