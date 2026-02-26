---
Title: Introducing PgBeam
Description: 
Author: Alexis Rico
Date: 2026-02-26T22:09:03.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The problem
</h2>

<p>If your PostgreSQL database is in <code>us-east-1</code> and a request comes from a serverless function in Tokyo, an edge worker in Mumbai, or an application server in Frankfurt, the network round trip is 100–200ms.</p>

<p>But a new PostgreSQL connection isn’t one round trip. It’s at least four: TCP handshake, TLS handshake (2 RTTs), then PG startup and auth. That’s 400–800ms before the first query even runs.</p>

<p>Read replicas help but add operational complexity: replication lag, connection routing, ORM-level read/write splitting. PgBouncer pools connections but doesn’t cache. Hyperdrive does both but only works from Cloudflare Workers.</p>

<h2>
  
  
  What PgBeam does
</h2>

<p>PgBeam is a PostgreSQL proxy that sits between your application and your database. It speaks the PG wire protocol, so any driver or ORM connects to it without code changes.</p>

<p>You only need to change the host in your connection string to route traffic through PgBeam:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code><span class="gd">- postgresql://user:pass@prod.c7k2dfh4jk3l.us-east-1.rds.amazonaws.com:5432/postgres
</span><span class="err">
</span><span class="gi">+ postgresql://user:pass@02ljaccjaffjy8xvsw1xq6fdra.gw.pgbeam.app:5432/postgres
</span></code></pre>

</div>



<p>PgBeam does three things:</p>

<ol>
<li><p>Routing. Via DNS, your connection string points to the nearest PgBeam proxy that has a pool of connections to your database.</p></li>
<li><p>Connection pooling. Maintains warm upstream connections so your app doesn’t pay the TLS handshake + auth cost on every query.</p></li>
<li><p>Query caching. SELECT results are cached at the edge. Writes, transactions, and queries with volatile functions (<code>NOW</code>, <code>RANDOM</code>, <code>NEXTVAL</code>) are never cached.</p></li>
</ol>

<h2>
  
  
  Benchmark
</h2>

<p>We publish a live latency benchmark at <a href="https://pgbeam.com/benchmark" rel="noopener noreferrer">https://pgbeam.com/benchmark</a> that runs serverless functions from 20 global regions. Each function opens a real TLS PostgreSQL connection to a database in us-east-1, once directly and once through PgBeam. It measures full connect time plus p50 query latency across 5 samples (first discarded as warmup). No synthetic data, no cherry-picked regions.</p>

<p>For distant regions like Tokyo, São Paulo, and Mumbai, PgBeam typically reduces total round-trip time by 3–5x on cached reads thanks to edge-local responses. Uncached queries still benefit from connection pooling, which eliminates the multi-RTT TLS + auth handshake on every request.</p>

<h2>
  
  
  What’s next
</h2>

<p>PgBeam is in technical preview. We’re looking for customers and design partners from the waitlist and collecting feedback on real workloads. Here’s what we’re working on:</p>

<ul>
<li><p>Dashboard. A web UI for managing projects, viewing cache hit rates, and configuring TTLs per query pattern.</p></li>
<li><p>More regions. We’re evaluating demand for additional regions to cover more of the globe with low latency.</p></li>
<li><p>Vercel Marketplace. One-click provisioning as a Vercel add-on with integrated billing.</p></li>
</ul>

<p>Running global infrastructure isn’t cheap. Right now, our focus is on validating PgBeam with real users and making sure the product delivers before scaling further. If you find this space interesting, please reach out.</p>

<p><a href="https://pgbeam.com" rel="noopener noreferrer">https://pgbeam.com</a></p>

