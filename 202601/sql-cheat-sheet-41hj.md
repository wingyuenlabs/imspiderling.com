---
Title: SQL Cheat Sheet
Description: 
Author: Shamil Suraweera
Date: 2026-01-19T21:09:26.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  SQL Code Library
</h1>

<p>A comprehensive guide to SQL (Structured Query Language) with practical examples, tutorials, and advanced techniques. This library covers SQL fundamentals, best practices, and implementations across different database systems.</p>




<h2>
  
  
  Table of Contents
</h2>

<ol>
<li>What is SQL?</li>
<li>SQL Basics</li>
<li>Major SQL Databases &amp; Versions</li>
<li>SQL vs NoSQL</li>
<li>Library Structure</li>
<li>Getting Started</li>
<li>Key SQL Concepts</li>
<li>Performance Tips</li>
<li>Resources</li>
</ol>




<h2>
  
  
  What is SQL?
</h2>

<h3>
  
  
  Definition
</h3>

<p><strong>SQL (Structured Query Language)</strong> is a standard programming language used to manage, query, and manipulate relational databases. It provides a way to store, retrieve, update, and delete data in a structured format.</p>

<h3>
  
  
  Key Characteristics
</h3>

<ul>
<li>
<strong>Declarative Language</strong>: You specify <em>what</em> you want, not <em>how</em> to get it</li>
<li>
<strong>ACID Compliance</strong>: Most SQL databases ensure Atomicity, Consistency, Isolation, Durability</li>
<li>
<strong>Standardized Syntax</strong>: Core SQL is consistent across different databases</li>
<li>
<strong>Powerful Data Manipulation</strong>: Complex queries, joins, aggregations, and transformations</li>
<li>
<strong>Scalable</strong>: Handles from kilobytes to terabytes of data</li>
</ul>

<h3>
  
  
  Why SQL?
</h3>

<ul>
<li>
<strong>Industry Standard</strong>: Used by 90%+ of organizations for data management</li>
<li>
<strong>Reliability</strong>: ACID properties ensure data integrity</li>
<li>
<strong>Security</strong>: Built-in user authentication and permission systems</li>
<li>
<strong>Performance</strong>: Optimized query engines for fast data retrieval</li>
<li>
<strong>Flexibility</strong>: Works with structured data of any complexity</li>
<li>
<strong>Interoperability</strong>: Consistent across different databases and platforms</li>
</ul>

<h3>
  
  
  Common Use Cases
</h3>

<ul>
<li>Web application backends</li>
<li>Business intelligence and reporting</li>
<li>Data warehousing</li>
<li>Financial systems</li>
<li>Healthcare information systems</li>
<li>E-commerce platforms</li>
<li>Content management systems</li>
<li>Data analysis and science</li>
</ul>




<h2>
  
  
  SQL Basics
</h2>

<h3>
  
  
  Core SQL Operations (CRUD)
</h3>

<h4>
  
  
  CREATE (INSERT)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">users</span> <span class="p">(</span><span class="n">name</span><span class="p">,</span> <span class="n">email</span><span class="p">,</span> <span class="n">age</span><span class="p">)</span> 
<span class="k">VALUES</span> <span class="p">(</span><span class="s1">'John Doe'</span><span class="p">,</span> <span class="s1">'john@example.com'</span><span class="p">,</span> <span class="mi">28</span><span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  READ (SELECT)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">users</span> <span class="k">WHERE</span> <span class="n">age</span> <span class="o">&gt;</span> <span class="mi">25</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  UPDATE
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">UPDATE</span> <span class="n">users</span> <span class="k">SET</span> <span class="n">age</span> <span class="o">=</span> <span class="mi">29</span> <span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  DELETE
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">DELETE</span> <span class="k">FROM</span> <span class="n">users</span> <span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  Basic SQL Syntax Structure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">column1</span><span class="p">,</span> <span class="n">column2</span>
<span class="k">FROM</span> <span class="k">table_name</span>
<span class="k">WHERE</span> <span class="n">condition</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">column1</span>
<span class="k">HAVING</span> <span class="n">aggregate_condition</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">column1</span> <span class="k">ASC</span>
<span class="k">LIMIT</span> <span class="mi">10</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  Major SQL Databases &amp; Versions
</h2>

<h3>
  
  
  1. MySQL
</h3>

<h4>
  
  
  Overview
</h4>

<ul>
<li>
<strong>Type</strong>: Open-source relational database</li>
<li>
<strong>Developer</strong>: Originally MySQL AB, now Oracle</li>
<li>
<strong>License</strong>: GPL (free)</li>
<li>
<strong>Popularity</strong>: One of the most popular databases for web applications</li>
</ul>

<h4>
  
  
  Major Versions
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version</th>
<th>Release Year</th>
<th>Key Features</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>5.7</strong></td>
<td>2013</td>
<td>JSON support, improved performance, native partitioning</td>
<td>Supported until Oct 2023</td>
</tr>
<tr>
<td><strong>8.0</strong></td>
<td>2018</td>
<td>Window functions, CTEs, JSON enhancements, improved InnoDB, default charset UTF-8MB4</td>
<td><strong>Current LTS</strong></td>
</tr>
<tr>
<td><strong>8.1-8.4</strong></td>
<td>2023-2024</td>
<td>Improved security, performance optimizations, new SQL features</td>
<td>Current</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  MySQL 8.0 Highlights
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Window Functions (NEW in 8.0)</span>
<span class="k">SELECT</span> <span class="n">name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span> 
       <span class="n">ROW_NUMBER</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">(</span><span class="k">PARTITION</span> <span class="k">BY</span> <span class="n">department</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span><span class="p">)</span> <span class="k">AS</span> <span class="n">rank</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>

<span class="c1">-- Common Table Expressions - CTE (NEW in 8.0)</span>
<span class="k">WITH</span> <span class="n">department_totals</span> <span class="k">AS</span> <span class="p">(</span>
  <span class="k">SELECT</span> <span class="n">department</span><span class="p">,</span> <span class="k">SUM</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">total</span>
  <span class="k">FROM</span> <span class="n">employees</span>
  <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">department</span>
<span class="p">)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">department_totals</span><span class="p">;</span>

<span class="c1">-- JSON Support</span>
<span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">JSON_EXTRACT</span><span class="p">(</span><span class="k">data</span><span class="p">,</span> <span class="s1">'$.name'</span><span class="p">)</span> <span class="k">AS</span> <span class="n">name</span>
<span class="k">FROM</span> <span class="n">users</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  MySQL Advantages
</h4>

<ul>
<li>Simple to install and use</li>
<li>Great for web development</li>
<li>Good performance for medium-scale applications</li>
<li>Strong community support</li>
<li>Wide hosting support</li>
</ul>

<h4>
  
  
  MySQL Limitations
</h4>

<ul>
<li>Not ideal for very large-scale operations compared to enterprise databases</li>
<li>Limited advanced analytics features compared to PostgreSQL</li>
<li>No true partitioning support (until recent versions)</li>
</ul>




<h3>
  
  
  2. PostgreSQL
</h3>

<h4>
  
  
  Overview
</h4>

<ul>
<li>
<strong>Type</strong>: Open-source object-relational database</li>
<li>
<strong>Developer</strong>: PostgreSQL Global Development Group</li>
<li>
<strong>License</strong>: PostgreSQL License (free)</li>
<li>
<strong>Popularity</strong>: Preferred for complex queries and data science</li>
</ul>

<h4>
  
  
  Major Versions
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version</th>
<th>Release Year</th>
<th>Key Features</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>9.6</strong></td>
<td>2016</td>
<td>Parallel query execution, logical replication</td>
<td>EOL (Oct 2021)</td>
</tr>
<tr>
<td><strong>10</strong></td>
<td>2017</td>
<td>Logical replication improvements, declarative partitioning</td>
<td>EOL (Oct 2022)</td>
</tr>
<tr>
<td><strong>11</strong></td>
<td>2018</td>
<td>JIT compilation, stored procedures (PL/pgSQL), partitioning improvements</td>
<td>Supported until Oct 2023</td>
</tr>
<tr>
<td><strong>12</strong></td>
<td>2019</td>
<td>Improved partitioning, better GENERATED columns</td>
<td>Supported until Oct 2024</td>
</tr>
<tr>
<td><strong>13-16</strong></td>
<td>2020-2024</td>
<td>Performance improvements, enhanced JSON, logical replication</td>
<td><strong>Current LTS</strong></td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  PostgreSQL Advanced Features
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Full-Text Search (native)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">articles</span> <span class="k">WHERE</span> <span class="n">search_vector</span> <span class="o">@@</span> <span class="n">to_tsquery</span><span class="p">(</span><span class="s1">'database'</span><span class="p">);</span>

<span class="c1">-- JSON/JSONB (more powerful than MySQL)</span>
<span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="k">data</span><span class="o">-&gt;&gt;</span><span class="s1">'name'</span> <span class="k">AS</span> <span class="n">name</span>
<span class="k">FROM</span> <span class="n">users</span>
<span class="k">WHERE</span> <span class="k">data</span> <span class="o">@&gt;</span> <span class="s1">'{"active": true}'</span><span class="p">;</span>

<span class="c1">-- Window Functions</span>
<span class="k">SELECT</span> <span class="n">name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span>
       <span class="n">RANK</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">(</span><span class="k">PARTITION</span> <span class="k">BY</span> <span class="n">department</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span><span class="p">)</span> <span class="k">AS</span> <span class="n">rank</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>

<span class="c1">-- Common Table Expressions (Recursive)</span>
<span class="k">WITH</span> <span class="k">RECURSIVE</span> <span class="n">numbers</span> <span class="k">AS</span> <span class="p">(</span>
  <span class="k">SELECT</span> <span class="mi">1</span> <span class="k">AS</span> <span class="n">n</span>
  <span class="k">UNION</span> <span class="k">ALL</span>
  <span class="k">SELECT</span> <span class="n">n</span> <span class="o">+</span> <span class="mi">1</span> <span class="k">FROM</span> <span class="n">numbers</span> <span class="k">WHERE</span> <span class="n">n</span> <span class="o">&lt;</span> <span class="mi">100</span>
<span class="p">)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">numbers</span><span class="p">;</span>

<span class="c1">-- Advanced Data Types</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">geo_data</span> <span class="p">(</span>
  <span class="n">id</span> <span class="nb">SERIAL</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span>
  <span class="k">location</span> <span class="n">POINT</span><span class="p">,</span>
  <span class="n">polygon_area</span> <span class="n">POLYGON</span><span class="p">,</span>
  <span class="n">json_data</span> <span class="n">JSONB</span>
<span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  PostgreSQL Advantages
</h4>

<ul>
<li>
<strong>Most Advanced Open-Source DB</strong>: Full ACID compliance, better concurrency</li>
<li>
<strong>Rich Data Types</strong>: Arrays, JSON, JSONB, geometric types, hstore, etc.</li>
<li>
<strong>Powerful Query Engine</strong>: Query planner handles complex queries efficiently</li>
<li>
<strong>Extensible</strong>: Can add custom data types, operators, and functions</li>
<li>
<strong>Superior Text Search</strong>: Built-in full-text search capabilities</li>
<li>
<strong>Better for Analytics</strong>: Window functions, CTEs, more sophisticated aggregations</li>
<li>
<strong>Replication &amp; High Availability</strong>: Logical replication, streaming replication</li>
</ul>

<h4>
  
  
  PostgreSQL Limitations
</h4>

<ul>
<li>Slightly slower write performance compared to MySQL for simple operations</li>
<li>Larger memory footprint</li>
<li>Steeper learning curve for beginners</li>
</ul>




<h3>
  
  
  3. Microsoft SQL Server
</h3>

<h4>
  
  
  Overview
</h4>

<ul>
<li>
<strong>Type</strong>: Commercial enterprise relational database</li>
<li>
<strong>Developer</strong>: Microsoft</li>
<li>
<strong>License</strong>: Commercial (Enterprise, Standard, Developer editions)</li>
<li>
<strong>Popularity</strong>: Dominant in enterprise environments</li>
</ul>

<h4>
  
  
  Major Versions
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version</th>
<th>Release Year</th>
<th>Key Features</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>2012</strong></td>
<td>2012</td>
<td>Column-store indexes, improved query optimization</td>
<td>EOL (July 2022)</td>
</tr>
<tr>
<td><strong>2014</strong></td>
<td>2014</td>
<td>In-memory OLTP, delayed durability</td>
<td>EOL (July 2024)</td>
</tr>
<tr>
<td><strong>2016</strong></td>
<td>2016</td>
<td>R integration, JSON support, temporal tables</td>
<td>Supported until Jan 2027</td>
</tr>
<tr>
<td><strong>2019</strong></td>
<td>2019</td>
<td>Graph database features, enhanced machine learning</td>
<td>Supported until Jan 2029</td>
</tr>
<tr>
<td><strong>2022</strong></td>
<td>2022</td>
<td>Azure integration, improved performance, T-SQL enhancements</td>
<td><strong>Current LTS</strong></td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  SQL Server Key Features
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Temporal Tables (tracks changes over time)</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">employees_temporal</span> <span class="p">(</span>
  <span class="n">id</span> <span class="nb">INT</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span>
  <span class="n">name</span> <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">100</span><span class="p">),</span>
  <span class="n">salary</span> <span class="nb">DECIMAL</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span><span class="mi">2</span><span class="p">),</span>
  <span class="n">ValidFrom</span> <span class="n">DATETIME2</span> <span class="k">GENERATED</span> <span class="n">ALWAYS</span> <span class="k">AS</span> <span class="k">ROW</span> <span class="k">START</span><span class="p">,</span>
  <span class="n">ValidTo</span> <span class="n">DATETIME2</span> <span class="k">GENERATED</span> <span class="n">ALWAYS</span> <span class="k">AS</span> <span class="k">ROW</span> <span class="k">END</span><span class="p">,</span>
  <span class="n">PERIOD</span> <span class="k">FOR</span> <span class="n">SYSTEM_TIME</span> <span class="p">(</span><span class="n">ValidFrom</span><span class="p">,</span> <span class="n">ValidTo</span><span class="p">)</span>
<span class="p">)</span> <span class="k">WITH</span> <span class="p">(</span><span class="n">SYSTEM_VERSIONING</span> <span class="o">=</span> <span class="k">ON</span><span class="p">);</span>

<span class="c1">-- Window Functions</span>
<span class="k">SELECT</span> <span class="n">name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span>
       <span class="n">ROW_NUMBER</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">(</span><span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span><span class="p">)</span> <span class="k">AS</span> <span class="n">rank</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>

<span class="c1">-- Graph Database</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">Person</span> <span class="p">(</span><span class="n">ID</span> <span class="nb">INT</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span> <span class="n">Name</span> <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">100</span><span class="p">));</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">Knows</span> <span class="k">AS</span> <span class="n">EDGE</span><span class="p">;</span>

<span class="c1">-- JSON Support</span>
<span class="k">SELECT</span> <span class="n">JSON_VALUE</span><span class="p">(</span><span class="k">data</span><span class="p">,</span> <span class="s1">'$.name'</span><span class="p">)</span> <span class="k">AS</span> <span class="n">name</span>
<span class="k">FROM</span> <span class="n">users</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  SQL Server Advantages
</h4>

<ul>
<li>
<strong>Enterprise-Grade</strong>: Robust, scalable, highly available</li>
<li>
<strong>Integration</strong>: Seamless with Microsoft ecosystem (Azure, Office, .NET)</li>
<li>
<strong>Security</strong>: Advanced encryption, row-level security, transparent data encryption</li>
<li>
<strong>Performance</strong>: Excellent for OLAP and OLTP workloads</li>
<li>
<strong>Advanced Features</strong>: Temporal tables, graph databases, machine learning integration</li>
<li>
<strong>Reporting</strong>: Excellent BI tools (Power BI, Reporting Services)</li>
<li>
<strong>Support</strong>: Professional enterprise support</li>
</ul>

<h4>
  
  
  SQL Server Limitations
</h4>

<ul>
<li>
<strong>Expensive</strong>: Licensing costs can be high</li>
<li>
<strong>Windows-Centric</strong>: Better support on Windows (though Linux support improving)</li>
<li>
<strong>Complexity</strong>: Steep learning curve for advanced features</li>
<li>
<strong>Not Open Source</strong>: Proprietary software</li>
</ul>




<h3>
  
  
  4. Oracle Database
</h3>

<h4>
  
  
  Overview
</h4>

<ul>
<li>
<strong>Type</strong>: Commercial enterprise relational database</li>
<li>
<strong>Developer</strong>: Oracle Corporation</li>
<li>
<strong>License</strong>: Commercial (very expensive)</li>
<li>
<strong>Popularity</strong>: Dominant in large enterprises, banking, government</li>
</ul>

<h4>
  
  
  Major Versions
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version</th>
<th>Release Year</th>
<th>Key Features</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>11g</strong></td>
<td>2007</td>
<td>Partitioning, RAC (Real Application Clusters)</td>
<td>EOL (Dec 2020)</td>
</tr>
<tr>
<td><strong>12c</strong></td>
<td>2013</td>
<td>Multi-tenancy, In-memory database, JSON support</td>
<td>Extended until Dec 2024</td>
</tr>
<tr>
<td><strong>18c</strong></td>
<td>2018</td>
<td>Cloud-optimized, improved performance, autonomous features</td>
<td>Supported</td>
</tr>
<tr>
<td><strong>19c</strong></td>
<td>2019</td>
<td>Enhanced security, better cloud integration</td>
<td><strong>Current LTS</strong></td>
</tr>
<tr>
<td><strong>21c</strong></td>
<td>2021</td>
<td>New SQL features, autonomous database, blockchain</td>
<td>Current</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  Oracle Advanced Features
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Window Functions</span>
<span class="k">SELECT</span> <span class="n">name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span>
       <span class="n">RANK</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">(</span><span class="k">PARTITION</span> <span class="k">BY</span> <span class="n">department</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span><span class="p">)</span> <span class="k">AS</span> <span class="n">rank</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>

<span class="c1">-- Recursive CTEs</span>
<span class="k">WITH</span> <span class="n">org_chart</span> <span class="k">AS</span> <span class="p">(</span>
  <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">manager_id</span><span class="p">,</span> <span class="mi">1</span> <span class="k">AS</span> <span class="k">level</span>
  <span class="k">FROM</span> <span class="n">employees</span>
  <span class="k">WHERE</span> <span class="n">manager_id</span> <span class="k">IS</span> <span class="k">NULL</span>
  <span class="k">UNION</span> <span class="k">ALL</span>
  <span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">name</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span><span class="p">,</span> <span class="n">oc</span><span class="p">.</span><span class="k">level</span> <span class="o">+</span> <span class="mi">1</span>
  <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
  <span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">org_chart</span> <span class="n">oc</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">oc</span><span class="p">.</span><span class="n">id</span>
<span class="p">)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">org_chart</span><span class="p">;</span>

<span class="c1">-- JSON Support</span>
<span class="k">SELECT</span> <span class="n">json_value</span><span class="p">(</span><span class="k">data</span><span class="p">,</span> <span class="s1">'$.name'</span><span class="p">)</span> <span class="k">AS</span> <span class="n">name</span>
<span class="k">FROM</span> <span class="n">users</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  Oracle Advantages
</h4>

<ul>
<li>
<strong>Most Powerful Enterprise DB</strong>: Handles massive data (petabytes)</li>
<li>
<strong>High Availability</strong>: RAC, Data Guard, clustering</li>
<li>
<strong>Security</strong>: Advanced encryption, audit trails, VPM</li>
<li>
<strong>Performance</strong>: Optimized for large-scale operations</li>
<li>
<strong>Autonomous Database</strong>: Self-managing, patching, upgrades</li>
<li>
<strong>Comprehensive Toolset</strong>: OEM, SQL Developer, performance tuning tools</li>
</ul>

<h4>
  
  
  Oracle Limitations
</h4>

<ul>
<li>
<strong>Very Expensive</strong>: Licensing costs can be prohibitive</li>
<li>
<strong>Complex</strong>: Steep learning curve, requires expertise</li>
<li>
<strong>Overkill for Small Projects</strong>: Too powerful for simple applications</li>
<li>
<strong>Vendor Lock-in</strong>: Difficult to migrate away from Oracle</li>
</ul>




<h3>
  
  
  5. SQLite
</h3>

<h4>
  
  
  Overview
</h4>

<ul>
<li>
<strong>Type</strong>: Embedded SQL database engine</li>
<li>
<strong>Developer</strong>: D. Richard Hipp</li>
<li>
<strong>License</strong>: Public Domain (free)</li>
<li>
<strong>Popularity</strong>: Most widely deployed database engine</li>
</ul>

<h4>
  
  
  Major Versions
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version</th>
<th>Release Year</th>
<th>Key Features</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>3.0</strong></td>
<td>2004</td>
<td>Full ACID compliance, cross-platform</td>
<td>Stable</td>
</tr>
<tr>
<td><strong>3.8+</strong></td>
<td>2013+</td>
<td>Window functions, CTEs, JSON support</td>
<td>Current</td>
</tr>
<tr>
<td><strong>3.35+</strong></td>
<td>2021+</td>
<td>Improved performance, RETURNING clause</td>
<td>Current</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  SQLite Features
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Window Functions (3.25+)</span>
<span class="k">SELECT</span> <span class="n">name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span>
       <span class="n">ROW_NUMBER</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">(</span><span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span><span class="p">)</span> <span class="k">AS</span> <span class="n">rank</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>

<span class="c1">-- JSON Support (3.38+)</span>
<span class="k">SELECT</span> <span class="n">json_extract</span><span class="p">(</span><span class="k">data</span><span class="p">,</span> <span class="s1">'$.name'</span><span class="p">)</span> <span class="k">AS</span> <span class="n">name</span>
<span class="k">FROM</span> <span class="n">users</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  SQLite Advantages
</h4>

<ul>
<li>
<strong>Zero Configuration</strong>: No setup, no server needed</li>
<li>
<strong>Lightweight</strong>: Single file database</li>
<li>
<strong>Portable</strong>: Works on any platform</li>
<li>
<strong>Fast</strong>: Excellent performance for single-user/small-scale</li>
<li>
<strong>ACID Compliant</strong>: Reliable transactions</li>
<li>
<strong>Widely Used</strong>: Mobile apps, IoT, embedded systems</li>
</ul>

<h4>
  
  
  SQLite Limitations
</h4>

<ul>
<li>
<strong>Not for Concurrent Access</strong>: Limited multi-user support</li>
<li>
<strong>No User Management</strong>: No built-in authentication</li>
<li>
<strong>Scale Limitations</strong>: Not suitable for large-scale applications</li>
<li>
<strong>No Remote Access</strong>: File-based only</li>
</ul>




<h2>
  
  
  Key Version Differences Summary
</h2>

<h3>
  
  
  Feature Comparison Across Major SQL Databases
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>MySQL 8.0</th>
<th>PostgreSQL 15+</th>
<th>SQL Server 2022</th>
<th>Oracle 21c</th>
<th>SQLite 3.35+</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Window Functions</strong></td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
</tr>
<tr>
<td><strong>CTEs (WITH clause)</strong></td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Recursive CTEs</strong></td>
<td>❌</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>❌</td>
</tr>
<tr>
<td><strong>JSON Support</strong></td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Full-Text Search</strong></td>
<td>❌</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>❌</td>
</tr>
<tr>
<td><strong>Graph Database</strong></td>
<td>❌</td>
<td>❌</td>
<td>✅</td>
<td>❌</td>
<td>❌</td>
</tr>
<tr>
<td><strong>Temporal Tables</strong></td>
<td>❌</td>
<td>❌</td>
<td>✅</td>
<td>❌</td>
<td>❌</td>
</tr>
<tr>
<td><strong>Partitioning</strong></td>
<td>Limited</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>❌</td>
</tr>
<tr>
<td><strong>Replication</strong></td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>❌</td>
</tr>
<tr>
<td><strong>Multi-User Support</strong></td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>Limited</td>
</tr>
<tr>
<td><strong>ACID Compliance</strong></td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Cost</strong></td>
<td>Free</td>
<td>Free</td>
<td>Expensive</td>
<td>Very Expensive</td>
<td>Free</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  SQL vs NoSQL
</h2>

<h3>
  
  
  Comparison Table
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Aspect</th>
<th>SQL (Relational)</th>
<th>NoSQL (Non-Relational)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Data Structure</strong></td>
<td>Tables with rows and columns</td>
<td>Documents, key-value, graphs, time-series</td>
</tr>
<tr>
<td><strong>Schema</strong></td>
<td>Fixed schema required</td>
<td>Flexible/dynamic schema</td>
</tr>
<tr>
<td><strong>ACID</strong></td>
<td>Strong ACID compliance</td>
<td>Often eventual consistency (BASE)</td>
</tr>
<tr>
<td><strong>Scaling</strong></td>
<td>Vertical (add more power)</td>
<td>Horizontal (add more servers)</td>
</tr>
<tr>
<td><strong>Query Language</strong></td>
<td>SQL (standardized)</td>
<td>Varies (MongoDB: JSON, Cassandra: CQL, etc.)</td>
</tr>
<tr>
<td><strong>Joins</strong></td>
<td>Efficient, common</td>
<td>Not native; requires denormalization</td>
</tr>
<tr>
<td><strong>Transactions</strong></td>
<td>Multi-row, complex</td>
<td>Limited or single-document</td>
</tr>
<tr>
<td><strong>Best For</strong></td>
<td>Structured, relational data</td>
<td>Big data, unstructured data, real-time</td>
</tr>
<tr>
<td><strong>Examples</strong></td>
<td>MySQL, PostgreSQL, Oracle, SQL Server</td>
<td>MongoDB, Cassandra, Redis, DynamoDB</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  When to Use SQL
</h3>

<ul>
<li>Financial systems requiring ACID compliance</li>
<li>Complex relationships between entities</li>
<li>Consistent, structured data</li>
<li>Complex reporting and analytics</li>
<li>When data integrity is critical</li>
</ul>

<h3>
  
  
  When to Use NoSQL
</h3>

<ul>
<li>Massive scale, distributed systems</li>
<li>Unstructured or semi-structured data</li>
<li>Real-time applications requiring horizontal scaling</li>
<li>High-velocity data (streaming, IoT)</li>
<li>Content management with varying structures</li>
</ul>




<h2>
  
  
  Library Structure
</h2>

<p>This code library is organized by SQL topics:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>├── 01.CREATE.sql              -- CREATE TABLE, database creation
├── 02.INSERT_INTO.sql         -- INSERT statements, data insertion
├── 03.SELECT.sql              -- SELECT queries, retrieving data
├── 04.WHERE.sql               -- WHERE clause, filtering
├── 05.LIKE.sql                -- Pattern matching with LIKE
├── 06.GROUP_BY.sql            -- Aggregation and grouping
├── 07.ORDER_BY.sql            -- Sorting results
├── 08.HAVING.sql              -- Filtering grouped data
├── 09.LIMIT.sql               -- Limiting result sets
├── 10.AS.sql                  -- Aliases for columns and tables
├── 11.JOIN.sql                -- INNER, LEFT, RIGHT, FULL joins
├── 12.UNION.sql               -- Combining result sets
├── 13.STRING.sql              -- String functions and manipulation
├── 14.CASE.sql                -- Conditional logic (CASE statements)
├── 15.SUBQUERY.sql            -- Nested queries
├── 16.WINDOW.sql              -- Window functions (MySQL 8.0+)
├── 17.CTE.sql                 -- Common Table Expressions
├── 18.TEMPORARY_TABLE.sql     -- Temporary tables
├── 19.PROCEDURE.sql           -- Stored procedures
├── 20.TRIGGER.sql             -- Database triggers
├── 21.EVENT.sql               -- Scheduled events
├── MySQL_README.md            -- MySQL-specific guide
└── MySQL/
    ├── 01.Installing          -- MySQL installation steps
    └── 02.Create_User.sql     -- User creation and privileges
</code></pre>

</div>



<p>Each file contains:</p>

<ul>
<li>
<strong>Comments</strong> explaining the concept</li>
<li>
<strong>Example Queries</strong> demonstrating the syntax</li>
<li>
<strong>Use Cases</strong> showing when to use each feature</li>
<li>
<strong>Best Practices</strong> for optimal performance</li>
</ul>




<h2>
  
  
  Getting Started
</h2>

<h3>
  
  
  1. Choose Your Database
</h3>

<ul>
<li>
<strong>For Web Development</strong>: MySQL or PostgreSQL</li>
<li>
<strong>For Learning SQL</strong>: SQLite or MySQL</li>
<li>
<strong>For Enterprise</strong>: SQL Server, Oracle, or PostgreSQL</li>
<li>
<strong>For Analytics</strong>: PostgreSQL or Snowflake</li>
</ul>

<h3>
  
  
  2. Install MySQL (Recommended for Learning)
</h3>

<p>See <a href="//./MySQL_README.md">MySQL_README.md</a> for installation and setup instructions.</p>

<h3>
  
  
  3. Learn SQL Fundamentals
</h3>

<p>Start with these files in order:</p>

<ol>
<li>
<code>01.CREATE.sql</code> — Create tables and databases</li>
<li>
<code>02.INSERT_INTO.sql</code> — Insert data</li>
<li>
<code>03.SELECT.sql</code> — Query data</li>
<li>
<code>04.WHERE.sql</code> — Filter results</li>
<li>
<code>05.LIKE.sql</code> — Pattern matching</li>
<li>
<code>06.GROUP_BY.sql</code> — Aggregation</li>
</ol>

<h3>
  
  
  4. Practice with Examples
</h3>

<ul>
<li>Run each query example in your SQL client</li>
<li>Modify queries to experiment</li>
<li>Create your own test data</li>
<li>Compare results to understand behavior</li>
</ul>

<h3>
  
  
  5. Learn Advanced Topics
</h3>

<ul>
<li>
<code>11.JOIN.sql</code> — Connect multiple tables</li>
<li>
<code>15.SUBQUERY.sql</code> — Nested queries</li>
<li>
<code>16.WINDOW.sql</code> — Advanced analytics</li>
<li>
<code>17.CTE.sql</code> — Complex query structures</li>
</ul>




<h2>
  
  
  Key SQL Concepts
</h2>

<h3>
  
  
  1. <strong>Normalization</strong>
</h3>

<p>Organizing data to reduce redundancy and improve data integrity.</p>

<h3>
  
  
  2. <strong>Indexing</strong>
</h3>

<p>Creating indexes on frequently queried columns for faster retrieval.</p>

<h3>
  
  
  3. <strong>Transactions</strong>
</h3>

<p>ACID-compliant operations ensuring data consistency:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">BEGIN</span> <span class="n">TRANSACTION</span><span class="p">;</span>
  <span class="k">UPDATE</span> <span class="n">accounts</span> <span class="k">SET</span> <span class="n">balance</span> <span class="o">=</span> <span class="n">balance</span> <span class="o">-</span> <span class="mi">100</span> <span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
  <span class="k">UPDATE</span> <span class="n">accounts</span> <span class="k">SET</span> <span class="n">balance</span> <span class="o">=</span> <span class="n">balance</span> <span class="o">+</span> <span class="mi">100</span> <span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="mi">2</span><span class="p">;</span>
<span class="k">COMMIT</span><span class="p">;</span>  <span class="c1">-- or ROLLBACK;</span>
</code></pre>

</div>



<h3>
  
  
  4. <strong>Joins</strong>
</h3>

<p>Combining data from multiple tables:</p>

<ul>
<li>
<strong>INNER JOIN</strong>: Only matching rows</li>
<li>
<strong>LEFT JOIN</strong>: All rows from left table</li>
<li>
<strong>RIGHT JOIN</strong>: All rows from right table</li>
<li>
<strong>FULL OUTER JOIN</strong>: All rows from both tables</li>
</ul>

<h3>
  
  
  5. <strong>Aggregation</strong>
</h3>

<p>Summarizing data with functions like SUM, AVG, COUNT, MIN, MAX:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">department</span><span class="p">,</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_salary</span>
<span class="k">FROM</span> <span class="n">employees</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">department</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  6. <strong>Subqueries</strong>
</h3>

<p>Nested queries for complex filtering:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">employees</span>
<span class="k">WHERE</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  7. <strong>Window Functions</strong> (MySQL 8.0+, PostgreSQL, SQL Server)
</h3>

<p>Advanced analytical functions without collapsing rows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span>
       <span class="n">ROW_NUMBER</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">(</span><span class="k">PARTITION</span> <span class="k">BY</span> <span class="n">department</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span><span class="p">)</span> <span class="k">AS</span> <span class="n">rank</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  8. <strong>Common Table Expressions (CTEs)</strong>
</h3>

<p>Named temporary result sets:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WITH</span> <span class="n">high_earners</span> <span class="k">AS</span> <span class="p">(</span>
  <span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="k">WHERE</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="mi">100000</span>
<span class="p">)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">high_earners</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  Performance Tips
</h2>

<h3>
  
  
  1. <strong>Use Indexes Wisely</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_email</span> <span class="k">ON</span> <span class="n">users</span><span class="p">(</span><span class="n">email</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  2. *<em>Avoid SELECT *</em>*
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Good</span>
<span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">email</span> <span class="k">FROM</span> <span class="n">users</span><span class="p">;</span>

<span class="c1">-- Avoid</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">users</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  3. <strong>Use LIMIT for Large Result Sets</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">users</span> <span class="k">LIMIT</span> <span class="mi">100</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  4. <strong>Optimize Joins</strong>
</h3>

<ul>
<li>Join on indexed columns</li>
<li>Join small tables first (in some databases)</li>
<li>Avoid cartesian products</li>
</ul>

<h3>
  
  
  5. <strong>Use Appropriate Data Types</strong>
</h3>

<ul>
<li>INT instead of VARCHAR for numeric IDs</li>
<li>DECIMAL for money (avoid FLOAT)</li>
<li>DATE/TIMESTAMP for temporal data</li>
</ul>

<h3>
  
  
  6. <strong>Regular Maintenance</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Analyze table statistics</span>
<span class="k">ANALYZE</span> <span class="k">TABLE</span> <span class="n">users</span><span class="p">;</span>

<span class="c1">-- Optimize table (MySQL)</span>
<span class="n">OPTIMIZE</span> <span class="k">TABLE</span> <span class="n">users</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  7. <strong>Use Prepared Statements</strong>
</h3>

<p>Protects against SQL injection and improves performance.</p>

<h3>
  
  
  8. <strong>Monitor Slow Queries</strong>
</h3>

<ul>
<li>Enable slow query logging</li>
<li>Use EXPLAIN ANALYZE to understand query execution</li>
</ul>




<h2>
  
  
  Resources
</h2>

<h3>
  
  
  Official Documentation
</h3>

<ul>
<li>
<strong>MySQL</strong>: <a href="https://dev.mysql.com/doc/" rel="noopener noreferrer">https://dev.mysql.com/doc/</a>
</li>
<li>
<strong>PostgreSQL</strong>: <a href="https://www.postgresql.org/docs/" rel="noopener noreferrer">https://www.postgresql.org/docs/</a>
</li>
<li>
<strong>SQL Server</strong>: <a href="https://docs.microsoft.com/en-us/sql/" rel="noopener noreferrer">https://docs.microsoft.com/en-us/sql/</a>
</li>
<li>
<strong>Oracle</strong>: <a href="https://docs.oracle.com/" rel="noopener noreferrer">https://docs.oracle.com/</a>
</li>
<li>
<strong>SQLite</strong>: <a href="https://www.sqlite.org/docs.html" rel="noopener noreferrer">https://www.sqlite.org/docs.html</a>
</li>
</ul>

<h3>
  
  
  Learning Platforms
</h3>

<ul>
<li>
<strong>W3Schools SQL Tutorial</strong>: <a href="https://www.w3schools.com/sql/" rel="noopener noreferrer">https://www.w3schools.com/sql/</a>
</li>
<li>
<strong>Mode Analytics SQL Tutorial</strong>: <a href="https://mode.com/sql-tutorial/" rel="noopener noreferrer">https://mode.com/sql-tutorial/</a>
</li>
<li>
<strong>LeetCode SQL</strong>: <a href="https://leetcode.com/discuss/interview-question?currentPage=1&amp;orderBy=most_votes&amp;topicTags=sql" rel="noopener noreferrer">https://leetcode.com/discuss/interview-question?currentPage=1&amp;orderBy=most_votes&amp;topicTags=sql</a>
</li>
<li>
<strong>HackerRank SQL</strong>: <a href="https://www.hackerrank.com/domains/sql" rel="noopener noreferrer">https://www.hackerrank.com/domains/sql</a>
</li>
</ul>

<h3>
  
  
  Books
</h3>

<ul>
<li>
<strong>"Learning SQL" by Alan Beaulieu</strong>: Beginner-friendly</li>
<li>
<strong>"SQL Performance Explained" by Markus Winand</strong>: Performance optimization</li>
<li>
<strong>"Advanced SQL" by Joe Celko</strong>: Complex queries and design patterns</li>
<li>
<strong>"SQL Antipatterns" by Bill Karwin</strong>: What NOT to do</li>
</ul>

<h3>
  
  
  Tools
</h3>

<ul>
<li>
<strong>DBeaver</strong>: Universal database IDE</li>
<li>
<strong>Adminer</strong>: Web-based database management</li>
<li>
<strong>DataGrip</strong>: JetBrains SQL IDE</li>
<li>
<strong>MySQL Workbench</strong>: MySQL-specific tool</li>
<li>
<strong>pgAdmin</strong>: PostgreSQL management</li>
</ul>




<h2>
  
  
  Contributing
</h2>

<p>Feel free to add more SQL examples, tips, and best practices to this library!</p>




<p><strong>Version</strong>: 1.0<br><br>
<strong>Last Updated</strong>: December 2025<br><br>
<strong>Maintainer</strong>: Shamil Suraweera<br><br>
<strong>License</strong>: Free to use and modify</p>

