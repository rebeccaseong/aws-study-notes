- **What is it? (1-Sentence Pitch):** AWS IAM Identity Center (successor to AWS Single Sign-On) provides centralized access management across all your AWS accounts and applications, enabling users to sign in once and access multiple AWS accounts and business applications.

- **Core Use Cases:**
    - Single sign-on across multiple AWS accounts in an organization
    - Centralized user and group management
    - Integration with external identity providers (Microsoft AD, Okta, Azure AD)
    - Accessing AWS CLI, SDK, and AWS Console with SSO credentials
    - Managing access to third-party SaaS applications (Salesforce, Microsoft 365, etc.)

- **Key Features & Concepts:**
    - **Centralized Access Portal:** Single URL for users to access all assigned AWS accounts and applications
    - **Multi-Account Management:** Manage permissions across all AWS accounts in your organization
    - **Identity Source:** Choose between IAM Identity Center directory, Active Directory, or external IdP (SAML 2.0)
    - **Permission Sets:** Collections of IAM policies that define user access in AWS accounts
    - **Automatic Provisioning (SCIM):** Sync users and groups from external identity providers
    - **MFA Support:** Enforce multi-factor authentication for sign-in
    - **No Additional Cost:** Free service (you only pay for underlying AWS services used)

- **How IAM Identity Center Works:**
    1. User signs in to IAM Identity Center portal (single URL)
    2. IAM Identity Center authenticates the user (using configured identity source)
    3. User sees all AWS accounts and applications they have access to
    4. User selects an AWS account/application and is granted temporary credentials
    5. User accesses AWS services using those temporary credentials

- **IAM Identity Center vs Traditional IAM:**
    | **Aspect** | **IAM Identity Center** | **IAM Users** |
    |---|---|---|
    | **Use Case** | **Multi-account** organizations | Single AWS account |
    | **User Management** | Centralized (one place) | Per-account (decentralized) |
    | **Identity Source** | External IdP, AD, or IAM Identity Center | IAM only |
    | **SSO** | Yes | No (separate login per account) |
    | **Best For** | **Organizations with many accounts** | Small setups, single account |
    | **Credentials** | Temporary (STS) | Long-term (access keys) |

- **Integration:**
    - **AWS Organizations:** Automatically integrates with all accounts in your organization
    - **Active Directory:** Connect to on-premises or AWS Managed Microsoft AD
    - **External IdPs:** Integrate with Okta, Azure AD, etc. (SAML 2.0)
    - **AWS CLI/SDK:** Users can authenticate via `aws sso login`
    - **Third-Party SaaS:** Provide SSO to Salesforce, Slack, Microsoft 365, etc.
    - **CloudTrail:** Audit all sign-in and access activities

- **Exam Tips / What to Remember:**
    - **IAM Identity Center (formerly AWS SSO)** is for **multi-account environments**
    - Exam scenario: **"Centralize login across 50 AWS accounts"** = IAM Identity Center
    - **Free service** (no additional cost)
    - Works best with **AWS Organizations** (multi-account management)
    - **Permission Sets** define what users can do in each AWS account
    - Can integrate with **external identity providers** (Microsoft AD, Okta, Azure AD)
    - Provides **temporary credentials** (more secure than long-term IAM access keys)
    - Replaces the need to create IAM users in every AWS account
    - Users get a **single sign-on portal** to access all accounts and applications
