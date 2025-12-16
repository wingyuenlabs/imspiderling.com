---
Title: How I Built a Python Library That Lets You Join MySQL, PostgreSQL, MongoDB, REST APIs, and Files in a Single SQL Query
Description: 
Author: Theodore P.
Date: 2025-12-16T21:55:43.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Join Data from Anywhere: The Streaming SQL Engine That Bridges Databases, APIs, and Files
</h1>

<p>Have you ever needed to join data from a MySQL database with a PostgreSQL database, a MongoDB collection, and a REST API all in one query? Traditional databases can't do this. That's why I built the Streaming SQL Engine.</p>

<h2>
  
  
  The Problem: Data Lives Everywhere
</h2>

<p>Modern applications don't store all their data in one place. You might have:</p>

<ul>
<li>User data in PostgreSQL</li>
<li>Order data in MySQL</li>
<li>Product catalog in MongoDB</li>
<li>Pricing information from a REST API</li>
<li>Inventory data in CSV files</li>
<li>Product feeds in XML files</li>
</ul>

<p><strong>The challenge:</strong> How do you join all this data together?</p>

<p>Traditional solutions require:</p>

<ul>
<li>Exporting data from each system</li>
<li>Importing into a central database</li>
<li>Writing complex ETL pipelines</li>
<li>Maintaining data synchronization</li>
</ul>

<p><strong>There had to be a better way.</strong></p>

<h2>
  
  
  The Solution: Streaming SQL Engine
</h2>

<p>I built a lightweight Python library that lets you join data from <strong>any source</strong> using standard SQL syntax without exporting, importing, or setting up infrastructure.</p>

<h2>
  
  
  Example:
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">streaming_sql_engine</span> <span class="kn">import</span> <span class="n">Engine</span>
<span class="kn">import</span> <span class="n">psycopg2</span>
<span class="kn">import</span> <span class="n">pymysql</span>
<span class="kn">from</span> <span class="n">pymongo</span> <span class="kn">import</span> <span class="n">MongoClient</span>
<span class="kn">import</span> <span class="n">requests</span>
<span class="kn">import</span> <span class="n">csv</span>

<span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">()</span>

<span class="c1"># Register PostgreSQL source (iterator function)
</span><span class="k">def</span> <span class="nf">postgres_users</span><span class="p">():</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">psycopg2</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">localhost</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">mydb</span><span class="sh">"</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="sh">"</span><span class="s">pass</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT id, name, email FROM users</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cursor</span><span class="p">:</span>
        <span class="k">yield</span> <span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="sh">"</span><span class="s">email</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">2</span><span class="p">]}</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">postgres_users</span><span class="sh">"</span><span class="p">,</span> <span class="n">postgres_users</span><span class="p">)</span>

<span class="c1"># Register MySQL source (iterator function)
</span><span class="k">def</span> <span class="nf">mysql_products</span><span class="p">():</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">pymysql</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">localhost</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">mydb</span><span class="sh">"</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="sh">"</span><span class="s">pass</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT id, name, price FROM products</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cursor</span><span class="p">:</span>
        <span class="k">yield</span> <span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="sh">"</span><span class="s">price</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">2</span><span class="p">]}</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">mysql_products</span><span class="sh">"</span><span class="p">,</span> <span class="n">mysql_products</span><span class="p">)</span>

<span class="c1"># Register MongoDB source (iterator function)
</span><span class="k">def</span> <span class="nf">mongo_inventory</span><span class="p">():</span>
    <span class="n">client</span> <span class="o">=</span> <span class="nc">MongoClient</span><span class="p">(</span><span class="sh">"</span><span class="s">mongodb://localhost:27017</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">doc</span> <span class="ow">in</span> <span class="n">client</span><span class="p">.</span><span class="n">mydb</span><span class="p">.</span><span class="n">inventory</span><span class="p">.</span><span class="nf">find</span><span class="p">():</span>
        <span class="k">yield</span> <span class="n">doc</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">mongo_inventory</span><span class="sh">"</span><span class="p">,</span> <span class="n">mongo_inventory</span><span class="p">)</span>

<span class="c1"># Register REST API source (iterator function)
</span><span class="k">def</span> <span class="nf">api_prices</span><span class="p">():</span>
    <span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">https://api.example.com/prices</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="nf">json</span><span class="p">():</span>
        <span class="k">yield</span> <span class="n">item</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">api_prices</span><span class="sh">"</span><span class="p">,</span> <span class="n">api_prices</span><span class="p">)</span>

<span class="c1"># Register CSV source (iterator function)
</span><span class="k">def</span> <span class="nf">csv_suppliers</span><span class="p">():</span>
    <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">suppliers.csv</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">csv</span><span class="p">.</span><span class="nc">DictReader</span><span class="p">(</span><span class="n">f</span><span class="p">):</span>
            <span class="k">yield</span> <span class="n">row</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">csv_suppliers</span><span class="sh">"</span><span class="p">,</span> <span class="n">csv_suppliers</span><span class="p">)</span>

<span class="c1"># Join them all in one SQL query!
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    SELECT
        mysql_products.name,
        postgres_users.email,
        mongo_inventory.quantity,
        api_prices.price,
        csv_suppliers.supplier_name
    FROM mysql_products
    JOIN postgres_users ON mysql_products.user_id = postgres_users.id
    JOIN mongo_inventory ON mysql_products.sku = mongo_inventory.sku
    JOIN api_prices ON mysql_products.sku = api_prices.sku
    JOIN csv_suppliers ON mysql_products.supplier_id = csv_suppliers.id
    WHERE api_prices.price &gt; 100
</span><span class="sh">"""</span>

<span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">engine</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="n">query</span><span class="p">):</span>
    <span class="nf">process</span><span class="p">(</span><span class="n">row</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>That's it.</strong> No clusters, no infrastructure, no data export - just pure Python and SQL.</p>

<h2>
  
  
  Why I Built This: The Problem That Needed Solving
</h2>

<p>I was working on a data reconciliation project where I needed to join data from multiple sources:</p>

<ul>
<li>MySQL database (product catalog)</li>
<li>PostgreSQL database (user data)</li>
<li>MongoDB collection (inventory)</li>
<li>REST API (pricing information)</li>
<li>CSV files (supplier data)</li>
</ul>

<p><strong>The challenge:</strong> Traditional databases can't join across different systems. I had three options:</p>

<ol>
<li>
<strong>Export everything to one database</strong> - Time-consuming, requires ETL pipelines, data becomes stale</li>
<li>
<strong>Write custom Python code</strong> - Complex, error-prone, hard to maintain</li>
<li>
<strong>Use existing tools</strong> - Spark/Flink require clusters, DuckDB requires data import, Presto needs infrastructure</li>
</ol>

<p><strong>None of these worked for my use case.</strong> I needed something that:</p>

<ul>
<li>Could join data from different systems without export</li>
<li>Was simple to use (SQL syntax)</li>
<li>Required zero infrastructure</li>
<li>Worked with Python natively</li>
<li>Processed data efficiently (streaming)</li>
</ul>

<p>So I built the Streaming SQL Engine.</p>

<h2>
  
  
  How I Built It: Architecture and Design Decisions
</h2>

<h3>
  
  
  Core Design Philosophy
</h3>

<p>The engine follows a <strong>pipeline architecture</strong> inspired by database query execution engines (like PostgreSQL and SQLite), but implemented in pure Python using iterators.</p>

<p><strong>Key insight:</strong> Python's iterator protocol is perfect for streaming data. Each operator in the pipeline is an iterator that processes rows one at a time.</p>

<h3>
  
  
  Why Iterator-Based Architecture?
</h3>

<p><strong>Key advantages:</strong></p>

<ol>
<li>
<strong>Memory efficiency:</strong> Only one row in memory at a time (except for join indexes)</li>
<li>
<strong>Lazy evaluation:</strong> Processing starts only when you iterate over results</li>
<li>
<strong>Composability:</strong> Operators can be chained arbitrarily</li>
<li>
<strong>Extensibility:</strong> Easy to add new operators</li>
<li>
<strong>Python-native:</strong> Uses standard Python iterator protocol</li>
</ol>

<p><strong>Trade-offs:</strong></p>

<ul>
<li>
<strong>Performance:</strong> Python iterators are slower than compiled code, but flexibility is worth it</li>
<li>
<strong>Memory:</strong> Join indexes require memory, but this is necessary for efficient joins</li>
<li>
<strong>Complexity:</strong> Iterator chains can be complex, but they're composable and testable</li>
</ul>

<h3>
  
  
  Expression Evaluation
</h3>

<p><strong>How WHERE clauses are evaluated:</strong></p>

<p>The engine uses recursive AST traversal to evaluate expressions:</p>

<p><strong>Why recursive:</strong> SQL expressions are trees. Recursive evaluation naturally handles nested expressions like <code>(a &gt; 10 AND b &lt; 20) OR c = 5</code>.</p>

<h3>
  
  
  Join Algorithm Selection
</h3>

<p><strong>How the engine chooses join algorithms:</strong></p>

<p>The engine follows this priority order when selecting a join algorithm:</p>

<ol>
<li>
<strong>Check if both sides are sorted</strong> (<code>ordered_by</code> metadata)</li>
</ol>

<ul>
<li>If yes, use <code>MergeJoinIterator</code> (most memory-efficient for sorted data)</li>
<li>Only used when <code>use_polars=False</code>
</li>
</ul>

<ol>
<li>
<strong>Check if right side is a file</strong> (<code>filename</code> metadata)</li>
</ol>

<ul>
<li>If yes, use <code>MmapLookupJoinIterator</code> (memory-mapped, 90-99% memory reduction)</li>
<li>Only used when <code>use_polars=False</code>
</li>
</ul>

<ol>
<li>
<strong>Check if Polars is available</strong> (<code>use_polars</code> flag)</li>
</ol>

<ul>
<li>If yes, use <code>PolarsLookupJoinIterator</code> (vectorized, SIMD-accelerated)</li>
<li>Used when <code>use_polars=True</code> is explicitly set</li>
</ul>

<ol>
<li>
<strong>Default fallback</strong>

<ul>
<li>Use <code>LookupJoinIterator</code> (Python hash-based, most compatible)</li>
</ul>
</li>
</ol>

<p><strong>Logic:</strong></p>

<p><strong>Note:</strong> When <code>use_polars=False</code> (default), Merge Join and MMAP Join are checked first before falling back to Python Lookup Join. When <code>use_polars=True</code> is explicitly set, the engine prioritizes Polars over MMAP and Merge Join.</p>

<h3>
  
  
  Protocol-Based Optimization
</h3>

<p><strong>The key innovation:</strong> Automatic optimization detection via function signature inspection.</p>

<p><strong>How it works:</strong></p>

<ol>
<li>Engine inspects source function signature using <code>inspect.signature()</code>
</li>
<li>If function accepts <code>dynamic_where</code> or <code>dynamic_columns</code> parameters, it supports optimizations</li>
<li>Engine passes optimization parameters automatically</li>
<li>Source function applies optimizations (filter pushdown, column pruning)</li>
</ol>

<p><strong>Why this approach:</strong></p>

<ul>
<li>
<strong>No flags needed:</strong> Detection is automatic</li>
<li>
<strong>Flexible:</strong> Any Python function can be a source</li>
<li>
<strong>Backward compatible:</strong> Simple sources still work (no optimizations)</li>
<li>
<strong>Extensible:</strong> Easy to add new optimization parameters</li>
</ul>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Simple source (no optimizations)
</span><span class="k">def</span> <span class="nf">simple_source</span><span class="p">():</span>
    <span class="k">return</span> <span class="nf">iter</span><span class="p">([{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Alice</span><span class="sh">"</span><span class="p">}])</span>

<span class="c1"># Optimized source (with protocol)
</span><span class="k">def</span> <span class="nf">optimized_source</span><span class="p">(</span><span class="n">dynamic_where</span><span class="o">=</span><span class="bp">None</span><span class="p">,</span> <span class="n">dynamic_columns</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
    <span class="n">query</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SELECT </span><span class="sh">"</span>
    <span class="k">if</span> <span class="n">dynamic_columns</span><span class="p">:</span>
        <span class="n">query</span> <span class="o">+=</span> <span class="sh">"</span><span class="s">, </span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">dynamic_columns</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">query</span> <span class="o">+=</span> <span class="sh">"</span><span class="s">*</span><span class="sh">"</span>
    <span class="n">query</span> <span class="o">+=</span> <span class="sh">"</span><span class="s"> FROM table</span><span class="sh">"</span>
    <span class="k">if</span> <span class="n">dynamic_where</span><span class="p">:</span>
        <span class="n">query</span> <span class="o">+=</span> <span class="sa">f</span><span class="sh">"</span><span class="s"> WHERE </span><span class="si">{</span><span class="n">dynamic_where</span><span class="si">}</span><span class="sh">"</span>
    <span class="k">return</span> <span class="nf">execute_query</span><span class="p">(</span><span class="n">query</span><span class="p">)</span>

<span class="c1"># Both work the same way:
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">users</span><span class="sh">"</span><span class="p">,</span> <span class="n">simple_source</span><span class="p">)</span>  <span class="c1"># No optimizations
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">optimized_source</span><span class="p">)</span>  <span class="c1"># Optimizations apply automatically!
</span></code></pre>

</div>



<h3>
  
  
  Memory Management
</h3>

<p><strong>Key strategies:</strong></p>

<ol>
<li>
<strong>Streaming:</strong> Process one row at a time, never load full tables</li>
<li>
<strong>Join indexes:</strong> Only right side of joins is materialized (necessary for lookups)</li>
<li>
<strong>Memory-mapped files:</strong> For large JSONL files, use OS virtual memory</li>
<li>
<strong>Column pruning:</strong> Only extract needed columns</li>
<li>
<strong>Filter pushdown:</strong> Filter at source, reduce data transfer</li>
</ol>

<p><strong>Memory footprint:</strong></p>

<ul>
<li>
<strong>Left side of join:</strong> O(1) - one row at a time</li>
<li>
<strong>Right side of join:</strong> O(n) - hash index in memory</li>
<li>
<strong>Total:</strong> O(n) where n = size of right side</li>
</ul>

<p><strong>Why this works:</strong> In most queries, the right side is smaller (e.g., joining large product table with small category table). The engine is designed to put smaller tables on the right side.</p>

<h3>
  
  
  Performance Optimizations
</h3>

<p><strong>1. Polars Vectorization</strong></p>

<p>When Polars is available, the engine uses vectorized operations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Instead of row-by-row filtering:
</span><span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">rows</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">row</span><span class="p">[</span><span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">]</span> <span class="o">&gt;</span> <span class="mi">100</span><span class="p">:</span>
        <span class="k">yield</span> <span class="n">row</span>

<span class="c1"># Use Polars batch filtering:
</span><span class="n">df</span> <span class="o">=</span> <span class="n">pl</span><span class="p">.</span><span class="nc">DataFrame</span><span class="p">(</span><span class="n">rows</span><span class="p">)</span>
<span class="n">filtered_df</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="n">pl</span><span class="p">.</span><span class="nf">col</span><span class="p">(</span><span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">100</span><span class="p">)</span>
<span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">filtered_df</span><span class="p">.</span><span class="nf">iter_rows</span><span class="p">(</span><span class="n">named</span><span class="o">=</span><span class="bp">True</span><span class="p">):</span>
    <span class="k">yield</span> <span class="n">row</span>
</code></pre>

</div>



<p><strong>Why faster:</strong> Polars uses SIMD instructions and columnar processing, 10-200x faster than row-by-row Python loops.</p>

<p><strong>2. Memory-Mapped Joins</strong></p>

<p>For large JSONL files, use OS virtual memory instead of loading into RAM:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Instead of loading entire file:
</span><span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">'</span><span class="s">large_file.jsonl</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">data</span> <span class="o">=</span> <span class="p">[</span><span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">line</span><span class="p">)</span> <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">f</span><span class="p">]</span>  <span class="c1"># 10GB in memory!
</span>
<span class="c1"># Use memory-mapped file:
</span><span class="kn">import</span> <span class="n">mmap</span>
<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">'</span><span class="s">large_file.jsonl</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">rb</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">mm</span> <span class="o">=</span> <span class="n">mmap</span><span class="p">.</span><span class="nf">mmap</span><span class="p">(</span><span class="n">f</span><span class="p">.</span><span class="nf">fileno</span><span class="p">(),</span> <span class="mi">0</span><span class="p">,</span> <span class="n">access</span><span class="o">=</span><span class="n">mmap</span><span class="p">.</span><span class="n">ACCESS_READ</span><span class="p">)</span>
    <span class="c1"># OS handles memory, can process files larger than RAM
</span></code></pre>

</div>



<p><strong>Why this works:</strong> OS virtual memory allows accessing file data without loading it all into RAM. 90-99% memory reduction for large files.</p>

<p><strong>3. First Match Only Optimization</strong></p>

<p>For joins where only the first match matters (prevents Cartesian products).</p>

<p><strong>Why useful:</strong> Prevents Cartesian products when right side has duplicate keys. Significantly reduces output size and processing time.</p>

<p><strong>4. Column Pruning</strong></p>

<p>Only extracts columns needed for the query, reducing I/O and memory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Query only requests 'name' and 'price'
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SELECT name, price FROM products</span><span class="sh">"</span>

<span class="c1"># Engine automatically requests only these columns from source
</span><span class="k">def</span> <span class="nf">source</span><span class="p">(</span><span class="n">dynamic_columns</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
    <span class="n">columns</span> <span class="o">=</span> <span class="sh">"</span><span class="s">, </span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">dynamic_columns</span><span class="p">)</span>  <span class="c1"># ['name', 'price']
</span>    <span class="n">query</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">SELECT </span><span class="si">{</span><span class="n">columns</span><span class="si">}</span><span class="s"> FROM table</span><span class="sh">"</span>
</code></pre>

</div>



<p><strong>5. Filter Pushdown</strong></p>

<p>Pushes WHERE conditions to data sources when possible:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Query has WHERE clause
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SELECT * FROM products WHERE price &gt; 100</span><span class="sh">"</span>

<span class="c1"># Engine automatically pushes filter to source
</span><span class="k">def</span> <span class="nf">source</span><span class="p">(</span><span class="n">dynamic_where</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
    <span class="n">query</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">SELECT * FROM table WHERE </span><span class="si">{</span><span class="n">dynamic_where</span><span class="si">}</span><span class="sh">"</span>
</code></pre>

</div>



<p><strong>6. Merge Joins</strong></p>

<p>Efficient joins for pre-sorted data (O(n+m) time complexity):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Register with ordered_by to enable merge join
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">users</span><span class="sh">"</span><span class="p">,</span> <span class="n">users_source</span><span class="p">,</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Engine uses merge join algorithm
# Both sides must be sorted by join key
</span></code></pre>

</div>



<h3>
  
  
  Design Decisions and Trade-offs
</h3>

<p><strong>1. Why Python iterators instead of compiled code?</strong></p>

<p><strong>Decision:</strong> Use Python iterators for flexibility</p>

<p><strong>Trade-off:</strong> Slower than compiled code, but:</p>

<ul>
<li>Works with any Python data source</li>
<li>Easy to extend and customize</li>
<li>No compilation step needed</li>
<li>Python-native integration</li>
</ul>

<p><strong>2. Why separate logical planning from execution?</strong></p>

<p><strong>Decision:</strong> Two-phase approach (plan then execute)</p>

<p><strong>Trade-off:</strong> Extra step, but:</p>

<ul>
<li>Enables query optimization</li>
<li>Easier to test and debug</li>
<li>Can reuse planning logic</li>
<li>Clear separation of concerns</li>
</ul>

<p><strong>3. Why protocol-based optimization instead of flags?</strong></p>

<p><strong>Decision:</strong> Automatic detection via function signature</p>

<p><strong>Trade-off:</strong> Slightly more complex detection, but:</p>

<ul>
<li>No flags to remember</li>
<li>Automatic optimization</li>
<li>Backward compatible</li>
<li>More Pythonic</li>
</ul>

<p><strong>4. Why materialize right side of joins?</strong></p>

<p><strong>Decision:</strong> Build hash index on right side</p>

<p><strong>Trade-off:</strong> Memory usage, but:</p>

<ul>
<li>O(1) lookup time per left row</li>
<li>Necessary for efficient joins</li>
<li>Can use memory-mapped files for large files</li>
<li>Standard database approach</li>
</ul>

<p><strong>5. Why limit SQL features (no GROUP BY, aggregations)?</strong></p>

<p><strong>Decision:</strong> Focus on joins and filtering</p>

<p><strong>Trade-off:</strong> Less SQL support, but:</p>

<ul>
<li>Simpler implementation</li>
<li>Faster execution</li>
<li>Focuses on core use case (cross-system joins)</li>
<li>Can add later if needed</li>
</ul>

<h3>
  
  
  The Result
</h3>

<p>A lightweight Python library that:</p>

<ul>
<li>Joins data from any source using SQL</li>
<li>Processes data row-by-row (streaming)</li>
<li>Requires zero infrastructure</li>
<li>Automatically optimizes when possible</li>
<li>Works with any Python iterator</li>
</ul>

<p><strong>The key insight:</strong> Python's iterator protocol is perfect for streaming SQL execution. By combining SQL parsing, logical planning, and iterator-based execution, I created a tool that solves a real problem: joining data from different systems without complex infrastructure.</p>

<h2>
  
  
  How It Works: Streaming Architecture
</h2>

<p>The engine processes data <strong>row-by-row</strong>, never loading entire tables into memory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SQL Query
    |
Parser -&gt; AST
    |
Planner -&gt; Logical Plan
    |
Executor -&gt; Iterator Pipeline
    |
Results (Generator)
</code></pre>

</div>



<p><strong>Iterator Pipeline:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ScanIterator -&gt; FilterIterator -&gt; JoinIterators -&gt; ProjectIterator -&gt; Results
</code></pre>

</div>



<p>Each iterator processes rows incrementally, enabling true streaming execution. This means:</p>

<ul>
<li>Low memory footprint</li>
<li>Can process data larger than RAM</li>
<li>Results yielded immediately</li>
<li>No buffering required</li>
</ul>

<h2>
  
  
  Supported Data Sources
</h2>

<p>The engine works with <strong>any Python iterator</strong>, making it incredibly flexible:</p>

<h3>
  
  
  Databases
</h3>

<p>All databases are accessed via Python iterator functions. The engine doesn't use connectors - it works with any Python function that returns an iterator:</p>

<ul>
<li>
<strong>PostgreSQL</strong> - Create iterator function that queries PostgreSQL and yields rows</li>
<li>
<strong>MySQL</strong> - Create iterator function that queries MySQL and yields rows</li>
<li>
<strong>MongoDB</strong> - Create iterator function that queries MongoDB and yields documents</li>
</ul>

<h3>
  
  
  Files
</h3>

<ul>
<li>
<strong>CSV</strong> - Standard CSV files</li>
<li>
<strong>JSONL</strong> - JSON Lines format with memory-mapped joins</li>
<li>
<strong>JSON</strong> - Standard JSON files</li>
<li>
<strong>XML</strong> - XML parsing with ElementTree</li>
</ul>

<h3>
  
  
  APIs
</h3>

<ul>
<li>
<strong>REST APIs</strong> - Any HTTP endpoint</li>
<li>
<strong>GraphQL</strong> - Via custom functions</li>
<li>
<strong>WebSockets</strong> - Streaming data sources</li>
</ul>

<h3>
  
  
  Custom Sources
</h3>

<ul>
<li>
<strong>Any Python function</strong> that returns an iterator</li>
<li>
<strong>Generators</strong> - Perfect for streaming data</li>
<li>
<strong>Custom transformations</strong> - Apply Python logic between joins</li>
</ul>

<h2>
  
  
  SQL Features
</h2>

<p>The engine supports standard SQL syntax:</p>

<h3>
  
  
  Supported Features
</h3>

<p><strong>SELECT</strong> - Column selection, aliasing, table-qualified columns<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">users</span><span class="p">.</span><span class="n">name</span><span class="p">,</span> <span class="n">orders</span><span class="p">.</span><span class="n">total</span> <span class="k">AS</span> <span class="n">order_total</span>
<span class="k">FROM</span> <span class="n">users</span>
<span class="k">JOIN</span> <span class="n">orders</span> <span class="k">ON</span> <span class="n">users</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="n">orders</span><span class="p">.</span><span class="n">user_id</span>
</code></pre>

</div>



<p><strong>JOIN</strong> - INNER JOIN and LEFT JOIN with equality conditions<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span>
<span class="k">FROM</span> <span class="n">table1</span> <span class="n">t1</span>
<span class="k">JOIN</span> <span class="n">table2</span> <span class="n">t2</span> <span class="k">ON</span> <span class="n">t1</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="n">t2</span><span class="p">.</span><span class="n">id</span>
<span class="k">LEFT</span> <span class="k">JOIN</span> <span class="n">table3</span> <span class="n">t3</span> <span class="k">ON</span> <span class="n">t1</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="n">t3</span><span class="p">.</span><span class="n">id</span>
</code></pre>

</div>



<p><strong>WHERE</strong> - Comparisons, boolean logic, NULL checks, IN clauses<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span>
<span class="k">FROM</span> <span class="n">products</span>
<span class="k">WHERE</span> <span class="n">price</span> <span class="o">&gt;</span> <span class="mi">100</span>
  <span class="k">AND</span> <span class="n">status</span> <span class="k">IN</span> <span class="p">(</span><span class="s1">'active'</span><span class="p">,</span> <span class="s1">'pending'</span><span class="p">)</span>
  <span class="k">AND</span> <span class="n">description</span> <span class="k">IS</span> <span class="k">NOT</span> <span class="k">NULL</span>
</code></pre>

</div>



<p><strong>Arithmetic</strong> - Addition, subtraction, multiplication, division, modulo<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span>
  <span class="n">price</span> <span class="o">-</span> <span class="n">discount</span> <span class="k">AS</span> <span class="n">final_price</span><span class="p">,</span>
  <span class="n">quantity</span> <span class="o">*</span> <span class="n">unit_price</span> <span class="k">AS</span> <span class="n">total</span>
<span class="k">FROM</span> <span class="n">orders</span>
</code></pre>

</div>



<h3>
  
  
  Not Supported
</h3>

<ul>
<li>GROUP BY and aggregations (COUNT, SUM, AVG)</li>
<li>ORDER BY</li>
<li>HAVING</li>
<li>Subqueries</li>
</ul>

<p><em>These limitations keep the engine focused on joins and filtering - its core strength.</em></p>

<h2>
  
  
  Real-World Examples
</h2>

<h3>
  
  
  Example 1: Microservices Data Integration
</h3>

<p>In a microservices architecture, data is distributed across services:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">streaming_sql_engine</span> <span class="kn">import</span> <span class="n">Engine</span>
<span class="kn">import</span> <span class="n">psycopg2</span>
<span class="kn">import</span> <span class="n">pymysql</span>
<span class="kn">import</span> <span class="n">requests</span>

<span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">()</span>

<span class="c1"># Service 1: User service (PostgreSQL) - iterator function
</span><span class="k">def</span> <span class="nf">users_source</span><span class="p">():</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">psycopg2</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">user-db</span><span class="sh">"</span><span class="p">,</span> <span class="n">port</span><span class="o">=</span><span class="mi">5432</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="sh">"</span><span class="s">pass</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">users_db</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT id, name, email FROM users</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cursor</span><span class="p">:</span>
        <span class="k">yield</span> <span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="sh">"</span><span class="s">email</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">2</span><span class="p">]}</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">users</span><span class="sh">"</span><span class="p">,</span> <span class="n">users_source</span><span class="p">)</span>

<span class="c1"># Service 2: Order service (MySQL) - iterator function
</span><span class="k">def</span> <span class="nf">orders_source</span><span class="p">():</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">pymysql</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">order-db</span><span class="sh">"</span><span class="p">,</span> <span class="n">port</span><span class="o">=</span><span class="mi">3306</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="sh">"</span><span class="s">pass</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">orders_db</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT id, user_id, total FROM orders</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cursor</span><span class="p">:</span>
        <span class="k">yield</span> <span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">user_id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="sh">"</span><span class="s">total</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">2</span><span class="p">]}</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">orders</span><span class="sh">"</span><span class="p">,</span> <span class="n">orders_source</span><span class="p">)</span>

<span class="c1"># Service 3: Payment service (REST API) - iterator function
</span><span class="k">def</span> <span class="nf">payment_source</span><span class="p">():</span>
    <span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">https://payments.service/api/transactions</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="nf">json</span><span class="p">():</span>
        <span class="k">yield</span> <span class="n">item</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">payments</span><span class="sh">"</span><span class="p">,</span> <span class="n">payment_source</span><span class="p">)</span>

<span class="c1"># Join across services
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    SELECT users.name, orders.total, payments.status
    FROM users
    JOIN orders ON users.id = orders.user_id
    JOIN payments ON orders.id = payments.order_id
</span><span class="sh">"""</span>
</code></pre>

</div>



<p><strong>Why this matters:</strong> No need for a shared database or complex ETL pipelines. The engine accepts any Python function that returns an iterator, making it incredibly flexible.</p>

<h3>
  
  
  Example 2: Real-Time Price Comparison
</h3>

<p>Compare prices from multiple XML feeds and match with MongoDB:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">parse_xml</span><span class="p">(</span><span class="n">filepath</span><span class="p">):</span>
    <span class="n">tree</span> <span class="o">=</span> <span class="n">ET</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="n">filepath</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">product</span> <span class="ow">in</span> <span class="n">tree</span><span class="p">.</span><span class="nf">findall</span><span class="p">(</span><span class="sh">'</span><span class="s">.//product</span><span class="sh">'</span><span class="p">):</span>
        <span class="k">yield</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">ean</span><span class="sh">'</span><span class="p">:</span> <span class="n">product</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="sh">'</span><span class="s">ean</span><span class="sh">'</span><span class="p">).</span><span class="n">text</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">product</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">).</span><span class="n">text</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">:</span> <span class="n">product</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">).</span><span class="n">text</span>
        <span class="p">}</span>

<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">xml1</span><span class="sh">"</span><span class="p">,</span> <span class="k">lambda</span><span class="p">:</span> <span class="nf">parse_xml</span><span class="p">(</span><span class="sh">"</span><span class="s">prices1.xml</span><span class="sh">"</span><span class="p">))</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">xml2</span><span class="sh">"</span><span class="p">,</span> <span class="k">lambda</span><span class="p">:</span> <span class="nf">parse_xml</span><span class="p">(</span><span class="sh">"</span><span class="s">prices2.xml</span><span class="sh">"</span><span class="p">))</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">mongo</span><span class="sh">"</span><span class="p">,</span> <span class="n">mongo_source</span><span class="p">)</span>

<span class="n">query</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    SELECT
        xml1.ean,
        xml1.price AS price1,
        xml2.price AS price2,
        mongo.sf_sku
    FROM xml1
    JOIN xml2 ON xml1.ean = xml2.ean
    JOIN mongo ON xml1.ean = mongo.ean
    WHERE xml1.price != xml2.price
</span><span class="sh">"""</span>
</code></pre>

</div>



<h3>
  
  
  Example 3: Python Processing Between Joins
</h3>

<p>Apply Python logic (ML models, custom functions) between joins:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">enriched_source</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Source that processes data with Python before joining</span><span class="sh">"""</span>
    <span class="kn">import</span> <span class="n">psycopg2</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">psycopg2</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">localhost</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">mydb</span><span class="sh">"</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="sh">"</span><span class="s">pass</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT id, name, category_id FROM products</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cursor</span><span class="p">:</span>
        <span class="n">product</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="sh">"</span><span class="s">category_id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">2</span><span class="p">]}</span>
        <span class="c1"># Apply Python logic
</span>        <span class="n">product</span><span class="p">[</span><span class="sh">'</span><span class="s">ml_score</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">ml_model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">product</span><span class="p">)</span>
        <span class="n">product</span><span class="p">[</span><span class="sh">'</span><span class="s">custom_field</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="nf">custom_function</span><span class="p">(</span><span class="n">product</span><span class="p">)</span>
        <span class="k">yield</span> <span class="n">product</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>

<span class="k">def</span> <span class="nf">categories_source</span><span class="p">():</span>
    <span class="kn">import</span> <span class="n">psycopg2</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">psycopg2</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">localhost</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">mydb</span><span class="sh">"</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="sh">"</span><span class="s">pass</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT id, name FROM categories</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cursor</span><span class="p">:</span>
        <span class="k">yield</span> <span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">category_name</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">]}</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>

<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">enriched_products</span><span class="sh">"</span><span class="p">,</span> <span class="n">enriched_source</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">categories</span><span class="sh">"</span><span class="p">,</span> <span class="n">categories_source</span><span class="p">)</span>

<span class="n">query</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    SELECT p.name, p.ml_score, c.category_name
    FROM enriched_products p
    JOIN categories c ON p.category_id = c.id
</span><span class="sh">"""</span>
</code></pre>

</div>



<p><strong>Why this matters:</strong> Seamless integration with Python ecosystem - use any library, apply any logic.</p>

<h2>
  
  
  Comparison with Alternatives
</h2>

<h3>
  
  
  vs DuckDB
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Streaming SQL Engine</th>
<th>DuckDB</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cross-system joins</td>
<td>✅ Direct</td>
<td>⚠️ Requires import</td>
</tr>
<tr>
<td>API + Database join</td>
<td>✅ Direct</td>
<td>❌ Must export API</td>
</tr>
<tr>
<td>Real-time streaming</td>
<td>✅ True streaming</td>
<td>⚠️ Buffering needed</td>
</tr>
<tr>
<td>Python processing</td>
<td>✅ Native</td>
<td>⚠️ Export/import</td>
</tr>
<tr>
<td>GROUP BY / Aggregations</td>
<td>❌ Not supported</td>
<td>✅ Full support</td>
</tr>
<tr>
<td>Performance (same DB)</td>
<td>⚠️ Moderate</td>
<td>✅ Very fast</td>
</tr>
</tbody>
</table></div>

<p><strong>Use Streaming SQL Engine when:</strong> You need to join data from different systems that can't be imported into DuckDB.</p>

<p><strong>Use DuckDB when:</strong> All data can be imported and you need aggregations.</p>

<h3>
  
  
  vs Pandas
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Streaming SQL Engine</th>
<th>Pandas</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cross-system joins</td>
<td>✅ Direct</td>
<td>❌ Must load all data</td>
</tr>
<tr>
<td>Memory efficiency</td>
<td>✅ Streaming</td>
<td>❌ Loads all in memory</td>
</tr>
<tr>
<td>SQL syntax</td>
<td>✅ Standard SQL</td>
<td>❌ DataFrame API</td>
</tr>
<tr>
<td>Large datasets</td>
<td>✅ Can exceed RAM</td>
<td>❌ Limited by RAM</td>
</tr>
<tr>
<td>Real-time data</td>
<td>✅ Streaming</td>
<td>❌ Batch only</td>
</tr>
<tr>
<td>File formats</td>
<td>✅ Any Python iterator</td>
<td>⚠️ Limited formats</td>
</tr>
<tr>
<td>Performance (large data)</td>
<td>✅ Streaming</td>
<td>⚠️ Slower for large data</td>
</tr>
</tbody>
</table></div>

<p><strong>Use Streaming SQL Engine when:</strong> You need to join data from different systems, process data larger than RAM, or use SQL syntax.</p>

<p><strong>Use Pandas when:</strong> All data fits in memory and you prefer DataFrame API.</p>

<h2>
  
  
  Are There Other Tools Like This?
</h2>

<p><strong>Short answer: Not exactly.</strong></p>

<p>While there are many tools that do <strong>parts</strong> of what Streaming SQL Engine does, none combine all these characteristics:</p>

<h3>
  
  
  What Makes Streaming SQL Engine Unique
</h3>

<ol>
<li><strong>Zero Infrastructure + Cross-System Joins</strong></li>
</ol>

<ul>
<li>Most tools require clusters (Spark, Flink, Drill)</li>
<li>Or require specific infrastructure (ksqlDB needs Kafka)</li>
<li>Streaming SQL Engine: Just Python</li>
</ul>

<ol>
<li><strong>Any Python Iterator as Data Source</strong></li>
</ol>

<ul>
<li>Most tools require specific connectors</li>
<li>Streaming SQL Engine: Any Python function works</li>
</ul>

<ol>
<li><strong>Direct API Joins</strong></li>
</ol>

<ul>
<li>Most tools can't join REST APIs directly</li>
<li>Streaming SQL Engine: Native support</li>
</ul>

<ol>
<li>
<strong>Python-Native Architecture</strong>

<ul>
<li>Most tools are Java/Rust with Python wrappers</li>
<li>Streaming SQL Engine: Pure Python, seamless integration</li>
</ul>
</li>
</ol>

<h3>
  
  
  Similar Tools (But Different)
</h3>

<p><strong>Apache Drill</strong> - Similar cross-system capability, but requires cluster and Java</p>

<p><strong>ksqlDB</strong> - Streaming SQL, but Kafka-only and requires infrastructure</p>

<p><strong>Materialize</strong> - Streaming database, but requires database server</p>

<p><strong>DataFusion</strong> - Fast SQL engine, but limited to Arrow/Parquet data</p>

<p><strong>Polars SQL</strong> - Fast SQL, but requires loading data into DataFrames first</p>

<p><strong>Presto/Trino</strong> - Cross-system SQL, but requires cluster infrastructure</p>

<p><strong>None of these</strong> combine:</p>

<ul>
<li>Zero infrastructure</li>
<li>Any Python iterator as source</li>
<li>Direct API joins</li>
<li>Pure Python implementation</li>
<li>Simple deployment</li>
</ul>

<p><strong>That's what makes Streaming SQL Engine unique.</strong></p>

<h2>
  
  
  Start Here: The Most Secure Way
</h2>

<h3>
  
  
  Step 1: Install and Basic Setup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>streaming-sql-engine
</code></pre>

</div>



<h2>
  
  
  Two Stable Join Options
</h2>

<p>The engine provides two stable join algorithms that work reliably with any data:</p>

<h3>
  
  
  Option 1: Lookup Join (Default)
</h3>

<p><strong>What it is:</strong> Python hash-based join - the default when no special metadata is provided.</p>

<p><strong>How to use:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">()</span>  <span class="c1"># use_polars=False (default)
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">products_source</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="n">images_source</span><span class="p">)</span>
<span class="c1"># No special metadata - uses Lookup Join automatically
</span></code></pre>

</div>



<p><strong>How it works:</strong></p>

<ol>
<li>Builds hash index on right side (loads all right rows into memory)</li>
<li>Streams left side row-by-row</li>
<li>Looks up matches in hash index (O(1) per lookup)</li>
</ol>

<p><strong>Performance (100K products, 300K images):</strong></p>

<ul>
<li>
<strong>Rows processed:</strong> 299,553</li>
<li>
<strong>Time:</strong> 39.63 seconds</li>
<li>
<strong>Memory overhead:</strong> 177.11 MB</li>
<li>
<strong>CPU:</strong> 2.8%</li>
<li>
<strong>Throughput:</strong> 7,559 rows/second</li>
</ul>

<p><strong>When to use:</strong></p>

<ul>
<li>Default choice - works with any data</li>
<li>Small to medium datasets (&lt; 1M rows)</li>
<li>When right side fits in memory</li>
<li>When data is not sorted</li>
</ul>

<p><strong>Pros:</strong></p>

<ul>
<li>✅ Most compatible (works with any data types)</li>
<li>✅ No special requirements</li>
<li>✅ Reliable and stable</li>
<li>✅ Good performance for medium datasets</li>
</ul>

<p><strong>Cons:</strong></p>

<ul>
<li>⚠️ Loads entire right side into memory</li>
<li>⚠️ Not optimal for very large right tables</li>
</ul>

<h3>
  
  
  Option 2: Merge Join (Sorted Data)
</h3>

<p><strong>What it is:</strong> Streaming merge join for pre-sorted data - most memory-efficient join algorithm.</p>

<p><strong>How to use:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">()</span>  <span class="c1"># use_polars=False (required for Merge Join)
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">products_source</span><span class="p">,</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="n">images_source</span><span class="p">,</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>
<span class="c1"># Both sides must be sorted by join key
</span></code></pre>

</div>



<p><strong>How it works:</strong></p>

<ol>
<li>Both sides stream simultaneously (no index needed)</li>
<li>Compares join keys and advances iterators</li>
<li>When keys match, joins the rows</li>
<li>Only buffers small groups of duplicate keys</li>
</ol>

<p><strong>Performance (100K products, 300K images, sorted):</strong></p>

<ul>
<li>
<strong>Rows processed:</strong> 179,871</li>
<li>
<strong>Time:</strong> 24.79 seconds</li>
<li>
<strong>Memory overhead:</strong> 0.87 MB (lowest!)</li>
<li>
<strong>CPU:</strong> 3.5%</li>
<li>
<strong>Throughput:</strong> 7,256 rows/second</li>
</ul>

<p><strong>When to use:</strong></p>

<ul>
<li>Data is already sorted (or can be sorted once)</li>
<li>Memory-constrained environments</li>
<li>Very large datasets (&gt; 1M rows)</li>
<li>When you want maximum memory efficiency</li>
</ul>

<p><strong>Pros:</strong></p>

<ul>
<li>✅ Lowest memory usage (O(1) - only buffers current rows)</li>
<li>✅ Fast for sorted data (O(n+m) single pass)</li>
<li>✅ True streaming (both sides stream simultaneously)</li>
<li>✅ Can handle datasets larger than RAM</li>
</ul>

<p><strong>Cons:</strong></p>

<ul>
<li>⚠️ Requires pre-sorted data</li>
<li>⚠️ Must specify <code>ordered_by</code> metadata</li>
<li>⚠️ Requires <code>use_polars=False</code>
</li>
</ul>

<p><strong>Example: Preparing Data for Merge Join</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Step 1: Sort your JSONL files
# Use the provided utility:
</span><span class="n">python</span> <span class="n">examples</span><span class="o">/</span><span class="n">sort_jsonl</span><span class="p">.</span><span class="n">py</span> <span class="n">products</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">products_sorted</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">product_id</span>
<span class="n">python</span> <span class="n">examples</span><span class="o">/</span><span class="n">sort_jsonl</span><span class="p">.</span><span class="n">py</span> <span class="n">images</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">images_sorted</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">product_id</span>

<span class="c1"># Step 2: Use sorted files with Merge Join
</span><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="k">lambda</span><span class="p">:</span> <span class="nf">load_jsonl</span><span class="p">(</span><span class="sh">"</span><span class="s">products_sorted.jsonl</span><span class="sh">"</span><span class="p">),</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="k">lambda</span><span class="p">:</span> <span class="nf">load_jsonl</span><span class="p">(</span><span class="sh">"</span><span class="s">images_sorted.jsonl</span><span class="sh">"</span><span class="p">),</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Step 3: Query - Merge Join will be used automatically
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    SELECT products.product_id, products.title, images.image
    FROM products
    LEFT JOIN images ON products.product_id = images.product_id
</span><span class="sh">"""</span>
</code></pre>

</div>



<p><strong>Sample Data Structure:</strong></p>

<p>For Merge Join to work, your data must be sorted. Example:</p>

<p><strong>products.jsonl (sorted by product_id):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{"product_id": 1, "title": "Product 1", "checked": 1}
{"product_id": 2, "title": "Product 2", "checked": 1}
{"product_id": 3, "title": "Product 3", "checked": 0}
{"product_id": 4, "title": "Product 4", "checked": 1}
</code></pre>

</div>



<p><strong>images.jsonl (sorted by product_id):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{"product_id": 1, "image": "img1.jpg", "image_type": "main"}
{"product_id": 1, "image": "img1_alt.jpg", "image_type": "thumbnail"}
{"product_id": 2, "image": "img2.jpg", "image_type": "main"}
{"product_id": 4, "image": "img4_1.jpg", "image_type": "main"}
{"product_id": 4, "image": "img4_2.jpg", "image_type": "gallery"}
</code></pre>

</div>



<p><strong>Result:</strong> Merge Join processes both files simultaneously, comparing <code>product_id</code> values and joining matching rows. Memory usage stays constant regardless of file size.</p>

<h3>
  
  
  Recommendation 1: Use Merge Join if Data is Sorted (Fastest + Lowest Memory)
</h3>

<p><strong>Best for:</strong> Pre-sorted data or data that can be sorted once</p>

<p><strong>Configuration:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>  <span class="c1"># Merge Join requires use_polars=False
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">products_source</span><span class="p">,</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="n">images_source</span><span class="p">,</span> <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Performance:</strong></p>

<ul>
<li>
<strong>Time:</strong> 159.51 seconds</li>
<li>
<strong>Memory overhead:</strong> 69.50 MB (lowest!)</li>
<li>
<strong>Speed:</strong> 941 rows/second</li>
<li>
<strong>CPU:</strong> 3.9%</li>
</ul>

<p><strong>Why it's best:</strong></p>

<ul>
<li>✅ Lowest memory overhead (69.50 MB vs 236+ MB for other options)</li>
<li>✅ True streaming (both sides stream simultaneously)</li>
<li>✅ Fast execution (comparable to other options)</li>
<li>✅ Can handle datasets larger than RAM</li>
</ul>

<p><strong>When to use:</strong></p>

<ul>
<li>Data is already sorted by join key</li>
<li>Memory is a concern</li>
<li>You can sort data once (e.g., sort JSONL files before processing)</li>
<li>Both sides of join are sorted</li>
</ul>

<p><strong>Example: Preparing Data for Merge Join</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Step 1: Sort your JSONL files (one-time operation)
# Use a utility script or database ORDER BY
</span><span class="n">python</span> <span class="n">examples</span><span class="o">/</span><span class="n">sort_jsonl</span><span class="p">.</span><span class="n">py</span> <span class="n">products</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">products_sorted</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">product_id</span>
<span class="n">python</span> <span class="n">examples</span><span class="o">/</span><span class="n">sort_jsonl</span><span class="p">.</span><span class="n">py</span> <span class="n">images</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">images_sorted</span><span class="p">.</span><span class="n">jsonl</span> <span class="n">product_id</span>

<span class="c1"># Step 2: Use sorted files with Merge Join
</span><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span>
    <span class="k">lambda</span><span class="p">:</span> <span class="nf">load_jsonl</span><span class="p">(</span><span class="sh">"</span><span class="s">products_sorted.jsonl</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span>
    <span class="k">lambda</span><span class="p">:</span> <span class="nf">load_jsonl</span><span class="p">(</span><span class="sh">"</span><span class="s">images_sorted.jsonl</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">ordered_by</span><span class="o">=</span><span class="sh">"</span><span class="s">product_id</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Step 3: Query - Merge Join will be used automatically
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    SELECT products.product_id, products.title, images.image
    FROM products
    LEFT JOIN images ON products.product_id = images.product_id
    WHERE products.checked = 1
</span><span class="sh">"""</span>
</code></pre>

</div>



<h3>
  
  
  Recommendation 2: Use MMAP Join if Data is NOT Sorted (Low Memory)
</h3>

<p><strong>Best for:</strong> Unsorted data, large files, memory-constrained environments</p>

<p><strong>Configuration:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>  <span class="c1"># MMAP Join requires use_polars=False
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">products_source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">products.jsonl</span><span class="sh">"</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="n">images_source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">images.jsonl</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Performance:</strong></p>

<ul>
<li>
<strong>Time:</strong> 165.74 seconds</li>
<li>
<strong>Memory overhead:</strong> 77.48 MB (very low!)</li>
<li>
<strong>Speed:</strong> 905 rows/second</li>
<li>
<strong>CPU:</strong> 5.0%</li>
</ul>

<p><strong>Why it's best:</strong></p>

<ul>
<li>✅ Low memory overhead (77.48 MB vs 236+ MB for Lookup Join)</li>
<li>✅ Works with unsorted data (no sorting required)</li>
<li>✅ OS-managed memory mapping (can handle files larger than RAM)</li>
<li>✅ 90-99% memory reduction compared to loading entire file</li>
</ul>

<p><strong>When to use:</strong></p>

<ul>
<li>Data is NOT sorted (or sorting is expensive)</li>
<li>Large files (&gt; 100MB)</li>
<li>Memory-constrained systems</li>
<li>Files larger than available RAM</li>
<li>You want low memory without sorting</li>
</ul>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">jsonl_source</span><span class="p">():</span>
    <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">products.jsonl</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">f</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">line</span><span class="p">.</span><span class="nf">strip</span><span class="p">():</span>
                <span class="k">yield</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">line</span><span class="p">)</span>

<span class="c1"># Register with filename parameter to enable MMAP Join
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">jsonl_source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">products.jsonl</span><span class="sh">"</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="n">images_source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">images.jsonl</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Recommendation 3: Use Polars + Optimizations for Best Throughput
</h3>

<p><strong>Best for:</strong> Maximum speed when you can filter early and normalize data types</p>

<p><strong>Configuration:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">first_match_only</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">optimized_source</span><span class="p">(</span><span class="n">dynamic_where</span><span class="o">=</span><span class="bp">None</span><span class="p">,</span> <span class="n">dynamic_columns</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    Source with all optimizations:
    - Filter pushdown (dynamic_where)
    - Column pruning (dynamic_columns)
    - Data normalization (for Polars)
    </span><span class="sh">"""</span>
    <span class="c1"># Build optimized query
</span>    <span class="n">query</span> <span class="o">=</span> <span class="nf">build_query</span><span class="p">(</span><span class="n">dynamic_where</span><span class="p">,</span> <span class="n">dynamic_columns</span><span class="p">)</span>

    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="nf">execute_query</span><span class="p">(</span><span class="n">query</span><span class="p">):</span>
        <span class="c1"># Normalize types for Polars stability
</span>        <span class="k">yield</span> <span class="nf">normalize_types</span><span class="p">(</span><span class="n">row</span><span class="p">)</span>

<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span> <span class="n">optimized_source</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">images</span><span class="sh">"</span><span class="p">,</span> <span class="n">images_source</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Performance (with early filtering):</strong></p>

<ul>
<li>
<strong>Time:</strong> 54.30 seconds (fastest!)</li>
<li>
<strong>Memory overhead:</strong> 46.68 MB (very low!)</li>
<li>
<strong>Speed:</strong> 922 rows/second</li>
<li>
<strong>CPU:</strong> 6.6%</li>
<li>
<strong>Rows processed:</strong> 50,089 (filtered early, vs 150,048 without filtering)</li>
</ul>

<p><strong>Why it's best:</strong></p>

<ul>
<li>✅ Fastest execution time (54.30s vs 157-165s for others)</li>
<li>✅ Low memory overhead (46.68 MB)</li>
<li>✅ Early filtering reduces data volume significantly</li>
<li>✅ Vectorized operations (SIMD acceleration)</li>
<li>✅ All optimizations combined (filter pushdown + column pruning)</li>
</ul>

<p><strong>When to use:</strong></p>

<ul>
<li>You can filter data early (WHERE clause can be pushed to source)</li>
<li>Data types are consistent (can normalize)</li>
<li>Speed is priority</li>
<li>Polars is available</li>
<li>You want maximum performance</li>
</ul>

<p><strong>Trade-offs:</strong></p>

<ul>
<li>⚠️ Requires protocol support in source function (<code>dynamic_where</code>, <code>dynamic_columns</code>)</li>
<li>⚠️ Requires data normalization for Polars stability</li>
<li>⚠️ Requires Polars dependency</li>
<li>⚠️ Processes fewer rows (because of early filtering)</li>
</ul>

<h3>
  
  
  Summary: Choosing the Right Configuration for 100K+ Records
</h3>

<p><strong>For 100K+ records, follow this decision tree:</strong></p>

<ol>
<li><strong>Is your data sorted (or can you sort it)?</strong></li>
</ol>

<ul>
<li>✅ <strong>YES</strong> Use <strong>Merge Join</strong> (<code>ordered_by</code> parameter)

<ul>
<li>Best memory efficiency (69.50 MB)</li>
<li>Fast execution (159.51s)</li>
<li>True streaming</li>
</ul>


</li>

</ul>

<ol>
<li><strong>Is your data NOT sorted?</strong></li>
</ol>

<ul>
<li>✅ <strong>YES</strong> Use <strong>MMAP Join</strong> (<code>filename</code> parameter)

<ul>
<li>Low memory (77.48 MB)</li>
<li>Works with unsorted data</li>
<li>Good for large files</li>
</ul>


</li>

</ul>

<ol>
<li>
<strong>Do you need maximum throughput and can filter early?</strong>

<ul>
<li>✅ <strong>YES</strong> Use <strong>Polars + Optimizations</strong> (<code>use_polars=True</code> + protocols)

<ul>
<li>Fastest execution (54.30s)</li>
<li>Low memory (46.68 MB)</li>
<li>Requires protocol support</li>
</ul>
</li>
</ul>
</li>
</ol>

<p><strong>Default fallback:</strong> If none of the above apply, use <strong>Lookup Join</strong> (default Python) - it's stable and works with any data (159.44s, 236.12 MB).</p>

<h2>
  
  
  Performance Comparison Table: 100K+ Records (150K+ Joined Rows)
</h2>

<p>Based on comprehensive benchmarks with <strong>150,048 joined rows</strong> (50,089 products with ~3 images each, filtered to <code>checked=1</code>):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Configuration</th>
<th>Rows Processed</th>
<th>Time (s)</th>
<th>Memory Overhead (MB)</th>
<th>CPU %</th>
<th>Throughput (rows/s)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1. Merge Join (Sorted Data)</strong></td>
<td>150,048</td>
<td>159.51</td>
<td><strong>69.50</strong></td>
<td>3.9%</td>
<td>941</td>
</tr>
<tr>
<td><strong>2. Lookup Join (Default Python)</strong></td>
<td>150,048</td>
<td>159.44</td>
<td>236.12</td>
<td>1.8%</td>
<td>941</td>
</tr>
<tr>
<td><strong>3. Polars Join</strong></td>
<td>150,048</td>
<td>157.74</td>
<td>285.64</td>
<td>4.1%</td>
<td>951</td>
</tr>
<tr>
<td><strong>4. MMAP Join</strong></td>
<td>150,048</td>
<td>165.74</td>
<td><strong>77.48</strong></td>
<td>5.0%</td>
<td>905</td>
</tr>
<tr>
<td><strong>5. Polars + Column Pruning</strong></td>
<td>150,048</td>
<td>157.72</td>
<td>243.85</td>
<td>3.1%</td>
<td><strong>951</strong></td>
</tr>
<tr>
<td><strong>6. Polars + Filter Pushdown</strong></td>
<td>150,048</td>
<td>157.74</td>
<td>239.38</td>
<td>2.0%</td>
<td>951</td>
</tr>
<tr>
<td><strong>7. All Optimizations Combined</strong></td>
<td>50,089</td>
<td><strong>54.30</strong></td>
<td><strong>46.68</strong></td>
<td>6.6%</td>
<td>922</td>
</tr>
</tbody>
</table></div>

<p><strong>Note:</strong> Configuration 7 processes fewer rows (50,089 vs 150,048) because it applies filter pushdown early, filtering products with <code>checked=1</code> before the join. This demonstrates the power of early filtering.</p>

<h3>
  
  
  Key Insights for 100K+ Records
</h3>

<p><strong>Fastest Execution:</strong> All Optimizations Combined (54.30s)</p>

<ul>
<li>Uses Polars Join + Column Pruning + Filter Pushdown + early filtering</li>
<li>Processes 50,089 rows (filtered early) vs 150,048 without filtering</li>
<li>Best for speed priority when you can filter early</li>
</ul>

<p><strong>Lowest Memory (Full Join):</strong> Merge Join (69.50 MB)</p>

<ul>
<li>True streaming, no index needed</li>
<li>Best for memory-constrained environments with sorted data</li>
<li>Processes all 150,048 rows with minimal memory</li>
</ul>

<p><strong>Lowest Memory (Unsorted Data):</strong> MMAP Join (77.48 MB)</p>

<ul>
<li>Works with unsorted data</li>
<li>OS-managed memory mapping</li>
<li>Best for large files when data is not sorted</li>
</ul>

<p><strong>Highest Throughput:</strong> Polars + Column Pruning (951 rows/s)</p>

<ul>
<li>Vectorized operations + reduced I/O</li>
<li>Best for processing large volumes</li>
<li>Processes all 150,048 rows efficiently</li>
</ul>

<p><strong>Best Balance:</strong> Merge Join (159.51s, 69.50 MB)</p>

<ul>
<li>Good speed with lowest memory overhead</li>
<li>Best for sorted data when memory matters</li>
<li>True streaming architecture</li>
</ul>

<h3>
  
  
  Recommendations for 100K+ Records
</h3>

<p><strong>For 100K+ records, use this decision tree:</strong></p>

<ol>
<li><strong>Data is sorted (or can be sorted)?</strong></li>
</ol>

<ul>
<li>✅ <strong>YES</strong> <strong>Merge Join</strong> (<code>ordered_by</code> parameter)

<ul>
<li>Memory: 69.50 MB (lowest)</li>
<li>Time: 159.51s</li>
<li>Best for: Memory-constrained + sorted data</li>
</ul>


</li>

</ul>

<ol>
<li><strong>Data is NOT sorted?</strong></li>
</ol>

<ul>
<li>✅ <strong>YES</strong> <strong>MMAP Join</strong> (<code>filename</code> parameter)

<ul>
<li>Memory: 77.48 MB (very low)</li>
<li>Time: 165.74s</li>
<li>Best for: Unsorted data + memory-constrained</li>
</ul>


</li>

</ul>

<ol>
<li>
<strong>Need maximum speed + can filter early?</strong>

<ul>
<li>✅ <strong>YES</strong> <strong>Polars + Optimizations</strong> (<code>use_polars=True</code> + protocols)

<ul>
<li>Memory: 46.68 MB (lowest with filtering)</li>
<li>Time: 54.30s (fastest!)</li>
<li>Best for: Speed priority + early filtering possible</li>
</ul>
</li>
</ul>
</li>
</ol>

<p><strong>Default fallback:</strong> <strong>Lookup Join</strong> (default Python) - stable and works with any data (159.44s, 236.12 MB).</p>

<h2>
  
  
  Performance Guide
</h2>

<h3>
  
  
  By Dataset Size
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Size</th>
<th>Configuration</th>
<th>Why</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>&lt; 10K rows</strong></td>
<td>
<code>use_polars=False</code> (default)</td>
<td>Fastest, most stable</td>
</tr>
<tr>
<td><strong>10K-100K rows</strong></td>
<td>
<code>use_polars=False</code> (default)</td>
<td>Still fastest, handles mixed types</td>
</tr>
<tr>
<td><strong>100K-1M rows</strong></td>
<td>See recommendations below</td>
<td>Choose based on data characteristics</td>
</tr>
<tr>
<td><strong>&gt; 1M rows</strong></td>
<td>All optimizations</td>
<td>Maximum performance</td>
</tr>
</tbody>
</table></div>

<p><strong>For 100K-1M rows, choose based on your data:</strong></p>

<ul>
<li>
<strong>Data is sorted</strong> <strong>Merge Join</strong> (<code>ordered_by</code> parameter) - 69.50 MB memory, 159.51s</li>
<li>
<strong>Data is NOT sorted</strong> <strong>MMAP Join</strong> (<code>filename</code> parameter) - 77.48 MB memory, 165.74s</li>
<li>
<strong>Need maximum speed + can filter early</strong> <strong>Polars + Optimizations</strong> (<code>use_polars=True</code> + protocols) - 46.68 MB memory, 54.30s</li>
</ul>

<h2>
  
  
  Common Pitfalls
</h2>

<h3>
  
  
  Pitfall 1: Using Polars Without Normalization
</h3>

<p><strong>Problem:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="c1"># Mixed types cause schema inference errors
</span></code></pre>

</div>



<p><strong>Solution:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">normalized_source</span><span class="p">():</span>
    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="nf">raw_source</span><span class="p">():</span>
        <span class="k">yield</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="nf">int</span><span class="p">(</span><span class="n">row</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">,</span> <span class="mi">0</span><span class="p">)),</span>
            <span class="sh">"</span><span class="s">price</span><span class="sh">"</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">row</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">price</span><span class="sh">"</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">)),</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Pitfall 2: Using MMAP Without Polars (Very Slow)
</h3>

<p><strong>Problem:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">table</span><span class="sh">"</span><span class="p">,</span> <span class="n">source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">data.jsonl</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Very slow!
</span></code></pre>

</div>



<p><strong>Solution:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># MMAP uses Polars internally for index building if available
# But you still need use_polars=False for MMAP Join algorithm
</span><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">use_polars</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>  <span class="c1"># MMAP Join requires this
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">table</span><span class="sh">"</span><span class="p">,</span> <span class="n">source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">data.jsonl</span><span class="sh">"</span><span class="p">)</span>
<span class="c1"># MMAP will use Polars internally for faster index building
</span></code></pre>

</div>



<h3>
  
  
  Pitfall 3: Using MMAP for Small Files
</h3>

<p><strong>Problem:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># MMAP overhead &gt; benefit for small files
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">table</span><span class="sh">"</span><span class="p">,</span> <span class="n">source</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">"</span><span class="s">small.jsonl</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Slower!
</span></code></pre>

</div>



<p><strong>Solution:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># No filename for small files
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">table</span><span class="sh">"</span><span class="p">,</span> <span class="n">source</span><span class="p">)</span>  <span class="c1"># Faster for &lt; 100MB
</span></code></pre>

</div>



<h2>
  
  
  Summary
</h2>

<h3>
  
  
  Start Here (Most Secure)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">engine</span> <span class="o">=</span> <span class="nc">Engine</span><span class="p">()</span>  <span class="c1"># Default: use_polars=False
</span><span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">table1</span><span class="sh">"</span><span class="p">,</span> <span class="n">source1</span><span class="p">)</span>
<span class="n">engine</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="sh">"</span><span class="s">table2</span><span class="sh">"</span><span class="p">,</span> <span class="n">source2</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Why:</strong> Most stable, handles all edge cases, works with any data types</p>

<p><strong>Performance:</strong> 39.63s, 177.11 MB memory, 7,559 rows/s</p>

<h3>
  
  
  Then Experiment
</h3>

<ol>
<li>
<strong>Add debug mode</strong>: <code>Engine(debug=True)</code> - See what's happening</li>
<li>
<strong>Try Polars</strong>: <code>Engine(use_polars=True)</code> - For large datasets (39.36s, 197.64 MB)</li>
<li>
<strong>Try MMAP</strong>: <code>filename="data.jsonl"</code> - For large files (19.65s, 43.50 MB)</li>
<li>
<strong>Try Merge Join</strong>: <code>ordered_by="key"</code> - For sorted data (24.79s, 0.87 MB)</li>
</ol>

<h2>
  
  
  Conclusion
</h2>

<p>The Streaming SQL Engine fills a unique niche: <strong>cross-system data integration</strong>. While it may not match the raw performance of specialized tools for their specific use cases, it excels at joining data from different systems - a problem that traditional databases cannot solve.</p>

<p><strong>Key strengths:</strong></p>

<ul>
<li>Cross-system joins (databases, APIs, files)</li>
<li>Zero infrastructure requirements</li>
<li>Memory-efficient streaming architecture</li>
<li>Python-native integration</li>
<li>Automatic optimizations</li>
<li>Simple deployment</li>
</ul>

<p><strong>Best suited for:</strong></p>

<ul>
<li>Microservices data aggregation</li>
<li>Cross-system ETL pipelines</li>
<li>Real-time data integration</li>
<li>Memory-constrained environments</li>
<li>Python-native workflows</li>
</ul>

<p>For cross-system data integration, the Streaming SQL Engine provides a unique solution that balances performance, simplicity, and flexibility.</p>

