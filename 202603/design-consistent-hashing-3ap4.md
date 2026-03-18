---
Title: Design Consistent Hashing
Description: 
Author: JosephAkayesi
Date: 2026-03-18T22:09:44.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Consistent Hashing — System Design Deep Dive
</h1>

<p>A <strong>consistent hashing</strong> algorithm is a technique that allows distributed systems to evenly distribute requests and data across a cluster of servers.</p>

<p>At scale, consistent hashing becomes a fundamental building block for building reliable and horizontally scalable systems.</p>




<h2>
  
  
  Why Do We Need Consistent Hashing?
</h2>

<p>Consistent hashing provides several important benefits:</p>

<ul>
<li>✅ Minimizes data redistribution when servers join or leave</li>
<li>✅ Promotes even distribution of load across servers</li>
<li>✅ Protects systems from cascading failures during topology changes</li>
<li>✅ Reduces unnecessary cache misses at scale</li>
</ul>




<h2>
  
  
  Real-World Examples
</h2>

<p>Consistent hashing appears everywhere in modern distributed systems:</p>

<ul>
<li>
<strong>Amazon DynamoDB</strong> uses it to partition and replicate data</li>
<li>
<strong>Apache Cassandra</strong> uses it to distribute data across nodes</li>
<li>
<strong>Akamai CDN</strong> uses it to route requests to edge servers</li>
</ul>

<p>In reality, the exact implementation depends entirely on your system's access patterns and infrastructure requirements.</p>




<h2>
  
  
  The Rehashing Problem
</h2>

<p>Before understanding consistent hashing, we need to understand the problem it solves.</p>

<p>There are <code>n</code> servers in your cluster. Each server's index is computed using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">server_idx</span> <span class="o">=</span> <span class="nf">hash</span><span class="p">(</span><span class="n">key</span><span class="p">)</span> <span class="o">%</span> <span class="n">n</span>

<span class="nb">hash</span> <span class="o">=</span> <span class="n">Hash</span> <span class="n">function</span> <span class="n">to</span> <span class="nb">hash</span> <span class="nb">all</span> <span class="n">keys</span>
<span class="n">n</span>    <span class="o">=</span> <span class="n">Size</span> <span class="n">of</span> <span class="n">your</span> <span class="n">server</span> <span class="n">pool</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpu8rxzjcv5hen4tvakz1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpu8rxzjcv5hen4tvakz1.png" alt=" " width="683" height="472"></a></p>

<p>To fetch a key we perform the following operation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>f(key) % 4
</code></pre>

</div>



<p>The output is the server on which that key is stored.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5l9nz387weg9nnevy6nd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5l9nz387weg9nnevy6nd.png" alt=" " width="681" height="344"></a></p>

<p>This approach works fine when the server pool is fixed and data is distributed evenly.</p>

<h3>
  
  
  Problem: Server Changes
</h3>

<p>When a server is added or removed, nearly all keys must be remapped.</p>

<p>Below is how server indexes are affected when we remove a server from the pool:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2fjkdq8d0582pxs77ea6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2fjkdq8d0582pxs77ea6.png" alt=" " width="681" height="447"></a></p>

<p>The new distribution of keys after the server is removed:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fonk0u8jdycsa9ny1iwy8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fonk0u8jdycsa9ny1iwy8.png" alt=" " width="676" height="388"></a></p>

<p>Most keys are redistributed — not only those that were previously stored on the server that went offline.</p>




<h2>
  
  
  Core Consistent Hashing Concepts
</h2>

<p>Most consistent hashing implementations are built on a few foundational ideas:</p>

<ul>
<li>Hash Space and Hash Ring</li>
<li>Hash Servers</li>
<li>Hash Keys</li>
<li>Server Lookup</li>
</ul>

<p>Let's walk through each one.</p>




<h2>
  
  
  Hash Space and Hash Ring
</h2>

<p>Imagine a hash algorithm whose output goes from <code>x0, x1 ... xn</code>.</p>

<p>For example, if our hash function is <code>hash(key) % 100</code>, the hash space runs from <code>0</code> to <code>99</code>. In real systems, the space is far larger — SHA-1 runs from <code>0</code> to <code>2^160 - 1</code>.</p>

<p>We then connect the two ends of the hash space to form a <strong>hash ring</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuxet91lwzthki73cbz9y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuxet91lwzthki73cbz9y.png" alt=" " width="684" height="451"></a></p>




<h2>
  
  
  Hash Servers
</h2>

<p>We use our hashing function to map servers onto the ring using either the server name or IP address.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4qpg2g3qge47ynwgeoiz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4qpg2g3qge47ynwgeoiz.png" alt=" " width="693" height="450"></a></p>

<blockquote>
<p><strong>Note:</strong> Most real-world systems use a different hashing function for servers than for keys.</p>
</blockquote>




<h2>
  
  
  Hash Keys
</h2>

<p>Keys are hashed and placed onto the same ring.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkoks0nw7jnxopdusmcpn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkoks0nw7jnxopdusmcpn.png" alt=" " width="697" height="404"></a></p>




<h2>
  
  
  Server Lookup
</h2>

<p>To determine which server a key belongs to, we move <strong>clockwise</strong> from the key's position on the ring until we hit the nearest server.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2vhcnhqbs1rf75kc9wac.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2vhcnhqbs1rf75kc9wac.png" alt=" " width="639" height="337"></a></p>




<h2>
  
  
  Adding and Removing Servers
</h2>

<h3>
  
  
  Add a Server
</h3>

<p>When a new server is added, only a subset of keys need to be redistributed — those whose clockwise path now reaches the new server first.</p>

<p>In the example below, only <code>key0</code> was redistributed. All other keys remain intact.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi2m1p60zrtnhlmzecqi9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi2m1p60zrtnhlmzecqi9.png" alt=" " width="709" height="491"></a></p>

<h3>
  
  
  Remove a Server
</h3>

<p>When a server is removed, only the keys that were mapped to it are redistributed to the next server clockwise.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnrmg6oqwtkesgsiur9lm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnrmg6oqwtkesgsiur9lm.png" alt=" " width="743" height="515"></a></p>




<h2>
  
  
  Two Issues in the Basic Approach
</h2>

<p>The basic consistent hashing approach introduces two problems:</p>

<p><strong>1. Uneven partition sizes</strong> — When servers are added or removed, partitions can become imbalanced, leading to uneven load.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1eyvggl4y1y5j5rzrn99.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1eyvggl4y1y5j5rzrn99.png" alt=" " width="714" height="377"></a></p>

<p><strong>2. Non-uniform key distribution</strong> — Keys may cluster on certain servers, leaving others underutilized.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbes7xfs02ran0fmps3e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbes7xfs02ran0fmps3e.png" alt=" " width="709" height="414"></a></p>

<p>A technique called <strong>virtual nodes</strong> is used to solve both of these problems.</p>




<h2>
  
  
  Virtual Nodes
</h2>

<p>In addition to real nodes, we place multiple <strong>virtual nodes</strong> for each server sparsely across the hash ring. Each server is now responsible for multiple partitions.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flyxi9zb10eyk6m25as6l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flyxi9zb10eyk6m25as6l.png" alt=" " width="723" height="474"></a></p>

<p>To find a key's server, we locate the nearest virtual node clockwise from the key's position.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2qssbwdgmebu6vtu69px.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2qssbwdgmebu6vtu69px.png" alt=" " width="717" height="427"></a></p>

<p>As the number of virtual nodes increases, key distribution becomes more balanced.</p>




<h2>
  
  
  Finding Affected Keys
</h2>

<h3>
  
  
  When a server is added:
</h3>

<p>Move <strong>anticlockwise</strong> from the new server's position to the nearest existing server. All keys in that range are redistributed onto the new server.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flk8xskqalq6hc8tr8jel.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flk8xskqalq6hc8tr8jel.png" alt=" " width="722" height="493"></a></p>

<h3>
  
  
  When a server is removed:
</h3>

<p>The affected range starts at the removed server and moves <strong>anticlockwise</strong> until the next server is found. Keys in that range are redistributed to the surviving server.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff9ypono8lgjkr6eds0lv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff9ypono8lgjkr6eds0lv.png" alt=" " width="727" height="463"></a></p>




<h2>
  
  
  Algorithm Comparison
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Algorithm</th>
<th>Redistribution on Change</th>
<th>Handles Hotspots</th>
<th>Complexity</th>
</tr>
</thead>
<tbody>
<tr>
<td>Simple Modulo Hashing</td>
<td>All keys remapped</td>
<td>❌ No</td>
<td>Low</td>
</tr>
<tr>
<td>Basic Consistent Hashing</td>
<td>Subset of keys</td>
<td>⚠️ Partial</td>
<td>Medium</td>
</tr>
<tr>
<td>Consistent Hashing + Virtual Nodes</td>
<td>Subset of keys</td>
<td>✅ Yes</td>
<td>Medium-High</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Request Flow
</h2>

<ol>
<li>Client sends a request with a key</li>
<li>Key is hashed onto the ring</li>
<li>System moves clockwise to find the nearest server (or virtual node)</li>
<li>Request is forwarded to that server</li>
<li>On topology change, only affected keys are redistributed</li>
</ol>




<h2>
  
  
  Consistent Hashing in Distributed Systems
</h2>

<p>Scaling introduces new challenges even with consistent hashing.</p>

<h3>
  
  
  Hotspot Problem
</h3>

<p>Even with virtual nodes, certain keys (e.g. celebrity data in social networks) can generate disproportionate traffic to one server.</p>

<p>Solutions:</p>

<ul>
<li>Further subdivide hotspot partitions</li>
<li>Add dedicated servers for high-traffic keys</li>
<li>Apply application-level caching in front of the ring</li>
</ul>

<h3>
  
  
  Replication
</h3>

<p>For fault tolerance, keys are typically replicated to the next <code>N</code> servers clockwise on the ring. This ensures data survives individual node failures without a full redistribution.</p>




<h2>
  
  
  Monitoring and Observability
</h2>

<p>After deployment, monitoring consistent hashing behavior is critical.</p>

<p>Track:</p>

<ul>
<li>Load distribution across nodes</li>
<li>Redistribution events on topology changes</li>
<li>Virtual node count and balance metrics</li>
<li>Replication lag across replicas</li>
<li>Hotspot frequency and severity</li>
</ul>

<p>Consistent hashing is not a "set and forget" mechanism — it requires continuous tuning of virtual node counts and replication factors as your cluster evolves.</p>




<h2>
  
  
  Final Thoughts
</h2>

<p>Consistent hashing is more than just a smarter way to distribute keys. It is a core scalability mechanism that:</p>

<ul>
<li>stabilizes systems during cluster topology changes,</li>
<li>ensures fair distribution of load across nodes,</li>
<li>and enables systems to scale horizontally with confidence.</li>
</ul>

<p>Choosing the right configuration of virtual nodes and replication strategy depends heavily on your traffic patterns, data size, and fault tolerance requirements.</p>

<p>Design it carefully — because at scale, <strong>consistent hashing becomes part of your system's scalability strategy</strong>.</p>

