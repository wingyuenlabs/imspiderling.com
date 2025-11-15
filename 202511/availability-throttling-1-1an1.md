---
Title: Availability — Throttling (1)
Description: 
Author: Udayan Sawant
Date: 2025-11-15T21:36:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>"<em>the "PLEASE CHILL” Pattern your services desperately need</em>"</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhesmxmxs7gfaqn311fxy.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhesmxmxs7gfaqn311fxy.jpg" alt="Throttling" width="739" height="500"></a></p>

<p>Imagine your service is a tiny café.</p>

<p>Most days it’s fine. A few customers, some coffee orders, a little latency but nothing dramatic.</p>

<p>Then one day you get featured on Hacker News. Suddenly 10,000 people show up, all yelling GET /coffee at the same time.</p>

<p>Options:</p>

<ul>
<li>You try to serve everyone → kitchen melts, nobody gets coffee.</li>
<li>You shut the door and deny everyone → users rage, business dies.</li>
<li>You let people in at a controlled rate → some wait, some get “come back later,” the kitchen keeps working.</li>
</ul>

<p>That third one is throttling.</p>

<p>In distributed systems, throttling is how we tell clients:</p>

<p>“<strong>You’re not wrong, you’re just early.</strong>”</p>

<p>Let’s unpack what throttling really is, how it differs from plain rate limiting, and how to design it cleanly in large systems.</p>




<p><strong>Throttling vs Rate Limiting vs “Just Autoscale It”</strong><br>
These terms get mixed a lot, so let’s carve out some boundaries.</p>

<p><strong>Rate limiting (from the client’s perspective)</strong><br>
Rate limiting is usually about enforcing a policy on how many requests a client is allowed to make over some time window:</p>

<p>“<em>User A can hit /search at most 100 requests per minute.</em>”</p>

<p>If the client exceeds that, we reject extra requests (often with HTTP<br>
<br>
 <code>429 Too Many Requests</code><br>
<br>
). Rate limiting is often part of API gateways and public-facing APIs.</p>

<p><strong>Throttling (from the system’s perspective)</strong><br>
Throttling is the system saying:</p>

<p>“<strong>Given my current resources, I’ll only process this many things right now.</strong>”</p>

<p>It’s not only about fairness across clients, but also about protecting dependencies, keeping latency under control, and staying alive under chaos.</p>

<p>Throttling might:</p>

<ul>
<li>Slow you down (queue or delay requests),</li>
<li>Reject you outright,</li>
<li>Or downgrade what you get (fallbacks, cached/stale responses).
Rate limiting is often policy-first (“free users get 10 requests/sec”).
Throttling is often health-first (“DB is unhappy, we’ll aggressively shed load until it recovers”).</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpejew4ei49j2hv0l1oat.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpejew4ei49j2hv0l1oat.PNG" alt="A conceptual graph showing how incoming traffic flows into these three mechanisms and what each one “guards.”" width="800" height="364"></a></p>

<p><strong>Why “just autoscale” isn’t enough</strong><br>
Autoscaling is great, but:</p>

<ul>
<li>It’s slow compared to traffic spikes.</li>
<li>Some resources don’t scale linearly (databases, legacy systems, third-party APIs).</li>
<li>You pay for overprovisioning.</li>
<li>There’s always a ceiling where more machines don’t help.</li>
</ul>

<p>Throttling is your first line of defense even in a fully auto-scaled world. Azure’s architecture docs explicitly recommend throttling as a complement/alternative to scaling when resources are finite.</p>




<p><strong>What Exactly Are We Throttling?</strong><br>
“Throttling” isn’t just “requests per second.” You can throttle pretty much any scarce thing:</p>

<ul>
<li>RPS per client: 100 req/min per API key, IP, user, tenant.</li>
<li>Global RPS: 50k req/sec across the service.</li>
<li>Concurrent work: max 500 in-flight queries to a DB, max 200 open HTTP connections to a dependency.</li>
<li>Resource usage: CPU, memory, I/O bandwidth, number of Kafka partitions you read from at once, etc.</li>
<li>Per-resource quotas: particular endpoints, particular queues, particular features.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs7spf2eol8ob3prm5zl3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs7spf2eol8ob3prm5zl3.png" alt="A simple hierarchy of what you can actually throttle, with sub-branches for types." width="800" height="192"></a></p>

<p>You also have to decide:</p>

<ul>
<li>Who gets limited? Per-API key, per-user, per-tenant, per-region, per-service, per-IP…</li>
<li>Where does it happen? Client SDK, API gateway, service layer, database gatekeeper, background worker pool.</li>
<li>What happens to excess? Drop immediately, queue, delay, or degrade.</li>
</ul>

<p><em>Throttling is half policy, half plumbing.</em></p>




<p><strong>Core Throttling Algorithms (Without Hand-Wavy Math)</strong><br>
Let’s go through the usual suspects in human terms.</p>

<p><strong>1. Fixed Window Counter</strong><br>
Policy: N requests per time window (say 100 req/min).</p>

<p>Implementation idea:</p>

<ul>
<li>Maintain a counter per key (like user_id).</li>
<li>Each minute, reset the counter.</li>
<li>If the count for this minute &gt; 100 → reject.</li>
</ul>

<p>It’s simple and fast but has a nasty edge case:</p>

<ul>
<li>User sends 100 requests at 12:00:59 and 100 at 12:01:01 → effectively 200 in ~2 seconds.</li>
</ul>

<p>Works fine for many systems, but not ideal if you care about burst control.</p>

<p><strong>2. Sliding Window (More Fair, Slightly More Work)</strong><br>
Same rule: 100 requests per minute, but we treat it as “in the last 60 seconds” instead of “in this calendar minute.”</p>

<p>Implementation variants:</p>

<ul>
<li>Sliding log: store timestamps for each request, prune anything older than 60s, count the rest.</li>
<li>Sliding window with buckets: split the minute into smaller buckets (e.g., 10 x 6-second buckets) and sum them.</li>
<li>Smoother, safer, but you trade off memory (for logs) or precision (for buckets).</li>
</ul>

<p><strong>3. Token Bucket (let bursts through, but only occasionally)</strong><br>
Token Bucket is the workhorse of modern rate limiting and throttling.</p>

<p>Think of it as:</p>

<ul>
<li>A bucket that holds at most capacity tokens.</li>
<li>Tokens drip in at some rate (r tokens/second).</li>
<li>Each request consumes one token.</li>
<li>If there are no tokens, you reject or delay the request.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fjke4ug6vbcwja7fnbp.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fjke4ug6vbcwja7fnbp.PNG" alt="Sequence diagram showing time adding tokens and clients consuming them" width="800" height="464"></a></p>

<p>Properties:</p>

<ul>
<li>Allows short bursts (up to capacity) if the client has been idle.</li>
<li>Enforces a long-term average rate (r).</li>
<li>Very well-suited to distributed caching stores like Redis / DynamoDB.</li>
</ul>

<p>Azure’s ARM throttling uses a token bucket model at regional level to enforce limits while allowing some burstiness.</p>

<p><strong>4. Leaky Bucket (smooths spikes aggressively)</strong><br>
Leaky Bucket is like a queue with a fixed drain rate:</p>

<ul>
<li>Requests enter a queue (the bucket).</li>
<li>The system processes them at a constant rate (the leak).</li>
<li>If the bucket fills up → drop or reject new arrivals.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F69jxve0qvfgd6slgw5ce.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F69jxve0qvfgd6slgw5ce.PNG" alt="Data flow from incoming requests into a queue/bucket that drains at a fixed rate, overflowing when full." width="800" height="209"></a></p>

<p>This is great for protecting a fragile downstream:</p>

<p>“<em>We’ll never send more than 200 writes/sec to this database, full stop.</em>”</p>

<p>It’s more about smoothing than fairness.</p>

<p><strong>5. Concurrency Limits (semaphores in a trench coat)</strong><br>
Sometimes RPS isn’t the right lever. You care about how many operations are currently in flight.</p>

<p>Classic pattern:</p>

<ul>
<li>Wrap access to a resource with a semaphore of size N.</li>
<li>Each request acquire()s a slot before calling the dependency.</li>
<li>When done, release().</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz38umz0pux3wg10kxm57.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz38umz0pux3wg10kxm57.PNG" alt="Sequence diagram showing the limiter mediating between service and dependency." width="800" height="619"></a></p>

<p>If no slot is available:</p>

<ul>
<li>Either queue until one frees up, or</li>
<li>Fail fast and tell the caller to back off.</li>
</ul>

<p>This is common for DB pools, file I/O, CPU-heavy tasks, and in thread-pool-based throttling patterns.</p>




<p>Coming up in Part 2<br>
This wraps up the core theory: what throttling is, how it differs from rate limiting, and the main algorithms (fixed/sliding windows, token bucket, leaky bucket, concurrency limits) that power it.</p>

<p>In Part 2, we’ll plug this into real-world architecture: where to place throttling in a distributed system, how to combine it with circuit breakers and load shedding, what actually happens at runtime (429s, backoff, queues), and a concrete design for a distributed throttling service.</p>

