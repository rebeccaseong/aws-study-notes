- **What is it? (1-Sentence Pitch):** VPC Peering is a networking connection between two VPCs that enables you to route traffic between them privately using private IP addresses, as if they were part of the same network.

- **Core Use Cases:**
    - Connecting VPCs within the same AWS account (e.g., Dev and Prod VPCs)
    - Connecting VPCs across different AWS accounts (e.g., partner integrations)
    - Connecting VPCs in different AWS regions (inter-region peering)
    - Sharing resources between VPCs (databases, file systems, etc.)

- **Key Features & Concepts:**
    - **One-to-One Connection:** Each VPC Peering connection links exactly two VPCs
    - **Non-Transitive:** VPC A peered with VPC B, and VPC B peered with VPC C does NOT mean A can reach C (must create A↔C peer)
    - **No Overlapping CIDR Blocks:** VPCs must have non-overlapping IP address ranges
    - **Private Connectivity:** Traffic stays on AWS private network (does not traverse the internet)
    - **Cross-Region Peering:** Supports peering across different AWS regions
    - **Cross-Account Peering:** Supports peering between VPCs in different AWS accounts
    - **Pricing:** Data transfer charges apply (especially for cross-region peering)

- **How VPC Peering Works:**
    1. **Request:** VPC A owner sends a peering request to VPC B
    2. **Accept:** VPC B owner accepts the peering request
    3. **Update Route Tables:** Both VPCs add routes pointing to each other's CIDR blocks
    4. **Update Security Groups:** Allow traffic between the VPCs (reference security group IDs if same region)

- **Limitations:**
    - **Not Transitive:** Requires full mesh for multiple VPCs (e.g., 3 VPCs = 3 peering connections)
    - **No Edge-to-Edge Routing:** Cannot use VPC A's Internet Gateway from VPC B
    - **No Overlapping CIDR:** VPCs with overlapping IP ranges cannot be peered
    - **DNS Resolution:** Requires enabling DNS resolution for cross-VPC queries

- **VPC Peering vs Transit Gateway vs PrivateLink:**
    | **Aspect** | **VPC Peering** | **Transit Gateway** | **PrivateLink** |
    |---|---|---|---|
    | **Connectivity** | **One-to-one** (pair of VPCs) | **Hub-and-spoke** (many VPCs) | **Service-level** (expose services) |
    | **Transitive** | No | **Yes** | N/A |
    | **Scalability** | Poor (full mesh needed) | Excellent (centralized hub) | N/A |
    | **Cost** | Data transfer charges | Hourly + data transfer | Hourly + data transfer |
    | **Use Case** | Simple VPC-to-VPC | **Many VPCs** or on-prem | Expose specific services privately |

- **Integration:**
    - **Route Tables:** Must update route tables in both VPCs to route traffic through peering connection
    - **Security Groups:** Can reference security group IDs across peered VPCs (same region only)
    - **DNS:** Enable DNS resolution to use DNS names across VPCs

- **Exam Tips / What to Remember:**
    - **VPC Peering = Direct connection between two VPCs**
    - **Not transitive:** A-B peering + B-C peering ≠ A-C connectivity
    - **No overlapping CIDR blocks** allowed between peered VPCs
    - Exam scenario: **"Connect two VPCs"** = VPC Peering
    - Exam scenario: **"Connect 10+ VPCs"** = Transit Gateway (not VPC Peering, too many connections)
    - Can peer across **regions** and **accounts**
    - **Requires route table updates** on both sides
    - **Cannot use one VPC's Internet Gateway from another** (no edge-to-edge routing)
    - For **many-to-many connectivity**, use **Transit Gateway** instead
