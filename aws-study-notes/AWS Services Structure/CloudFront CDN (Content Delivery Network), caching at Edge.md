- **What is it? (1-Sentence Pitch):** A fast Content Delivery Network (CDN) that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds by caching content in worldwide edge locations.
- **How CloudFront Works: The Global Network:** CloudFront's performance comes from its three-tiered global network
    1. **Edge Locations (Points of Presence - PoPs):** These are thousands of data centers globally. When a user requests your content, they are routed to the nearest Edge Location, which dramatically reduces latency.
    2. **Regional Edge Caches (Mid-Tier Cache):** These are larger caches located between the Edge Locations and your Origin. They hold a larger amount of less-frequently accessed content. When an Edge Location doesn't have a file, it checks the Regional Edge Cache before going to the Origin, which helps reduce the load on your Origin.
    3. **Origin:** This is the source of your content. It can be an **S3 bucket**, an **EC2 instance**, an **Elastic Load Balancer (ELB)**, or any custom HTTP server (even on-premises).
    - **Request Flow:** User → Nearest Edge Location → Regional Edge Cache (if not at Edge) → Origin (if not in Regional Cache)
- **The Multi-Tier Cache (Regional Edge Caches)**
    - This multi-tier architecture is designed to improve performance and reduce load on your origin.
    - **How it Works:**
        1. A user's request hits the nearest **Edge Location**.
        2. If the content is not cached at the Edge (a "cache miss"), the Edge Location forwards the request to the corresponding **Regional Edge Cache**.
        3. If the content is in the Regional Edge Cache (a "cache hit"), it is sent back to the Edge Location, which then serves it to the user and caches it for future requests.
        4. Only if the content is *also* not in the Regional Edge Cache does the request get forwarded to the **Origin** server.
    - **Key Benefits:**
        - **Reduced Origin Load:** Instead of thousands of Edge Locations hitting your origin for the same file, they are consolidated at the Regional Edge Cache. This dramatically reduces traffic to your origin, saving costs and preventing performance bottlenecks.
        - **Improved Cache-Hit Ratio:** Regional Caches have a larger storage capacity than Edge Locations, allowing them to hold less-frequently accessed content for longer. This increases the chance of a cache hit, even for less popular assets.
        - **Better Latency on Cache Misses:** When an edge location misses, it retrieves the content from a geographically closer Regional Cache instead of making a long-haul trip back to the origin's region.
    - **Content That Bypasses the Regional Edge Cache:**
        - Not all traffic flows through the Regional Edge Cache. The following content types are sent **directly from the Edge Location to the Origin**:
        - **Dynamic Content(Forwarding Headers/Cookies):** Any request for content where the cache behavior is configured to forward all headers, cookies, or query strings is considered dynamic and non-cacheable by the Regional Cache.
            - Analogy
                
                Imagine a cache is like a waiter who has pre-poured glasses of water (a popular, generic item) ready to hand out instantly. This is very fast.
                
                Now, imagine a customer comes in and says, "I want water, but it must be from Fiji, at exactly 8°C, with a slice of lemon, in a blue glass."
                
                The waiter can't use the pre-poured glasses. He has to go all the way back to the kitchen (the **Origin**) and make this highly specific, custom drink from scratch.
                
                In this analogy:
                
                - **The pre-poured water** is static content (like an image).
                - **The custom drink order** is a dynamic request.
                - **The list of specific requirements** (Fiji, 8°C, lemon, blue glass) are the **headers and cookies**.
                
                When you tell CloudFront to "forward all headers and cookies," you're telling it: "Pay attention to every single custom requirement. Don't use a pre-made copy. Go to the origin every time to get a fresh, personalized version." Because of this, the Regional Cache is skipped, as its job is to store common, reusable items, not one-of-a-kind creations.
                
            - **Real-Life Example: An Amazon Shopping Cart**
                1. You log into Amazon. Your browser stores a **cookie** that says, "This is user123."
                2. You click on your shopping cart icon.
                3. Your browser sends a request to CloudFront for the shopping cart page. It also sends your "user123" cookie along with it (this is a **header**).
                4. CloudFront is configured to see that shopping cart pages are dynamic, so it **forwards the cookie** to the Amazon server (the Origin).
                5. The Amazon server sees the "user123" cookie, looks up your specific cart in its database, and builds a webpage showing the exact items *you* added.
                6. This page is sent back to you.
                
                It can't be cached because my shopping cart page is completely different from yours. Forwarding the cookie ensures you see *your* items, not a generic or incorrect cart.
                
        - **Proxy Methods:** Client requests using PUT, POST, PATCH, DELETE, or OPTIONS are proxied directly to the origin.
            - Analogy
                
                Think of a public bulletin board at a community center (the Cache).
                
                - A **GET** request is like **reading** a notice on the board. You're just retrieving information. It's safe for many people to read the same notice. This is what caches are for.
                
                Now, think about actions that *change* the board:
                
                - A **POST** request is like **pinning a brand new notice** to the board.
                - A **PUT** request is like **replacing an existing notice** with a new version.
                - A **DELETE** request is like **taking a notice down**.
                
                You wouldn't want the front-desk person (the Edge Location) to just pretend these actions happened. The request must go directly to the person in charge of the official board (the **Origin**) to make the actual change. Caching an "action" makes no sense; the action itself must be performed.
                
                Therefore, CloudFront doesn't interfere with these action-oriented methods. It acts like a simple messenger ("proxy") and sends the request straight to the origin to be executed.
                
            - **Real-Life Example: Posting a Comment on a Blog**
                1. You read a blog post (your browser used a GET request to fetch the article, which was likely cached by CloudFront).
                2. You type a comment in the comment box and hit "Submit."
                3. Your browser sends a **POST** request. This request contains the text of your comment.
                4. CloudFront sees this is a POST request, not a GET. It knows this is an action, not a read request.
                5. It immediately passes the POST request directly to the blog's server (the **Origin**).
                6. The server receives the comment, saves it to the database, and then the new comment appears on the page for everyone else.
                
                CloudFront didn't try to cache your "submit" action. It let the origin do its job of actually adding the comment.
                
- **Core Components & Configuration**
    
    
    | **Component** | **Description** | **Key Exam Points & Real-World Use** |
    | --- | --- | --- |
    | **Distribution** | The fundamental configuration unit in CloudFront. It defines all the settings for how your content is delivered. | You'll create a "distribution" for each application or website you want to accelerate. |
    | **Origin** | The source of the files. You can have multiple origins for a single distribution (e.g., S3 for images, ELB for API calls). | **S3 Origin:** For static files. **Custom Origin:** For dynamic content (EC2, ELB, API Gateway). |
    | **Origin Access** | A critical security feature to restrict direct access to your S3 bucket, forcing users to access content only through CloudFront. | **Origin Access Control (OAC):** The modern, recommended method. It's more secure and supports advanced scenarios like server-side encryption with KMS. **Origin Access Identity (OAI):** The legacy method. **Exam Tip:** If you see a question about securing S3 content served via CloudFront, the answer is always OAC/OAI. |
    | **Cache Behavior** | A set of rules that defines how CloudFront handles requests based on the URL path pattern (e.g., /images/*, *.js, /api/*). | • **Path Pattern:** Determines which behavior applies.
    • **Viewer Protocol Policy:** Redirect HTTP to HTTPS.
    • **Allowed HTTP Methods:** GET/HEAD (cacheable), GET/HEAD/OPTIONS (cacheable), or All (for dynamic content, usually not cached). |
    | **Cache Policies** | Fine-grained rules that determine the **Cache Key**. The cache key is the unique identifier for a cached object, composed of the URL plus selected headers, cookies, and query strings. | • **Managed Policies:** AWS provides pre-configured policies (e.g., CachingOptimized, CachingDisabled). 
    • **Custom Policies:** You can define exactly which headers, cookies, or query strings should be part of the cache key. This is key for dynamic content that varies based on these values. |
    | **Time to Live (TTL)** | How long CloudFront caches an object. The order of precedence is:  
    1. Cache-Control header from the Origin (most important). 
    2. Min/Default/Max TTL settings in your CloudFront Cache Policy. | Cache-Control: max-age=3600 from the origin tells CloudFront to cache the file for 3600 seconds. This gives you granular control at the object level. |
    | **Invalidation** | A request to forcibly remove an object from all Edge Caches before its TTL expires. | Use sparingly. **Invalidations cost money.** The best practice is to use **versioned filenames** instead (e.g., style-v1.css, style-v2.css). Invalidation is for emergency situations. |
- **CloudFront Security**
    
    
    | **Feature** | **What It's For** | **Key Use Case** |
    | --- | --- | --- |
    | **HTTPS/SSL/TLS** | Encrypting data in transit between the viewer and CloudFront. | Use **AWS Certificate Manager (ACM)** to provision and manage free SSL/TLS certificates for your custom domain. **This is a standard practice.** |
    | **Signed URLs & Signed Cookies** | Protecting content from unauthorized access. Creates a temporary, private token for access. | • **Signed URLs:** For restricting access to a single file (e.g., a download link for purchased software, a private video stream).  
    • **Signed Cookies:** For providing access to multiple restricted files (e.g., an entire premium video library or online course materials for a logged-in user). |
    | **Geo-Restrictions** | Restricting content access based on the user's geographic location. | **Whitelist:** Allow users from specific countries only (e.g., for content licensing reasons). 
    **Blacklist:** Block users from specific countries. |
    | **AWS WAF Integration** | Protecting against common web attacks at the edge, before they reach your origin. | Block malicious traffic like **SQL Injection (SQLi)** and **Cross-Site Scripting (XSS)**. Implement rate-based rules to block IP addresses that make excessive requests (DDoS protection). |
    
    ---
    
- **Performance, Advanced Features & Cost**
    - **Caching Dynamic Content:**
        - Dynamic content (e.g., API responses, personalized pages) often bypasses caches.
        - Requests with PUT/POST/PATCH/DELETE methods and requests configured to forward all headers are sent directly to the origin, **skipping the Regional Edge Cache.**
    - **Origin Shield:**
        - An additional caching layer that sits in front of your origin in an AWS Region you choose.
        - It further reduces origin load by consolidating requests from all Regional Edge Caches into a single request to the origin for a given asset. It's your "shield" for your origin.
    - **Price Classes:**
        - Control your costs by selecting the geographic scope of your distribution.
        - **Price Class All:** Best performance, highest cost (uses all edge locations).
        - **Price Class 200:** Excludes the most expensive regions (Australia, South America).
        - **Price Class 100:** Lowest cost, worst performance (only USA, Canada, Europe).
    - **Lambda@Edge vs. CloudFront Functions:** Run code at the edge to modify requests and responses.
        
        
        | **Feature** | **CloudFront Functions** | **Lambda@Edge** |
        | --- | --- | --- |
        | **Purpose** | Lightweight, high-volume transformations. | Complex, compute-heavy logic. |
        | **Triggers** | Viewer Request, Viewer Response | Viewer Request, **Origin Request**, **Origin Response**, Viewer Response |
        | **Runtime** | JavaScript only | Node.js, Python |
        | **Execution Time** | Sub-millisecond | Can run for several seconds. |
        | **Use Case** | Header manipulation, URL rewrites/redirects, simple access authorization. | A/B testing, advanced image optimization, complex authentication workflows. |