---
Title: Horizontal Scaling PostgreSQL with Citus: A Practical Deep Dive
Description: 
Author: William Kwabena Akoto
Date: 2026-01-28T20:45:01.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>As databases grow beyond the capabilities of a single server, teams face a critical decision: scale vertically by adding more resources to one machine, or scale horizontally by distributing data across multiple servers. While vertical scaling eventually hits physical and economic limits, horizontal scaling offers virtually unlimited growth potential.</p>

<p>In this hands-on guide, we'll explore Citus—an open-source extension that transforms PostgreSQL into a distributed database. We'll build a real Citus cluster from scratch, demonstrate data distribution, implement foreign key relationships across distributed tables, and understand the principles that make horizontal scaling work.</p>

<p>By the end, you'll have practical experience with distributed databases and understand both their power and their limitations.</p>




<h2>
  
  
  What is Horizontal Scaling?
</h2>

<p>Imagine a pizza restaurant with one chef making all the pizzas. During rush hour, orders pile up because one person can only work so fast. This is <strong>vertical scaling</strong>—you could train the chef to work faster (add more CPU), give them better tools (add more RAM), or expand their workspace (add more storage). But eventually, you hit limits.</p>

<p>Now imagine hiring three chefs, each handling different orders. This is <strong>horizontal scaling</strong>—you distribute the workload across multiple workers. The more customers you have, the more chefs you can add.</p>

<p>In database terms:</p>

<ul>
<li>
<strong>Vertical scaling</strong>: Bigger server (more CPU, RAM, storage)</li>
<li>
<strong>Horizontal scaling</strong>: More servers working together</li>
</ul>

<p>Citus brings horizontal scaling to PostgreSQL by:</p>

<ol>
<li>Splitting your data across multiple worker nodes (sharding)</li>
<li>Keeping related data together on the same node (co-location)</li>
<li>Maintaining PostgreSQL's ACID guarantees and SQL compatibility</li>
</ol>




<h2>
  
  
  Architecture: Understanding Citus Components
</h2>

<p>A Citus cluster consists of at least three components:</p>

<h3>
  
  
  1. Coordinator Node
</h3>

<p>The coordinator is like a project manager—it doesn't do the heavy lifting but knows where everything is. When you connect to a Citus cluster, you connect to the coordinator. It:</p>

<ul>
<li>Routes queries to the appropriate workers</li>
<li>Combines results from multiple workers</li>
<li>Manages distributed transactions</li>
<li>Stores metadata about data distribution</li>
</ul>

<h3>
  
  
  2. Worker Nodes
</h3>

<p>Workers are the muscle of your cluster—they store the actual data and execute queries. Each worker:</p>

<ul>
<li>Holds a subset of your data (shards)</li>
<li>Executes queries locally on its data</li>
<li>Communicates with other workers when needed</li>
<li>Functions as a full PostgreSQL instance</li>
</ul>

<h3>
  
  
  3. Shards
</h3>

<p>Shards are pieces of your distributed tables. Think of them as mini-tables that together form your complete dataset. When you distribute a table:</p>

<ul>
<li>Citus creates 32 shards by default (configurable)</li>
<li>Shards are distributed across workers</li>
<li>Each row goes to exactly one shard based on a hash function</li>
<li>Related data can be co-located on the same worker</li>
</ul>




<h2>
  
  
  Setting Up a Citus Cluster on Digital Ocean
</h2>

<p>For this demonstration, we'll create a three-node cluster:</p>

<ul>
<li>
<strong>citus-01</strong>: Coordinator node</li>
<li>
<strong>citus-02</strong>: Worker 1 </li>
<li>
<strong>citus-03</strong>: Worker 2 </li>
</ul>

<h3>
  
  
  Step 1: Install PostgreSQL and Citus Extension
</h3>

<p>On all three droplets:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Add PostgreSQL repository</span>
<span class="nb">sudo </span>apt-get update
<span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> postgresql-common
<span class="nb">sudo</span> /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

<span class="c"># Install PostgreSQL 16</span>
<span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> postgresql-16

<span class="c"># Install Citus extension</span>
curl https://install.citusdata.com/community/deb.sh | <span class="nb">sudo </span>bash
<span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> postgresql-16-citus-12.1

<span class="c"># Configure PostgreSQL to load Citus</span>
<span class="nb">echo</span> <span class="s2">"shared_preload_libraries = 'citus'"</span> | <span class="nb">sudo tee</span> <span class="nt">-a</span> /etc/postgresql/16/main/postgresql.conf

<span class="c"># Restart PostgreSQL</span>
<span class="nb">sudo </span>systemctl restart postgresql
</code></pre>

</div>



<h3>
  
  
  Step 2: Configure Network Access
</h3>

<p>Edit <code>/etc/postgresql/16/main/postgresql.conf</code> on all nodes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight conf"><code><span class="n">listen_addresses</span> = <span class="s1">'*'</span>
</code></pre>

</div>



<p>Edit <code>/etc/postgresql/16/main/pg_hba.conf</code> on all nodes to allow connections:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight conf"><code><span class="n">host</span>    <span class="n">all</span>             <span class="n">all</span>             <span class="m">0</span>.<span class="m">0</span>.<span class="m">0</span>.<span class="m">0</span>/<span class="m">0</span>               <span class="n">md5</span>
</code></pre>

</div>



<p>Restart PostgreSQL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>systemctl restart postgresql
</code></pre>

</div>



<h3>
  
  
  Step 3: Initialize the Coordinator
</h3>

<p>On the coordinator node (citus-01):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Connect as postgres user</span>
<span class="n">sudo</span> <span class="o">-</span><span class="n">u</span> <span class="n">postgres</span> <span class="n">psql</span>

<span class="c1">-- Create the Citus extension</span>
<span class="k">CREATE</span> <span class="n">EXTENSION</span> <span class="n">citus</span><span class="p">;</span>

<span class="c1">-- Add worker nodes to the cluster</span>
<span class="k">SELECT</span> <span class="n">citus_add_node</span><span class="p">(</span><span class="s1">'10.114.0.11'</span><span class="p">,</span> <span class="mi">5432</span><span class="p">);</span>
<span class="k">SELECT</span> <span class="n">citus_add_node</span><span class="p">(</span><span class="s1">'10.114.0.12'</span><span class="p">,</span> <span class="mi">5432</span><span class="p">);</span>

<span class="c1">-- Verify worker registration</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_get_active_worker_nodes</span><span class="p">();</span>
</code></pre>

</div>



<p>You should see both worker nodes listed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  node_name  | node_port 
-------------+-----------
 10.114.0.11 |      5432
 10.114.0.12 |      5432
</code></pre>

</div>



<h3>
  
  
  Step 4: Initialize Workers
</h3>

<p>On each worker node (citus-02 and citus-03):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">sudo</span> <span class="o">-</span><span class="n">u</span> <span class="n">postgres</span> <span class="n">psql</span>
<span class="k">CREATE</span> <span class="n">EXTENSION</span> <span class="n">citus</span><span class="p">;</span>
</code></pre>

</div>



<p>Your Citus cluster is now operational!</p>




<h2>
  
  
  Demonstration 1: Basic Data Distribution
</h2>

<p>Let's create a simple distributed table and observe how data spreads across workers.</p>

<h3>
  
  
  Creating a Distributed Table
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- On the coordinator</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">citus_demo</span> <span class="p">(</span>
    <span class="n">id</span> <span class="nb">SERIAL</span><span class="p">,</span>
    <span class="n">user_name</span> <span class="nb">TEXT</span><span class="p">,</span>
    <span class="n">city</span> <span class="nb">TEXT</span><span class="p">,</span>
    <span class="n">age</span> <span class="nb">INT</span><span class="p">,</span>
    <span class="n">created_at</span> <span class="nb">TIMESTAMP</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">()</span>
<span class="p">);</span>

<span class="c1">-- Distribute the table by 'id' column</span>
<span class="k">SELECT</span> <span class="n">create_distributed_table</span><span class="p">(</span><span class="s1">'citus_demo'</span><span class="p">,</span> <span class="s1">'id'</span><span class="p">);</span>
</code></pre>

</div>



<p>The <code>create_distributed_table</code> function tells Citus to:</p>

<ol>
<li>Create 32 shards (mini-tables) by default</li>
<li>Distribute these shards across available workers</li>
<li>Route future queries based on the distribution column (id)</li>
</ol>

<h3>
  
  
  Inserting Data
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">citus_demo</span> <span class="p">(</span><span class="n">user_name</span><span class="p">,</span> <span class="n">city</span><span class="p">,</span> <span class="n">age</span><span class="p">)</span> <span class="k">VALUES</span>
    <span class="p">(</span><span class="s1">'Alice'</span><span class="p">,</span> <span class="s1">'New York'</span><span class="p">,</span> <span class="mi">25</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Bob'</span><span class="p">,</span> <span class="s1">'Chicago'</span><span class="p">,</span> <span class="mi">30</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Charlie'</span><span class="p">,</span> <span class="s1">'Seattle'</span><span class="p">,</span> <span class="mi">27</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'David'</span><span class="p">,</span> <span class="s1">'Boston'</span><span class="p">,</span> <span class="mi">35</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Emma'</span><span class="p">,</span> <span class="s1">'Denver'</span><span class="p">,</span> <span class="mi">22</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Frank'</span><span class="p">,</span> <span class="s1">'Portland'</span><span class="p">,</span> <span class="mi">29</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Grace'</span><span class="p">,</span> <span class="s1">'Miami'</span><span class="p">,</span> <span class="mi">28</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Henry'</span><span class="p">,</span> <span class="s1">'Atlanta'</span><span class="p">,</span> <span class="mi">31</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Iris'</span><span class="p">,</span> <span class="s1">'Phoenix'</span><span class="p">,</span> <span class="mi">26</span><span class="p">),</span>
    <span class="p">(</span><span class="s1">'Jack'</span><span class="p">,</span> <span class="s1">'Dallas'</span><span class="p">,</span> <span class="mi">33</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Examining Distribution
</h3>

<p>Check how shards are distributed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_shards</span> <span class="k">WHERE</span> <span class="k">table_name</span><span class="p">::</span><span class="nb">text</span> <span class="o">=</span> <span class="s1">'citus_demo'</span><span class="p">;</span>
</code></pre>

</div>



<p>Result (abbreviated):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> table_name | shardid |    shard_name     | nodename    | nodeport | shard_size 
------------+---------+-------------------+-------------+----------+------------
 citus_demo |  102488 | citus_demo_102488 | 10.114.0.11 |     5432 |      16384
 citus_demo |  102489 | citus_demo_102489 | 10.114.0.12 |     5432 |      16384
 citus_demo |  102490 | citus_demo_102490 | 10.114.0.11 |     5432 |       8192
 ... (32 shards total)
</code></pre>

</div>



<p>We see 32 shards split between two workers. Some shards have data (16 KB), others are empty (8 KB).</p>

<h3>
  
  
  Understanding Shard Assignment
</h3>

<p>See which shard each record belongs to:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">user_name</span><span class="p">,</span> <span class="n">city</span><span class="p">,</span>
       <span class="n">get_shard_id_for_distribution_column</span><span class="p">(</span><span class="s1">'citus_demo'</span><span class="p">,</span> <span class="n">id</span><span class="p">)</span> <span class="k">as</span> <span class="n">shard_id</span>
<span class="k">FROM</span> <span class="n">citus_demo</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">id</span><span class="p">;</span>
</code></pre>

</div>



<p>Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> id | user_name |   city   | shard_id 
----+-----------+----------+----------
  1 | Alice     | New York |   102489
  2 | Bob       | Chicago  |   102512
  3 | Charlie   | Seattle  |   102503
  4 | David     | Boston   |   102496
  5 | Emma      | Denver   |   102494
  6 | Frank     | Portland |   102508
  7 | Grace     | Miami    |   102496
  8 | Henry     | Atlanta  |   102488
  9 | Iris      | Phoenix  |   102516
 10 | Jack      | Dallas   |   102492
</code></pre>

</div>



<p>Notice that David (id=4) and Grace (id=7) share shard 102496—this happens when their IDs hash to the same shard. With only 10 records across 32 shards, collisions and empty shards are normal.</p>

<h3>
  
  
  Viewing Data on Individual Workers
</h3>

<p>Connect directly to Worker 1 (10.114.0.11):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Query a specific shard</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_demo_102488</span><span class="p">;</span>
</code></pre>

</div>



<p>Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> id | user_name |  city   | age |         created_at         
----+-----------+---------+-----+----------------------------
  8 | Henry     | Atlanta |  31 | 2026-01-23 22:42:50.523619
</code></pre>

</div>



<p>Worker 1 only stores the data in its shards. Query other shards on this worker:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_demo_102512</span><span class="p">;</span>  <span class="c1">-- Bob</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_demo_102496</span><span class="p">;</span>  <span class="c1">-- David and Grace</span>
</code></pre>

</div>



<p>Connect to Worker 2 (10.114.0.12) and query:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_demo_102489</span><span class="p">;</span>  <span class="c1">-- Alice</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_demo_102503</span><span class="p">;</span>  <span class="c1">-- Charlie</span>
</code></pre>

</div>



<p><strong>Key insight</strong>: Each worker only stores a subset of the data. When you query the main table through the coordinator, Citus automatically fetches data from all workers and combines the results.</p>




<h2>
  
  
  Demonstration 2: Foreign Keys and Co-location
</h2>

<p>One of the biggest challenges in distributed databases is maintaining relationships between tables. Let's explore how Citus handles foreign keys through co-location.</p>

<h3>
  
  
  The Challenge
</h3>

<p>Imagine you have users and their orders:</p>

<ul>
<li>Users table: Alice, Bob, Charlie</li>
<li>Orders table: Order 1 (Alice's pizza), Order 2 (Bob's burger), Order 3 (Alice's soda)</li>
</ul>

<p>In a distributed setup, what if:</p>

<ul>
<li>Alice's user record is on Worker 1</li>
<li>Alice's orders are on Worker 2</li>
</ul>

<p>When inserting an order for Alice, Worker 2 must verify Alice exists—but Alice is on Worker 1! This requires expensive cross-node communication for every foreign key check.</p>

<h3>
  
  
  The Solution: Co-location
</h3>

<p>Co-location ensures related data lives on the same worker. Citus achieves this by distributing both tables using the same column.</p>

<h3>
  
  
  Creating Co-located Tables
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Create users table</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">users</span> <span class="p">(</span>
    <span class="n">user_id</span> <span class="nb">INT</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span> 
    <span class="n">name</span> <span class="nb">TEXT</span>
<span class="p">);</span>

<span class="c1">-- Create orders table with composite primary key</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">orders</span> <span class="p">(</span>
    <span class="n">order_id</span> <span class="nb">SERIAL</span><span class="p">,</span>
    <span class="n">user_id</span> <span class="nb">INT</span><span class="p">,</span>
    <span class="n">item</span> <span class="nb">TEXT</span><span class="p">,</span>
    <span class="n">created_at</span> <span class="nb">TIMESTAMP</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">(),</span>
    <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="p">(</span><span class="n">order_id</span><span class="p">,</span> <span class="n">user_id</span><span class="p">)</span>  <span class="c1">-- Must include distribution column</span>
<span class="p">);</span>

<span class="c1">-- Distribute both tables by user_id</span>
<span class="k">SELECT</span> <span class="n">create_distributed_table</span><span class="p">(</span><span class="s1">'users'</span><span class="p">,</span> <span class="s1">'user_id'</span><span class="p">);</span>
<span class="k">SELECT</span> <span class="n">create_distributed_table</span><span class="p">(</span><span class="s1">'orders'</span><span class="p">,</span> <span class="s1">'user_id'</span><span class="p">);</span>

<span class="c1">-- Add foreign key constraint</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">orders</span> 
<span class="k">ADD</span> <span class="k">CONSTRAINT</span> <span class="n">fk_user</span> 
<span class="k">FOREIGN</span> <span class="k">KEY</span> <span class="p">(</span><span class="n">user_id</span><span class="p">)</span> <span class="k">REFERENCES</span> <span class="n">users</span><span class="p">(</span><span class="n">user_id</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Important</strong>: The orders primary key includes <code>user_id</code> because Citus requires the distribution column in unique constraints. This ensures uniqueness can be verified locally on each worker.</p>

<h3>
  
  
  Inserting Related Data
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">users</span> <span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">name</span><span class="p">)</span> <span class="k">VALUES</span>
    <span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s1">'Alice'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s1">'Bob'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">'Charlie'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="s1">'David'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="s1">'Emma'</span><span class="p">);</span>

<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">orders</span> <span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">item</span><span class="p">)</span> <span class="k">VALUES</span>
    <span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s1">'Pizza'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s1">'Soda'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s1">'Ice Cream'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s1">'Burger'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s1">'Fries'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">'Taco'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">'Burrito'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">'Nachos'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="s1">'Pasta'</span><span class="p">),</span>
    <span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="s1">'Salad'</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Verifying Co-location
</h3>

<p>Check which shards contain user and order data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">u</span><span class="p">.</span><span class="n">user_id</span><span class="p">,</span>
    <span class="n">u</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
    <span class="n">o</span><span class="p">.</span><span class="n">item</span><span class="p">,</span>
    <span class="n">get_shard_id_for_distribution_column</span><span class="p">(</span><span class="s1">'users'</span><span class="p">,</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span><span class="p">)</span> <span class="k">as</span> <span class="n">user_shard</span><span class="p">,</span>
    <span class="n">get_shard_id_for_distribution_column</span><span class="p">(</span><span class="s1">'orders'</span><span class="p">,</span> <span class="n">o</span><span class="p">.</span><span class="n">user_id</span><span class="p">)</span> <span class="k">as</span> <span class="n">order_shard</span>
<span class="k">FROM</span> <span class="n">users</span> <span class="n">u</span>
<span class="k">JOIN</span> <span class="n">orders</span> <span class="n">o</span> <span class="k">ON</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span> <span class="o">=</span> <span class="n">o</span><span class="p">.</span><span class="n">user_id</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span><span class="p">,</span> <span class="n">o</span><span class="p">.</span><span class="n">order_id</span><span class="p">;</span>
</code></pre>

</div>



<p>Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> user_id |  name   |   item    | user_shard | order_shard 
---------+---------+-----------+------------+-------------
       1 | Alice   | Pizza     |     102521 |      102553
       1 | Alice   | Soda      |     102521 |      102553
       1 | Alice   | Ice Cream |     102521 |      102553
       2 | Bob     | Burger    |     102544 |      102576
       2 | Bob     | Fries     |     102544 |      102576
       3 | Charlie | Taco      |     102535 |      102567
       3 | Charlie | Burrito   |     102535 |      102567
       3 | Charlie | Nachos    |     102535 |      102567
</code></pre>

</div>



<p><strong>Wait, the shard IDs are different!</strong> This might seem wrong, but it's actually correct.</p>

<h3>
  
  
  Understanding Different Shard IDs
</h3>

<p>The <code>users</code> table has its own set of shards (102521, 102544, 102535...), and the <code>orders</code> table has different shard IDs (102553, 102576, 102567...). However, what matters is that they're on the <strong>same physical worker node</strong>.</p>

<p>Verify this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="k">table_name</span><span class="p">,</span>
    <span class="n">shardid</span><span class="p">,</span>
    <span class="n">nodename</span>
<span class="k">FROM</span> <span class="n">citus_shards</span> 
<span class="k">WHERE</span> <span class="n">shardid</span> <span class="k">IN</span> <span class="p">(</span><span class="mi">102521</span><span class="p">,</span> <span class="mi">102553</span><span class="p">)</span>  <span class="c1">-- Alice's user and order shards</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">nodename</span><span class="p">,</span> <span class="k">table_name</span><span class="p">;</span>
</code></pre>

</div>



<p>Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> table_name | shardid |  nodename   
------------+---------+-------------
 orders     |  102553 | 10.114.0.11
 users      |  102521 | 10.114.0.11
</code></pre>

</div>



<p>Both shards are on Worker 1 (10.114.0.11)! This is co-location—different shard tables, same physical location.</p>

<p>Think of it like two filing cabinets (users and orders) in the same office. Even though they're separate cabinets with different drawer labels, they're in the same room, so finding related information is instant.</p>

<h3>
  
  
  Testing Foreign Key Constraints
</h3>

<p>Try inserting an order for a non-existent user:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">orders</span> <span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">item</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="mi">999</span><span class="p">,</span> <span class="s1">'Ghost Pizza'</span><span class="p">);</span>
</code></pre>

</div>



<p>Error:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ERROR:  insert or update on table "orders" violates foreign key constraint "orders_user_id_fkey"
DETAIL:  Key (user_id)=(999) is not present in table "users".
</code></pre>

</div>



<p>Perfect! The foreign key constraint works. Because user_id=999 doesn't exist, and thanks to co-location, the worker can check this locally without network calls to other workers.</p>

<h3>
  
  
  Performing JOINs
</h3>

<p>Count orders per user:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">u</span><span class="p">.</span><span class="n">user_id</span><span class="p">,</span> 
    <span class="n">u</span><span class="p">.</span><span class="n">name</span><span class="p">,</span> 
    <span class="k">COUNT</span><span class="p">(</span><span class="n">o</span><span class="p">.</span><span class="n">order_id</span><span class="p">)</span> <span class="k">as</span> <span class="n">num_orders</span>
<span class="k">FROM</span> <span class="n">users</span> <span class="n">u</span>
<span class="k">LEFT</span> <span class="k">JOIN</span> <span class="n">orders</span> <span class="n">o</span> <span class="k">ON</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span> <span class="o">=</span> <span class="n">o</span><span class="p">.</span><span class="n">user_id</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span><span class="p">,</span> <span class="n">u</span><span class="p">.</span><span class="n">name</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span><span class="p">;</span>
</code></pre>

</div>



<p>Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> user_id |  name   | num_orders 
---------+---------+------------
       1 | Alice   |          3
       2 | Bob     |          2
       3 | Charlie |          3
       4 | David   |          1
       5 | Emma    |          1
</code></pre>

</div>



<p>This JOIN is fast because each worker can join its local user and order data without coordinating with other workers. This is the power of co-location.</p>




<h2>
  
  
  Key Concepts and Best Practices
</h2>

<h3>
  
  
  1. Distribution Column Selection
</h3>

<p>Choose your distribution column carefully:</p>

<p><strong>Good choices:</strong></p>

<ul>
<li>
<code>user_id</code> for multi-tenant applications (each tenant's data together)</li>
<li>
<code>account_id</code> for SaaS applications</li>
<li>
<code>customer_id</code> for e-commerce</li>
</ul>

<p><strong>Bad choices:</strong></p>

<ul>
<li>
<code>created_at</code> (causes temporal hotspots)</li>
<li>Low-cardinality columns (poor distribution)</li>
<li>Columns that change frequently</li>
</ul>

<p><strong>Rule of thumb</strong>: Pick a column that:</p>

<ul>
<li>Appears in most queries (for query efficiency)</li>
<li>Enables co-location of related tables</li>
<li>Has high cardinality (many unique values)</li>
<li>Remains stable (doesn't change)</li>
</ul>

<h3>
  
  
  2. Co-location Requirements
</h3>

<p>For foreign keys to work across distributed tables:</p>

<ol>
<li>
<strong>Same distribution column</strong>: Both parent and child tables must be distributed by the same column</li>
<li>
<strong>Include in constraints</strong>: The distribution column must be part of PRIMARY KEY and UNIQUE constraints</li>
<li>
<strong>Foreign key on distribution column</strong>: The foreign key must reference the distribution column</li>
</ol>

<h3>
  
  
  3. Shard Count
</h3>

<p>Default: 32 shards per table</p>

<ul>
<li>
<strong>Too few shards</strong>: Limits parallelism and future scalability</li>
<li>
<strong>Too many shards</strong>: Increases metadata overhead and query planning time</li>
</ul>

<p>For most applications, 32-128 shards works well. Adjust based on:</p>

<ul>
<li>Expected data size</li>
<li>Number of workers (more workers = more shards beneficial)</li>
<li>Query patterns</li>
</ul>

<h3>
  
  
  4. When to Use Citus
</h3>

<p><strong>Good use cases:</strong></p>

<ul>
<li>Multi-tenant SaaS applications</li>
<li>Real-time analytics on time-series data</li>
<li>High-throughput transactional workloads</li>
<li>Applications that exceed single-server capacity</li>
<li>Workloads with natural sharding keys</li>
</ul>

<p><strong>Not ideal for:</strong></p>

<ul>
<li>Small databases (&lt; 100GB)</li>
<li>Workloads requiring many cross-shard JOINs</li>
<li>Applications with no clear distribution key</li>
<li>Use cases where a single PostgreSQL instance suffices</li>
</ul>




<h2>
  
  
  Performance Considerations
</h2>

<h3>
  
  
  Query Routing
</h3>

<p>Citus routes queries differently based on their scope:</p>

<p><strong>1. Single-shard queries</strong> (fastest):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Routed to one worker because user_id=1 uniquely determines shard</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">orders</span> <span class="k">WHERE</span> <span class="n">user_id</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>2. Co-located JOINs</strong> (fast):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Each worker joins its local data, then results are combined</span>
<span class="k">SELECT</span> <span class="n">u</span><span class="p">.</span><span class="n">name</span><span class="p">,</span> <span class="k">COUNT</span><span class="p">(</span><span class="n">o</span><span class="p">.</span><span class="o">*</span><span class="p">)</span>
<span class="k">FROM</span> <span class="n">users</span> <span class="n">u</span>
<span class="k">JOIN</span> <span class="n">orders</span> <span class="n">o</span> <span class="k">ON</span> <span class="n">u</span><span class="p">.</span><span class="n">user_id</span> <span class="o">=</span> <span class="n">o</span><span class="p">.</span><span class="n">user_id</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">u</span><span class="p">.</span><span class="n">name</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>3. Broadcast queries</strong> (slower):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Must query all shards and combine results</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">orders</span> <span class="k">WHERE</span> <span class="n">item</span> <span class="o">=</span> <span class="s1">'Pizza'</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>4. Cross-shard queries</strong> (slowest):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Requires moving data between workers</span>
<span class="k">SELECT</span> <span class="n">o</span><span class="p">.</span><span class="n">item</span><span class="p">,</span> <span class="n">p</span><span class="p">.</span><span class="n">product_name</span>
<span class="k">FROM</span> <span class="n">orders</span> <span class="n">o</span>
<span class="k">JOIN</span> <span class="n">products</span> <span class="n">p</span> <span class="k">ON</span> <span class="n">o</span><span class="p">.</span><span class="n">product_id</span> <span class="o">=</span> <span class="n">p</span><span class="p">.</span><span class="n">id</span>
<span class="k">WHERE</span> <span class="n">o</span><span class="p">.</span><span class="n">user_id</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
<span class="c1">-- Only works if products is a reference table or co-located</span>
</code></pre>

</div>



<h3>
  
  
  Reference Tables
</h3>

<p>For small lookup tables (countries, products, categories), use reference tables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">countries</span> <span class="p">(</span>
    <span class="n">code</span> <span class="nb">CHAR</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span>
    <span class="n">name</span> <span class="nb">TEXT</span>
<span class="p">);</span>

<span class="c1">-- Replicate to all workers</span>
<span class="k">SELECT</span> <span class="n">create_reference_table</span><span class="p">(</span><span class="s1">'countries'</span><span class="p">);</span>
</code></pre>

</div>



<p>Reference tables are fully replicated to every worker, enabling efficient JOINs with distributed tables.</p>




<h2>
  
  
  Monitoring and Maintenance
</h2>

<h3>
  
  
  Checking Cluster Health
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- View active workers</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_get_active_worker_nodes</span><span class="p">();</span>

<span class="c1">-- Check shard distribution</span>
<span class="k">SELECT</span> <span class="n">nodename</span><span class="p">,</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">as</span> <span class="n">shard_count</span>
<span class="k">FROM</span> <span class="n">citus_shards</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">nodename</span><span class="p">;</span>

<span class="c1">-- Monitor distributed queries</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_stat_statements</span><span class="p">;</span>

<span class="c1">-- Check replication lag (if using HA setup)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_replication_status</span><span class="p">();</span>
</code></pre>

</div>



<h3>
  
  
  Rebalancing Shards
</h3>

<p>As you add workers, rebalance shards:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Move shards to balance load</span>
<span class="k">SELECT</span> <span class="n">citus_rebalance_start</span><span class="p">();</span>

<span class="c1">-- Check rebalance progress</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">citus_rebalance_status</span><span class="p">();</span>
</code></pre>

</div>









<h2>
  
  
  Conclusion
</h2>

<p>We've successfully demonstrated horizontal scaling with Citus by:</p>

<ol>
<li>
<strong>Building a distributed cluster</strong> with one coordinator and two workers</li>
<li>
<strong>Creating distributed tables</strong> and observing how data spreads across shards</li>
<li>
<strong>Implementing foreign key relationships</strong> through co-location</li>
<li>
<strong>Verifying co-location</strong> by checking that related data resides on the same workers</li>
<li>
<strong>Testing foreign key constraints</strong> to ensure data integrity</li>
<li>
<strong>Performing distributed JOINs</strong> that execute efficiently due to co-location</li>
</ol>

<p>Citus transforms PostgreSQL into a horizontally scalable database while maintaining SQL compatibility and ACID guarantees. By distributing data intelligently and co-locating related tables, it achieves impressive performance even as datasets grow beyond single-server capacity.</p>

<h3>
  
  
  The Elephant in the Room: Single Points of Failure
</h3>

<p>However, our current setup has a critical limitation: <strong>every node is a single point of failure</strong>.</p>

<p>Consider these scenarios:</p>

<ul>
<li>
<strong>Coordinator failure</strong>: If citus-01 goes down, your entire application stops. No queries can be routed, no data can be accessed, even though workers are healthy.</li>
<li>
<strong>Worker failure</strong>: If citus-02 crashes, all data on that worker becomes unavailable. Half your users suddenly can't access their orders.</li>
<li>
<strong>Maintenance downtime</strong>: Need to apply a security patch? You'll have to take the coordinator offline, causing application downtime.</li>
</ul>

<p>In production environments, single points of failure are unacceptable. Your database must survive:</p>

<ul>
<li>Hardware failures</li>
<li>Network issues</li>
<li>Planned maintenance</li>
<li>Software crashes</li>
<li>Data center outages</li>
</ul>

<h3>
  
  
  What's Next: High Availability with Patroni
</h3>

<p>In our next article, we'll solve these availability challenges by integrating <strong>Patroni</strong>—an open-source high availability solution for PostgreSQL. We'll transform our vulnerable single-node cluster into a highly available system:</p>

<p><strong>Architecture we'll build:</strong></p>

<ul>
<li>
<strong>Coordinator cluster</strong>: Primary + Standby (automatic failover)</li>
<li>
<strong>Worker 1 cluster</strong>: Primary + Standby
</li>
<li>
<strong>Worker 2 cluster</strong>: Primary + Standby</li>
<li>
<strong>etcd cluster</strong>: Distributed consensus for leader election</li>
</ul>

<p><strong>What you'll learn:</strong></p>

<ul>
<li>Setting up Patroni with etcd for distributed consensus</li>
<li>Configuring automatic failover for coordinators and workers</li>
<li>Testing failure scenarios (simulating crashes)</li>
<li>Monitoring cluster health and replication lag</li>
<li>Performing switchovers for maintenance</li>
<li>Understanding trade-offs between availability and consistency</li>
</ul>

<p>By combining Citus's horizontal scalability with Patroni's high availability, you'll have a production-ready distributed database that can:</p>

<ul>
<li>Scale to billions of rows</li>
<li>Handle millions of queries per second</li>
<li>Survive node failures automatically</li>
<li>Support zero-downtime maintenance</li>
<li>Provide strong consistency guarantees</li>
</ul>

<p>Stay tuned for <strong>"High Availability for Citus: Implementing Automatic Failover with Patroni"</strong> where we'll make this scalable database truly resilient.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<strong>Citus Documentation</strong>: <a href="https://docs.citusdata.com" rel="noopener noreferrer">https://docs.citusdata.com</a>
</li>
<li>
<strong>Citus GitHub</strong>: <a href="https://github.com/citusdata/citus" rel="noopener noreferrer">https://github.com/citusdata/citus</a>
</li>
<li>
<strong>PostgreSQL Documentation</strong>: <a href="https://www.postgresql.org/docs/" rel="noopener noreferrer">https://www.postgresql.org/docs/</a>
</li>
<li>
<strong>Digital Ocean Citus Tutorial</strong>: <a href="https://docs.digitalocean.com/products/databases/postgresql/" rel="noopener noreferrer">https://docs.digitalocean.com/products/databases/postgresql/</a>
</li>
</ul>




<p><em>Special thanks to the Citus team for building such an elegant distributed database solution.</em></p>

