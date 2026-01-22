- **What is it? (1-Sentence Pitch):** Auto Scaling Groups automatically adjust the number of EC2 instances in your application based on demand, ensuring you have the right capacity at the right time while optimizing costs.

- **Core Use Cases:**
    - Maintaining application availability by replacing unhealthy instances
    - Scaling compute capacity up during traffic spikes and down during quiet periods
    - Cost optimization by running only the instances you need
    - Integrating with Elastic Load Balancers for distributing traffic

- **Key Features & Concepts:**
    - **Launch Template/Configuration:** Defines the instance configuration (AMI, instance type, security groups, etc.)
    - **Desired Capacity:** The number of instances ASG attempts to maintain
    - **Min/Max Capacity:** The boundaries for scaling (e.g., min 2, max 10)
    - **Health Checks:** EC2 status checks or ELB health checks to determine instance health
    - **Scaling Policies:**
        - **Target Tracking:** Scale based on a target metric (e.g., maintain 70% CPU utilization)
        - **Step Scaling:** Scale in steps based on CloudWatch alarm thresholds
        - **Simple Scaling:** Add/remove a fixed number of instances
        - **Scheduled Scaling:** Scale based on predictable time patterns
    - **Cooldown Period:** Prevents ASG from launching/terminating instances too quickly

- **Integration:**
    - Works with **Elastic Load Balancing (ALB/NLB)** to distribute traffic
    - Uses **CloudWatch** metrics and alarms for scaling decisions
    - Integrates with **Launch Templates** for instance configuration
    - Can span multiple **Availability Zones** for high availability

- **Exam Tips / What to Remember:**
    - **Target Tracking is the easiest and most common** scaling policy (set a target and ASG figures out the rest)
    - ASG itself is **free** - you only pay for the EC2 instances launched
    - **Desired capacity** can be manually set or automatically adjusted by scaling policies
    - Instances are automatically replaced if they become unhealthy
    - **Launch Templates are preferred over Launch Configurations** (newer, more features)
    - Can use **predictive scaling** (machine learning) to forecast traffic patterns
