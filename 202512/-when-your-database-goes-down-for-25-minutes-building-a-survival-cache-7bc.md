---
Title: When Your Database Goes Down for 25 Minutes: Building a Survival Cache
Description: 
Author: Sivagurunathan Velayutham
Date: 2025-12-29T21:20:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>In microservice architectures, config services are critical infrastructure. They store feature flags, API endpoints, and runtime settings that services query constantly on startup, during requests, when auto-scaling. Most are backed by a database with aggressive caching. Everything works beautifully, until your database goes down.</p>

<p><strong>Here's the nightmare scenario:</strong> Your cache has a 5-minute TTL. Your database outage lasts 25 minutes. At the 5-minute mark, cache entries start expiring. Services start failing. New instances can't bootstrap. Your availability drops to zero.</p>

<p>This is the story of building a cache that survives prolonged database outages by persisting stale data to disk and the hard lessons learned along the way.</p>

<h2>
  
  
  The Problem Nobody Talks About
</h2>

<p>Everyone tells you to cache your database. "Just use Redis!" "Throw some Caffeine in there!" And they're right for normal operations.</p>

<p>But here's what the tutorials don't cover: <strong>What happens when your cache expires during a prolonged outage?</strong></p>

<p>The failure sequence looks like this:</p>

<ul>
<li>
<strong>T+0 min</strong>: Database goes down. Cache still serving traffic (100% hit rate).</li>
<li>
<strong>T+5 min</strong>: First cache entries expire. Cache misses start happening.</li>
<li>
<strong>T+6 min</strong>: Cache miss → try database → timeout. Service starts returning errors.</li>
<li>
<strong>T+10 min</strong>: Most cache entries expired. Availability plummets.</li>
<li>
<strong>T+15 min</strong>: Auto-scaling spins up new instances. They can't fetch configs. Immediate crash.</li>
<li>
<strong>T+25 min</strong>: Database finally recovers. You've been down for 20 minutes.</li>
</ul>

<p>The traditional solution is replication i.e Aurora multi-region, DynamoDB global tables, all that good stuff. But replication has its own problems:</p>

<p><strong>Cost</strong>: You're running duplicate infrastructure 24/7 for failure scenarios that happen 2-3 times per year.</p>

<p><strong>Complexity</strong>: Cross-region replication, failover logic, data consistency concerns, network latency.</p>

<p><strong>Partial protection</strong>: Regional outages still take you down. Replication lag can be seconds to minutes.</p>

<p><strong>There had to be a simpler approach.</strong></p>

<h2>
  
  
  The Core Insight: Stale Data Beats No Data
</h2>

<p>Here's the controversial take that changed everything: <strong>For read-heavy config services, serving 10-minute-old data during an outage is infinitely better than serving nothing.</strong></p>

<p>Think about what your config service actually stores:</p>

<ul>
<li>
<strong>Feature flags</strong>: Don't change every second</li>
<li>
<strong>Service endpoints</strong>: Relatively stable</li>
<li>
<strong>API rate limits</strong>: Rarely updated mid-incident</li>
<li>
<strong>Routing rules</strong>: Can tolerate brief staleness</li>
</ul>

<p>Sure, you might serve a feature flag that was disabled 5 minutes ago. But that's better than taking down your entire service because the config is unreachable.</p>

<p>The question became: <em>How do I serve stale data when my cache is empty and my database is unavailable?</em></p>

<p>The answer: <strong>Persist cache evictions to local disk.</strong></p>

<h2>
  
  
  Architecture: The Three-Tier Survival Strategy
</h2>

<p>I built what I call a "tier cache"—three layers of defense against database failures:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw19bg7o19o1bx7alxi3v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw19bg7o19o1bx7alxi3v.png" alt="Architecture" width="800" height="1453"></a></p>

<p><strong>Normal Operation Flow:</strong></p>

<ol>
<li>Request comes in → check L1 (memory)</li>
<li>Cache hit (99% of the time) → return immediately in ~2.5μs</li>
<li>Cache miss → fetch from L2 (database)</li>
<li>Write to L1 for fast access</li>
<li>
<strong>Asynchronously write to L3 (disk)</strong> for outage protection</li>
</ol>

<p><strong>Outage Operation Flow:</strong></p>

<ol>
<li>Request comes in → check L1 (memory)</li>
<li>Cache miss → try L2 (database) → connection timeout</li>
<li>
<strong>Fall back to L3 (disk)</strong> → serve stale data</li>
<li>Service stays alive with degraded data</li>
</ol>

<p><strong>The key innovation</strong>: Every cache eviction gets persisted to disk. When the database is unreachable, we serve from this stale disk cache. It's not perfect data, but it keeps services running.</p>

<h2>
  
  
  Why RocksDB?
</h2>

<p>My first instinct was simple file serialization. Why not just dump everything to JSON?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">File</span> <span class="n">cacheFile</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">File</span><span class="o">(</span><span class="s">"cache-backup.json"</span><span class="o">);</span>
<span class="n">objectMapper</span><span class="o">.</span><span class="na">writeValue</span><span class="o">(</span><span class="n">cacheFile</span><span class="o">,</span> <span class="n">cacheData</span><span class="o">);</span>
</code></pre>

</div>



<p>This worked great for 100 entries in my test. Then I tried 10,000 realistic config objects:</p>

<ul>
<li>
<strong>File size</strong>: 45MB of verbose JSON</li>
<li>
<strong>Write time</strong>: 280ms (blocking the cache)</li>
<li>
<strong>Read time</strong>: 380ms (sequential scan to find one key)</li>
</ul>

<p><strong>Completely unusable.</strong></p>

<p>I needed something that could:</p>

<ul>
<li>
<strong>Read individual keys fast</strong> without scanning the entire file</li>
<li>
<strong>Compress data</strong> since config JSON is highly repetitive</li>
<li>
<strong>Handle writes efficiently</strong> without blocking cache operations</li>
<li>
<strong>Survive crashes</strong> without losing all data</li>
</ul>

<p>After researching embedded databases, RocksDB emerged as the clear winner:</p>

<p><strong>Compression</strong>: My 45MB JSON dump compressed to ~8MB with LZ4 (5.6x reduction). Real-world compression varies by data patterns—typically in 2-4x.</p>

<p><strong>Fast random reads</strong>: Log-Structured Merge (LSM) tree design optimized for key-value lookups. 10-50μs to fetch any key.</p>

<p><strong>Write-optimized</strong>: Writes go to memory first, then flush to disk in batches. No blocking on individual writes.</p>

<p><strong>Battle-tested</strong>: Powers production systems at Facebook, LinkedIn, Netflix. If it's good enough for them, it's good enough for my config service.</p>

<p><strong>Crash safety</strong>: Write-Ahead Logging (WAL) ensures durability even if the process crashes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">RocksDBDiskStore</span> <span class="kd">implements</span> <span class="nc">AutoCloseable</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">RocksDB</span> <span class="n">db</span><span class="o">;</span>
    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">ObjectMapper</span> <span class="n">mapper</span><span class="o">;</span>

    <span class="kd">public</span> <span class="nf">RocksDBDiskStore</span><span class="o">(</span><span class="nc">String</span> <span class="n">path</span><span class="o">)</span> <span class="kd">throws</span> <span class="nc">RocksDBException</span> <span class="o">{</span>
        <span class="n