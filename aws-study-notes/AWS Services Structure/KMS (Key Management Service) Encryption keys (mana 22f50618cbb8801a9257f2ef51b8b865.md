# KMS (Key Management Service): Encryption keys (managed).

- **What is it? (1-Sentence Pitch):** KMS is a managed service that makes it easy for you to create and **control the encryption keys** used to protect your data, while its use of FIPS 140-2 validated Hardware Security Modules (HSMs) helps you meet compliance requirements.
- **Exam Rule of Thumb:** If a question mentions needing to **control key rotation**, **define specific IAM permissions for the key**, or **share the key with another account**, the answer is always a **Customer Managed Key**.
- **The Core Concept: Envelope Encryption**
    - KMS does not encrypt your large data objects directly. Instead, it uses a highly secure and efficient method called **Envelope Encryption**. This is the single most important concept to understand.
    - **Analogy:** Imagine you have a very important, large document (your data).
        1. You put the document in a standard lockbox and lock it with a small, disposable key. This key is called a **Data Key**.
        2. Now you have to protect the Data Key itself. You take this small key to a high-tech bank vault. This vault is your **KMS Key** (formerly called Customer Master Key or CMK).
        3. The vault manager (KMS) locks your Data Key inside the vault. You get a "receipt" which is the **Encrypted Data Key**.
        4. You can now safely store your locked document box (Encrypted Data) along with the vault receipt (Encrypted Data Key).
        5. To read your document later, you take the vault receipt back to the vault manager (KMS). KMS uses the vault's master key to unlock your Data Key and hands it back to you temporarily. You use this plaintext Data Key to open your document box.
    - **The Process**
        1. Your application asks KMS for a data key for a specific KMS Key.
        2. KMS returns two versions of the Data Key: a **Plaintext version** and an **Encrypted version**.
        3. You use the **Plaintext Data Key** in your application's memory to encrypt your data (e.g., a 10 GB file).
        4. You then **destroy the Plaintext Data Key** from memory as soon as possible.
        5. You store the **Encrypted Data** and the **Encrypted Data Key** together (e.g., in S3).
- **Types of KMS Keys (CRITICAL KNOWLEDGE)**
    - **Exam Rule of Thumb:** If a question mentions needing to **control key rotation**, **define specific IAM permissions for the key**, or **share the key with another account**, the answer is always a **Customer Managed Key**.
    
    | **Key Type** | **Managed By** | **Use Case & Characteristics** |
    | --- | --- | --- |
    | **AWS Managed Key** | **AWS** | **Free.** Created on your behalf when you first choose to encrypt a resource (e.g., an EBS volume). You can't change its key policy. Rotation is automatic and mandatory every 3 years. |
    | **Customer Managed Key** | **You** (The Customer) | **You have full control.** You create, manage, and define the key policy and IAM permissions. You can enable/disable, set up automatic rotation (every 1 year), and use it across services. **There is a cost** per key per month. |
    | **AWS Owned Key** | **AWS** (for its own use) | Used by AWS services to protect resources in the background (e.g., S3's SSE-S3 encryption). You cannot view, manage, or use these keys. They are not in your account. |
- **Security & Access Control (IAM vs. Key Policies)**
    - **Key Policy:** The **primary and most important** access control document. It is attached directly to the KMS Key. It defines **who can use** and **who can manage** the key.
    - **IAM Policies:** Used to **refine or delegate permissions** that are already granted in the Key Policy. An IAM policy alone is **not sufficient** to grant access to a KMS key.
    - **The Golden Rule:** To use a KMS Key, permission must be granted by **EITHER the Key Policy OR an IAM policy (but only if the Key Policy allows it).**
    - **The Gate Analogy**
        1. The **Key Policy** is the main gate to a property. If the Key Policy doesn't let your user ("iam:user/Bob") onto the property, they can't get in. Period.
        2. The **IAM Policy** is a key to a specific building *inside* the property.
        3. The Key Policy has a crucial statement: Allow root user of the account to control this key. This statement allows the account's root user to use IAM policies to delegate permissions to other users and roles. **If you delete this statement, you can lock yourself out of your own key!**
- **Integration with Other AWS Services (How it's really used)**
    - You rarely use the KMS Encrypt API directly. Instead, you tell other services to use KMS for Server-Side Encryption (SSE-KMS).
        - **Amazon S3 (SSE-KMS):** When you upload an object, you specify a KMS key. S3 calls KMS to get a unique data key, encrypts your object with it, and stores the encrypted data key as metadata with the object.
        - **Amazon EBS (Encrypted Volumes):** When you create an encrypted EBS volume, all data written to the disk is automatically encrypted using data keys managed by the specified KMS key.
        - **Amazon RDS:** Encrypts your database instances and snapshots at rest.
        - **AWS Secrets Manager:** Uses KMS by default to encrypt the values of the secrets you store.
- **Advanced Features for Engineers**
    - **Key Rotation:** For Customer Managed Keys, you can enable automatic annual rotation. KMS generates new backing key material, but the **KMS Key ID and ARN remain the same**, so you don't have to change your application code.
    - **Importing Key Material (BYOK - Bring Your Own Key):** For compliance scenarios where you must generate the key material yourself, outside of AWS. You can import it into a KMS key. AWS never sees your key in plaintext.
    - **Multi-Region Keys:** Special KMS keys that can be replicated into multiple AWS Regions. They share the same Key ID. This is useful for client-side encryption scenarios or DR strategies where you need the same key in different regions.
    - **Custom Key Stores:** An advanced feature that allows you to connect your KMS keys to a **CloudHSM cluster** that you own and manage. This gives you exclusive, single-tenant control over your HSMs while still integrating with AWS services.
- **Operations & Auditing**
    
    • **AWS CloudTrail:** KMS is fully integrated with CloudTrail. **Every single API call** (Encrypt, Decrypt, GenerateDataKey, DisableKey, etc.) is logged. This provides a critical audit trail for compliance, showing who used which key on what resource and when.