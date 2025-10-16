---
Title: Real-time Data Analytics at Scale: Integrating Apache Flink and Apache Doris with Flink Doris Connector and Flink CDC
Description: 
Author: Apache Doris
Date: 2025-10-16T22:05:31.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  I. Introduction to Apache Doris
</h2>

<p>Apache Doris is a high-performance, real-time analytical database based on the MPP architecture. Its overall architecture is streamlined, consisting of only two system modules: FE and BE. The FE (Frontend) mainly handles request reception, query parsing, metadata management, and task scheduling, while the BE (Backend) mainly handles query execution and data storage. Apache Doris supports standard SQL and is fully compatible with the MySQL protocol, allowing databases stored in Apache Doris to be accessed through various client tools and BI software that support the MySQL protocol.</p>

<p>In typical data integration and processing pipelines, data sources such as TP databases, user behavior logs, time-series data, and local files are often collected. These data sources are processed by data integration tools or ETL tools and then written into the real-time data warehouse, Apache Doris. Doris provides querying and analysis capabilities for downstream data applications, covering typical scenarios such as BI report analysis, OLAP multi-dimensional analysis, Ad-hoc queries, and log search and analysis.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg465b4s6toivbj3js9ea.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg465b4s6toivbj3js9ea.png" alt=" " width="800" height="295"></a></p>

<p>Flink-Doris-Connector is an integration solution between Apache Doris and Apache Flink for real-time data processing ETL scenarios. It leverages Flink's real-time computing capabilities to build an efficient data processing and analysis pipeline. The main use cases for Flink-Doris-Connector are divided into three types:</p>

<ol>
<li><p>Scan: Typically used for data synchronization or joint analysis with other data sources.</p></li>
<li><p>Lookup Join: Joins data from real-time streams with dimension tables in Doris.</p></li>
<li><p>Real-time ETL: Uses Flink to clean data and then writes it into Doris in real-time.</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo9r0s68imftadta0i295.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo9r0s68imftadta0i295.png" alt=" " width="800" height="365"></a></p>

<h2>
  
  
  II. Design and Implementation of Flink-Doris-Connector in Typical Scenarios
</h2>

<p>This section introduces the design and implementation of Flink-Doris-Connector in three scenarios: Scan, Lookup Join, and Write.</p>

<h3>
  
  
  01 Scan Scenario
</h3>

<p>The Scan scenario refers to quickly extracting existing data from Doris. When reading large amounts of data from Doris, traditional JDBC methods may face performance bottlenecks. Therefore, the Flink-Doris-Connector utilizes the Doris Source to take full advantage of Doris's distributed architecture and Flink's parallel processing capabilities, achieving more efficient data synchronization.</p>

<p>Doris Source Reading Process:</p>

<ul>
<li><p>The Job Manager sends a request for a query plan to the FE (Frontend), which returns the corresponding BE (Backend) and Tablet for the data to be queried.</p></li>
<li><p>Requests are distributed to different TaskManagers based on the different BEs.</p></li>
<li><p>Through the TaskManager, data from each Tablet on the corresponding BE is read directly.</p></li>
</ul>

<p>By using this method, we can leverage Flink's distributed processing capabilities to improve the efficiency of the entire data synchronization process.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmth2jmao0fwmekanh13y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmth2jmao0fwmekanh13y.png" alt=" " width="800" height="362"></a></p>

<h3>
  
  
  02 Lookup Join Scenarios
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw1etvak5d9cu4slarhou.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw1etvak5d9cu4slarhou.png" alt=" " width="800" height="386"></a></p>

<p>For scenarios where dimension tables are stored in Doris, real-time stream data can be joined with Doris dimension tables through Lookup Join.</p>

<h4>
  
  
  Limitations of JDBC Connector
</h4>

<p>Doris supports the MySQL protocol, so it can directly use the JDBC Connector for Lookup Join. However, this method has certain limitations:</p>

<ul>
<li><p>The Lookup Join in the JDBC Connector performs synchronous queries, which means each piece of data in the real-time stream must wait for the query result from Doris, increasing latency.</p></li>
<li><p>It only supports querying one record at a time. When the upstream data throughput is high, this can easily lead to performance bottlenecks and backpressure.</p></li>
</ul>

<h4>
  
  
  Optimizations in Flink-Doris-Connector
</h4>

<p>To address the limitations of the Lookup Join scenario, the Flink-Doris-Connector implements asynchronous Lookup Join and batch accumulation query optimizations:</p>

<ul>
<li><p>Support for Asynchronous Lookup Join: Asynchronous Lookup Join means that data in the real-time stream does not need to explicitly wait for the query result of each record, significantly reducing latency.</p></li>
<li><p>Support for Batch Accumulation Query: Real-time stream data is appended to a queue. A background listener thread (Watcher) retrieves data from the queue and pushes it to a query execution worker thread pool. The worker threads concatenate the received batch of data into a single <code>Union All</code> query and send this query to Doris.</p></li>
</ul>

<p>By implementing asynchronous Lookup Join and batch accumulation queries, the throughput of dimension table joins can be significantly improved when upstream data volume is large, ensuring efficient data reading and processing.</p>

<h3>
  
  
  03 Real-time ETL Scenario
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F877guq4phaodxg7rh332.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F877guq4phaodxg7rh332.png" alt=" " width="800" height="215"></a></p>

<p>For real-time writing, the write operation of the Doris Sink is implemented using the Stream Load import method. Stream Load is one of the most common data import methods in Apache Doris, supporting the import of local files or data streams into Doris via the HTTP protocol. The main process is as follows:</p>

<ul>
<li><p>After receiving the data, the Sink will initiate a long-lived Stream Load request. During the Checkpoint interval, it continuously sends the received data to Doris in chunks.</p></li>
<li><p>At the time of Checkpoint, the previously initiated Stream Load request is committed. Only after the commit is completed will the data become visible.</p></li>
</ul>

<h4>
  
  
  How to Ensure End-to-End Exactly-Once Semantics for Data Writing
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frmv6izckg22yb4t8dzgy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frmv6izckg22yb4t8dzgy.png" alt=" " width="800" height="344"></a></p>

<p>To ensure end-to-end exactly-once semantics for data writing, let's take the example of synchronizing data from Kafka to Doris during the Checkpoint process:</p>

<ol>
<li><p>Checkpoint Time: At checkpoint time, the Source will receive a Checkpoint Barrier.</p></li>
<li><p>Source Receives Barrier: After receiving the Barrier, the Source first takes a snapshot of its own state and then passes the Checkpoint Barrier down to the Sink.</p></li>
<li><p>Sink Receives Barrier: Upon receiving the Barrier, the Sink performs a Pre-commit operation. If successful, the data is fully written to Doris. Since this is a pre-commit, the data is not yet visible to users in Doris.</p></li>
<li><p>Save Pre-Commit Transaction ID: The transaction ID of the successfully pre-committed operation is saved to the state.</p></li>
<li><p>All Operator Checkpoints Complete: Once all operator checkpoints are completed, the Job Manager sends out a notification that the current Checkpoint has been successfully completed.</p></li>
<li><p>Final Commit by Sink: The Sink then commits the previously pre-committed transaction.</p></li>
</ol>

<p>By using this two-phase commit process, end-to-end exactly-once semantics can be achieved, ensuring that each record is processed exactly once, even in the event of failures.</p>

<h4>
  
  
  Real-Time Processing and Exactly-Once
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fijrtykq16qff9pmbp1kh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fijrtykq16qff9pmbp1kh.png" alt=" " width="800" height="165"></a></p>

<p>As mentioned above, the writing operation on the Doris Sink side is bound to Checkpoint, and the delay of data writing to Doris depends on the Checkpoint interval. However, in some users' scenarios, they hope that data can be written in real-time, but Checkpoint cannot be performed too frequently. Meanwhile, for some jobs, overly frequent Checkpoint will consume a lot of resources. To address this situation, Flink-Doris-Connector has introduced a batching mechanism to balance the contradiction between real-time performance and resource consumption.</p>

<p>The implementation principle of batching is that after the Sink side receives upstream data, it does not immediately write each piece of data to Doris individually. Instead, it first caches the data in the memory, and then submits the cached data to Doris through corresponding parameter settings. Combining batch writing with the primary key model in Doris can ensure the idempotency of data writing.</p>

<p>The introduction of the batching mechanism not only meets users' needs for real-time data writing but also avoids the resource consumption problem caused by frequent Checkpoint, thereby achieving the optimization of performance and efficiency.</p>

<h2>
  
  
  III. Full Database Synchronization Solution Based on Flink CDC
</h2>

<p>The above is an introduction to the typical scenarios and implementation principles of Flink-Doris-Connector. Next, let's look at one of its important applications in actual business—full database synchronization. Compared with the underlying implementation, full database synchronization is more inclined to specific usage scenarios. Based on the capabilities introduced earlier, we will further explore how to realize efficient and automated synchronization from TP databases to Doris through Flink CDC.</p>

<h3>
  
  
  01 Pain Points of Full Database Synchronization
</h3>

<p>During data migration, users usually hope to migrate data to Doris as soon as possible. However, when synchronizing TP databases, full database synchronization often faces the following challenges:</p>

<ul>
<li>
<p>Table Creation:</p>

<ul>
<li>Rapid batch creation of existing tables: There are often thousands of tables in TP databases, with different structures. For existing tables, corresponding table structures need to be created one by one in Doris;</li>
<li>Automatic creation and synchronization of new tables after the synchronization task is started: To ensure data integrity and real-time performance, the synchronization tool needs to monitor changes in the TP database in real-time and automatically create and synchronize new tables in Doris.</li>
</ul>


</li>

<li><p>Metadata Mapping: Convenient mapping of field metadata between upstream and downstream, including conversion of field types, corresponding modification of field names, etc.</p></li>

<li><p>Automatic DDL Synchronization: Operations such as adding or deleting columns will change the database structure, which in turn affects data synchronization. Therefore, the synchronization tool needs to be able to capture DDL in real-time and dynamically update the Doris table structure to ensure data accuracy and consistency.</p></li>

<li><p>Out-of-the-Box: Zero code, low threshold. An ideal synchronization tool can realize data migration and synchronization with simple configuration.</p></li>

</ul>

<h3>
  
  
  02 Implementing Full Database Synchronization Based on Flink CDC
</h3>

<p>In terms of data extraction, Flink-Doris-Connector leverages the characteristic capabilities of Flink CDC:</p>

<ul>
<li>
<p>Incremental Snapshot Reading:</p>

<ul>
<li>Lock-free reading and concurrent reading: No matter how large the amount of existing data is, the data reading speed can be improved by horizontally increasing the concurrency of Flink.</li>
<li>Resumable transmission: When the amount of existing data is large, synchronization interruption may occur, and CDC supports the connection and synchronization of interrupted tasks.</li>
</ul>


</li>

<li><p>Rich Data Source Support: Flink CDC supports various databases such as MySQL, Oracle, SQLServer, etc.</p></li>

<li><p>Seamless integration with the existing Flink ecosystem, facilitating combination with existing Flink Sources and Sinks.</p></li>

</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fek57e3l6zo174b7rq76g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fek57e3l6zo174b7rq76g.png" alt=" " width="800" height="210"></a></p>

<h4>
  
  
  One-Click Table Creation and Automatic Metadata Mapping
</h4>

<p>Flink-Doris-Connector integrates capabilities such as Flink CDC, enabling users to perform full-database synchronization with a single operation. The main principle is that after the Flink CDC Source receives the upstream data source, it will perform shunting processing, with different tables using different Sinks. Meanwhile, in the latest version of the Connector, it also supports a single Sink to synchronize multiple tables and supports the creation and synchronization of new tables.</p>

<p>After integrating the functions of Flink CDC, users only need to submit tasks through Flink-Doris-Connector to automatically create the required tables in Doris, without configuring explicit associations between upstream and downstream tables, thus realizing rapid data synchronization.</p>

<p>When the Flink task starts, the Doris-Flink-Connector will automatically identify whether the corresponding Doris table exists. If the table does not exist, the Doris Flink Connector will automatically create the table and perform shunting according to the table name, so as to realize the Sink access of multiple downstream tables; if the table exists, the synchronization task will start directly.</p>

<p>This improvement not only simplifies the configuration process but also makes the creation and synchronization of new tables more convenient, thereby improving the overall efficiency of data processing.</p>

<h4>
  
  
  Light Schema Change and Automatic DDL Synchronization
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvm2h9kifla5yshfizqzx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvm2h9kifla5yshfizqzx.png" alt=" " width="800" height="620"></a></p>

<p>Before the Apache Doris 1.2 version, the Schema Change operation was quite cumbersome, requiring manual addition or modification of data columns. When the table structure of the upstream TP database changed, it was necessary to pause the data synchronization task, wait for the Schema Change in Doris to complete, and then restart the task.</p>

<p>Schema Change Process:</p>

<ul>
<li><p>The client initiates a request to add or delete columns to the FE;</p></li>
<li><p>After receiving the request, the FE modifies the current metadata and persists the latest Schema;</p></li>
<li><p>The FE synchronizes the result of the Schema Change to the client;</p></li>
</ul>

<p>Data Load Process:</p>

<ul>
<li><p>When a subsequent import task is initiated, the FE sends the import task and the latest Schema information to the BE;</p></li>
<li><p>During data writing, each Rowset of the BE stores the Schema information of the current import;</p></li>
</ul>

<p>Query Process:</p>

<ul>
<li><p>The FE sends the query plan along with the latest Schema to the BE;</p></li>
<li><p>The BE executes the query plan using the latest Schema;</p></li>
</ul>

<p>Compaction Process:</p>

<ul>
<li><p>In the BE, compare the versions of the Rowsets involved in the merge;</p></li>
<li><p>Perform data merging according to the latest Schema Change information.</p></li>
</ul>

<p>Tests have shown that compared with the early Schema Change, the data synchronization performance of Light Schema Change has been improved by hundreds of times.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffqjrjepu3ce38l2wbucr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffqjrjepu3ce38l2wbucr.png" alt=" " width="800" height="418"></a></p>

<p>The combination of Light Schema Change and Flink-Doris-Connector enables automatic synchronization of DDL through Flink CDC, with the specific steps as follows:</p>

<ol>
<li><p>The Source side captures the upstream Schema Change information and starts DDL change synchronization;</p></li>
<li><p>The Doris Sink side identifies and parses DDL operations (adding or deleting columns);</p></li>
<li><p>Table verification is performed to determine whether Light Schema Change can be carried out;</p></li>
<li><p>Initiate the Schema Change operation;</p></li>
</ol>

<p>Based on this implementation, Doris can automatically obtain DDL statements and complete the Schema Change operation in milliseconds. When the table structure of the upstream TP database changes, there is no need to pause the data synchronization task.</p>

<h4>
  
  
  Out-of-the-Box: Example of MySQL Full Database Synchronization
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flb8sjmnabo311nqjmw2h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flb8sjmnabo311nqjmw2h.png" alt=" " width="800" height="629"></a></p>

<p>For users, as long as they have a Flink client, they can submit a full database synchronization job through the operations shown in the above figure. It supports passing in Flink configurations, such as concurrency settings and Checkpoint intervals. It also allows using regular expressions to configure the tables that need to be synchronized. Meanwhile, the configurations of Flink CDC Source and Doris Sink can be directly passed through to the specific Connector. In this way, users can conveniently submit full database synchronization jobs.</p>

<h3>
  
  
  03 Core Advantages of Flink-Doris-Connector
</h3>

<p>The above optimizations perfectly address user pain points:</p>

<ol>
<li><p>Automatic table creation, including automatic creation of existing and incremental tables, eliminating the need for users to pre-create corresponding table structures in Doris.</p></li>
<li><p>Automatic mapping of upstream and downstream fields, eliminating the need to manually write matching rules between upstream and downstream fields, saving significant labor costs.</p></li>
<li><p>Seamless synchronization of column additions and deletions: Immediately receiving upstream DDL statements and automatically implementing millisecond-level schema changes in Doris, eliminating the need for service downtime and ensuring smooth data synchronization.</p></li>
<li><p>Out-of-the-box operation reduces learning costs and allows users to focus more on their business.</p></li>
</ol>

<h2>
  
  
  IV. Future Plans
</h2>

<p>Future capabilities planned for the Flink-Doris-Connector are as follows:</p>

<ol>
<li><p>Support for real-time reads. Currently, the Doris source only scans data, reading from a bounded stream. CDC scenarios will be supported in the future, allowing Flink to stream data from Doris.</p></li>
<li><p>Multiple-table sinks. Currently, the Flink-Doris-Connector supports synchronizing multiple tables with a single sink, but the Stream Load import method only supports importing a single table. Therefore, when there are many tables, it is necessary to maintain a large number of Stream Load connections on the sink side. In the future, a single Stream Load connection will support writing to multiple tables.</p></li>
<li><p>For full-database synchronization, support will be provided for more upstream data sources, meeting more data synchronization scenarios.</p></li>
</ol>

