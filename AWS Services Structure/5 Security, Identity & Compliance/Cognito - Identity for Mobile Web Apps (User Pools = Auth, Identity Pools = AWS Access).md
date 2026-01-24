# Cognito - Identity for Mobile/Web Apps (User Pools = Auth, Identity Pools = AWS Access)

- **What is it? (1-Sentence Pitch):** A managed service that provides authentication, authorization, and user management for your web and mobile applications.
- **Core Use Cases:** Adding sign-up and sign-in functionality to your app, federating identities from social providers (Google, Facebook) or enterprise providers (SAML), granting mobile users temporary AWS credentials to access AWS resources.
- **Key Distinction:** **IAM** is for controlling access to *AWS resources* by AWS users/roles. **Cognito** is for managing *your application's end users* (customers).

---

## What Cognito Does

Cognito provides two main services:
1. **User Pools:** User directory with authentication (sign-up, sign-in, password reset)
2. **Identity Pools:** Grant temporary AWS credentials to authenticated users

**Key Point:** These are **two separate services** that can be used together or independently.

---

## User Pools vs Identity Pools (CRITICAL!)

**This is the most tested distinction on the exam.**

```
┌─────────────────────────────────────────────────────────┐
│              Cognito Components                         │
├────────────────────────┬────────────────────────────────┤
│     User Pools         │     Identity Pools             │
│  (Authentication)      │     (Authorization)            │
├────────────────────────┼────────────────────────────────┤
│ "Who are you?"         │ "What can you access?"         │
│ Returns: JWT tokens    │ Returns: AWS credentials (STS) │
│ Sign-up/sign-in        │ Access S3, DynamoDB, etc.      │
└────────────────────────┴────────────────────────────────┘
```

### **Quick Comparison**

| Aspect | User Pools | Identity Pools |
|--------|-----------|----------------|
| **Purpose** | **Authentication** (who are you?) | **Authorization** (what AWS resources can you access?) |
| **Returns** | **JWT tokens** | **Temporary AWS credentials** (via STS) |
| **Use Case** | Sign-in for your app | Access AWS services directly from app (S3, DynamoDB) |
| **Analogy** | The bouncer checking your ID | The valet giving you a parking pass |
| **Standalone** | ✅ Yes (works alone) | ✅ Yes (can use without User Pools) |
| **Together** | ✅ Yes (User Pool authenticates → Identity Pool grants AWS access) |

---

### **Exam Trigger Phrases**

| Phrase in Question | Answer |
|-------------------|---------|
| "Sign-in for mobile app" | **User Pools** |
| "Upload files to S3 from mobile app" | **Identity Pools** (grants AWS credentials) |
| "Social login (Google, Facebook)" | **User Pools** (federated identities) |
| "Access DynamoDB directly from browser" | **Identity Pools** |
| "MFA, password policies" | **User Pools** |
| "Temporary AWS credentials for mobile users" | **Identity Pools** |

---

## 1. User Pools (Authentication)

**What It Is:** A user directory that provides sign-up and sign-in functionality for your app.

### **Key Features**

| Feature | Description |
|---------|-------------|
| **User Directory** | Store user profiles (username, email, phone, custom attributes) |
| **Sign-up / Sign-in** | Built-in UI or custom UI |
| **Password Policies** | Minimum length, complexity requirements |
| **MFA** | SMS, TOTP (Time-based One-Time Password) |
| **Email/SMS Verification** | Verify email or phone number |
| **Password Recovery** | Forgot password flow |
| **Social Sign-in** | Google, Facebook, Amazon, Apple |
| **SAML/OIDC** | Enterprise identity providers (Okta, Auth0, Active Directory) |
| **Custom Authentication** | Lambda triggers for custom auth flows |

---

### **How User Pools Work**

```
User (Mobile/Web App)
      ↓
Cognito User Pool (authenticate)
      ↓
Returns: JWT tokens (ID token, Access token, Refresh token)
      ↓
App uses tokens to call API Gateway, AppSync, or custom backend
```

**JWT Tokens:**
- **ID Token:** Contains user identity information (username, email, etc.)
- **Access Token:** Used to authorize API calls (API Gateway, AppSync)
- **Refresh Token:** Used to get new access tokens (long-lived)

---

### **Authentication Methods**

| Method | Description | Use Case |
|--------|-------------|----------|
| **Username/Password** | Traditional sign-in | Basic authentication |
| **Email/Phone + Password** | Sign-in with email or phone | User-friendly |
| **Social Sign-in** | Google, Facebook, Amazon, Apple | Reduce friction (users don't create new account) |
| **SAML** | Enterprise identity providers (Okta, Azure AD) | Corporate apps |
| **OIDC** | OpenID Connect providers | Modern OAuth2 flows |
| **Custom Authentication** | Lambda triggers | Custom auth logic (e.g., challenge-response) |

---

### **User Pool Hosted UI**

Cognito provides a **pre-built, customizable hosted UI** for sign-up and sign-in.

**Features:**
- Fully managed (no code required)
- Customizable logo, colors, CSS
- Supports all authentication methods
- OAuth 2.0 / OIDC compliant

**Use Case:** Quick setup without building custom login UI

---

### **Lambda Triggers**

Customize the authentication flow with Lambda functions:

| Trigger | When It Runs | Use Case |
|---------|-------------|----------|
| **Pre-sign-up** | Before user is created | Custom validation, deny sign-ups from specific domains |
| **Post-confirmation** | After user confirms account | Send welcome email, add to CRM |
| **Pre-authentication** | Before sign-in | Custom validation, block suspicious IPs |
| **Post-authentication** | After successful sign-in | Log login events, update last login time |
| **Custom message** | Before sending verification email/SMS | Customize email/SMS templates |
| **Define Auth Challenge** | Custom authentication | Implement CAPTCHA, custom MFA |

---

### **User Pool Groups**

Organize users into groups for role-based access control.

**Use Cases:**
- Admin group (higher privileges)
- Premium users (access premium features)
- Free tier users (limited access)

**Integration:** Groups can map to IAM roles (when combined with Identity Pools)

---

## 2. Identity Pools (Authorization)

**What It Is:** Grants temporary AWS credentials to users so they can access AWS resources directly (S3, DynamoDB, etc.).

### **How Identity Pools Work**

```
User authenticates (User Pool, Google, Facebook, or unauthenticated)
      ↓
Identity Pool verifies authentication
      ↓
Identity Pool assumes IAM role via STS
      ↓
Returns: Temporary AWS credentials (Access Key, Secret Key, Session Token)
      ↓
User accesses AWS resources directly (S3, DynamoDB, IoT, etc.)
```

**Key Point:** Identity Pools use **AWS STS (Security Token Service)** to provide temporary credentials.

---

### **Identity Providers Supported**

Identity Pools can authenticate users from multiple sources:

| Provider | Description | Use Case |
|----------|-------------|----------|
| **Cognito User Pool** | Authenticate via User Pool first | Full Cognito integration |
| **Social Providers** | Google, Facebook, Amazon, Apple (direct) | Social login |
| **SAML** | Enterprise identity providers | Corporate apps |
| **OIDC** | OpenID Connect providers | OAuth2 flows |
| **Unauthenticated** | Guest access (no login required) | Allow limited access for anonymous users |

---

### **Authenticated vs Unauthenticated Roles**

Identity Pools support two types of users:

| Type | Description | IAM Role |
|------|-------------|----------|
| **Authenticated** | Users who signed in (via User Pool, Google, Facebook, etc.) | Higher privileges (e.g., write to S3, read/write DynamoDB) |
| **Unauthenticated (Guest)** | Users who didn't sign in | Limited privileges (e.g., read-only access to public S3 bucket) |

**Example:**
- **Authenticated users:** Upload photos to S3, write to DynamoDB
- **Unauthenticated users:** View public content, read from public S3 bucket

---

### **IAM Roles and Policies**

Identity Pools assume IAM roles to grant permissions.

**Example Policy (Authenticated Role):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-user-uploads/${cognito-identity.amazonaws.com:sub}/*"
    }
  ]
}
```

**Key:** `${cognito-identity.amazonaws.com:sub}` = Unique user ID (ensures users can only access their own files)

---

### **Fine-Grained Access Control**

Use **policy variables** to restrict access per user:

**Example:** User can only access their own folder in S3
```
Resource: arn:aws:s3:::my-bucket/${cognito-identity.amazonaws.com:sub}/*
```

**Result:**
- User A can access `s3://my-bucket/user-a-id/`
- User B can access `s3://my-bucket/user-b-id/`
- Users cannot access each other's folders

---

## 3. User Pools + Identity Pools (Together)

**Most Common Pattern:** Use both together for full authentication and authorization.

### **Workflow**

```
1. User signs in via User Pool
   ↓ Returns JWT tokens
2. App sends JWT token to Identity Pool
   ↓ Identity Pool verifies token
3. Identity Pool assumes IAM role
   ↓ Returns temporary AWS credentials
4. User accesses AWS resources (S3, DynamoDB, etc.)
```

**Use Case:** Mobile app with user sign-in and AWS resource access

**Example:**
1. User signs up and signs in (User Pool)
2. App authenticates user, gets JWT token
3. App calls Identity Pool with JWT token
4. Identity Pool grants AWS credentials
5. User uploads profile picture to S3

---

## Common Use Cases

### **Use Case 1: Mobile App with User Sign-in**

**Requirements:**
- Users can sign up and sign in
- Users can upload photos to S3
- Each user can only access their own photos

**Solution:**
1. **User Pool:** Handles sign-up, sign-in, MFA
2. **Identity Pool:** Grants AWS credentials
3. **IAM Policy:** Restrict S3 access to user-specific folder
4. **S3 Bucket:** Stores user photos

---

### **Use Case 2: Social Login**

**Requirements:**
- Users sign in with Google or Facebook
- No separate registration required

**Solution:**
1. **User Pool:** Configure Google/Facebook as identity providers
2. **Identity Pool:** Accept User Pool tokens, grant AWS credentials

---

### **Use Case 3: Guest Access**

**Requirements:**
- Unauthenticated users can browse catalog
- Authenticated users can add to cart and checkout

**Solution:**
1. **Identity Pool:** Enable unauthenticated access
2. **IAM Roles:**
   - Unauthenticated: Read-only DynamoDB access (view catalog)
   - Authenticated: Read/write DynamoDB access (add to cart)

---

### **Use Case 4: Enterprise SSO**

**Requirements:**
- Employees sign in with corporate credentials (Active Directory)
- Access AWS resources based on their role

**Solution:**
1. **User Pool:** Federate with SAML identity provider (Azure AD, Okta)
2. **User Pool Groups:** Map AD groups to Cognito groups
3. **Identity Pool:** Map groups to IAM roles

---

## Security Best Practices

### **1. Use MFA**
- Enable MFA for sensitive applications
- SMS or TOTP (more secure)

### **2. Enforce Strong Password Policies**
- Minimum 8 characters
- Require uppercase, lowercase, numbers, symbols

### **3. Use HTTPS**
- All communication encrypted in transit

### **4. Rotate Refresh Tokens**
- Set expiration for refresh tokens
- Force re-authentication periodically

### **5. Use Fine-Grained Access Control**
- Policy variables to restrict access per user
- Principle of least privilege

### **6. Monitor with CloudTrail**
- Track Cognito API calls
- Detect suspicious activity

---

## Exam Scenarios

| Scenario | Solution |
|----------|----------|
| "Mobile app needs user sign-up and sign-in" | **User Pools** |
| "Users upload files to S3 from mobile app" | **Identity Pools** (grants AWS credentials) |
| "Social login with Google and Facebook" | **User Pools** (federated identities) |
| "Web app users directly access DynamoDB" | **Identity Pools** |
| "Guest users can browse, authenticated users can purchase" | **Identity Pools** (authenticated + unauthenticated roles) |
| "Enterprise employees sign in with Active Directory" | **User Pools** (SAML federation) |
| "Users can only access their own S3 folder" | **Identity Pools** with policy variables (`${cognito-identity.amazonaws.com:sub}`) |
| "Add MFA to login flow" | **User Pools** (enable MFA) |
| "Custom authentication logic (CAPTCHA, custom validation)" | **User Pools** with **Lambda triggers** |
| "Mobile app with sign-in AND AWS resource access" | **User Pools + Identity Pools** (together) |

---

## User Pools vs Identity Pools (Detailed)

### **User Pools**

| Feature | Details |
|---------|---------|
| **Purpose** | User directory + authentication |
| **Returns** | JWT tokens (ID, Access, Refresh) |
| **Standalone** | ✅ Yes (use without Identity Pools) |
| **Authenticates Against** | User Pool database, Google, Facebook, SAML, OIDC |
| **Provides** | User profile, MFA, password policies, Lambda triggers |
| **Use Case** | Sign-up/sign-in for your app |

**Example Flow:**
```
User → Sign in → User Pool → JWT Token → App uses token to call API Gateway
```

---

### **Identity Pools**

| Feature | Details |
|---------|---------|
| **Purpose** | Grant temporary AWS credentials |
| **Returns** | AWS credentials (Access Key, Secret Key, Session Token) |
| **Standalone** | ✅ Yes (can use Google/Facebook directly, or unauthenticated) |
| **Authenticates Against** | User Pool, Google, Facebook, SAML, OIDC, or unauthenticated |
| **Provides** | Temporary AWS credentials (via STS AssumeRole) |
| **Use Case** | Access AWS resources from app (S3, DynamoDB, IoT) |

**Example Flow:**
```
User → Authenticate → Identity Pool → AWS Credentials → User uploads to S3
```

---

## Key Exam Tips

1. **User Pools = Authentication** ("Who are you?") → Returns JWT tokens
2. **Identity Pools = Authorization** ("What can you access?") → Returns AWS credentials
3. **"Sign-in for mobile app"** → User Pools
4. **"Upload to S3 from mobile app"** → Identity Pools
5. **"Social login"** → User Pools (federated identities)
6. **"Guest access"** → Identity Pools (unauthenticated role)
7. **User Pools + Identity Pools:** Most common pattern (authenticate first, then get AWS credentials)
8. **Fine-grained access:** Use policy variables (`${cognito-identity.amazonaws.com:sub}`)
9. **MFA, password policies, Lambda triggers:** User Pools
10. **Temporary credentials:** Identity Pools (uses STS AssumeRole)
11. **Unauthenticated access:** Identity Pools only (not User Pools)
12. **SAML/OIDC federation:** User Pools
13. **Groups:** User Pools (can map to IAM roles via Identity Pools)
14. **Hosted UI:** User Pools (pre-built sign-in page)
15. **JWT tokens:** User Pools (ID token, Access token, Refresh token)

---

## Analogy (Easy to Remember)

**Scenario:** You're going to a concert venue.

**User Pool (Authentication):**
- The **ticket booth** where you prove who you are
- Shows your ID (username/password, Google, Facebook)
- Gets a **wristband** (JWT token) that proves you're allowed in

**Identity Pool (Authorization):**
- The **VIP area attendant** who checks your wristband
- Gives you a **special pass** (AWS credentials) to access VIP areas
- The pass has **specific permissions** (you can access lounge, but not backstage)

**Together:**
1. Get wristband from ticket booth (User Pool → JWT token)
2. Show wristband to VIP attendant (Identity Pool verifies JWT)
3. Get special pass (Identity Pool → AWS credentials)
4. Access VIP lounge (User accesses S3, DynamoDB)

---

## Summary

**Cognito** provides two services for managing application users:

| Service | Purpose | Returns | Use Case |
|---------|---------|---------|----------|
| **User Pools** | Authentication (who are you?) | JWT tokens | Sign-up/sign-in for your app |
| **Identity Pools** | Authorization (what can you access?) | AWS credentials | Access AWS resources from app |

**Bottom Line:**
- **User Pools:** Manage users, handle sign-in, return JWT tokens
- **Identity Pools:** Grant temporary AWS credentials, enable direct AWS resource access
- **Together:** Complete solution for authenticated users accessing AWS resources

**Exam Strategy:** If the question mentions "sign-in" or "authentication" → User Pools. If it mentions "access S3" or "AWS credentials" → Identity Pools.
