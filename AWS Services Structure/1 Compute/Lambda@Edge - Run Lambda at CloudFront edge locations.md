- **What is it? (1-Sentence Pitch):** Lambda@Edge lets you run Lambda functions at AWS CloudFront edge locations (closer to your users) to customize content delivery and reduce latency.

- **Core Use Cases:**
    - Customizing content based on user location, device, or headers
    - A/B testing by routing users to different versions of content
    - Authentication and authorization at the edge
    - URL redirects and rewrites
    - Adding security headers to responses
    - Generating HTTP responses without reaching the origin server

- **Key Features & Concepts:**
    - **Runs at Edge Locations:** Functions execute at CloudFront edge locations (200+ locations worldwide)
    - **Four CloudFront Events:**
        1. **Viewer Request:** After CloudFront receives a request from a viewer
        2. **Origin Request:** Before CloudFront forwards the request to the origin
        3. **Origin Response:** After CloudFront receives the response from the origin
        4. **Viewer Response:** Before CloudFront returns the response to the viewer
    - **Low Latency:** Execute code close to users for faster response times
    - **Limitations (vs regular Lambda):**
        - Max execution time: **30 seconds** (Viewer Request/Response: 5 seconds)
        - Max memory: **10 GB** (Viewer Request/Response: 128 MB)
        - No environment variables
        - Deployed globally (takes time to propagate)

- **Integration:**
    - **CloudFront:** Lambda@Edge is triggered by CloudFront distributions
    - **S3:** Can modify requests/responses for S3-backed CloudFront distributions
    - **CloudWatch Logs:** Logs are written to the region where the function executes (edge location's region)

- **Lambda@Edge vs CloudFront Functions:**
    | **Aspect** | **Lambda@Edge** | **CloudFront Functions** |
    |---|---|---|
    | **Language** | Node.js, Python | JavaScript only |
    | **Execution Time** | Up to 30 seconds (5 sec for Viewer events) | Sub-millisecond |
    | **Triggers** | All 4 CloudFront events | Viewer Request/Response only |
    | **Use Case** | More complex logic, external calls | Lightweight transforms, header manipulation |
    | **Cost** | Higher | Lower (1/6 the price) |

- **Exam Tips / What to Remember:**
    - **Use Lambda@Edge when you need to customize content at the edge** (closer to users)
    - **Four trigger points:** Viewer Request/Response, Origin Request/Response
    - **CloudFront Functions are cheaper and faster** but more limited (JavaScript only, viewer events only)
    - Common exam scenario: **"Customize content based on user's country/device"** = Lambda@Edge or CloudFront Functions
    - **Logs are in the region where the function executed** (not in your home region)
    - Use cases: A/B testing, auth at edge, adding security headers, bot detection
