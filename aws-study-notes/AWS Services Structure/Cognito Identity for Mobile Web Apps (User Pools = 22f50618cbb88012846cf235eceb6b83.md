# Cognito: Identity for Mobile/Web Apps (User Pools = Auth, Identity Pools = AWS Access).

- **What is it? (1-Sentence Pitch):** A managed service that provides authentication, authorization, and user management for your web and mobile applications.
- **Core Use Cases:**
    - Adding sign-up and sign-in functionality to your app.
    - Federating identities from social providers (like Google, Facebook) or enterprise providers (SAML).
- **Key Features & Concepts (CRITICAL DISTINCTION):**
    - **User Pools:** The user directory. Handles user registration, sign-in, and account recovery.
    - **Identity Pools:** Grants your users temporary AWS credentials to access AWS resources directly (e.g., upload a file to S3).
- **Key Distinction:** **IAM** is for controlling access to *AWS resources*. **Cognito** is for managing *your application's users*.