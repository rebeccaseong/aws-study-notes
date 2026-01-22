- **What is it? (1-Sentence Pitch):** AWS X-Ray is a distributed tracing service that helps developers analyze and debug production, distributed applications (such as microservices and serverless architectures) by providing end-to-end request tracing.

- **Core Use Cases:**
    - Tracing requests across microservices architectures
    - Identifying performance bottlenecks and latency issues
    - Debugging errors in distributed applications
    - Understanding dependencies between services
    - Root cause analysis for failures in complex systems

- **Key Features & Concepts:**
    - **Service Map:** Visual representation of application architecture and dependencies
    - **Trace:** End-to-end record of a request as it travels through your application
    - **Segments:** Data recorded by each service component (e.g., Lambda function, API Gateway, DynamoDB)
    - **Subsegments:** More granular detail within a segment (e.g., database query, HTTP call)
    - **Annotations:** Key-value pairs for indexing traces (searchable)
    - **Metadata:** Additional data for context (not indexed/searchable)
    - **Sampling:** Control which requests to trace (reduce cost, avoid noise)

- **How X-Ray Works:**
    1. **Instrument Application:** Add X-Ray SDK to your code or enable X-Ray for AWS services
    2. **Generate Trace Data:** Application sends trace data to X-Ray daemon/agent
    3. **X-Ray Service:** Aggregates and stores trace data
    4. **Service Map & Analysis:** Visualize service map, analyze latency, errors, and dependencies

- **X-Ray Integration with AWS Services:**
    | **Service** | **Integration** |
    |---|---|
    | **Lambda** | Enable X-Ray tracing in function configuration |
    | **API Gateway** | Enable X-Ray tracing for stage |
    | **ECS/Fargate** | Run X-Ray daemon as sidecar container |
    | **EC2** | Install X-Ray daemon on instance |
    | **Elastic Beanstalk** | Enable X-Ray in environment configuration |
    | **ALB** | Automatically traces requests to backend services |

- **X-Ray vs CloudWatch vs CloudTrail:**
    | **Service** | **What It Does** | **Focus Area** |
    |---|---|---|
    | **X-Ray** | **Request tracing** (track requests across services) | **Performance, bottlenecks, errors** |
    | **CloudWatch** | **Metrics and logs** (monitor performance) | **System health, metrics, alarms** |
    | **CloudTrail** | **API audit logging** (who did what) | **Compliance, auditing** |

- **Key Concepts:**
    - **Trace ID:** Unique identifier for a request's journey through the system
    - **Sampling Rules:** Determine which requests to trace (e.g., 5% of all requests, 100% of errors)
    - **Filter Expressions:** Search traces by response time, error status, annotations
    - **Service Graph:** Visualize microservices and their dependencies
    - **Error Analysis:** Identify which services are causing errors

- **Integration:**
    - **Lambda:** Built-in X-Ray support (just enable it)
    - **API Gateway:** Enable X-Ray tracing per stage
    - **ECS/Fargate:** Run X-Ray daemon container alongside application
    - **EC2:** Install X-Ray daemon on instances
    - **SDK Support:** Java, Node.js, Python, Ruby, .NET, Go
    - **CloudWatch ServiceLens:** Combines X-Ray traces with CloudWatch metrics

- **Exam Tips / What to Remember:**
    - **X-Ray = Distributed tracing** for microservices and serverless applications
    - Exam scenario: **"Trace requests across microservices"** or **"Find performance bottlenecks"** = X-Ray
    - **Service Map** visualizes application architecture and dependencies
    - **CloudWatch monitors metrics**, **X-Ray traces requests** (different purposes!)
    - **CloudTrail logs API calls**, **X-Ray traces application requests**
    - **Segments** = service-level data, **Subsegments** = granular data (database calls, HTTP requests)
    - **Sampling** reduces cost by tracing only a percentage of requests
    - Integrated with **Lambda, API Gateway, ECS, EC2, Elastic Beanstalk**
    - **X-Ray daemon** must be running to collect trace data (automatic for Lambda/API Gateway)
