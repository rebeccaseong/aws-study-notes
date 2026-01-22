- **What is it? (1-Sentence Pitch):** AWS Budgets enables you to set custom cost and usage budgets that alert you when your costs or usage exceed (or are forecasted to exceed) your budgeted amount, helping you stay on top of your AWS spending.

- **Core Use Cases:**
    - Setting monthly cost budgets for AWS accounts
    - Monitoring Reserved Instance and Savings Plans utilization
    - Alerting when costs exceed thresholds (actual or forecasted)
    - Tracking usage budgets for specific services (e.g., EC2 hours, S3 storage)
    - Implementing cost governance across teams and projects

- **Key Features & Concepts:**
    - **Budget Types:**
        1. **Cost Budgets:** Monitor total costs (or specific service costs)
        2. **Usage Budgets:** Monitor usage of specific resources (e.g., EC2 hours, S3 GB)
        3. **RI/SP Utilization Budgets:** Monitor Reserved Instance or Savings Plans utilization
        4. **RI/SP Coverage Budgets:** Monitor how much of your usage is covered by RIs/SPs
    - **Alerts:** Receive notifications when budget thresholds are exceeded (actual or forecasted)
    - **Forecasting:** Budgets can alert based on forecasted spend (before it happens)
    - **Multiple Thresholds:** Set multiple alert thresholds (e.g., 50%, 80%, 100%)
    - **SNS Integration:** Send alerts via email, SMS, or trigger automated actions

- **How AWS Budgets Works:**
    1. **Create Budget:** Define budget type, amount, and time period (monthly, quarterly, annually)
    2. **Set Thresholds:** Configure alert thresholds (e.g., alert at 80% and 100%)
    3. **Configure Alerts:** Choose how to be notified (email, SNS topic)
    4. **Monitor:** AWS Budgets tracks your costs/usage and compares to budget
    5. **Alert:** Receive alerts when thresholds are exceeded (actual or forecasted)

- **Budget Actions (Automated Response):**
    - **Apply IAM Policy:** Automatically restrict permissions when budget exceeds
    - **Apply SCP (Service Control Policy):** Restrict actions at organization level
    - **Stop EC2/RDS Instances:** Automatically stop instances to prevent overspend

- **AWS Budgets vs Cost Explorer vs CloudWatch Billing Alarms:**
    | **Aspect** | **AWS Budgets** | **Cost Explorer** | **CloudWatch Billing Alarms** |
    |---|---|---|---|
    | **Purpose** | **Set budgets and alerts** | **Visualize and analyze** costs | Simple billing alerts |
    | **Forecasting** | **Yes** (alert before overspend) | **Yes** (forecast future costs) | No |
    | **Budget Types** | Cost, Usage, RI/SP utilization/coverage | N/A | Billing total only |
    | **Granularity** | Service, tag, account level | Highly customizable | Account-wide total only |
    | **Automation** | **Budget Actions** (IAM, SCP, stop instances) | No | No |
    | **Free Tier** | First 2 budgets free | Free | Free |

- **Integration:**
    - **SNS:** Send budget alerts to SNS topics (trigger Lambda, email, SMS)
    - **Cost Explorer:** Use Cost Explorer data to set realistic budgets
    - **IAM/SCP:** Budget Actions can automatically apply policies
    - **EventBridge:** Trigger custom workflows when budgets are exceeded
    - **Chatbot:** Send budget alerts to Slack or Microsoft Teams

- **Exam Tips / What to Remember:**
    - **AWS Budgets = Proactive cost control** (set budgets, get alerted)
    - Exam scenario: **"Alert when monthly costs exceed $1000"** = AWS Budgets
    - **Forecasting:** Can alert based on **forecasted spend** (before you overspend)
    - **Budget Actions** can automatically restrict permissions or stop instances
    - **First 2 budgets are free**, additional budgets cost $0.02/day each
    - **Multiple alert thresholds** (e.g., 50%, 80%, 100%, 120%)
    - **CloudWatch Billing Alarms** are simpler but less flexible (account-wide only, no forecasting)
    - Can create budgets for **specific services, tags, linked accounts, or entire organization**
    - **RI/SP Utilization budgets** ensure you're getting value from Reserved Instances/Savings Plans
