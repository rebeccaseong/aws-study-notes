- **What is it? (1-Sentence Pitch):** A fully managed service that acts as a "front door" for applications to access data, business logic, or functionality from your backend services (like Lambda or EC2).
- **The Three Types of APIs**
    
    
    | **API Type** |  | **Use Case / Description** | **Key Characteristics** |
    | --- | --- | --- | --- |
    | **REST API** | **Stateless**
     | The traditional, feature-rich choice for building RESTful APIs. | Offers the most features: API keys, per-client throttling, request validation, caching, etc. The most flexible option. |
    | **HTTP API** | **Stateless** | A newer, lighter-weight, and cheaper alternative to REST APIs. | **Lower cost & lower latency** than REST APIs, but with a reduced feature set. Ideal for simple proxying to Lambda or HTTP backends. |
    | **WebSocket API** | **Stateful** | For building real-time, two-way communication applications. | Stateful connections (unlike REST/HTTP). Used for chat apps, real-time dashboards, and streaming updates. |
    - **Stateless VS Stateful**
        
        
        | **Characteristic** | **Stateless (Vending Machine)** | **Stateful (Barista)** |
        | --- | --- | --- |
        | **Server Memory** | The server forgets the client after every request.
        Every single request from the client must contain all the information the server needs to fulfill it (like authentication tokens, what data is being requested, etc.). | The server remembers the client's state between requests.
        The history of previous requests can influence how the current request is handled. |
        | **Client's Job** | Must send a complete, self-contained request every time. | Can send simpler requests that rely on stored context. |
        | **Scalability** | **Easy to scale.** Any request can go to any server, making it simple to add more servers. | **Harder to scale.** If a client's state is on Server A, all their future requests must go to Server A ("stickiness"). |
        | **Resilience** | **High.** If a server fails, the client's request can just be sent to another identical server. | **Low.** If the server holding the state fails, that state (and the session) is lost. |
        | **Primary Example** | **REST APIs (HTTP)** | **WebSocket APIs**, RDP/SSH sessions, FTP connections. |
- **Core API Gateway Concepts (The Building Blocks)**
    - **API Endpoint:** The public-facing URL of your API (e.g., https://xyz.execute-api.us-east-1.amazonaws.com/prod).
    - **Resource:** A path component of your API's URL (e.g., /users or /products/{productID}).
    - **Method:** The HTTP verb associated with a resource (e.g., GET, POST, DELETE).
    - **Integration:** The backend service that API Gateway calls to fulfill the request. This is the "workhorse."
    - **Deployment Stage:** A snapshot of your API configuration, like a version (e.g., dev, test, v1, prod). You can have different settings (like throttling or logging) for each stage.
- **Security & Authorization (CRITICAL KNOWLEDGE)**
    
    
    | **Authorization Method** | **How it Works** | **Common Use Case** |
    | --- | --- | --- |
    | **IAM Roles & Policies** | The request is signed with AWS credentials (Access/Secret Keys). API Gateway validates the signature and checks IAM permissions. | **Internal traffic** within AWS. Securing communication between different AWS services (e.g., EC2 instance calling an API). |
    | **Cognito User Pools** | The user signs into your application via Cognito. Cognito returns a JWT token, which is included in the API call's Authorization header. | **Securing APIs for your own web/mobile application users.** You manage the users, Cognito handles the authentication. |
    | **Lambda Authorizer** (formerly Custom Authorizer) | The **most flexible** option. API Gateway invokes a Lambda function you write. This function receives the request's token (e.g., OAuth, SAML, JWT) and returns an IAM policy document (Allow or Deny). | Integrating with third-party identity providers (Auth0, Okta), implementing custom authentication logic, or validating bearer tokens. |
    | **API Keys** | A simple string (key) that is passed in the x-api-key header. Used for **identification and throttling**, not strong authentication. | Granting API access to third-party developers or customers. Used with **Usage Plans**. |
- **Endpoint Types (Deployment Strategy)**
    - **Edge-Optimized (Default):** The API is deployed to the global CloudFront network. Best for **geographically distributed users** as it provides the lowest latency for clients around the world.
    - **Regional:** The API is deployed only within the specified AWS Region. Best for clients that are also in the **same region**, as it avoids the extra hop to a CloudFront edge location.
    - **Private:** The API is only accessible from within your Amazon VPC via a **VPC Endpoint (VPCE)**. Used for building secure, internal microservices that are not exposed to the public internet.
- **Integration Types (The Backend Connection)**
    - **Lambda Function (Proxy Integration):** The most common serverless pattern. API Gateway passes the entire request to a Lambda function and returns the function's response.
    - **HTTP (Proxy Integration):** Forwards the request to another public HTTP endpoint (e.g., an application running on EC2 behind an ALB, or a third-party API).
    - **AWS Service Integration:** Allows API Gateway to directly call other AWS service APIs without needing a Lambda function in between (e.g., directly putting a message into an SQS queue or reading an item from DynamoDB).
    - **VPC Link:** The secure way to connect a **Private API** or a **Regional API** to private resources (like an Application Load Balancer or an ECS service) inside your VPC **without** exposing those resources to the internet.
- **Performance & Management**
    - **Throttling & Usage Plans:**
        - **Throttling:** Protects your backend from being overwhelmed. You can set a **rate** (requests per second) and a **burst** limit.
        - **Usage Plans & API Keys:** The mechanism for monetizing your API or managing third-party access. You create a usage plan (e.g., "Gold Tier" with 100 req/sec and a 10,000 request/month quota) and associate API Keys with that plan.
    - **Caching:** API Gateway can cache responses at the edge for a configurable Time-to-Live (TTL). This improves performance and reduces the number of calls to your backend. Caching is configured per-stage.
    - **CORS (Cross-Origin Resource Sharing):** A browser security feature. If your API is called from a web browser hosted on a different domain, you must enable CORS. API Gateway has a simple "Enable CORS" button that sets up the required OPTIONS pre-flight method and response headers.
- **The Big Comparison: API Gateway vs. Application Load Balancer (ALB)**
    
    
    | **Feature** | **Amazon API Gateway** | **Application Load Balancer (ALB)** |
    | --- | --- | --- |
    | **Primary Job** | **Building & Managing APIs** | **Load Balancing** HTTP/HTTPS traffic |
    | **Key Features** | Authentication (IAM, Cognito, Lambda), Throttling, Caching, API Keys, Staging. | Routing based on path/host, health checks, sticky sessions. |
    | **Backend Targets** | Lambda, HTTP endpoints, AWS Services. | EC2, ECS, Fargate, Lambda, IP Addresses. |
    | **Model** | **Fully Serverless**. Pay-per-call. | You manage the ALB, but backend targets (like EC2) require provisioning. |