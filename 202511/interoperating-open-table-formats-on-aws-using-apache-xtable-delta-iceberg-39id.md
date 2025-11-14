---
Title: Interoperating Open Table Formats on AWS Using Apache XTable (Delta Iceberg)
Description: 
Author: Aki
Date: 2025-11-14T21:56:39.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Original Japanese article</strong>: <a href="https://zenn.dev/penginpenguin/articles/caa7278ebd2b5d" rel="noopener noreferrer">Apache XTableを使ったAWS上でのOpen Table Format相互運用（Delta→Iceberg）</a></p>
</blockquote>

<h1>
  
  
  Introduction
</h1>

<p>I'm Aki, an AWS Community Builder (<a href="https://x.com/jitepengin" rel="noopener noreferrer">@jitepengin</a>).</p>

<p>Lakehouse architectures have become increasingly common in modern data platforms. In many cases, multiple Open Table Format (OTF) are used simultaneously, such as:</p>

<ul>
<li>Delta Lake</li>
<li>Apache Iceberg</li>
<li>Apache Hudi</li>
</ul>

<p>For example:<br>
<em>“Our existing lakehouse uses Delta Lake, but the new project wants to adopt Iceberg…”</em><br>
This kind of scenario is becoming more frequent.</p>

<p>When this happens, <strong>interoperability</strong> between OTF becomes a key challenge.<br>
Interoperability refers to the ability to integrate different table formats and lakehouses smoothly and seamlessly.</p>

<p>For instance, Microsoft Fabric provides partial Delta ⇔ Iceberg sync via the Iceberg Shortcut feature.<br>
However, it still suffers from type conversion errors, and full bidirectional sync is not yet possible.</p>

<p>This is where <strong>Apache XTable</strong> has been gaining attention.<br>
In this post, I’ll walk through how I used XTable to convert and synchronize Delta Lake tables to Apache Iceberg on AWS.</p>


<h1>
  
  
  What Is Apache XTable?
</h1>

<p>Apache XTable is an open-source project designed to provide seamless interoperability between the following OTF:<br>
<a href="https://xtable.apache.org/" rel="noopener noreferrer">https://xtable.apache.org/</a></p>

<ul>
<li>Apache Hudi</li>
<li>Delta Lake</li>
<li>Apache Iceberg</li>
</ul>

<p>Key features include:</p>
<h2>
  
  
  TableFormatSync
</h2>

<p>Converts table metadata between formats.<br>
For example, you can read Delta Lake tables as Iceberg tables.</p>
<h2>
  
  
  CatalogSync
</h2>

<p>Synchronizes metadata between multiple external catalogs.<br>
Currently supported:</p>

<ul>
<li>Hive Metastore (HMS)</li>
<li>AWS Glue Data Catalog</li>
</ul>

<p>Upcoming support includes Unity Catalog, Polaris, Gravitino, DataHub, and more.</p>

<p>Polaris Catalog, which focuses on Iceberg interoperability, is also worth watching.</p>

<p>With these capabilities, you can combine the strengths of each OTF—Hudi’s real-time ingestion, Delta’s feature-rich capabilities, Iceberg’s flexibility—within a unified catalog.</p>


<h1>
  
  
  Interoperating with AWS Glue Catalog
</h1>
<h2>
  
  
  Architecture
</h2>

<p>I tested the following setup:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flkzookc7ess6wxpoeko5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flkzookc7ess6wxpoeko5.png" width="800" height="248"></a></p>

<ul>
<li>Create Delta Lake tables with Databricks (stored in S3)</li>
<li>Run XTable on EC2</li>
<li>Convert Delta → Iceberg</li>
<li>Sync the Iceberg table to AWS Glue Catalog</li>
</ul>

<p>Unity Catalog cannot be used as a source yet, so I used Delta tables directly stored in S3.</p>


<h1>
  
  
  Configuration
</h1>

<p>Below is an example configuration.<br>
The <code>sourceCatalog</code> uses a storage backend, and the target catalog is Glue.<br>
The dataset section controls the Delta → Iceberg conversion.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">sourceCatalog</span><span class="pi">:</span>
   <span class="na">catalogId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">source-catalog-id"</span>
   <span class="na">catalogType</span><span class="pi">:</span> <span class="s2">"</span><span class="s">STORAGE"</span>
   <span class="na">catalogProperties</span><span class="pi">:</span> <span class="pi">{}</span>

<span class="na">targetCatalogs</span><span class="pi">:</span>
   <span class="pi">-</span> <span class="na">catalogId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">target-catalog-id-glue"</span>
     <span class="na">catalogSyncClientImpl</span><span class="pi">:</span> <span class="s2">"</span><span class="s">org.apache.xtable.glue.GlueCatalogSyncClient"</span>
     <span class="na">catalogProperties</span><span class="pi">:</span>
        <span class="na">externalCatalog.glue.region</span><span class="pi">:</span> <span class="s2">"</span><span class="s">ap-northeast-1"</span>

<span class="na">datasets</span><span class="pi">:</span>
   <span class="pi">-</span> <span class="na">sourceCatalogTableIdentifier</span><span class="pi">:</span>
        <span class="na">storageIdentifier</span><span class="pi">:</span>
           <span class="na">tableBasePath</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://your-source-bucket"</span>
           <span class="na">tableName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">source-table"</span>
           <span class="na">tableFormat</span><span class="pi">:</span> <span class="s2">"</span><span class="s">DELTA"</span>

     <span class="na">targetCatalogTableIdentifiers</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">catalogId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">target-catalog-id-glue"</span>
          <span class="na">tableFormat</span><span class="pi">:</span> <span class="s2">"</span><span class="s">ICEBERG"</span>
          <span class="na">tableIdentifier</span><span class="pi">:</span>
             <span class="na">hierarchicalId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">db.delta_table"</span>
</code></pre>

</div>






<h1>
  
  
  Running XTable
</h1>

<p>Execute XTable with the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>java -cp "xtable-utilities/target/xtable-utilities_2.12-0.2.0-SNAPSHOT-bundled.jar:xtable-aws/target/xtable-aws-0.2.0-SNAPSHOT-bundled.jar:hudi-hive-sync-bundle-0.14.0.jar" \
  org.apache.xtable.utilities.RunCatalogSync \
  --catalogSyncConfig my_config_catalog.yaml
</code></pre>

</div>






<h1>
  
  
  Results
</h1>

<h2>
  
  
  Databricks Source Table
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuq4ugrei3oo82282mw05.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuq4ugrei3oo82282mw05.png" width="800" height="693"></a></p>

<h2>
  
  
  XTable Execution
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwclnncu230t0bgvboxwm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwclnncu230t0bgvboxwm.png" width="800" height="338"></a></p>

<h2>
  
  
  Glue Catalog
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpt6hfzzjezo9k6c19uje.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpt6hfzzjezo9k6c19uje.png" width="800" height="420"></a></p>

<p>The Iceberg table is successfully registered in the Glue Data Catalog!</p>

<h2>
  
  
  Athena Query
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiu3mp6ezlpxqojzrywb6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiu3mp6ezlpxqojzrywb6.png" width="800" height="401"></a></p>

<p>Athena can query the Iceberg table via Glue without any issues.</p>




<h1>
  
  
  Alternative Approach (EventBridge × Lambda)
</h1>

<p>Although true real-time sync is difficult, you can still implement periodic synchronization using EventBridge and Lambda.<br>
For near-real-time workflows, S3 event triggers work well.</p>

<h2>
  
  
  Scheduled Sync Architecture
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzeruug7vyk7v5510jf8y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzeruug7vyk7v5510jf8y.png" width="800" height="370"></a></p>

<ul>
<li>Data written to Delta Lake (S3)</li>
<li>EventBridge triggers Lambda</li>
<li>Lambda runs XTable Java libraries via JPype</li>
<li>Converts Delta → Iceberg and syncs metadata to Glue Catalog</li>
</ul>

<h2>
  
  
  Event-Driven Architecture (Near Real-Time)
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsunidml5n7chmaqm0wfp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsunidml5n7chmaqm0wfp.png" width="800" height="248"></a></p>

<ul>
<li>Data written to Delta Lake (S3)</li>
<li>S3 event triggers Lambda</li>
<li>Lambda uses JPype to run XTable</li>
<li>Syncs Iceberg/Delta metadata to Glue Catalog</li>
</ul>

<p>This enables low-cost near–real-time synchronization.</p>




<h1>
  
  
  What I Learned While Using XTable
</h1>

<h2>
  
  
  Small EC2 Instances Are Not Enough
</h2>

<p>XTable loads multiple dependencies:</p>

<ul>
<li>Java 11</li>
<li>Maven</li>
<li>hudi-hive-sync</li>
<li>Various XTable JARs</li>
</ul>

<p>As a result, <strong>t3.micro or t3.small instances run out of memory and fail frequently</strong>.</p>

<p>Typical issues:</p>

<ul>
<li>Java heap errors</li>
<li>Spark startup failures</li>
<li>JAR load errors</li>
<li>Class conflicts</li>
</ul>

<p>→ <strong>Use t3.medium or larger</strong></p>




<h2>
  
  
  Type Conversion Errors Are Common
</h2>

<p>Just like Microsoft Fabric’s Iceberg Shortcut, type conversion compatibility is still immature.<br>
Cross-OTF schema compatibility is challenging, and full bidirectional sync remains difficult.</p>

<p>Even though OTF emphasize schema evolution, interoperability introduces new considerations.</p>




<h1>
  
  
  Conclusion
</h1>

<p>In this post, I introduced Apache XTable — a powerful OSS for interoperating between different Open Table Formats on a unified catalog.</p>

<p>However, from hands-on experience:</p>

<ul>
<li>Type conversion between Delta and Iceberg is still unstable</li>
<li>You need sufficiently large EC2 instances</li>
<li>Schema evolution requires careful testing</li>
<li>And the project is still incubating</li>
</ul>

<p>So full production adoption is difficult at this stage.</p>

<p>That said, XTable is one of the fastest-growing solutions in the OTF interoperability space.<br>
It has the potential to break down the walls between lakehouses, reduce data silos, and enable more flexible data architectures in the future.</p>

<p>I hope this article helps anyone exploring modern data platform design or multi-OTF environments.</p>

