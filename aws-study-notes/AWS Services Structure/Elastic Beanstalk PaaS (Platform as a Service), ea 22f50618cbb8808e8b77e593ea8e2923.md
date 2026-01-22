# Elastic Beanstalk: PaaS (Platform as a Service), easy deployment.

- **What is it? (1-Sentence Pitch):** A Platform-as-a-Service (PaaS) that lets developers easily deploy and manage web applications without worrying about the underlying infrastructure.
- **Core Use Cases:**
    - Rapidly deploying a standard web application (e.g., Java, .NET, Node.js, Python, Ruby).
    - For teams that want to focus on writing code, not managing AWS resources.
- **Key Features & Concepts:**
    - **You provide the code, Beanstalk handles the rest:** It provisions EC2 instances, load balancers, auto-scaling, and monitoring for you.
    - **Deployment Models:** All at once, Rolling, Rolling with additional batch, Immutable.
    - You still have full control over the underlying resources if you need it.
- **Key Distinction:** **Beanstalk vs. CloudFormation:** Beanstalk is a PaaS for standard web apps. CloudFormation is an IaC tool to provision *any* AWS resource. Beanstalk uses CloudFormation under the hood.