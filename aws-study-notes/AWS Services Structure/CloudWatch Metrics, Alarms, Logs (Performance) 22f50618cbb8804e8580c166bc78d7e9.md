# CloudWatch: Metrics, Alarms, Logs (Performance).

- **What is it? (1-Sentence Pitch):** The central monitoring and observability service for AWS resources and the applications you run on AWS.
- **Key Features & Concepts:**
    - **Metrics:** The core of CloudWatch. A time-ordered set of data points (e.g., EC2 CPU Utilization, Lambda Invocations).
    - **Alarms:** Automatically perform actions (e.g., send an SNS notification, trigger an Auto Scaling action) when a metric crosses a threshold.
    - **Logs:** A centralized repository to store, monitor, and access log files from EC2 instances, Lambda, CloudTrail, and other sources.
    - **Events (now part of EventBridge):** Delivers a near real-time stream of system events that describe changes in AWS resources.