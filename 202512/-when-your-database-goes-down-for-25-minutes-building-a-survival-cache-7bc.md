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
        <span class="nc">RocksDB</span><span class="o">.</span><span class="na">loadLibrary</span><span class="o">();</span>

        <span class="nc">Options</span> <span class="n">options</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Options</span><span class="o">()</span>
            <span class="o">.</span><span class="na">setCreateIfMissing</span><span class="o">(</span><span class="kc">true</span><span class="o">)</span>
            <span class="o">.</span><span class="na">setCompressionType</span><span class="o">(</span><span class="nc">CompressionType</span><span class="o">.</span><span class="na">LZ4_COMPRESSION</span><span class="o">)</span>
            <span class="o">.</span><span class="na">setMaxOpenFiles</span><span class="o">(</span><span class="mi">256</span><span class="o">)</span>
            <span class="o">.</span><span class="na">setWriteBufferSize</span><span class="o">(</span><span class="mi">8</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="o">);</span> <span class="c1">// 8MB buffer</span>

        <span class="k">this</span><span class="o">.</span><span class="na">db</span> <span class="o">=</span> <span class="nc">RocksDB</span><span class="o">.</span><span class="na">open</span><span class="o">(</span><span class="n">options</span><span class="o">,</span> <span class="n">path</span><span class="o">);</span>
        <span class="k">this</span><span class="o">.</span><span class="na">mapper</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ObjectMapper</span><span class="o">();</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Disk Management Built-In
</h3>

<p><strong>Implementation</strong>: RocksDB has a configurable background cleanup thread:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// From RocksDBDiskStore.java</span>
<span class="k">if</span><span class="o">(</span><span class="n">cleanupDuration</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">this</span><span class="o">.</span><span class="na">scheduler</span> <span class="o">=</span> <span class="nc">Executors</span><span class="o">.</span><span class="na">newSingleThreadScheduledExecutor</span><span class="o">(</span><span class="n">r</span> <span class="o">-&gt;</span> <span class="o">{</span>
        <span class="nc">Thread</span> <span class="n">thread</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Thread</span><span class="o">(</span><span class="n">r</span><span class="o">,</span> <span class="s">"RocksDB-Cleanup"</span><span class="o">);</span>
        <span class="n">thread</span><span class="o">.</span><span class="na">setDaemon</span><span class="o">(</span><span class="kc">true</span><span class="o">);</span>
        <span class="k">return</span> <span class="n">thread</span><span class="o">;</span>
    <span class="o">});</span>
    <span class="k">this</span><span class="o">.</span><span class="na">scheduler</span><span class="o">.</span><span class="na">scheduleAtFixedRate</span><span class="o">(</span>
        <span class="k">this</span><span class="o">::</span><span class="n">cleanup</span><span class="o">,</span> 
        <span class="n">cleanupDuration</span><span class="o">,</span> 
        <span class="n">cleanupDuration</span><span class="o">,</span> 
        <span class="n">unit</span>
    <span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This daemon thread runs periodic cleanup to prevent unbounded disk growth. You configure the cleanup frequency when initializing the disk store, ensuring L3 doesn't consume all server disk space over time.</p>

<h2>
  
  
  Cache Eviction: The Secret Sauce
</h2>

<p>The clever part is <em>when</em> data gets written to RocksDB. I don't persist every cache write—that would be wasteful. Instead, I persist on <strong>cache eviction</strong>.</p>

<p>Caffeine's removal listener is the key:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="k">this</span><span class="o">.</span><span class="na">cache</span> <span class="o">=</span> <span class="nc">Caffeine</span><span class="o">.</span><span class="na">newBuilder</span><span class="o">()</span>
            <span class="o">.</span><span class="na">maximumSize</span><span class="o">(</span><span class="n">maxSize</span><span class="o">)</span>
            <span class="o">.</span><span class="na">expireAfterWrite</span><span class="o">(</span><span class="n">ttl</span><span class="o">)</span>
            <span class="o">.</span><span class="na">evictionListener</span><span class="o">((</span><span class="n">key</span><span class="o">,</span> <span class="n">value</span><span class="o">,</span> <span class="n">cause</span><span class="o">)</span> <span class="o">-&gt;</span> <span class="o">{</span>
                <span class="k">this</span><span class="o">.</span><span class="na">diskStore</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">key</span><span class="o">,</span> <span class="n">value</span><span class="o">);</span> <span class="c1">// write to RocksDB        </span>
            <span class="o">})</span>
            <span class="o">.</span><span class="na">build</span><span class="o">();</span>
</code></pre>

</div>



<p><strong>When does eviction happen?</strong></p>

<ol>
<li>
<strong>Time-based expiry</strong>: Entry sits unused for X minutes → TTL expires → eviction</li>
<li>
<strong>Size-based eviction</strong>: Cache hits 10,000 entries → least recently used gets evicted</li>
</ol>

<p><strong>Why this approach is efficient:</strong></p>

<p><strong>Hot data stays in memory</strong>: Frequently accessed configs never touch disk.</p>

<p><strong>Cold data gets archived</strong>: When a config entry expires from L1, it gets persisted to L3 for outage scenarios.</p>

<p><strong>Eviction-triggered persistence</strong>: Data is written to disk when evicted from memory, not on every cache operation.</p>

<p><strong>During normal operations</strong>: L3 is write-mostly, read-rarely. The database is healthy, so cache misses go to L2, not L3.</p>

<p><strong>During outages</strong>: L3 becomes read-heavy. Cache misses can't reach L2 (database down), so they fall back to L3 for stale data.</p>

<p>This design means your disk isn't constantly thrashing with writes—it only persists data that's already being evicted from memory anyway.</p>

<h2>
  
  
  Benchmarking: Does This Actually Work?
</h2>

<p>I built a test harness to simulate realistic failure scenarios. Here are the results that convinced me this approach works:</p>

<h3>
  
  
  Test 1: Long Outage Resilience (25-min database failure)
</h3>

<p><strong>Setup</strong>: 10K cache entries, 5-min TTL, simulated database outage at T+0</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Time Elapsed</th>
<th>Tier Cache</th>
<th>EhCache (disk)</th>
<th>Caffeine Only</th>
</tr>
</thead>
<tbody>
<tr>
<td>3 minutes</td>
<td>100%</td>
<td>100%</td>
<td>100%</td>
</tr>
<tr>
<td>5 minutes</td>
<td>100%</td>
<td>0%</td>
<td>0%</td>
</tr>
<tr>
<td>7 minutes</td>
<td>100%</td>
<td>0%</td>
<td>0%</td>
</tr>
<tr>
<td>10 minutes</td>
<td>100%</td>
<td>0%</td>
<td>0%</td>
</tr>
<tr>
<td>25 minutes</td>
<td>100%</td>
<td>0%</td>
<td>0%</td>
</tr>
</tbody>
</table></div>

<p><strong>Key finding</strong>: Tier cache maintained availability <strong>for previously-cached keys</strong> by <br>
serving from L3 (RocksDB) after L1 expired.This assumes all requested keys were previously cached. In reality, newly added configs or never-requested keys won't be in L3 and will fail. This represents typical production traffic patterns.</p>

<p>Why did EhCache fail? Its disk persistence is designed for overflow, not outage recovery. When the cache expires, it tries to fetch from the database (which is down) rather than serving stale disk data.</p