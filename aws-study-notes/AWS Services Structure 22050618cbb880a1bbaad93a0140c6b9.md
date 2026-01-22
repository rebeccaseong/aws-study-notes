# AWS Services Structure

- AWS Service Outline
    - 0: Concepts: Availability Zone and Region
    - **1: Compute:**
        - **EC2 (Elastic Compute Cloud):** Instances, AMIs, User Data, Spot vs. On-Demand vs. Reserved.
        - **Auto Scaling Groups (ASG):** Scaling policies (Target Tracking), Launch Templates.
        - **Containers:**
            - **ECS (Elastic Container Service):** Docker management.
            - **EKS (Elastic Kubernetes Service):** Kubernetes management.
            - **AWS Fargate:** Serverless compute for containers (Exam favorite).
        - **Serverless:**
            - **Lambda:** Event-driven code, limits (15 min), Cold starts.
            - **Lambda@Edge:** Run Lambda at CloudFront edge locations.
        - **Other:**
            - **Elastic Beanstalk:** PaaS (Platform as a Service), easy deployment.
            - **AWS Batch:** Batch computing (HPC) using Spot instances.
            - **AWS Outposts:** Run AWS infrastructure on-premises.
            - **Local Zones:** Extend AWS regions closer to end users.
            - **Wavelength:** Ultra-low latency for 5G devices.
    - **2: Storage & File Systems:**
        - Object Storage:
            - S3: Standard, Intelligent-Tiering, Glacier (Archive), Lifecycle Policies, Versioning.
        - Block Storage:
            - EBS: Persistent, Network-attached (gp3, io2).
            - Instance Store: Ephemeral, High Random I/O, locally attached.
        - File Storage:
            - EFS: Shared Linux (NFS), cross-AZ.
            - FSx: Windows (SMB), Lustre (HPC), NetApp ONTAP.
        - Hybrid Storage:
            - Storage Gateway: File (S3), Volume (iSCSI), Tape (VTL).
    - **3: Databases, Caching & Analytics:**
        - **Relational Databases (SQL):**
            - **RDS:** Managed Postgres/MySQL/etc., Multi-AZ (Disaster Recovery) vs. Read Replicas (Performance).
            - **Aurora:** Cloud-native, high performance, Serverless v2, Global Database.
        - **NoSQL Databases:**
            - **DynamoDB:** Key-value, single-digit millisecond latency, DAX, Streams, Global Tables.
        - **Caching:**
            - **ElastiCache:** Redis (complex data structures) vs. Memcached (simple).
            - **DAX:** Caching specifically for DynamoDB.
        - Batch Analytics (Big Data):
            - **Redshift:** Data Warehouse, Columnar storage (OLAP).
            - **EMR:** Hadoop/Spark clusters, high maintenance.
            - **Glue:** Serverless ETL, Data Catalog.
            - **Athena:** Serverless SQL on S3 files.
            - **OpenSearch (formerly Elasticsearch):** Log analytics, search, dashboards.
        - Streaming Analytics (Real-Time):
            - Kinesis Data Streams: Real-time ingestion, requires shard management ("The Pipe").
            - Kinesis Data Firehose: Near real-time delivery to S3/Redshift, zero admin, supports Lambda transformation ("The Delivery Truck").
            - Kinesis Data Analytics: SQL/Flink analysis *inside* the stream.
            - The Kinesis Cheat Sheet: Which one to pick?
            - Amazon MSK: Managed Kafka (alternative to Kinesis Data Streams).
    - **4: Networking & Content Delivery:**
        - **VPC Fundamentals:** CIDR, Subnets (Public vs. Private), Route Tables, Internet Gateway (IGW).
        - Security Groups vs NACLs (Critical Comparison)
        - **Connectivity:**
            - **NAT Gateway:** Allows private subnets to talk to the internet.
            - **VPC Endpoints:** PrivateLink (Interface) vs. Gateway (S3/DynamoDB).
            - **VPC Peering:** Connect two VPCs.
            - **Transit Gateway:** Hub-and-spoke topology for many VPCs.
            - **VPN vs. Direct Connect:** Public internet encrypted tunnel vs. Private physical fiber.
        - **Delivery & Traffic:**
            - **Route 53:** DNS records (A, CNAME, Alias), Routing Policies (Failover, Latency, Geolocation).
            - **CloudFront:** CDN (Content Delivery Network), caching at Edge.
            - **Global Accelerator:** Static IP, improves performance via AWS backbone (not caching).
            - **Load Balancing:** ALB (Layer 7/HTTP), NLB (Layer 4/TCP/UDP), GWLB (Security appliances).
    - **5: Security, Identity & Compliance:**
        - **Identity:**
            - **IAM:** Users, Roles, Policies, MFA.
            - **Cognito:** Identity for Mobile/Web Apps (User Pools = Auth, Identity Pools = AWS Access).
            - **IAM Identity Center (SSO):** Centralized login for organizations.
            - **STS (Security Token Service):** Temporary credentials, AssumeRole.
        - **Data Protection:**
            - **KMS:** Encryption keys (managed).
            - **CloudHSM:** Dedicated hardware (compliance).
            - **Secrets Manager:** Rotate DB credentials automatically.
            - **Systems Manager Parameter Store:** Store config/secrets (free tier).
            - **AWS Certificate Manager (ACM):** Free SSL/TLS certs for ALB, CloudFront, API Gateway.
        - **Protection Services:**
            - **Shield:** DDoS protection (Standard vs. Advanced).
            - **WAF:** Web Application Firewall (Block SQL injection/XSS).
            - **GuardDuty:** Intelligent threat detection (logs analysis).
            - **Macie:** PII/Sensitive data discovery in S3.
            - **Inspector:** EC2 vulnerability scanning.
            - **AWS Firewall Manager:** Centrally manage WAF/Shield across accounts.
    - **6: Monitoring, Management & Governance:**
        - **Monitoring:**
            - **CloudWatch:** Metrics, Alarms, Logs (Performance).
            - **X-Ray:** Tracing and debugging distributed apps/Lambda.
        - **Audit & Governance:**
            - **CloudTrail:** "Who did what?" (API Auditing).
            - **AWS Config:** "What does my infrastructure look like?" (Compliance/Rules).
            - **Trusted Advisor:** Best practice checklist.
        - **Management:**
            - **CloudFormation:** Infrastructure as Code (JSON/YAML).
            - **Systems Manager (SSM):** Patch Manager, Session Manager (No SSH needed).
            - **Organizations:** SCPs (Service Control Policies), Consolidated Billing.
            - **Cost Explorer & Compute Optimizer:** Saving money.
            - **AWS Budgets:** Set alerts when costs exceed thresholds.
    - **7: Migration & Transfer:**
        - **Migration:**
            - **DMS (Database Migration Service):** Move DBs while keeping source live.
            - **MGN (Application Migration Service):** Lift-and-shift servers.
            - **Schema Conversion Tool (SCT):** Convert Oracle to Aurora.
        - **Transfer:**
            - **Snow Family:** Physical devices (Snowcone, Snowball Edge, Snowmobile) for massive data.
            - **DataSync:** Automated data transfer (NAS to S3).
            - **Transfer Family:** FTP/SFTP to S3.
    - **8: Application Integration:**
        - **Messaging:**
            - **SQS:** Decoupling, Queueing (Standard vs. FIFO).
            - **SNS:** Pub/Sub, Notifications to Email/SMS/Lambda.
            - **Amazon MQ:** Broker for industry standards (MQTT, AMQP) - "Lift and shift" legacy apps.
        - **Events & Workflow:**
            - **EventBridge:** Serverless Event Bus, Rules, Scheduler.
            - **Step Functions:** Visual workflow orchestration (State Machine).
            - **API Gateway:** REST/HTTP APIs, Throttling, API Keys.
    - **9: Developer Tools:**
        - **CI/CD Pipeline:**
            - **CodeCommit:** Git repo.
            - **CodeBuild:** Build and test.
            - **CodeDeploy:** Deploy to EC2/Lambda.
            - **CodePipeline:** Orchestrate the flow.
    
- AWS Service Outline Detailed
    - 0: Concepts:
        
        [Availability Zone and Region](AWS%20Services%20Structure/Availability%20Zone%20and%20Region%2024b50618cbb880c4b68df386643c872d.md)
        
    - **1: Compute:**
        
        [ **EC2 (Elastic Compute Cloud):** Instances, AMIs, User Data, Spot vs. On-Demand vs. Reserved.  ](AWS%20Services%20Structure/EC2%20(Elastic%20Compute%20Cloud)%20Instances,%20AMIs,%20User%20%2022f50618cbb88044b80fec4b587a9322.md)
        
        - **Auto Scaling Groups (ASG):** Scaling policies (Target Tracking), Launch Templates.
        - **Containers:**
            
            [ECS **(Elastic Container Service):** Docker management.](AWS%20Services%20Structure/ECS%20(Elastic%20Container%20Service)%20Docker%20management%2022f50618cbb880448cfacce65da81644.md)
            
            [**EKS (Elastic Kubernetes Service):** Kubernetes management.](AWS%20Services%20Structure/EKS%20(Elastic%20Kubernetes%20Service)%20Kubernetes%20manage%2022f50618cbb880958ca5d185bf5db690.md)
            
            - **Fargate:** Serverless compute for containers (Exam favorite).
        - **Serverless:**
            
            [Lambda**:** Event-driven code, limits (15 min), Cold starts.](AWS%20Services%20Structure/Lambda%20Event-driven%20code,%20limits%20(15%20min),%20Cold%20st%2022f50618cbb8800988acfdd20bbdab77.md)
            
            - **Lambda@Edge:** Run Lambda at CloudFront edge locations.
        - **Other:**
            
            [Elastic Beanstalk**:** PaaS (Platform as a Service), easy deployment.](AWS%20Services%20Structure/Elastic%20Beanstalk%20PaaS%20(Platform%20as%20a%20Service),%20ea%2022f50618cbb8808e8b77e593ea8e2923.md)
            
            - **AWS Batch:** Batch computing (HPC) using Spot instances.
            - **AWS Outposts:** Run AWS infrastructure on-premises.
            - **Local Zones:** Extend AWS regions closer to end users.
            - **Wavelength:** Ultra-low latency for 5G devices.
    - **2: Storage & File Systems:**
        - Object Storage:
            
            [**S3 (Simple Storage Service)**: Standard, Intelligent-Tiering, Glacier (Archive), Lifecycle Policies, Versioning.](AWS%20Services%20Structure/S3%20(Simple%20Storage%20Service)%20Standard,%20Intelligent-%2022f50618cbb88067a645eb9f0cf92d5a.md)
            
        - Block Storage:
            
            [**EBS (Elastic Block Store)**: Persistent, Network-attached (gp3, io2).](AWS%20Services%20Structure/EBS%20(Elastic%20Block%20Store)%20Persistent,%20Network-atta%2022f50618cbb88094b407c36ca4528f28.md)
            
            - Instance Store: Ephemeral, High Random I/O, locally attached.
        - File Storage:
            
            [**EFS (Elastic File System)** — *Linux:* : Shared Linux (NFS), cross-AZ.](AWS%20Services%20Structure/EFS%20(Elastic%20File%20System)%20%E2%80%94%20Linux%20Shared%20Linux%20(NF%2022f50618cbb88007ac88f387571fdeb3.md)
            
            [**FSx (File System for X)** — *Windows:* : Windows (SMB), Lustre (HPC), NetApp ONTAP.](AWS%20Services%20Structure/FSx%20(File%20System%20for%20X)%20%E2%80%94%20Windows%20Windows%20(SMB),%20L%2022f50618cbb880a18478ce94efa996cb.md)
            
        - Hybrid Storage:
            
            [**Storage Gateway**: File (S3), Volume (iSCSI), Tape (VTL).](AWS%20Services%20Structure/Storage%20Gateway%20File%20(S3),%20Volume%20(iSCSI),%20Tape%20(V%202eb50618cbb880e8acc0ea341241026b.md)
            
        - **Storage Cheat Sheet for SAA Exam**
            
            
            | **If the question mentions...** | **The answer is likely...** |
            | --- | --- |
            | Storing files, static website hosting, infinite scale | **S3** |
            | The "hard drive" or boot volume for a single EC2 instance | **EBS** |
            | A shared network drive for many **Linux** EC2 instances | **EFS** (NFS) |
            | A shared network drive for many **Windows** EC2 instances | **FSx for Windows** (SMB) |
            | High-performance computing (HPC) or video rendering | **FSx for Lustre** |
            | Long-term, low-cost data archival | **S3 Glacier** |
            | Extreme performance (High Random I/O) or "Ephemeral" storage | Instance Store |
            | Migrating on-premise VMware/Hyper-V disks to AWS | Storage Gateway (Volume) |
            | Replacing physical tape backups | Storage Gateway (Tape) |
    - **3: Databases, Caching & Analytics:**
        - **Relational Databases (SQL):**
            
            [**RDS (Relational Database Service):** Managed Postgres/MySQL/etc., Multi-AZ (Disaster Recovery) vs. Read Replicas (Performance).](AWS%20Services%20Structure/RDS%20(Relational%20Database%20Service)%20Managed%20Postgres%2022f50618cbb880b0a040ec511f10a521.md)
            
            [**Aurora:** Cloud-native, high performance, Serverless v2, Global Database.](AWS%20Services%20Structure/Aurora%20Cloud-native,%20high%20performance,%20Serverless%20%2022f50618cbb8802ba6c3e1b40d5935fe.md)
            
        - **NoSQL Databases:**
            
            [**DynamoDB:** Key-value, single-digit millisecond latency, DAX, Streams, Global Tables.](AWS%20Services%20Structure/DynamoDB%20Key-value,%20single-digit%20millisecond%20laten%2022f50618cbb880d286e8df4df3fbe8e6.md)
            
        - **Caching:**
            
            [**ElastiCache:** Redis (complex data structures) vs. Memcached (simple).](AWS%20Services%20Structure/ElastiCache%20Redis%20(complex%20data%20structures)%20vs%20Mem%2022f50618cbb88040a00dfdaa236b9d40.md)
            
            - **DAX:** Caching specifically for DynamoDB.
        - Batch Analytics (Big Data):
            
            [**Redshift:** Data Warehouse, Columnar storage (OLAP).](AWS%20Services%20Structure/Redshift%20Data%20Warehouse,%20Columnar%20storage%20(OLAP)%2022f50618cbb880e8912efc4b32478752.md)
            
            [EMR (Elastic MapReduce)**:** Hadoop/Spark clusters, high maintenance.](AWS%20Services%20Structure/EMR%20(Elastic%20MapReduce)%20Hadoop%20Spark%20clusters,%20hig%2024750618cbb880878975ef2e84bc19b0.md)
            
            [Glue**:** Serverless ETL, Data Catalog.](AWS%20Services%20Structure/Glue%20Serverless%20ETL,%20Data%20Catalog%2024750618cbb880588644ea84aff3e0ac.md)
            
            [**Athena: Serverless SQL on S3 files.**](AWS%20Services%20Structure/Athena%20Serverless%20SQL%20on%20S3%20files%2022f50618cbb8802da308c6b35912a0ed.md)
            
            - **OpenSearch (formerly Elasticsearch):** Log analytics, search, dashboards.
        - Streaming Analytics (Real-Time):
            
            [**Kinesis Data Streams**: Real-time ingestion, requires shard management ("The Pipe").](AWS%20Services%20Structure/Kinesis%20Data%20Streams%20Real-time%20ingestion,%20requires%2023150618cbb880988df6df4f290ff205.md)
            
            [**Kinesis Data Firehose**: Near real-time delivery to S3/Redshift, zero admin, supports Lambda transformation ("The Delivery Truck").](AWS%20Services%20Structure/Kinesis%20Data%20Firehose%20Near%20real-time%20delivery%20to%20S%2024b50618cbb8809c8db4ee3d5f43c124.md)
            
            [**Kinesis Data Analytics**: SQL/Flink analysis *inside* the stream.](AWS%20Services%20Structure/Kinesis%20Data%20Analytics%20SQL%20Flink%20analysis%20inside%20t%202eb50618cbb880828482d35ebc22c4fd.md)
            
            [The Kinesis Cheat Sheet: Which one to pick?](AWS%20Services%20Structure/The%20Kinesis%20Cheat%20Sheet%20Which%20one%20to%20pick%202eb50618cbb8808d892acbf001bb6681.md)
            
            - Amazon MSK: Managed Kafka (alternative to Kinesis Data Streams).
    - **4: Networking & Content Delivery:**
        
        [VPC **(Virtual Private Cloud) Fundamentals:** CIDR, Subnets (Public vs. Private), Route Tables, Internet Gateway (IGW).](AWS%20Services%20Structure/VPC%20(Virtual%20Private%20Cloud)%20Fundamentals%20CIDR,%20Sub%2022f50618cbb880009ddac683ccb779f3.md)
        
        - Security Groups vs NACLs (Critical Comparison)
        - **Connectivity:**
            - **NAT Gateway:** Allows private subnets to talk to the internet.
            - **VPC Endpoints:** PrivateLink (Interface) vs. Gateway (S3/DynamoDB).
            - **VPC Peering:** Connect two VPCs.
            - **Transit Gateway:** Hub-and-spoke topology for many VPCs.
            
            [**VPN vs. Direct Connect:** Public internet encrypted tunnel vs. Private physical fiber.](AWS%20Services%20Structure/VPN%20vs%20Direct%20Connect%20Public%20internet%20encrypted%20tu%2022f50618cbb8805da645ef3fd12792de.md)
            
        - **Delivery & Traffic:**
            
            [Route 53**:** DNS records (A, CNAME, Alias), Routing Policies (Failover, Latency, Geolocation).](AWS%20Services%20Structure/Route%2053%20DNS%20records%20(A,%20CNAME,%20Alias),%20Routing%20Po%2022f50618cbb8809f9edde4cd712f6922.md)
            
            [CloudFront**:** CDN (Content Delivery Network), caching at Edge.](AWS%20Services%20Structure/CloudFront%20CDN%20(Content%20Delivery%20Network),%20caching%2022f50618cbb8803d9686e8929d9aeda0.md)
            
            - **Global Accelerator:** Static IP, improves performance via AWS backbone (not caching).
            
            [**Elastic Load Balancing (ELB)**](AWS%20Services%20Structure/Elastic%20Load%20Balancing%20(ELB)%2022f50618cbb88087acece63fad193bbd.md)
            
            - **Load Balancing:** ALB (Layer 7/HTTP), NLB (Layer 4/TCP/UDP), GWLB (Security appliances).
    - **5: Security, Identity & Compliance:**
        - **Identity:**
            
            [IAM **(Identity and Access Management):** Users, Roles, Policies, MFA.](AWS%20Services%20Structure/IAM%20(Identity%20and%20Access%20Management)%20Users,%20Roles,%2022f50618cbb88089802ecc57ec943a97.md)
            
            [Cognito**:** Identity for Mobile/Web Apps (User Pools = Auth, Identity Pools = AWS Access).](AWS%20Services%20Structure/Cognito%20Identity%20for%20Mobile%20Web%20Apps%20(User%20Pools%20=%2022f50618cbb88012846cf235eceb6b83.md)
            
            - **IAM Identity Center (SSO):** Centralized login for organizations.
            - **STS (Security Token Service):** Temporary credentials, AssumeRole.
        - **Data Protection:**
            
            [KMS **(Key Management Service): Encryption keys (managed).**](AWS%20Services%20Structure/KMS%20(Key%20Management%20Service)%20Encryption%20keys%20(mana%2022f50618cbb8801a9257f2ef51b8b865.md)
            
            [CloudHSM **(Hardware Security Module):** Dedicated hardware (compliance).](AWS%20Services%20Structure/CloudHSM%20(Hardware%20Security%20Module)%20Dedicated%20hard%2022f50618cbb8809bac07e5ca4a97de64.md)
            
            [**Secrets Manager:** Rotate DB credentials automatically.](AWS%20Services%20Structure/Secrets%20Manager%20Rotate%20DB%20credentials%20automaticall%2022f50618cbb88003922be9fe534ebb9e.md)
            
            - **Systems Manager Parameter Store:** Store config/secrets (free tier).
            - **AWS Certificate Manager (ACM):** Free SSL/TLS certs for ALB, CloudFront, API Gateway.
        - **Protection Services:**
            
            [Shield: DDoS protection (Standard vs. Advanced).](AWS%20Services%20Structure/Shield%20DDoS%20protection%20(Standard%20vs%20Advanced)%2022f50618cbb8801fbf18d83b7d6c93d4.md)
            
            [WAF **(Web Application Firewall)**(Block SQL injection/XSS)](AWS%20Services%20Structure/WAF%20(Web%20Application%20Firewall)(Block%20SQL%20injection%2022f50618cbb880b48efadab947be6248.md)
            
            [GuardDuty**:** Intelligent threat detection (logs analysis).](AWS%20Services%20Structure/GuardDuty%20Intelligent%20threat%20detection%20(logs%20analy%2022f50618cbb88025a7caf4fc8e17357a.md)
            
            [**Macie:** PII/Sensitive data discovery in S3.](AWS%20Services%20Structure/Macie%20PII%20Sensitive%20data%20discovery%20in%20S3%2022f50618cbb880c3b114f7bb4bee8a12.md)
            
            - **Inspector:** EC2 vulnerability scanning.
            - **AWS Firewall Manager:** Centrally manage WAF/Shield across accounts.
    - **6: Monitoring, Management & Governance:**
        - **Monitoring:**
            
            [CloudWatch**:** Metrics, Alarms, Logs (Performance).](AWS%20Services%20Structure/CloudWatch%20Metrics,%20Alarms,%20Logs%20(Performance)%2022f50618cbb8804e8580c166bc78d7e9.md)
            
            - **X-Ray:** Tracing and debugging distributed apps/Lambda.
        - **Audit & Governance:**
            
            [CloudTrail**:** "Who did what?" (API Auditing).](AWS%20Services%20Structure/CloudTrail%20Who%20did%20what%20(API%20Auditing)%2022f50618cbb8808ebf65eb32b3da1f7c.md)
            
            [Config**:** "What does my infrastructure look like?" (Compliance/Rules).](AWS%20Services%20Structure/Config%20What%20does%20my%20infrastructure%20look%20like%20(Comp%2022f50618cbb880c8b456d34fc04e0b71.md)
            
            [Trusted Advisor**:** Best practice checklist.](AWS%20Services%20Structure/Trusted%20Advisor%20Best%20practice%20checklist%2022f50618cbb880bbb7caf2e42ebaf11c.md)
            
        - **Management:**
            
            [**CloudFormation:** Infrastructure as Code (JSON/YAML).](AWS%20Services%20Structure/CloudFormation%20Infrastructure%20as%20Code%20(JSON%20YAML)%2022f50618cbb880eeba29fb3cbc7d5076.md)
            
            [**Systems Manager (SSM):** Patch Manager, Session Manager (No SSH needed).](AWS%20Services%20Structure/Systems%20Manager%20(SSM)%20Patch%20Manager,%20Session%20Manag%2022f50618cbb88003a1a3c338838836bd.md)
            
            [**Organizations & Control Tower:** SCPs (Service Control Policies), Consolidated Billing.](AWS%20Services%20Structure/Organizations%20&%20Control%20Tower%20SCPs%20(Service%20Contro%2022f50618cbb8800fa96cf72ac5a11db6.md)
            
            - **Cost Explorer & Compute Optimizer:** Saving money.
            - **AWS Budgets:** Set alerts when costs exceed thresholds.
    - **7: Migration & Transfer:**
        - **Migration:**
            
            [DMS **(Database Migration Service):** Move DBs while keeping source live.](AWS%20Services%20Structure/DMS%20(Database%20Migration%20Service)%20Move%20DBs%20while%20ke%2022f50618cbb880ba8820ef2d5682fc5e.md)
            
            [**MGN (Application Migration Service):** Lift-and-shift servers.](AWS%20Services%20Structure/MGN%20(Application%20Migration%20Service)%20Lift-and-shift%2022f50618cbb880dfade1dd0e63f71fcb.md)
            
            - **Schema Conversion Tool (SCT):** Convert Oracle to Aurora.
        - **Transfer:**
            - **Snow Family:** Physical devices (Snowcone, Snowball Edge, Snowmobile) for massive data.
            
            [DataSync**:** Automated data transfer (NAS to S3).](AWS%20Services%20Structure/DataSync%20Automated%20data%20transfer%20(NAS%20to%20S3)%2022f50618cbb880279c65e826f4bf6659.md)
            
            [Transfer Family**:** FTP/SFTP to S3.](AWS%20Services%20Structure/Transfer%20Family%20FTP%20SFTP%20to%20S3%2022f50618cbb880d29833c2f4a54b0df8.md)
            
            [Snowball](AWS%20Services%20Structure/Snowball%2022f50618cbb8801d963edf9fe4c62966.md)
            
            [Storage Gateway](AWS%20Services%20Structure/Storage%20Gateway%2022f50618cbb8804d8291cb5db9949e66.md)
            
    - **8: Application Integration:**
        - **Messaging:**
            
            [SQS **(Simple Queue Service):** Decoupling, Queueing (Standard vs. FIFO).](AWS%20Services%20Structure/SQS%20(Simple%20Queue%20Service)%20Decoupling,%20Queueing%20(S%2022f50618cbb88087abbaf358d2fa6fe1.md)
            
            [SNS **(Simple Notification Service):** Pub/Sub, Notifications to Email/SMS/Lambda.](AWS%20Services%20Structure/SNS%20(Simple%20Notification%20Service)%20Pub%20Sub,%20Notific%2022f50618cbb880a19ad0c1573652a066.md)
            
            - **Amazon MQ:** Broker for industry standards (MQTT, AMQP) - "Lift and shift" legacy apps.
        - **Events & Workflow:**
            
            [EventBridge**:** Serverless Event Bus, Rules, Scheduler.](AWS%20Services%20Structure/EventBridge%20Serverless%20Event%20Bus,%20Rules,%20Scheduler%2022f50618cbb88008981ed3264b8965f6.md)
            
            [Step Functions**:** Visual workflow orchestration (State Machine).](AWS%20Services%20Structure/Step%20Functions%20Visual%20workflow%20orchestration%20(Stat%2022f50618cbb88018956eebd11475b339.md)
            
            [SWF **(Simple Workflow Service)**](AWS%20Services%20Structure/SWF%20(Simple%20Workflow%20Service)%2022f50618cbb880688420c2233121fd1f.md)
            
            [API Gateway**:** REST/HTTP APIs, Throttling, API Keys.](AWS%20Services%20Structure/API%20Gateway%20REST%20HTTP%20APIs,%20Throttling,%20API%20Keys%2022f50618cbb880b58b13f3f261176032.md)
            
    - **9: Developer Tools:**
        - **CI/CD Pipeline:**
            
            [CodeCommit**:** Git repo.](AWS%20Services%20Structure/CodeCommit%20Git%20repo%2022f50618cbb8802a9133e4ee01129f87.md)
            
            [CodeBuild**:** Build and test.](AWS%20Services%20Structure/CodeBuild%20Build%20and%20test%2022f50618cbb880c19318c31d7c486823.md)
            
            [CodeDeploy**:** Deploy to EC2/Lambda.](AWS%20Services%20Structure/CodeDeploy%20Deploy%20to%20EC2%20Lambda%2022f50618cbb880109ceaf756b0664a9f.md)
            
            [CodePipeline**:** Orchestrate the flow.](AWS%20Services%20Structure/CodePipeline%20Orchestrate%20the%20flow%2022f50618cbb8802ba737ed2ceb9bcf6e.md)
            
    

- Practice Exams | AWS Certified Solutions Architect Associate
    
    ## Practice Exams 1
    
    - 1
        - CDN
            - To optimize the website loading times for users in Asia while keeping the backend in the United States, the recommended immediate solution is to use a **Content Delivery Network (CDN)** like Amazon CloudFront.
            - A CDN caches content at edge locations closer to users, significantly reducing latency and improving load times
        - CloudFront (AWS CDN Service)
            - **Use Amazon CloudFront with a custom origin pointing to the on-premises servers (Correct)**
                - A **custom origin** is any HTTP(S) server that is not an Amazon S3 bucket. This could be a web server running on an on-premises infrastructure or hosted on another cloud provider. When you set up a CloudFront distribution, you can specify this custom origin by providing its publicly resolvable DNS name. This allows CloudFront to fetch content from your on-premises server when users request it
                - When you configure CloudFront to **point to the on-premises servers**, you are essentially telling CloudFront where to retrieve the original content that it will distribute to users. This involves specifying the URL of your on-premises server in the CloudFront distribution settings. By doing this, CloudFront can cache static content closer to users in various geographic locations, while still retrieving dynamic content from your existing servers located in the United States
            - **Use Amazon CloudFront with a custom origin pointing to the DNS record of the website on Amazon Route 53**
                - This option has been added as a distractor. CloudFront cannot have a custom origin pointing to the DNS record of the website on Route 53.
                - Why Directly Pointing to the On-Premises Servers Is Correct
                    - **CloudFront's custom origin** should be set to the actual public DNS name or IP address of your **on-premises servers**. This ensures CloudFront fetches content directly from the source, allowing it to cache and accelerate delivery to users in Asia.
                - **What Happens If You Point to a Route 53 DNS Record Instead?**
                    - If you configure CloudFront's origin as a **Route 53 DNS record** that resolves to your on-premises website, CloudFront must perform a DNS lookup each time it needs to fetch content from the origin.
                    - This can introduce **additional DNS resolution latency** and potential points of failure, especially if the Route 53 record is misconfigured or uses advanced routing policies that could inadvertently direct CloudFront to the wrong endpoint.
                    - Using Route 53 as an intermediary does **not provide any acceleration or caching benefit**; it simply adds an extra layer of DNS resolution between CloudFront and your origin.
            - Leverage an Amazon Route 53 geo-proximity routing policy pointing to on-premises servers
                - While this option could theoretically route traffic based on geographic proximity, it is not optimal for immediate performance improvements. Geo-proximity routing relies heavily on DNS resolution, which may not always direct users to the nearest resource due to DNS caching and other factors. Additionally, this approach does not provide caching benefits that a CDN like CloudFront offers, which would store content closer to users and reduce load times significantly. Therefore, relying solely on Route 53 for routing would not achieve the same level of performance optimization as using CloudFront with a custom origin
                - **Geo-proximity routing is not appropriate because:** It is designed for situations where you have multiple resources in different locations and want to route users to the nearest one. Here, there is only one backend resource (in the US), so geo-proximity routing offers no benefit
            - **Migrate the website to Amazon S3. Use S3 cross-region replication (S3 CRR) between AWS Regions in the US and Asia**
                - This option is incorrect because migrating the entire website to S3 and implementing cross-region replication is a significant undertaking that may not be feasible within a short timeframe, especially since the website is launching imminently. Moreover, S3 CRR is designed for object storage and does not directly address dynamic content delivery or caching, which are critical for optimizing loading times. This migration would also require rearchitecting parts of the application and could introduce additional latency during replication processes. In contrast, using CloudFront with a custom origin allows for immediate optimization without such extensive changes
    - 2
        - The company is migrating **SMB(Server Message Block) file shares**, which are natively supported by **Amazon FSx for Windows File Server**. This service provides a fully managed Windows file system with support for the SMB protocol, Active Directory integration, and Windows-native features
        - **Amazon Storage Gateway’s File Gateway** is designed to provide on-premises access to data stored in **Amazon S3** using NFS or SMB protocols. It is not intended to connect directly to **Amazon FSx for Windows File Server**
        - **Use Amazon Storage Gateway’s File Gateway to provide low-latency, on-premises access to fully managed file shares in Amazon FSx for Windows File Server. The applications deployed on AWS can access this data directly from Amazon FSx in AWS**
            - When you need to access S3 using a file system protocol, you should use File Gateway. You get a local cache in the gateway that provides high throughput and low latency over SMB. AWS Storage Gateway’s File Gateway does not support file shares in Amazon FSx for Windows File Server, so this option is incorrect.
                1. **The Standard "File Gateway"**: This tool is designed to connect your office to Amazon S3. Amazon S3 is like a giant, basic warehouse for storing files. The File Gateway makes this warehouse look and act like a normal shared folder on your office network. Its main job is to work with S3.
                2. **The Special "Amazon FSx File Gateway"**: This is a different, more specialized tool. It's designed to connect your office specifically to Amazon FSx for Windows File Server. FSx is not just a basic warehouse; it's a fully-featured "cloud hard drive" that works exactly like a traditional Windows server.
                
                The option is incorrect because it suggests using the wrong tool for the job. It says to use the standard "File Gateway" (the S3 tool) to connect to FSx (the advanced "cloud hard drive"). That connection is not supported.
                
    - 10
        
        Here is how it solves the problem using **Path-based Routing**:
        
        1. **Listener:** The ALB has a "Listener" that waits for incoming traffic on a specific port (in this case, port 443 for HTTPS).
        2. **Rules:** The Listener has a set of "Rules" that it evaluates in order of priority. Each rule is a simple "IF-THEN" statement.
        3. **Conditions (The "IF"):** The "IF" part of the rule is the condition. For this use case, you would create rules with a **path condition**:
            - Rule 1 Condition: **IF** the URL path is /orders/*...
            - Rule 2 Condition: **IF** the URL path is /products/*...
        4. **Actions (The "THEN"):** The "THEN" part of the rule is the action. The action tells the ALB what to do with the traffic that matches the condition.
            - Rule 1 Action: ...**THEN** forward the request to the orders-microservice-target-group.
            - Rule 2 Action: ...**THEN** forward the request to the products-microservice-target-group.
        
        **Target Groups** are simply the collections of EC2 instances that are running each specific microservice.
        
    - 3
        - **ElastiCache Memcached: The Simple Bulletin Board**
            - Think of **Memcached** as a simple **bulletin board** on the counter.
            - **What it does:** You can pin a piece of paper (your data) to the board with a label (a "key"). You can quickly grab the paper by looking for its label. That's it.
            - **Simplicity is Speed:** Because it's so simple, it is incredibly fast.
            - **Temporary:** If you turn off the power (restart the service), the bulletin board is wiped clean. It's only for temporary, quick-access storage.
            - **Best for:** Simple caching. When you just need to store something and get it back as fast as possible, like the results of a database query or a user's session information.
            - **In short: Memcached is a very fast, very simple, temporary memory holder.**
        - **ElastiCache Redis: The Smart Organizer Shelf**
            - Think of **Redis** as a **smart, multi-functional organizer shelf** on the counter.
            - **What it does:** It's not just a bulletin board. It has special compartments. You can store a simple item, but you can also store organized **lists** (like a queue of drink orders), **leaderboards** (like a sorted list of top customers), and unique collections of items.
            - **More Features:** It's a "Swiss Army Knife." It can do more than just store and retrieve. It can act as a message system, run counters, and manage more complex data.
            - **Durable:** You can tell this shelf to periodically take a picture of its contents and save it. If the power goes out, it can look at the picture and put everything back. Your data isn't lost.
            - **Best for:** Complex caching, leaderboards, real-time messaging, and situations where you need data to survive a restart.
            - **In short: Redis is a powerful, fast, multi-tool data holder that can store complex data and can save it permanently.**
    - 4
    - 5
        - Unlike other S3 Storage Classes which store data in a minimum of three Availability Zones (AZs), Amazon S3 One Zone-IA stores data in a single Availability Zone (AZ) and costs 20% less than Amazon S3 Standard-IA.
    -