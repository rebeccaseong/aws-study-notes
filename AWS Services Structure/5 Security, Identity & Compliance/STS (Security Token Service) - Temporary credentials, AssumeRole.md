- **What is it? (1-Sentence Pitch):** AWS Security Token Service (STS) enables you to request temporary, limited-privilege credentials for IAM users or for users who authenticate through external identity providers (federation).

- **Core Use Cases:**
    - Cross-account access (users in Account A access resources in Account B)
    - Granting temporary access to AWS resources for federated users (e.g., corporate SSO)
    - Providing temporary access to mobile/web applications
    - Assuming IAM roles for elevated permissions
    - EC2 instances accessing AWS services using instance roles

- **Key Features & Concepts:**
    - **Temporary Credentials:** Short-lived credentials (15 minutes to 12 hours, default 1 hour)
    - **AssumeRole:** Primary API for obtaining temporary credentials by assuming an IAM role
    - **No Long-Term Credentials:** More secure than using permanent IAM access keys
    - **Session Token:** STS returns Access Key ID + Secret Access Key + Session Token
    - **Policy-Based Access:** Credentials respect both the role's permissions and optional session policies
    - **MFA Support:** Can require MFA for assuming roles (conditional access)

- **Common STS API Actions:**
    1. **AssumeRole:** Assume an IAM role (most common)
    2. **AssumeRoleWithSAML:** For federated users via SAML 2.0 (corporate SSO)
    3. **AssumeRoleWithWebIdentity:** For federated users via web identity providers (Google, Facebook, Amazon)
    4. **GetSessionToken:** Get temporary credentials for IAM users (with optional MFA)
    5. **GetFederationToken:** Get credentials for federated users (legacy, use AssumeRole instead)

- **How AssumeRole Works:**
    1. User or application calls **AssumeRole** API, specifying the role ARN
    2. STS verifies that the caller has permission to assume the role (via role's trust policy)
    3. STS returns temporary credentials (Access Key, Secret Key, Session Token)
    4. User uses temporary credentials to access AWS resources
    5. Credentials expire after the specified duration (default 1 hour)

- **AssumeRole Trust Policy Example:**
    ```json
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:user/Alice"
      },
      "Action": "sts:AssumeRole"
    }
    ```
    This allows user Alice from account 111122223333 to assume the role.

- **STS Use Cases:**
    | **Scenario** | **STS API** |
    |---|---|
    | Cross-account access | **AssumeRole** |
    | EC2 instance accessing AWS services | **AssumeRole** (via instance profile) |
    | Corporate SSO (SAML 2.0) | **AssumeRoleWithSAML** |
    | Mobile app users (Google/Facebook login) | **AssumeRoleWithWebIdentity** (or use Cognito) |
    | Temporary elevated permissions | **AssumeRole** |

- **Integration:**
    - **IAM Roles:** STS is the mechanism for assuming IAM roles
    - **IAM Identity Center:** Uses STS to provide temporary credentials
    - **EC2 Instance Profiles:** EC2 uses STS to get credentials from instance role
    - **Cognito:** Uses STS to provide temporary AWS credentials to app users
    - **Lambda:** Execution role credentials obtained via STS

- **Exam Tips / What to Remember:**
    - **STS provides temporary credentials** (more secure than long-term access keys)
    - **AssumeRole is the most common STS API** (cross-account access, assuming roles)
    - Exam scenario: **"Grant access to users from another AWS account"** = AssumeRole with trust policy
    - **Temporary credentials expire** (15 min to 12 hours, default 1 hour)
    - **Trust policy** defines who can assume the role
    - **Session policies** can further restrict permissions beyond the role's permissions
    - STS is **global service** (endpoints in all regions, but also a global endpoint)
    - **AssumeRoleWithWebIdentity** is legacy for web/mobile apps - **use Cognito instead**
    - EC2 instance profiles and Lambda execution roles use **STS behind the scenes**
