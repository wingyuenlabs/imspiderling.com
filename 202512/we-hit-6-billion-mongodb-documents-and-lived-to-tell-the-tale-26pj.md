---
Title: We Hit 6 Billion MongoDB Documents (And Lived to Tell the Tale)
Description: 
Author: Milinda Biswas
Date: 2025-12-18T21:34:57.000Z
Robots: noindex,nofollow
Template: index
---


<p>So here's the thing about running a database at scale – nobody tells you about the weird stuff until you're already knee-deep in it. At <a href="https://avluz.com/" rel="noopener noreferrer">Avluz.com</a>, we crossed 6 billion documents in our <a href="https://www.mongodb.com/" rel="noopener noreferrer">MongoDB</a> cluster this year, and honestly? It was equal parts terrifying and fascinating.</p>

<p>We started on AWS like everyone else. Three years in, our monthly bill hit $7,500 and our CFO was giving me <em>that look</em> in every meeting. We moved everything to <a href="https://www.ovhcloud.com/" rel="noopener noreferrer">OVH</a>, spent four months optimizing with help from <a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark AI</a>, and now we're paying $2,180/month for better performance.</p>

<p>Here's what actually happened.</p>

<h2>The "Oh Crap" Moment</h2>

<p>Picture this: It's 2AM, I'm getting Slack alerts that queries are timing out, and our main collection just hit 4.8 billion documents. Our carefully-tuned indexes? Useless. Our query times went from "pretty good" to "are you even trying?" in about two weeks.</p>

<p>That's when we knew we had to do something drastic. The AWS bills were bad enough, but watching our p95 query times climb to 8 seconds? That was the real pain.</p>

<h2>Why We Ditched AWS for OVH</h2>

<p>Look, AWS is great. Their managed services are top-notch. But when you're running dedicated <a href="https://www.mongodb.com/" rel="noopener noreferrer">MongoDB</a> instances and you know what you're doing, the cost difference is just… brutal.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-1.png" alt="Cost Comparison" width="800" height="450"></a></p>

<p>Check out these numbers:</p>

<ul>
<li>
<strong>AWS:</strong> $7,500/month for our replica set</li>
<li>
<strong>GCP:</strong> $6,800/month (we checked)</li>
<li>
<strong><a href="https://www.ovhcloud.com/" rel="noopener noreferrer">OVH</a>:</strong> $2,180/month for the same specs</li>
</ul>

<p>That's not a typo. Same hardware – 32 cores, 256GB RAM per node, NVMe storage. The catch? <a href="https://www.ovhcloud.com/" rel="noopener noreferrer">OVH</a> doesn't hold your hand as much. But that's fine because we were already managing everything manually anyway.</p>

<p>The real kicker? <a href="https://www.ovhcloud.com/en/network/vrack/" rel="noopener noreferrer">OVH's vRack private network</a> between servers is completely free. AWS was charging us $459/month just for replication traffic between nodes. That's $5,500 a year on network transfers alone. For data that never even leaves the datacenter!</p>

<h2>Our Sharding Disaster (And Recovery)</h2>

<p>When you hit billions of documents, <a href="https://www.mongodb.com/docs/manual/sharding/" rel="noopener noreferrer">sharding</a> isn't optional. But man, can you screw it up.</p>

<p>Our first attempt:</p>

<pre><code>sh.shardCollection("avluz.events", { user_id: 1 })
</code></pre>

<p>Seemed logical, right? Every query filters by user_id, so let's shard on that. Except we have power users who generate 10x more data than normal users. Within a week, some shards were at 900GB while others were chilling at 120GB.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-2.png" alt="Sharding Distribution" width="800" height="450"></a></p>

<p>Queries on the hot shards were dying. The <a href="https://www.mongodb.com/docs/manual/core/sharding-balancer-administration/" rel="noopener noreferrer">MongoDB balancer</a> was moving chunks around constantly, making everything worse. It was a mess.</p>

<p>I spent three days reading documentation, blog posts, and MongoDB's forums. Then I just asked <a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark</a>:</p>

<blockquote>
<p>"I have 6B MongoDB documents with user_id and timestamp. Queries filter by user_id and date range. My shards are super unbalanced. What do I do?"</p>
</blockquote>

<p>It came back with a <a href="https://www.mongodb.com/docs/manual/core/sharding-shard-key/#compound-shard-keys" rel="noopener noreferrer">compound sharding key</a> strategy I hadn't even considered:</p>

<pre><code>sh.shardCollection("avluz.events", { 
  user_id: "hashed", 
  timestamp: 1 
})
</code></pre>

<p><a href="https://www.mongodb.com/docs/manual/core/hashed-sharding/" rel="noopener noreferrer">Hashing the user_id</a> distributes power users evenly across shards. The timestamp as secondary key helps with our range queries.</p>

<p>Resharding 6 billion documents took 72 hours of nail-biting, but the results were immediate:</p>

<ul>
<li>Shard sizes went from all over the place to within 85GB of each other</li>
<li>Query latency dropped from 2.4 seconds to 78ms</li>
<li>The balancer finally calmed down</li>
</ul>

<h2>The Index Audit From Hell</h2>

<p>We had 47 indexes across our collections. I knew some were probably useless, but which ones? Going through <a href="https://www.mongodb.com/" rel="noopener noreferrer">MongoDB</a> logs manually would've taken weeks.</p>

<p>So I dumped our <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/indexStats/" rel="noopener noreferrer">index stats</a> and slow query logs into a file and asked <a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark</a> to analyze it. Twenty minutes later, it told me:</p>

<ul>
<li>12 indexes had literally zero accesses in 30 days</li>
<li>8 were redundant (covered by other <a href="https://www.mongodb.com/docs/manual/core/indexes/index-types/index-compound/" rel="noopener noreferrer">compound indexes</a>)</li>
<li>6 had fields in the wrong order for our query patterns</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-3.png" alt="Index Optimization" width="800" height="450"></a></p>

<p>I'll be honest – I felt pretty dumb. Some of these were obvious in hindsight. But when you're managing dozens of indexes across multiple collections, obvious things stop being obvious.</p>

<p>We dropped the useless indexes and reordered the problematic ones. Results:</p>

<ul>
<li>Index storage: 840GB → 380GB (55% reduction!)</li>
<li>Write performance: +28% faster</li>
<li>My stress level: way down</li>
</ul>

<p>The whole process took maybe four hours of actual work. Doing this manually would've been a multi-week project involving spreadsheets, meetings, and probably a few arguments.</p>

<h2>That Time We Melted Our Cache</h2>

<p>MongoDB's <a href="https://www.mongodb.com/docs/manual/core/wiredtiger/" rel="noopener noreferrer">WiredTiger storage engine</a> cache is supposed to be magical. By default it uses 50% of your RAM. We have 256GB per server, so that's 128GB of cache. Should be plenty, right?</p>

<p>Wrong.</p>

<p>Our <a href="https://www.mongodb.com/docs/manual/administration/analyzing-mongodb-performance/#cache-hit-ratio" rel="noopener noreferrer">cache hit ratio</a> was stuck at 72%. That means 28% of reads were going to disk. With billions of documents and NVMe storage, it wasn't <em>terrible</em>, but it wasn't great either. Queries were averaging 145ms when we knew they could be faster.</p>

<p>I mentioned this to <a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark</a> while troubleshooting something else, and it suggested bumping the cache to 80% of RAM. I was skeptical – doesn't the OS need memory? But the logic made sense: we're not running anything else heavy on these boxes, and Linux is pretty good at managing tight memory.</p>

<p>Changed one config line:</p>

<pre><code>storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 205
</code></pre>

<p>Restarted the nodes one by one (because downtime is scary), and watched the metrics:</p>

<ul>
<li>Cache hit ratio: 72% → 94%</li>
<li>Disk IOPS: dropped by 77%</li>
<li>Query latency: 145ms → 78ms</li>
</ul>

<p>Sometimes the simple fixes are the best ones. The <a href="https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.wiredTiger.engineConfig.cacheSizeGB" rel="noopener noreferrer">MongoDB documentation</a> recommends 50% but notes you can go higher if your workload benefits from it. Ours definitely did.</p>

<h2>Connection Pool Shenanigans</h2>

<p>This one almost took us down completely. We have 200 application servers connecting to MongoDB. Each one had a <a href="https://www.mongodb.com/docs/drivers/node/current/fundamentals/connection/connection-options/#connection-pool-options" rel="noopener noreferrer">connection pool</a> of 100 connections because… honestly? That's what some blog post recommended three years ago and nobody questioned it.</p>

<p>Do the math: 200 servers × 100 connections = 20,000 connections trying to hit MongoDB.</p>

<p>MongoDB started refusing connections around 15,000. Things got weird. Queries would randomly fail. Connections would hang. Our on-call person (me) was having a bad week.</p>

<p><a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark</a> suggested dropping the pool size way down:</p>

<pre><code>const client = new MongoClient(uri, {
  maxPoolSize: 25,      // was 100
  minPoolSize: 5,
  maxIdleTimeMS: 30000
})
</code></pre>

<p>And bumping MongoDB's <a href="https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.maxIncomingConnections" rel="noopener noreferrer">connection limit</a>:</p>

<pre><code>net:
  maxIncomingConnections: 5000
</code></pre>

<p>Now we're at about 5,000 total connections across all apps. MongoDB's CPU usage dropped 40%. Connection errors disappeared. Query latency improved 35%.</p>

<p>Turns out you don't need a massive connection pool per instance. You just need enough to handle your concurrent queries. Who knew? (Everyone who actually read the documentation properly, probably.)</p>

<h2>The Dashboard That Actually Matters</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Favluz.com%2Fblog%2Fwp-content%2Fuploads%2F2025%2F12%2Fimage-4.png" alt="Performance Dashboard" width="800" height="446"></a></p>

<p>After dealing with all this, I realized we were monitoring way too much noise. Most metrics don't matter until they're already broken. We rebuilt our primary dashboard to show just five things:</p>

<ol>
<li>
<strong>Query time (p95):</strong> Currently 78ms. If it hits 200ms, something's wrong</li>
<li>
<strong>Cache hit ratio:</strong> Sitting at 94%. Below 90% means we're thrashing disk</li>
<li>
<strong>Active connections:</strong> 3,847 right now. Over 4,500 and we start investigating</li>
<li>
<strong><a href="https://www.mongodb.com/docs/manual/replication/#replication-lag" rel="noopener noreferrer">Replication lag</a>:</strong> 2.1 seconds. Over 10 seconds means a node is struggling</li>
<li>
<strong>Disk space per shard:</strong> Alert at 15% free (learned this one the hard way)</li>
</ol>

<p>Everything else is in a secondary dashboard that we check during incidents. But this five-metric view? It tells us 95% of what we need to know at a glance.</p>

<p><a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark</a> helped us design this after I fed it six months of incident logs and asked "which metrics actually predicted our outages?" Turns out most of them didn't. These five did.</p>

<h2>How AI Actually Saved Us Weeks</h2>

<p>Let me be real about the <a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark</a> thing. It didn't write our code. It didn't magically fix our database. But here's what it did:</p>

<p><strong>Index optimization:</strong> Would've taken me two weeks of analyzing logs, testing changes, measuring results. With GenSpark analyzing patterns? Four hours.</p>

<p><strong>Schema redesign:</strong> We were restructuring our product catalog. Normally this is weeks of research, testing different approaches, measuring performance. GenSpark gave us three solid approaches with pros/cons in minutes. We tested the best one, it worked, done in three days.</p>

<p><strong>Query optimization:</strong> Our analytics queries were slow. I'd spend a day staring at <a href="https://www.mongodb.com/docs/manual/reference/method/db.collection.explain/" rel="noopener noreferrer">EXPLAIN output</a> trying to figure out why. Now I paste the EXPLAIN into GenSpark, it tells me "you're doing a collection scan on 2M documents, add this index," and I'm done in an hour.</p>

<p><strong>Connection tuning:</strong> This would've been pure trial and error. Test a pool size, monitor for a day, try another. GenSpark gave us a sensible starting point based on our query patterns and we only had to tweak it once.</p>

<p>Total time saved on this project? About 7-8 weeks of work compressed into a week and a half. That's not hype – that's me being able to ship this whole migration in four months instead of six.</p>

<p>The key is knowing what to ask and when. GenSpark isn't magic. But it's like having a senior database engineer available 24/7 to sanity-check your ideas and point out things you missed.</p>

<h2>The Weird Stuff Nobody Warns You About</h2>

<p><strong>Aggregation queries that lie:</strong> MongoDB will happily try to <a href="https://www.mongodb.com/docs/manual/core/aggregation-pipeline/" rel="noopener noreferrer">aggregate</a> billions of documents in memory, fail silently when it hits the <a href="https://www.mongodb.com/docs/manual/core/aggregation-pipeline-limits/#aggregation-pipeline-memory-restrictions" rel="noopener noreferrer">memory limit</a>, and return incomplete results. Always use <code>{ allowDiskUse: true }</code> on big aggregations. Always.</p>

<p><strong>The balancer is chaos:</strong> When resharding, the <a href="https://www.mongodb.com/docs/manual/core/sharding-balancer-administration/" rel="noopener noreferrer">balancer</a> moves chunks between shards automatically. Sounds great! Except it hammers your cluster and makes everything slow. Schedule <a href="https://www.mongodb.com/docs/manual/tutorial/manage-sharded-cluster-balancer/#schedule-the-balancing-window" rel="noopener noreferrer">balancing windows</a> or you'll get surprise performance hits at random times.</p>

<p><strong>Backups at this scale are weird:</strong> Our full backup is 14.5TB. Restoring from backup takes 8 hours. We test this quarterly because the one time you don't test is the one time you'll need it and discover it's broken.</p>

<p><strong><a href="https://www.ovhcloud.com/en/support-levels/" rel="noopener noreferrer">OVH support</a> is… different:</strong> They're helpful, but you need to know what you're doing. If you're used to AWS holding your hand, OVH will make you Google some stuff. That's the tradeoff for paying a third of the price.</p>

<h2>What We'd Do Differently</h2>

<p>If I could go back and redo this whole thing?</p>

<ol>
<li>
<strong>Start with <a href="https://www.mongodb.com/docs/manual/core/hashed-sharding/" rel="noopener noreferrer">hashed compound sharding keys</a>.</strong> Don't wait until you have problems.</li>
<li>
<strong>Audit indexes every quarter.</strong> They accumulate like junk in a garage. Use <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/indexStats/" rel="noopener noreferrer">$indexStats</a> regularly.</li>
<li>
<strong>Set up <a href="https://www.genspark.ai/" rel="noopener noreferrer">GenSpark AI</a> queries earlier.</strong> Would've saved us from at least two mistakes.</li>
<li>
<strong><a href="https://www.ovhcloud.com/" rel="noopener noreferrer">OVH</a> from day one?</strong> Maybe. AWS was nice for the early days when we didn't know what we were doing. But once we hit any real scale, the cost difference is just too big to ignore.</li>
</ol>

<h2>The Numbers That Matter</h2>

<p>Just to close this out with some actual data:</p>

<ul>
<li>
<strong>Total documents:</strong> 6.2 billion (and growing 85M/day)</li>
<li>
<strong>Storage:</strong> 14.5 TB compressed with <a href="https://www.mongodb.com/docs/manual/core/wiredtiger/" rel="noopener noreferrer">WiredTiger</a>
</li>
<li>
<strong>Queries per day:</strong> 1.2 million</li>
<li>
<strong>p95 query time:</strong> 78ms</li>
<li>
<strong>Monthly cost:</strong> $2,180 on <a href="https://www.ovhcloud.com/" rel="noopener noreferrer">OVH</a> vs $7,500 on AWS</li>
<li>
<strong>Time to migrate:</strong> 4 months</li>
<li>
<strong>Times we thought it would never work:</strong> at least 6</li>
<li>
<strong>Times we almost gave up and just paid AWS:</strong> 2</li>
<li>
<strong>Current stress level:</strong> manageable</li>
</ul>

<p>Running 6 billion documents at <a href="https://avluz.com/" rel="noopener noreferrer">Avluz.com</a> taught us that scaling isn't about perfect architecture or having infinite budget. It's about making smart tradeoffs, knowing when to ask for help (even from <a href="https://www.genspark.ai/" rel="noopener noreferrer">AI</a>), and being willing to spend a weekend resharding your database when you have to.</p>

<p>Also, monitor your cache hit ratio. Seriously.</p>




<p><em>Originally published at <a href="https://avluz.com/blog/we-hit-6-billion-mongodb-documents-and-lived-to-tell-the-tale/" rel="noopener noreferrer">Avluz.com</a></em></p>

