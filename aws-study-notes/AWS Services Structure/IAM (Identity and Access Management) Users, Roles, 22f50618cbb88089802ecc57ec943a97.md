# IAM (Identity and Access Management): Users, Roles, Policies, MFA.

- **What is it? (1-Sentence Pitch):** The core security service that centrally manages access to all AWS services and resources.
- **Core Use Cases:** Controlling who (or what) can perform which actions on which AWS resources.
- **Key Features & Concepts:**
    - **Users:** A permanent identity representing a person or application.
    - **Groups:** A collection of users, used to simplify permission management.
    - **Roles:** A temporary identity that can be assumed by trusted entities (EC2 instances, Lambda functions, users from another account). **CRITICAL CONCEPT:** Use roles for services, not access keys.
    - **Policies:** A JSON document that explicitly defines permissions (Allow/Deny).
    - **MFA (Multi-Factor Authentication):** Provides an extra layer of security for user sign-in.