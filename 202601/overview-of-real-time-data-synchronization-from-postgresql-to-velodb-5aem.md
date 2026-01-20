---
Title: Overview of Real-Time Data Synchronization from PostgreSQL to VeloDB
Description: 
Author: Apache Doris
Date: 2026-01-20T22:14:48.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Overview
</h1>

<p>In the process of migrating data from PostgreSQL (including PostgreSQL-compatible Amazon Aurora) to VeloDB, Flink can be introduced as a real-time data synchronization engine to ensure data consistency and real-timeliness. Flink possesses high-throughput and low-latency stream processing capabilities, enabling efficient full-data loading and incremental change handling for databases.</p>

<p>For real-time synchronization scenarios, PostgreSQL's Logical Replication can be enabled to capture CDC (Change Data Capture) events. Whether it is a self-hosted PostgreSQL or cloud-based Amazon Aurora-PostgreSQL, you can subscribe with Flink CDC by enabling the logical decoding plugin and creating a Replication Slot, thereby achieving:</p>

<ul>
<li><p>Full data initial load: First import business data from PostgreSQL/Aurora into VeloDB</p></li>
<li><p>Real-time synchronization of incremental changes: Capture Insert/Update/Delete operations based on Logical Replication and continuously write them to VeloDB</p></li>
</ul>

<p>The following takes Amazon Aurora-PostgreSQL as an example to demonstrate how to use Flink CDC to subscribe to Aurora changes and synchronize them to VeloDB in real time.</p>

<h1>
  
  
  Example
</h1>

<h2>
  
  
  1. Create an AWS RDS Aurora PostgreSQL instance
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft3nplp5dpn0lqx2ju1jy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft3nplp5dpn0lqx2ju1jy.png" alt=" " width="800" height="324"></a></p>

<h2>
  
  
  2. Create a VeloDB warehouse
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F88qwgmdt5sc9vi340nv2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F88qwgmdt5sc9vi340nv2.png" alt=" " width="800" height="210"></a></p>

<h2>
  
  
  3. Create a PostgreSQL database and corresponding tables
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="k">CREATE</span> <span class="k">DATABASE</span> <span class="n">test_db</span><span class="p">;</span>

<span class="c1">-- Create table</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">test_db</span><span class="p">.</span><span class="k">public</span><span class="p">.</span><span class="n">student</span> <span class="p">(</span>
    <span class="n">id</span> <span class="nb">INT</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span>
    <span class="n">name</span> <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">100</span><span class="p">)</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">age</span> <span class="nb">INT</span><span class="p">,</span>
    <span class="n">email</span> <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">255</span><span class="p">),</span>
    <span class="n">phone</span> <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">20</span><span class="p">),</span>
    <span class="n">score</span> <span class="nb">NUMERIC</span><span class="p">(</span><span class="mi">5</span><span class="p">,</span><span class="mi">2</span><span class="p">),</span>
    <span class="n">created_at</span> <span class="nb">TIMESTAMP</span> <span class="k">DEFAULT</span> <span class="k">CURRENT_TIMESTAMP</span>
<span class="p">);</span>

<span class="c1">-- Load data</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">test_db</span><span class="p">.</span><span class="k">public</span><span class="p">.</span><span class="n">student</span> <span class="p">(</span><span class="n">id</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">age</span><span class="p">,</span> <span class="n">email</span><span class="p">,</span> <span class="n">phone</span><span class="p">,</span> <span class="n">score</span><span class="p">,</span> <span class="n">created_at</span><span class="p">)</span> 
<span class="k">VALUES</span> 
<span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s1">'Alice Zhang'</span><span class="p">,</span> <span class="mi">22</span><span class="p">,</span> <span class="s1">'alice@example.com'</span><span class="p">,</span> <span class="s1">'13800138000'</span><span class="p">,</span> <span class="mi">89</span><span class="p">.</span><span class="mi">50</span><span class="p">,</span> <span class="n">NOW</span><span class="p">()),</span>
<span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s1">'Bob Li'</span><span class="p">,</span> <span class="mi">21</span><span class="p">,</span> <span class="s1">'bob@example.com'</span><span class="p">,</span> <span class="s1">'13900139000'</span><span class="p">,</span> <span class="mi">76</span><span class="p">.</span><span class="mi">80</span><span class="p">,</span> <span class="n">NOW</span><span class="p">()),</span>
<span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">'Charlie Wang'</span><span class="p">,</span> <span class="mi">23</span><span class="p">,</span> <span class="s1">'charlie@example.com'</span><span class="p">,</span> <span class="s1">'13600136000'</span><span class="p">,</span> <span class="mi">92</span><span class="p">.</span><span class="mi">00</span><span class="p">,</span> <span class="n">NOW</span><span class="p">()),</span>
<span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="s1">'David Chen'</span><span class="p">,</span> <span class="mi">20</span><span class="p">,</span> <span class="s1">'david@example.com'</span><span class="p">,</span> <span class="s1">'13500135000'</span><span class="p">,</span> <span class="mi">85</span><span class="p">.</span><span class="mi">60</span><span class="p">,</span> <span class="n">NOW</span><span class="p">()),</span>
<span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="s1">'Emma Liu'</span><span class="p">,</span> <span class="mi">22</span><span class="p">,</span> <span class="s1">'emma@example.com'</span><span class="p">,</span> <span class="s1">'13700137000'</span><span class="p">,</span> <span class="mi">78</span><span class="p">.</span><span class="mi">90</span><span class="p">,</span> <span class="n">NOW</span><span class="p">());</span>
</code></pre>

</div>



<h2>
  
  
  4. Enable PostgreSQL Logical Replication
</h2>

<p>Create a parameter group and modify the rds.logical_replication configuration</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fktav4qgd42kxpxloophx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fktav4qgd42kxpxloophx.png" alt=" " width="800" height="381"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wh63dd8gn23ae8y40df.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wh63dd8gn23ae8y40df.png" alt=" " width="800" height="155"></a></p>

<p>Modify the PostgreSQL configuration: replace the DB Cluster Parameter with the parameter group created just now, apply the changes, and restart the service</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdqsmhscpfd8c3pn3art2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdqsmhscpfd8c3pn3art2.png" alt=" " width="800" height="169"></a></p>

<h2>
  
  
  5. Install Flink With Doris Connector
</h2>

<h3>
  
  
  5.1 Download the pre-defined installation package
</h3>

<p>Based on Flink 1.17, we provide a pre-defined installation package that can be directly downloaded and decompressed.</p>

<h3>
  
  
  5.2 Manual installation
</h3>

<p>If you already have a Flink environment or need another version of Flink, you can use the manual installation method.Taking Flink 1.17 as an example here, download the Flink installation package and its dependencies.</p>

<ul>
<li><p>Flink 1.17</p></li>
<li><p>Flink Postgres CDC Connector</p></li>
<li><p>Flink Doris Connector</p></li>
</ul>

<p>After the download is complete, extract the Flink installation package.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="nb">tar</span> <span class="nt">-zxvf</span> flink-1.17.2-bin-scala_2.12.tgz
</code></pre>

</div>



<p>Meanwhile, place the Flink PostgreSQL CDC Connector and Doris Connector into the flink-1.17.2-bin/lib directory.</p>

<p>As follows:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo2oq577dh7dkjn4hvhm3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo2oq577dh7dkjn4hvhm3.png" alt=" " width="800" height="269"></a></p>

<h2>
  
  
  6. Submit the Flink synchronization job
</h2>

<p>When submitting the job, the Doris Connector will automatically create corresponding tables in VeloDB in advance based on the table structure of the upstream PostgreSQL database.</p>

<p>Flink supports job submission and operation in modes such as Local, Standalone, and Yarn. If you already have a Flink environment, you can directly submit the job to your own Flink environment.</p>

<h3>
  
  
  6.1 Local Environment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="nb">cd </span>flink-1.17.2-bin
bin/flink run <span class="nt">-t</span> <span class="nb">local</span> <span class="se">\</span>
    <span class="nt">-Dexecution</span>.checkpointing.interval<span class="o">=</span>10s <span class="se">\ </span>                                                                                                              
    <span class="nt">-Dparallelism</span>.default<span class="o">=</span>1 <span class="se">\ </span>                                                                                                                             
    <span class="nt">-c</span> org.apache.doris.flink.tools.cdc.CdcTools <span class="se">\ </span>                                                                                                        
    lib/flink-doris-connector-1.17-25.1.0.jar <span class="se">\ </span>                                                                                                           
    postgres-sync-database <span class="se">\ </span>                                                                                                                                 
    <span class="nt">--database</span> test_db <span class="se">\ </span>                                                                                                                                  
    <span class="nt">--postgres-conf</span> <span class="nb">hostname</span><span class="o">=</span>database-test.xxx.us-east-1.rds.amazonaws.com <span class="se">\ </span>                                                                                                                     
    <span class="nt">--postgres-conf</span> <span class="nv">port</span><span class="o">=</span>3306 <span class="se">\ </span>                                                                                                                              
    <span class="nt">--postgres-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                          
    <span class="nt">--postgres-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                        
    <span class="nt">--postgres-conf</span> database-name<span class="o">=</span><span class="nb">test</span> <span class="se">\ </span>                                                                                                                     
    <span class="nt">--including-tables</span> <span class="s2">"student"</span> <span class="se">\ </span>                                                                                                                       
    <span class="nt">--sink-conf</span> <span class="nv">fenodes</span><span class="o">=</span>lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:8080 <span class="se">\ </span>                                                                                                                
    <span class="nt">--sink-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                           
    <span class="nt">--sink-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                               
    <span class="nt">--sink-conf</span> jdbc-url<span class="o">=</span>jdbc:mysql://lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:9030 <span class="se">\ </span>                                                                                                  
    <span class="nt">--sink-conf</span> sink.label-prefix<span class="o">=</span>label
</code></pre>

</div>



<h3>
  
  
  6.2 Standalone Environment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="nb">cd </span>flink-1.17.2-bin
bin/flink run <span class="nt">-t</span> remote <span class="se">\</span>
    <span class="nt">-Dexecution</span>.checkpointing.interval<span class="o">=</span>10s <span class="se">\ </span>                                                                                                              
    <span class="nt">-Dparallelism</span>.default<span class="o">=</span>1 <span class="se">\ </span>                                                                                                                             
    <span class="nt">-c</span> org.apache.doris.flink.tools.cdc.CdcTools <span class="se">\ </span>                                                                                                        
    lib/flink-doris-connector-1.17-25.1.0.jar <span class="se">\ </span>                                                                                                           
    postgres-sync-database <span class="se">\ </span>                                                                                                                                 
    <span class="nt">--database</span> test_db <span class="se">\ </span>                                                                                                                                  
    <span class="nt">--postgres-conf</span> <span class="nb">hostname</span><span class="o">=</span>database-test.xxx.us-east-1.rds.amazonaws.com <span class="se">\ </span>                                                                                                                     
    <span class="nt">--postgres-conf</span> <span class="nv">port</span><span class="o">=</span>3306 <span class="se">\ </span>                                                                                                                              
    <span class="nt">--postgres-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                          
    <span class="nt">--postgres-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                        
    <span class="nt">--postgres-conf</span> database-name<span class="o">=</span><span class="nb">test</span> <span class="se">\ </span>                                                                                                                     
    <span class="nt">--including-tables</span> <span class="s2">"student"</span> <span class="se">\ </span>                                                                                                                       
    <span class="nt">--sink-conf</span> <span class="nv">fenodes</span><span class="o">=</span>lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:8080 <span class="se">\ </span>                                                                                                                
    <span class="nt">--sink-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                           
    <span class="nt">--sink-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                               
    <span class="nt">--sink-conf</span> jdbc-url<span class="o">=</span>jdbc:mysql://lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:9030 <span class="se">\ </span>                                                                                                  
    <span class="nt">--sink-conf</span> sink.label-prefix<span class="o">=</span>label
</code></pre>

</div>



<h3>
  
  
  6.3 Yarn Environment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="nb">cd </span>flink-1.17.2-bin
bin/flink run <span class="nt">-t</span> yarn-per-job <span class="se">\</span>
    <span class="nt">-Dexecution</span>.checkpointing.interval<span class="o">=</span>10s <span class="se">\ </span>                                                                                                              
    <span class="nt">-Dparallelism</span>.default<span class="o">=</span>1 <span class="se">\ </span>                                                                                                                             
    <span class="nt">-c</span> org.apache.doris.flink.tools.cdc.CdcTools <span class="se">\ </span>                                                                                                        
    lib/flink-doris-connector-1.17-25.1.0.jar <span class="se">\ </span>                                                                                                           
    postgres-sync-database <span class="se">\ </span>                                                                                                                                 
    <span class="nt">--database</span> test_db <span class="se">\ </span>                                                                                                                                  
    <span class="nt">--postgres-conf</span> <span class="nb">hostname</span><span class="o">=</span>database-test.xxx.us-east-1.rds.amazonaws.com <span class="se">\ </span>                                                                                                                     
    <span class="nt">--postgres-conf</span> <span class="nv">port</span><span class="o">=</span>3306 <span class="se">\ </span>                                                                                                                              
    <span class="nt">--postgres-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                          
    <span class="nt">--postgres-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                        
    <span class="nt">--postgres-conf</span> database-name<span class="o">=</span><span class="nb">test</span> <span class="se">\ </span>                                                                                                                     
    <span class="nt">--including-tables</span> <span class="s2">"student"</span> <span class="se">\ </span>                                                                                                                       
    <span class="nt">--sink-conf</span> <span class="nv">fenodes</span><span class="o">=</span>lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:8080 <span class="se">\ </span>                                                                                                                
    <span class="nt">--sink-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                           
    <span class="nt">--sink-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                               
    <span class="nt">--sink-conf</span> jdbc-url<span class="o">=</span>jdbc:mysql://lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:9030 <span class="se">\ </span>                                                                                                  
    <span class="nt">--sink-conf</span> sink.label-prefix<span class="o">=</span>label
</code></pre>

</div>



<h3>
  
  
  6.4 K8S Environment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="nb">cd </span>flink-1.17.2-bin
bin/flink run <span class="nt">-t</span> kubernetes-session <span class="se">\</span>
    <span class="nt">-Dexecution</span>.checkpointing.interval<span class="o">=</span>10s <span class="se">\ </span>                                                                                                              
    <span class="nt">-Dparallelism</span>.default<span class="o">=</span>1 <span class="se">\ </span>                                                                                                                             
    <span class="nt">-c</span> org.apache.doris.flink.tools.cdc.CdcTools <span class="se">\ </span>                                                                                                        
    lib/flink-doris-connector-1.17-25.1.0.jar <span class="se">\ </span>                                                                                                           
    postgres-sync-database <span class="se">\ </span>                                                                                                                                 
    <span class="nt">--postgres</span> test_db <span class="se">\ </span>                                                                                                                                  
    <span class="nt">--postgres-conf</span> <span class="nb">hostname</span><span class="o">=</span>database-test.xxx.us-east-1.rds.amazonaws.com <span class="se">\ </span>                                                                                                                     
    <span class="nt">--postgres-conf</span> <span class="nv">port</span><span class="o">=</span>3306 <span class="se">\ </span>                                                                                                                              
    <span class="nt">--postgres-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                          
    <span class="nt">--postgres-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                        
    <span class="nt">--postgres-conf</span> database-name<span class="o">=</span><span class="nb">test</span> <span class="se">\ </span>                                                                                                                     
    <span class="nt">--including-tables</span> <span class="s2">"student"</span> <span class="se">\ </span>                                                                                                                       
    <span class="nt">--sink-conf</span> <span class="nv">fenodes</span><span class="o">=</span>lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:8080 <span class="se">\ </span>                                                                                                                
    <span class="nt">--sink-conf</span> <span class="nv">username</span><span class="o">=</span>admin <span class="se">\ </span>                                                                                                                           
    <span class="nt">--sink-conf</span> <span class="nv">password</span><span class="o">=</span>123456 <span class="se">\ </span>                                                                                                                               
    <span class="nt">--sink-conf</span> jdbc-url<span class="o">=</span>jdbc:mysql://lb-40579077-a97732bc6c030909.elb.us-east-1.amazonaws.com:9030 <span class="se">\ </span>                                                                                                  
    <span class="nt">--sink-conf</span> sink.label-prefix<span class="o">=</span>label
</code></pre>

</div>



<p>Note: For more parameters of the Connector, refer to this link.</p>

<h2>
  
  
  7. Verify Historical Data Synchronization
</h2>

<p>The Flink job will synchronize full historical data for the first time. Check the data synchronization status in VeloDB.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5vbxrqb52w0m6fqi56xi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5vbxrqb52w0m6fqi56xi.png" alt=" " width="800" height="202"></a></p>

<h2>
  
  
  8. Verify Real-Time Data Synchronization
</h2>

<p>For scenarios requiring capture of deleted data, enable the following configuration in PostgreSQL<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="k">public</span><span class="p">.</span><span class="n">student</span> <span class="n">REPLICA</span> <span class="k">IDENTITY</span> <span class="k">FULL</span>
</code></pre>

</div>



<p>For details, refer to this link.</p>

<p>Perform data modifications in PostgreSQL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">student</span> <span class="p">(</span><span class="n">id</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">age</span><span class="p">,</span> <span class="n">email</span><span class="p">,</span> <span class="n">phone</span><span class="p">,</span> <span class="n">score</span><span class="p">,</span> <span class="n">created_at</span><span class="p">)</span> 
<span class="k">VALUES</span> 
<span class="p">(</span><span class="mi">6</span><span class="p">,</span> <span class="s1">'Frank Zhao'</span><span class="p">,</span> <span class="mi">24</span><span class="p">,</span> <span class="s1">'frank@example.com'</span><span class="p">,</span> <span class="s1">'13400134000'</span><span class="p">,</span> <span class="mi">88</span><span class="p">.</span><span class="mi">75</span><span class="p">,</span> <span class="n">NOW</span><span class="p">());</span>

<span class="k">DELETE</span> <span class="k">FROM</span> <span class="n">student</span> <span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="mi">3</span><span class="p">;</span>

<span class="k">UPDATE</span> <span class="n">student</span> 
<span class="k">SET</span> <span class="n">score</span> <span class="o">=</span> <span class="mi">95</span><span class="p">.</span><span class="mi">00</span><span class="p">,</span> <span class="n">age</span> <span class="o">=</span> <span class="mi">23</span> 
<span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="mi">2</span><span class="p">;</span>
</code></pre>

</div>



<p>Verify data changes in VeloDB:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftifdboaadc92ivtsw2vk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftifdboaadc92ivtsw2vk.png" alt=" " width="800" height="208"></a></p>

