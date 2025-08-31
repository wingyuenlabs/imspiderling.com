---
Title: Understanding Rate Limiting: 5 Essential Algorithms for Protecting Your APIs day 41 of sytem design Basics
Description: 
Author: Vincent Tommi
Date: 2025-08-31T21:47:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Rate limiting is a critical technique for safeguarding web services from being overwhelmed by excessive requests from a single client. Without it, your servers could crash, cloud costs could skyrocket, and legitimate users might experience degraded performance. By restricting the number of requests a client can make within a specific time frame, rate limiting ensures system stability and fair resource allocation.</p>

<p>In this article, we’ll explore five of the most common rate limiting algorithms, diving into how they work, their pros and cons, and providing diagram for more illustrations. Whether you're building a high-traffic API or a small-scale service, understanding these algorithms will empower you to choose the right one for your needs. </p>

<p><strong>Why Rate Limiting Matters</strong></p>

<p>Imagine a bot hammering your API with thousands of requests per second. This could:</p>

<ul>
<li><p>Consume all available server resources, leading to crashes.</p></li>
<li><p>Inflate cloud costs due to excessive API usage.</p></li>
<li><p>Degrade performance for legitimate users.</p></li>
</ul>

<p>Rate limiting mitigates these risks by assigning a request quota (e.g., 100 requests per minute) to each user or IP address. If the quota is exceeded, the server temporarily blocks additional requests and returns an HTTP 429 (Too Many Requests) response. Clear communication of rate limits, typically via response headers, allows API users to implement retry and backoff strategies effectively.</p>

<p>Now, let’s dive into the five most popular rate limiting algorithms.</p>

<ol>
<li>
<strong>Token Bucket</strong>
The Token Bucket algorithm is a widely used approach due to its simplicity and ability to handle bursts of traffic.</li>
</ol>

<p>How It Works</p>

<ul>
<li><p>Imagine a bucket that holds tokens, with a fixed maximum capacity.</p></li>
<li><p>Tokens are added to the bucket at a constant rate (e.g., 10 tokens per second).</p></li>
</ul>

<p>Each incoming request consumes a token. If tokens are available, the request is allowed, and a token is removed. If no tokens are available, the request is rejected.</p>

<p>Visual Illustration</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7r9v7up4uz2231na50ap.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7r9v7up4uz2231na50ap.png" alt=" " width="800" height="1200"></a></p>

<p>Description: This flowchart shows tokens being added to a bucket at a steady rate. An incoming request checks if there are enough tokens, leading to either allowing the request (and removing a token) or rejecting it.</p>

<p><strong>Pros</strong></p>

<ul>
<li><p>Simple to implement and understand.</p></li>
<li><p>Supports bursts of requests up to the bucket’s capacity, accommodating short-term traffic spikes.</p></li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li><p>Memory usage scales with the number of users when implemented per-user.</p></li>
<li><p>Does not guarantee perfectly smooth request rates.</p></li>
</ul>

<ol>
<li>
<strong>Leaky Bucket</strong>
The Leaky Bucket algorithm is similar to Token Bucket but prioritizes smoothing out bursty traffic for a consistent request rate.</li>
</ol>

<p>How It Works</p>

<ul>
<li><p>Picture a bucket with a small hole at the bottom.</p></li>
<li><p>Requests enter the bucket from the top.</p></li>
<li><p>The bucket processes requests at a constant rate through the hole (the "leak").</p></li>
</ul>

<p>If the bucket is full, new requests are discarded.</p>

<p>Visual Illustration</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4gp0wmgo0sxmoq3hvgu7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4gp0wmgo0sxmoq3hvgu7.png" alt=" " width="800" height="533"></a></p>

<p>Description: This flowchart depicts requests entering a bucket and being processed at a constant rate through a "leak." If the bucket is full, excess requests are discarded.</p>

<p><strong>Pros</strong></p>

<ul>
<li><p>Ensures a steady request rate, preventing sudden bursts from overwhelming the system.</p></li>
<li><p>Provides predictable and consistent processing.</p></li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li><p>Poor handling of sudden request bursts; excess requests are immediately dropped.</p></li>
<li><p>Slightly more complex to implement than Token Bucket.</p></li>
</ul>

<ol>
<li><strong>Fixed Window Counter</strong></li>
</ol>

<p>The Fixed Window Counter algorithm divides time into fixed intervals and tracks requests within each window.</p>

<p>How It Works</p>

<ul>
<li><p>Time is split into fixed windows (e.g., 1-minute intervals).</p></li>
<li><p>Each window starts with a counter set to zero.</p></li>
<li><p>Incoming requests increment the counter for the current window.</p></li>
</ul>

<p>If the counter exceeds the limit, requests are denied until the next window begins.</p>

<p>Visual Illustration</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8psqa78bnme702atvuhv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8psqa78bnme702atvuhv.png" alt=" " width="800" height="1200"></a></p>

<p>Description: This flowchart shows a time window with a counter that increments with each request. If the counter exceeds the limit, requests are rejected. A new window resets the counter.</p>

<p><strong>Pros</strong></p>

<ul>
<li><p>Easy to implement and understand.</p></li>
<li><p>Provides clear rate limits for each time window.</p></li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li>Struggles with bursts at window boundaries, potentially allowing twice the intended rate at the edges of windows.</li>
</ul>

<ol>
<li>Sliding Window Log</li>
</ol>

<p>The Sliding Window Log algorithm maintains a log of request timestamps to enforce precise rate limits.</p>

<p>How It Works</p>

<ul>
<li><p>Store a log of timestamps for each request.</p></li>
<li><p>When a new request arrives, remove timestamps older than the window size.</p></li>
</ul>

<p>Count the remaining timestamps.</p>

<p>If the count is below the limit, allow the request and add its timestamp to the log. Otherwise, deny the request.</p>

<p>Visual Illustration</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg58c2chyggtnsqkykodd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg58c2chyggtnsqkykodd.png" alt=" " width="800" height="1200"></a></p>

<p>Description: This flowchart illustrates a sliding window moving along a timeline, removing old timestamps and counting those within the window to decide whether to allow or reject a new request.</p>

<p><strong>Pros</strong></p>

<ul>
<li><p>Highly accurate, with no issues at window boundaries.</p></li>
<li><p>Ideal for low-volume APIs.</p></li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li><p>Memory-intensive for high-volume APIs due to storing timestamps.</p></li>
<li><p>Requires searching through timestamps, which can impact performance</p></li>
</ul>

<ol>
<li>Sliding Window Counter</li>
</ol>

<p>The Sliding Window Counter algorithm combines the simplicity of Fixed Window Counter with the accuracy of Sliding Window Log.</p>

<p>How It Works</p>

<ul>
<li>Track request counts for the current and previous time windows.</li>
</ul>

<p>Calculate a weighted sum of requests based on the overlap with the sliding window. For example, if 75% of the current window has elapsed, 25% of the weight comes from the previous window, and 75% from the current window:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>weight = (100 - 75)% * previousWindowRequests + currentWindowRequests
</code></pre>

</div>



<p>If the weighted sum plus the new request exceeds the limit, reject the request.</p>

<p>Visual Illustration</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5bhxfuc73dtf8d56xuf5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5bhxfuc73dtf8d56xuf5.png" alt=" " width="800" height="533"></a></p>

<p>Description: This flowchart shows a sliding window overlapping two fixed windows (previous and current), calculating a weighted sum of request counts to determine if a new request is allowed.</p>

<p><strong>Pros</strong></p>

<ul>
<li><p>More accurate than Fixed Window Counter.</p></li>
<li><p>More memory-efficient than Sliding Window Log.</p></li>
<li><p>Smooths out request rates at window boundaries.</p></li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li>Slightly more complex to implement than Fixed Window Counter.</li>
</ul>

<p><strong>Choosing the Right Algorithm</strong></p>

<ul>
<li><p>When selecting a rate limiting algorithm, consider:</p></li>
<li><p>System Scale: High-traffic systems may favor Token Bucket or Sliding Window Counter for efficiency.</p></li>
<li><p>Traffic Patterns: Bursty traffic may benefit from Token Bucket, while Leaky Bucket suits steady rates.</p></li>
<li><p>Granularity Needs: Sliding Window Log offers precision but requires more memory.</p></li>
<li><p>Implementation Complexity: Fixed Window Counter is simplest, while Sliding Window Counter balances accuracy and efficiency.</p></li>
</ul>

<p>Additionally, always communicate rate limits clearly to API users via response headers (e.g., X-Rate-Limit-Remaining, X-Rate-Limit-Reset). This helps clients implement retry and backoff strategies effectively.</p>

<p><strong>Conclusion</strong><br>
Rate limiting is an essential tool for protecting APIs and ensuring fair resource usage. By understanding the strengths and weaknesses of Token Bucket, Leaky Bucket, Fixed Window Counter, Sliding Window Log, and Sliding Window Counter, you can choose the algorithm that best fits your system’s needs. Each algorithm offers a unique balance of simplicity, accuracy, and performance, making them suitable for different use cases.</p>

