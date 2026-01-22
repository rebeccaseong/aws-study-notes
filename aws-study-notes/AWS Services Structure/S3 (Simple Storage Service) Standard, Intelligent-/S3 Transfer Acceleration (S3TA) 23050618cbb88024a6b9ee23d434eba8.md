# S3 Transfer Acceleration (S3TA)

- Both CloudFront and S3TA use the **AWS Global Edge Network** to improve performance for users who are geographically far from your resources.
    - **CloudFront:** Primarily for speeding up **reading/downloading** content (GET, HEAD requests).
    - **S3 Transfer Acceleration:** For speeding up **writing/uploading** and **downloading** content (PUT, POST, GET requests).
    - **S3TA:** Acts as a proxy or a "fast lane." It doesn't store (cache) your data at the edge. Every request goes through the fast lane all the way to the S3 bucket. This is why it's great for **uploads (PUTs/POSTs)** and for downloading frequently changing data.
    - **CloudFront:** Acts as a cache. It stores a copy of your popular files at the edge. It's best for **repeated downloads** of the same static content.
- Smart Cost Model
    - **You Pay for Speed:** You are only charged the extra S3TA fee if the transfer is actually faster than a standard S3 transfer over the public internet.
    - **No Benefit, No Fee:** If AWS determines that S3TA will not improve performance for a specific request (e.g., the user is in the same region as the bucket), it routes the data normally, and **you are not charged the acceleration fee**.
    - **Speed Check Tool:** AWS provides a browser-based [**S3 Transfer Acceleration Speed Comparison tool**](https://www.google.com/url?sa=E&q=https%3A%2F%2Fs3-accelerate-speedtest.s3-accelerate.amazonaws.com%2Fen%2Faccelerate-speed-test.html) that you can use to see the performance benefit from different locations around the world.
- **S3TA and Multi-Part Upload are a Perfect Pair**
    - **Not an "Either/Or" Choice:** You don't choose between S3TA and Multi-part Upload; you use them **together**.
    - **How they work together:** Multi-part Upload breaks a large file into smaller chunks and uploads them in parallel. S3TA gives each of those parallel streams a faster, more reliable network path.
    - **Best Practice:** For uploading large files (over 100MB) over long distances, the best practice is to use **Multi-part Upload through the S3 Transfer Acceleration endpoint**.