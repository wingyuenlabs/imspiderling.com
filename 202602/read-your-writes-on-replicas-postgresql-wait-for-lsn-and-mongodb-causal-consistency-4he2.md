---
Title: Read‑your‑writes on replicas: PostgreSQL WAIT FOR LSN and MongoDB Causal Consistency
Description: 
Author: Franck Pachot
Date: 2026-02-21T21:36:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>In databases designed for high availability and scalability, secondary nodes can fall behind the primary. Typically, a quorum of nodes is updated synchronously to guarantee durability while maintaining availability, while remaining standby instances are eventually consistent to handle partial failures. To balance availability with performance, synchronous replicas acknowledge a write only when it is durable and recoverable, even if it is not yet readable.</p>

<p>As a result, if your application writes data and then immediately queries another node, it may still see stale data.</p>

<p>Here’s a common anomaly: you commit an order on the primary and then try to retrieve it from a reporting system. The order is missing because the read replica has not yet applied the write.</p>

<p>PostgreSQL and MongoDB tackle this problem in different ways:</p>

<ul>
<li>
<strong>PostgreSQL 19</strong> should introduce a <code>WAIT FOR LSN</code> command, allowing applications to explicitly coordinate reads after writes.</li>
<li>
<strong>MongoDB</strong> provides causal consistency within sessions using the <code>afterClusterTime</code> read concern.</li>
</ul>

<p>Both approaches track when your write occurred and ensure subsequent reads observe at least that point. Let’s look at how each database does this.</p>




<h2>
  
  
  PostgreSQL: <code>WAIT FOR LSN</code> (PG19 - currently in development)
</h2>

<p>PostgreSQL records every change in the Write‑Ahead Log (WAL). Each WAL record has a Log Sequence Number (LSN): a 64‑bit position, typically displayed as two hexadecimal halves such as <code>0/40002A0</code> (high/low 32 bits).</p>

<p>Streaming replication ships WAL records from the primary to standbys, which then:</p>

<ol>
<li>
<strong>Write</strong> WAL records to disk
</li>
<li>
<strong>Flush</strong> them to durable storage
</li>
<li>
<strong>Replay</strong> them, applying changes to data files
</li>
</ol>

<p>The write position determines what can be recovered after a database crash. The flush position defines the recovery point for a compute instance failure. The replay position determines what queries can see on a standby.</p>

<p>A recent commit for the next PostgreSQL release (19) adds a <code>WAIT FOR LSN</code> command to allow a session to block until one of these points reaches a target LSN:</p>

<ul>
<li>
<code>standby_write</code> → WAL written to disk on the standby (not yet flushed)
</li>
<li>
<code>standby_flush</code> → WAL flushed to durable storage on the standby
</li>
<li>
<code>standby_replay</code> (default) → WAL replayed into data files and visible to readers
</li>
<li>
<code>primary_flush</code> → WAL flushed on the primary (useful when <code>synchronous_commit = off</code> and a durability barrier is needed)</li>
</ul>

<p>A typical flow is to write on the primary, commit, and then fetch the current WAL insert LSN:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">pg19rw</span><span class="o">=*#</span> <span class="k">BEGIN</span><span class="p">;</span>

<span class="k">BEGIN</span>

<span class="n">pg19rw</span><span class="o">=*#</span> <span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">orders</span> <span class="k">VALUES</span> <span class="p">(</span><span class="mi">123</span><span class="p">,</span> <span class="s1">'widget'</span><span class="p">);</span>

<span class="k">INSERT</span> <span class="mi">0</span> <span class="mi">1</span>

<span class="n">pg19rw</span><span class="o">=*#</span> <span class="k">COMMIT</span><span class="p">;</span>

<span class="k">COMMIT</span>

<span class="n">pg19rw</span><span class="o">=#</span> <span class="k">SELECT</span> <span class="n">pg_current_wal_insert_lsn</span><span class="p">();</span>

 <span class="n">pg_current_wal_insert_lsn</span>
<span class="c1">---------------------------</span>
 <span class="mi">0</span><span class="o">/</span><span class="mi">18724</span><span class="n">C0</span>

<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>
</code></pre>

</div>



<p>That LSN is then used to block reads on a replica until it has caught up:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="n">pg19ro</span><span class="o">=#</span> <span class="n">WAIT</span> <span class="k">FOR</span> <span class="n">LSN</span> <span class="s1">'0/18724C0'</span>
  <span class="k">WITH</span> <span class="p">(</span><span class="k">MODE</span> <span class="s1">'standby_replay'</span><span class="p">,</span> <span class="n">TIMEOUT</span> <span class="s1">'2s'</span><span class="p">);</span>

</code></pre>

</div>



<p>This LSN‑based read‑your‑writes pattern in PostgreSQL requires extra round‑trips: capturing the LSN on the primary and explicitly waiting on the standby. For many workloads, reading from the primary is simpler and faster.</p>

<p>The pattern becomes valuable when expensive reads must be offloaded to replicas while still preserving read‑your‑writes semantics, or in event‑driven and CQRS designs where the LSN itself serves as a change marker for downstream consumers.</p>




<h2>
  
  
  MongoDB: Causal Consistency
</h2>

<p>While PostgreSQL reasons in WAL positions, MongoDB tracks causality using oplog timestamps and a hybrid logical clock.</p>

<p>In a replica set, each write on the primary produces an entry in <code>local.oplog.rs</code>, a capped collection. These entries are rewritten to be idempotent (for example, <code>$inc</code> becomes <code>$set</code>) so they can be safely reapplied. Each entry carries a Hybrid Logical Clock (HLC) timestamp that combines physical time with a logical counter, producing a monotonically increasing cluster time. Replica set members apply oplog entries in timestamp order.</p>

<p>Because MongoDB allows concurrent writes, temporary “oplog holes” can appear: a write with a later timestamp may commit before another write with an earlier timestamp. A naïve reader scanning the oplog could skip the earlier operation.</p>

<p>MongoDB prevents this by tracking an <code>oplogReadTimestamp</code>, the highest hole‑free point in the oplog. Secondaries are prevented from reading past this point until all prior operations are visible, ensuring causal consistency even in the presence of concurrent commits.</p>

<p>Causal consistency in MongoDB is enforced by attaching an <code>afterClusterTime</code> to reads:</p>

<ul>
<li>Drivers track the <code>operationTime</code> of the last operation in a session.
</li>
<li>When a session is created with <code>causalConsistency: true</code>, the driver automatically includes an <code>afterClusterTime</code> equal to the highest known cluster time on subsequent reads.
</li>
<li>The server blocks the read until its cluster time has advanced beyond <code>afterClusterTime</code>.</li>
</ul>

<p>With any read preference that allows reading from secondaries as well as the primary, this guarantees read‑your‑writes behavior:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>
<span class="c1">// Start a causally consistent session</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="nx">client</span><span class="p">.</span><span class="nf">startSession</span><span class="p">({</span> <span class="na">causalConsistency</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>

<span class="kd">const</span> <span class="nx">coll</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nf">collection</span><span class="p">(</span><span class="dl">"</span><span class="s2">orders</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Write in this session</span>
<span class="k">await</span> <span class="nx">coll</span><span class="p">.</span><span class="nf">insertOne</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="mi">123</span><span class="p">,</span> <span class="na">product</span><span class="p">:</span> <span class="dl">"</span><span class="s2">widget</span><span class="dl">"</span> <span class="p">},</span> <span class="p">{</span> <span class="nx">session</span> <span class="p">});</span>

<span class="c1">// The driver automatically injects afterClusterTime into the read concern</span>
<span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">coll</span><span class="p">.</span><span class="nf">findOne</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="mi">123</span> <span class="p">},</span> <span class="p">{</span> <span class="nx">session</span> <span class="p">});</span>

</code></pre>

</div>



<p>Causal consistency is not limited to snapshot reads. It applies across read concern levels. The key point is that the session ensures later reads observe at least the effects of earlier writes, regardless of which replica serves the read.</p>




<h2>
  
  
  Conclusion
</h2>

<p>Here is a simplified comparison:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>PostgreSQL <code>WAIT FOR LSN</code>
</th>
<th>MongoDB Causal Consistency</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Clock type</strong></td>
<td>Physical byte offset in the WAL (LSN)</td>
<td>Hybrid Logical Clock (HLC)</td>
</tr>
<tr>
<td><strong>Mechanism</strong></td>
<td>Block until replay/write/flush LSN reached</td>
<td>Block until <code>afterClusterTime</code> is visible</td>
</tr>
<tr>
<td><strong>Tracking</strong></td>
<td>Application captures LSN</td>
<td>Driver tracks <code>operationTime</code>
</td>
</tr>
<tr>
<td><strong>Granularity</strong></td>
<td>WAL record position</td>
<td>Oplog timestamp</td>
</tr>
<tr>
<td><strong>Replication model</strong></td>
<td>Physical streaming</td>
<td>Logical oplog application</td>
</tr>
<tr>
<td><strong>Hole handling</strong></td>
<td>N/A (serialized WAL)</td>
<td><code>oplogReadTimestamp</code></td>
</tr>
<tr>
<td><strong>Failover handling</strong></td>
<td>Error unless <code>NO_THROW</code>
</td>
<td>Session continues, bounded by replication state</td>
</tr>
</tbody>
</table></div>

<p>Both PostgreSQL’s <code>WAIT FOR LSN</code> and MongoDB’s causal consistency ensure reads can observe prior writes, but at different layers:</p>

<ul>
<li>PostgreSQL offers manual, WAL‑level precision.</li>
<li>MongoDB provides automatic, session‑level guarantees.</li>
</ul>

<p>If you want read‑your‑writes semantics to “just work” without additional coordination calls, MongoDB’s session‑based model is a strong fit. Despite persistent myths about consistency, MongoDB delivers strong consistency in a horizontally scalable system with a simple developer experience.</p>

