---
Title: Performance Tuning 101: Finding and Fixing Backend Bottlenecks
Description: 
Author: Zainab Firdous
Date: 2026-02-15T21:22:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever fetched your order history on Amazon? <br>
That one click triggers a chain of backend operations that can take <strong>~5ms</strong> or keep you waiting <strong>for over a second</strong>.<br>
At a high level, your request typically follows a path like the one shown below</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmrc3id8dfp5213ez3pml.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmrc3id8dfp5213ez3pml.jpg" alt="API Workflow" width="742" height="301"></a><br>
Load tests reveal that, in most cases, the order API is taking over a second to respond. You're assigned to identify the bottleneck and fix it <strong>ASAP</strong>, as it's now impacting the business. <br>
If this sounds daunting - or uncomfortably familiar - you're not alone.<br>
Let's break it down with a <strong>101 guide to performance tuning any application</strong>.</p>

<p><strong>SOS/Emergency Vertical Scaling</strong> -  If you are under immediate production pressure, temporarily increase CPU/RAM of your server to stabilise the application. This will create the headroom for you to safely investigate the root cause and implement a proper fix. Treat this as a short-term measure only: once the incident is under control and you've fixed the underlying cause, scale resources back down to an efficient level to avoid unnecessary cost. </p>

<p>Performance bottlenecks can typically be grouped into <strong>three broad categories</strong>:</p>

<ol>
<li>Backend service optimisation</li>
<li>Database optimisation</li>
<li>Backend infrastructure optimisation</li>
</ol>

<p>Developers rarely have generous deadlines to fix these performance issues - so the first step is to run load tests and analyse application logs and server observability. <br>
<strong>To quickly narrow down which category is the bottleneck, consider the following steps</strong>:</p>

<ul>
<li>Do you notice low throughput in load test? Is the CPU utilisation high in both backend server and database? If yes, start with database optimisation. </li>
<li>If only backend CPU utilisation is high, start with backend service optimisation. Finally, look into server configuration.</li>
<li>Is the memory utilisation high? Check the application logs to find memory leaks.
</li>
<li>Are any external APIs involved? Check the response time of external API via browser dev tool. You can consider caching response, changing CND, efficient retry mechanism and async calls to improve the external API response.</li>
</ul>

<h2>
  
  
  <strong>Backend Service</strong>
</h2>

<p>In development, functionality often takes priority over efficiency -which is usually acceptable. But under tight deadlines, inefficient code often only reveals itself under production load.<br>
Start by <strong>identifying the APIs with the worst performance</strong> and tackle them first. Before reaching for caching or pagination, which usually mask the underlying issues - verify the following:</p>

<p><strong>1.Asynchronous programming</strong> </p>

<ul>
<li>In today's system, even seconds of load time is unacceptable.</li>
<li>Ensure your code does not block unnecessarily, especially during database/network operations. </li>
<li>Use await (or equivalent) correctly so multiple operations can run concurrently without changing the application logic.</li>
<li>Avoid calling async APIs and then blocking on them synchronously, as this defeats the purpose and kills throughput.</li>
</ul>

<p><strong>2.Algorithmic Efficiency</strong></p>

<ul>
<li>Many performance issues come from poor iteration patterns. For example, database insert/update calls inside a nested loop can kill throughput. </li>
<li>Ensure your logic is time-optimised, loops are inevitable but inefficient logic over large datasets can cause serious performance issues.</li>
<li>Consider using flat loops, maps, sets and even multiple if statements, instead of nested loops wherever possible.</li>
</ul>

<p><strong>3.Limit payload size</strong></p>

<ul>
<li>Sending entire data between frontend and backend (even when small portion is updated), increases serialisation/deserialisation overhead, consumes unnecessary network bandwidth and can expose sensitive data.</li>
<li>Ensure you have proper strategy to handle save/edit use-cases.</li>
<li>Send and receive only necessary data or data that has changed.</li>
<li>Use partial updates or delta updates wherever necessary.</li>
<li>consider incorporating draft mechanism with database views or flags in the table, so that in-progress changes do not require full dataset exchange.</li>
</ul>

<p><strong>4.Batch processing</strong></p>

<ul>
<li>Real-world applications deal with huge data, so bulk operations outperform multiple network calls. </li>
<li>While batching can improve performance, under-fetching can increase network calls and degrades performance. So choose the batch size intentionally.</li>
<li>Ensure heavy database write operations are batched properly, instead of executing them one by one.</li>
<li>In case of heavy reads, consider reading data in small chunks, so the application layer is not overwhelmed. </li>
<li>Operations occurring in close intervals can be captured and executed at regular intervals to reduce load. </li>
<li>Ensure that you design batch processing to gracefully handle failures. Failure should not cause heap growth, memory leaks or inconsistent state. Use transactions where necessary and implement retry logic with proper memory cleanup.</li>
</ul>

<p>These quick checks alone can significantly improve request latency. Once these are in place, you can properly plan caching to reduce frequent database calls and paginate responses that return large datasets.</p>

<h2>
  
  
  <strong>Database Optimisation</strong>
</h2>

<p>Databases can hide massive performance gains, but they often receive less attention than application code during design. <br>
Before you dive into optimising your queries, profile your slowest query using tools like pg_stat_statements for PostgreSQL or equivalent. <br>
<strong>1.Connection Pool</strong></p>

<ul>
<li>High request latency but low database CPU/IO? Long waits before queries execute? It's likely your connection pool is exhausted.
</li>
<li>Increase your connection pool size or implement efficient request handling. </li>
<li>You can consider using tools like PgBouncer for PostgreSQL or equivalent, that will allow you to have thousands of connections without overwhelming the database. </li>
</ul>

<p><strong>2.Query optimisation</strong></p>

<ul>
<li>Analyse your slowest query with Explain/ Explain Analyze to get the query plan and execution costs. </li>
<li>Look for n+1 queries, expensive joins and subqueries ,unnecessary functions or computations. </li>
<li>Consider whether moving certain operations to application code is more efficient than letting the database handle them.</li>
</ul>

<p><strong>3.Indexing</strong> </p>

<ul>
<li>Does your query plan show long sequential scans? Do you usually deal with large dataset per query? </li>
<li>If yes, Add indexes on columns used in WHERE, GROUP BY, ORDER BY, and JOIN clauses. </li>
<li>If you have 2 columns that are often read together, consider a composite index. </li>
<li>Index add storage overhead, so ensure they are used effectively.</li>
</ul>

<p><strong>4.Normalised vs Denormalised Table</strong></p>

<ul>
<li>Do you often require multiple joins to serve common read patterns? If yes, then having a denormalised table is better since it reduces join that can slow the query. </li>
<li>If you must keep the tables separate, consider creating a view that has combined data for simplifying the read operation.</li>
<li>But if you require highly consistent data at all times, keep the normalised table.</li>
</ul>

<h2>
  
  
  <strong>Backend Infrastructure</strong>
</h2>

<p>A system can only handle as many requests as its infrastructure allows. Whether you're using a VM, managed instance groups or another setup, having right CPU and Memory configuration is crucial. <br>
There is no universal correct configuration, the CPU and RAM utilisation will always depend on your specific workload and traffic pattern. To find your <strong>unique CPU and RAM configuration</strong>, follow this <strong>three-phase formula</strong>: </p>

<p><strong>Phase 1 - Measure Peak CPU and RAM Usage</strong>:<br>
The first step is to load test your application or the specific API your optimising. Monitor the CPU and RAM utilisation closely during the test. <br>
Rule of Thumb: if your CPU/RAM utilisation exceeds ~80%, it's usually a sign that more resources may be needed. Adjust based on your workload.</p>

<p><strong>Phase 2 - Allocate the desired CPU/RAM</strong>: <br>
A simple starting point for allocation is <strong>peak usage * 2</strong><br>
This "2x" is just a guideline - sometimes 1.5x is sufficient, and other workloads may require more than 2x. Use load tests to find right combination and avoid surprises in production.</p>

<p><strong>Phase 3 - Load Test with New configuration</strong>: <br>
Finally, benchmark your application performance with new CPU and RAM configurations to ensure the changes actually improve throughput and reduce latency.</p>

<p>This strategy works well with vertical-scaling, if the configuration issues still persist consider horizontal-scaling.<br>
If you're using managed instance groups - ensure min-instance is set so that cold start issues are handled gracefully. </p>

