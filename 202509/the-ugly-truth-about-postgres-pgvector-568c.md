---
Title: The Ugly Truth About Postgres & pgvector
Description: 
Author: Chris Sean 🪐
Date: 2025-09-22T21:04:17.000Z
Robots: noindex,nofollow
Template: index
---
<h4>
  
  
  The Single-Box Tax: Why My Friend Was Sketching Shards at Olive Garden
</h4>

<p>True story: I’m at dinner with a friend. Halfway through his pasta, he tells me about a “simple” weekend project where he'd make search feel smarter. Keep Postgres because he knows it well, add pgvector for embeddings, ship by Monday. Clean plan. Familiar tools.</p>

<p>It works… for about two days.</p>

<p>By Wednesday the loading spinner is basically the main character. Support screenshots all look the same. He tweaks settings, rents a bigger box, pushes a few jobs to a side worker. Things calm down, then spike again. At some point he pulls out a pen and starts sketching a sharding plan on the Olive Garden napkin. We laugh, because nobody wants to do shard math between breadsticks.</p>

<p>Here’s the important part: his plan wasn’t wrong. Postgres on a single machine is a tank, and pgvector is solid. The problem was the shape of the workload. Like regular relational rows plus embeddings, bursty writes, spiky reads. Vector indexes are wide. Autovacuum and checkpoints compete with live traffic. You get bloat, WAL pressure, and stalls right when usage spikes. Read replicas buy headroom but introduce staleness. Then come upgrade windows, “vacuum sprints,” hand-sharding, and reviews waiting on fresh data. That operational drag is the single-box tax.</p>

<h4>
  
  
  The What-If Thought Experiment
</h4>

<p>I work at TiDB, so I played a game with him: keep the same app and queries, but change the foundation.</p>

<p>What if storage split itself into ranges, kept three copies via consensus, and moved hot ranges off busy nodes automatically? What if the SQL layer were stateless so you could add capacity during a spike and scale it back down after? What if your vector embeddings lived in the same database as your business data and ANN indexes you configure once, a distance metric you choose up front, and a planner that can mix similarity search with normal filters? No sidecar, no ceremony, no napkin shard plan.</p>

<p>That’s the core idea behind TiDB Cloud.</p>

<h4>
  
  
  How TiDB Handles This
</h4>

<p>Requests land on a stateless SQL layer, which you can scale up or down. Data lives in a replicated key-value store for durability. When load shifts, hot ranges split and move so a single key doesn’t become a hotspot. Analytical replicas use a columnar layout for fast scans but read the same snapshot as transactional queries, so OLTP and analytics agree on the truth. Transactions give you snapshot reads and isolation that reduce the “is the replica caught up?” moments. When the graph gets bumpy on a random Wednesday, you change topology instead of firefighting at midnight.</p>

<h4>
  
  
  When Postgres &amp; pgvector Is Absolutely the Right Call
</h4>

<p>There’s a sweet spot where Postgres + pgvector shines:</p>

<ul>
<li>Your data fits comfortably in memory.</li>
<li>Traffic is predictable.</li>
<li>Your team knows Postgres inside out.</li>
<li>Index maintenance isn’t a weekly chore.</li>
</ul>

<p>In that world, keep the simplest thing that works. Reach for a distributed system when the coordination tax shows up.. like when autovacuum falls behind after a heavy write day, replica lag becomes user-visible, or product plans start bending around database limits.</p>

<h4>
  
  
  Why TiDB Then?
</h4>

<p>Because the big win isn’t only performance, it’s how failures behave and how you spend your time. No more extension-upgrade calendars. No more choosing between fresh reads and safe reads. No more late-night shard rebalance. The conversation shifts from “how do we hide the spikes?” to “how often does the tail cross the line where users notice?” Observability gets intentionally boring: keep an eye on slow queries and traffic spikes; let the system carry shard placement, replica lag, and failover.</p>

<p>Boring ops = more time for useful features.</p>

<h4>
  
  
  When Not to Use TiDB
</h4>

<p>There are clear cases where TiDB isn’t the right fit:</p>

<ul>
<li>Batch-first workloads where staleness never blocks users (nightly warehouse jobs are fine).</li>
<li>Apps that lean on Postgres-specific features you cannot or don’t want to replace.</li>
<li>Teams that value a fixed monthly bill over elasticity for policy or budgeting reasons.</li>
</ul>

<p>Don’t move because it’s new. Move when the old plan starts charging interest in nights and weekends.</p>

<p>Is Distributed More Complicated?</p>

<p>Sometimes, sure. But the complexity you carry as a human can get smaller. You stop carrying extension lifecycles in your head. You stop carrying shard placement. You stop carrying replica lag. Those become system concerns. Your levers feel like product levers, not plumbing.</p>

<p>If my friend had started his week on TiDB Cloud instead of a single Postgres box with pgvector, the ideal week would look like this: Monday ships and stays shipped. Wednesday night doesn’t page anyone. Friday’s review uses this morning’s data, not last night’s dump. The sprint ends with a feature, not a new runbook.</p>

<h4>
  
  
  Closing Thoughts
</h4>

<p>My friend’s “weekend project” wasn’t naive; it was normal. We all start with the tools we know, and we level up when reality asks us to. If you’re paying the single-box tax vacuum sprints, hand-sharding, replica-lag debates consider flipping the foundation, not the feature list.</p>

<p>If you want to try it, TiDB Cloud’s Starter plan includes 5 free GB's. More important than the free tier, though, is whether your Wednesday graph gets quieter. If it does, keep your napkins for marinara, not shard math.</p>

