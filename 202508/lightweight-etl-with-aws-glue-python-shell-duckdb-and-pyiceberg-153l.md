---
Title: Lightweight ETL with AWS Glue Python Shell, DuckDB, and PyIceberg
Description: 
Author: Aki
Date: 2025-08-13T21:30:34.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Introduction
</h1>

<p>I'm Aki, an AWS Community Builder (<a href="https://x.com/jitepengin" rel="noopener noreferrer">@jitepengin</a>).</p>

<p>I previously posted an article about implementing ETL with AWS Lambda × DuckDB × PyIceberg:<br>
<a href="https://zenn.dev/penginpenguin/articles/77d4a9b1e90e3a" rel="noopener noreferrer">https://zenn.dev/penginpenguin/articles/77d4a9b1e90e3a</a></p>

<p>In this post I’ll implement an ETL that writes data in Apache Iceberg format using AWS Glue Python Shell, which—like Lambda—is often chosen for lightweight ETL workloads.</p>

<p>When Glue (Spark) or EMR is too costly, and Lambda’s 15-minute timeout is a concern, AWS Glue Python Shell can be an effective middle ground.</p>


<h1>
  
  
  What is AWS Glue Python Shell?
</h1>

<p>AWS Glue Python Shell is a Glue execution environment that runs Python scripts without Apache Spark. Compared with Lambda’s 15-minute limit, it supports much longer batch jobs—the default maximum is 48 hours—so it’s suitable for longer-running ETL tasks. It’s serverless, so you don’t manage infrastructure and you pay for what you use.</p>

<p>Compared to Glue Spark or EMR, Glue Python Shell is often cheaper for lightweight ETL and data transformations. However, one downside versus Lambda is fewer native trigger options: Glue Python Shell does not directly support S3 object-created triggers or EventBridge triggers in the same way Lambda does. Typical approaches are to invoke Glue Python Shell via Lambda or Glue Workflows. Despite constraints, when used well it can significantly expand your processing options.</p>


<h1>
  
  
  Architecture used in this article
</h1>

<p>Here’s the architecture covered in this post: when a file is uploaded to S3, a Lambda function is triggered. The Lambda only starts a Glue Python Shell job and passes the S3 path; the actual ETL runs inside the Glue Python Shell and writes Iceberg-format data back to S3. This lets you process workloads that exceed Lambda’s 15-minute limit.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffelegjqfop11ba2yal07.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffelegjqfop11ba2yal07.png" alt="architecture" width="800" height="185"></a></p>

<p>Key libraries used:</p>

<ul>
<li>
<strong>DuckDB</strong> — A fast in-memory SQL database engine.</li>
<li>
<strong>PyArrow</strong> — Efficient data transformation and transfer via the Arrow format.</li>
<li>
<strong>PyIceberg</strong> — Enables working with Iceberg tables using AWS Glue Catalog.</li>
</ul>


<h1>
  
  
  About the Libraries
</h1>
<h2>
  
  
  DuckDB
</h2>

<p>DuckDB is an embedded OLAP database engine designed for analytical queries.<br><br>
It’s lightweight, runs in memory, and works efficiently even in restricted environments like Lambda.<br><br>
It’s particularly well-suited for batch analytics and ETL workloads.<br><br>
<a href="https://duckdb.org/" rel="noopener noreferrer">https://duckdb.org/</a></p>
<h2>
  
  
  PyArrow
</h2>

<p>PyArrow is the Python binding for Apache Arrow, a high-performance columnar memory format optimized for analytics and distributed computing.<br><br>
It allows fast, zero-copy data transfer between systems and languages.<br><br>
<a href="https://arrow.apache.org/" rel="noopener noreferrer">https://arrow.apache.org/</a></p>
<h2>
  
  
  PyIceberg
</h2>

<p>PyIceberg is the Python implementation of Apache Iceberg, a table format designed for large, cloud-native data lakes.<br><br>
It enables Python applications to read, write, and manage Iceberg tables seamlessly.<br><br>
<a href="https://py.iceberg.apache.org/" rel="noopener noreferrer">https://py.iceberg.apache.org/</a></p>


<h1>
  
  
  Additional Python modules path
</h1>

<p>Set the Additional Python modules path to include:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>pyiceberg[glue]==0.9.1,pydantic==2.5.0,pydantic-core==2.14.1,annotated-types==0.6.0,duckdb==0.9.2,pyarrow==14.0.1,typing-extensions==4.8.0
</code></pre>

</div>



<p>Be mindful—dependency management can be tricky here.</p>




<h1>
  
  
  Sample Code
</h1>

<h2>
  
  
  Lambda trigger function
</h2>

<p>A simple Lambda that receives an S3 event and starts the Glue job:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">boto3</span>

<span class="k">def</span> <span class="nf">lambda_handler</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">):</span>
    <span class="n">glue</span> <span class="o">=</span> <span class="n">boto3</span><span class="p">.</span><span class="nf">client</span><span class="p">(</span><span class="sh">"</span><span class="s">glue</span><span class="sh">"</span><span class="p">)</span>

    <span class="n">s3_bucket</span> <span class="o">=</span> <span class="n">event</span><span class="p">[</span><span class="sh">'</span><span class="s">Records</span><span class="sh">'</span><span class="p">][</span><span class="mi">0</span><span class="p">][</span><span class="sh">'</span><span class="s">s3</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">bucket</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">]</span>
    <span class="n">s3_object_key</span> <span class="o">=</span> <span class="n">event</span><span class="p">[</span><span class="sh">'</span><span class="s">Records</span><span class="sh">'</span><span class="p">][</span><span class="mi">0</span><span class="p">][</span><span class="sh">'</span><span class="s">s3</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">object</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">key</span><span class="sh">'</span><span class="p">]</span>

    <span class="n">s3_input_path</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">s3://</span><span class="si">{</span><span class="n">s3_bucket</span><span class="si">}</span><span class="s">/</span><span class="si">{</span><span class="n">s3_object_key</span><span class="si">}</span><span class="sh">"</span>

    <span class="n">response</span> <span class="o">=</span> <span class="n">glue</span><span class="p">.</span><span class="nf">start_job_run</span><span class="p">(</span>
        <span class="n">JobName</span><span class="o">=</span><span class="sh">"</span><span class="s">Iceberg shell</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">Arguments</span><span class="o">=</span><span class="p">{</span>
            <span class="sh">"</span><span class="s">--s3_input</span><span class="sh">"</span><span class="p">:</span> <span class="n">s3_input_path</span>
        <span class="p">}</span>
    <span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Glue Job started: </span><span class="si">{</span><span class="n">response</span><span class="p">[</span><span class="sh">'</span><span class="s">JobRunId</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">response</span>
</code></pre>

</div>



<h2>
  
  
  AWS Glue Python Shell ETL
</h2>

<p>The Glue Python Shell job is started from Lambda with the S3 input path; the job uses DuckDB to read the file and PyIceberg to write into an Iceberg table. In this example the DuckDB query filters VendorID = 1 to demonstrate SQL-style cleansing/filters.</p>

<p>This example uses GlueCatalog access. You can also use the Glue Iceberg REST catalog approach (REST Catalog) depending on your requirements—see the link below for differences.</p>

<p>Reference about catalog access modes:<br>
<a href="https://zenn.dev/penginpenguin/articles/e44880aaa2d5e3" rel="noopener noreferrer">https://zenn.dev/penginpenguin/articles/e44880aaa2d5e3</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">boto3</span>
<span class="kn">import</span> <span class="n">sys</span>
<span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">awsglue.utils</span> <span class="kn">import</span> <span class="n">getResolvedOptions</span>


<span class="k">def</span> <span class="nf">get_job_parameters</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Get job parameters passed from the Glue job arguments.</span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">required_args</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">s3_input</span><span class="sh">'</span><span class="p">]</span>

        <span class="n">args</span> <span class="o">=</span> <span class="nf">getResolvedOptions</span><span class="p">(</span><span class="n">sys</span><span class="p">.</span><span class="n">argv</span><span class="p">,</span> <span class="n">required_args</span><span class="p">)</span>

        <span class="n">s3_file_path</span> <span class="o">=</span> <span class="n">args</span><span class="p">[</span><span class="sh">'</span><span class="s">s3_input</span><span class="sh">'</span><span class="p">]</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">object: </span><span class="si">{</span><span class="n">s3_file_path</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">s3_file_path</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">parameters error: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">raise</span>

<span class="k">def</span> <span class="nf">setup_duckdb_environment</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Properly set up the DuckDB environment on the local filesystem (/tmp).</span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">home_dir</span> <span class="o">=</span> <span class="sh">'</span><span class="s">/tmp</span><span class="sh">'</span>
        <span class="n">duckdb_dir</span> <span class="o">=</span> <span class="sh">'</span><span class="s">/tmp/.duckdb</span><span class="sh">'</span>
        <span class="n">extensions_dir</span> <span class="o">=</span> <span class="sh">'</span><span class="s">/tmp/.duckdb/extensions</span><span class="sh">'</span>

        <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">'</span><span class="s">HOME</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">home_dir</span>
        <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">'</span><span class="s">DUCKDB_HOME</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">duckdb_dir</span>
        <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">'</span><span class="s">DUCKDB_CONFIG_PATH</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">duckdb_dir</span>
        <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">'</span><span class="s">DUCKDB_EXTENSION_DIRECTORY</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">extensions_dir</span>

        <span class="n">os</span><span class="p">.</span><span class="nf">makedirs</span><span class="p">(</span><span class="n">duckdb_dir</span><span class="p">,</span> <span class="n">exist_ok</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
        <span class="n">os</span><span class="p">.</span><span class="nf">makedirs</span><span class="p">(</span><span class="n">extensions_dir</span><span class="p">,</span> <span class="n">exist_ok</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
        <span class="n">os</span><span class="p">.</span><span class="nf">chmod</span><span class="p">(</span><span class="n">duckdb_dir</span><span class="p">,</span> <span class="mo">0o755</span><span class="p">)</span>
        <span class="n">os</span><span class="p">.</span><span class="nf">chmod</span><span class="p">(</span><span class="n">extensions_dir</span><span class="p">,</span> <span class="mo">0o755</span><span class="p">)</span>

        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">DuckDB environment setup completed: </span><span class="si">{</span><span class="n">duckdb_dir</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="bp">True</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">DuckDB environment setup error: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="bp">False</span>

<span class="k">def</span> <span class="nf">read_parquet_with_duckdb</span><span class="p">(</span><span class="n">s3_input</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Read Parquet file from S3 using DuckDB and return a PyArrow table.</span><span class="sh">"""</span>
    <span class="kn">import</span> <span class="n">duckdb</span>

    <span class="n">con</span> <span class="o">=</span> <span class="n">duckdb</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="sh">'</span><span class="s">:memory:</span><span class="sh">'</span><span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SET extension_directory=</span><span class="sh">'</span><span class="s">/tmp/.duckdb/extensions</span><span class="sh">'</span><span class="s">;</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">INSTALL httpfs;</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">LOAD httpfs;</span><span class="sh">"</span><span class="p">)</span>

        <span class="n">session</span> <span class="o">=</span> <span class="n">boto3</span><span class="p">.</span><span class="nc">Session</span><span class="p">()</span>
        <span class="n">credentials</span> <span class="o">=</span> <span class="n">session</span><span class="p">.</span><span class="nf">get_credentials</span><span class="p">()</span>

        <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">SET s3_region=</span><span class="sh">'</span><span class="s">ap-northeast-1</span><span class="sh">'</span><span class="s">;</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">SET s3_access_key_id=</span><span class="sh">'</span><span class="si">{</span><span class="n">credentials</span><span class="p">.</span><span class="n">access_key</span><span class="si">}</span><span class="sh">'</span><span class="s">;</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">SET s3_secret_access_key=</span><span class="sh">'</span><span class="si">{</span><span class="n">credentials</span><span class="p">.</span><span class="n">secret_key</span><span class="si">}</span><span class="sh">'</span><span class="s">;</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">credentials</span><span class="p">.</span><span class="n">token</span><span class="p">:</span>
            <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">SET s3_session_token=</span><span class="sh">'</span><span class="si">{</span><span class="n">credentials</span><span class="p">.</span><span class="n">token</span><span class="si">}</span><span class="sh">'</span><span class="s">;</span><span class="sh">"</span><span class="p">)</span>

        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Reading data from S3: </span><span class="si">{</span><span class="n">s3_input</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">sql</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">SELECT * FROM read_parquet(</span><span class="sh">'</span><span class="si">{</span><span class="n">s3_input</span><span class="si">}</span><span class="sh">'</span><span class="s">) WHERE VendorID = 1</span><span class="sh">"</span>
        <span class="n">res</span> <span class="o">=</span> <span class="n">con</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="n">sql</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">res</span><span class="p">.</span><span class="nf">arrow</span><span class="p">()</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">DuckDB error: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">raise</span>
    <span class="k">finally</span><span class="p">:</span>
        <span class="n">con</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>

<span class="k">def</span> <span class="nf">create_and_write_iceberg_table_fixed</span><span class="p">(</span><span class="n">arrow_table</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Create (or load) an Iceberg table and append an Arrow table to it.</span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Fixed Iceberg table creation and writing started...</span><span class="sh">"</span><span class="p">)</span>

        <span class="kn">from</span> <span class="n">pyiceberg.catalog</span> <span class="kn">import</span> <span class="n">load_catalog</span>
        <span class="kn">from</span> <span class="n">pyiceberg.schema</span> <span class="kn">import</span> <span class="n">Schema</span>
        <span class="kn">from</span> <span class="n">pyiceberg.types</span> <span class="kn">import</span> <span class="n">NestedField</span><span class="p">,</span> <span class="n">StringType</span><span class="p">,</span> <span class="n">IntegerType</span><span class="p">,</span> <span class="n">DoubleType</span><span class="p">,</span> <span class="n">TimestampType</span>

        <span class="n">catalog_config</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">glue</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">s3://your-bucket/your-warehouse/</span><span class="sh">"</span><span class="p">,</span> <span class="c1"># Adjust to your environment.
</span>            <span class="sh">"</span><span class="s">region</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">ap-northeast-1</span><span class="sh">"</span>
        <span class="p">}</span>

        <span class="n">catalog</span> <span class="o">=</span> <span class="nf">load_catalog</span><span class="p">(</span><span class="sh">"</span><span class="s">glue_catalog</span><span class="sh">"</span><span class="p">,</span> <span class="o">**</span><span class="n">catalog_config</span><span class="p">)</span>

        <span class="n">table_identifier</span> <span class="o">=</span> <span class="sh">"</span><span class="s">icebergdb.yellow_tripdata</span><span class="sh">"</span>

        <span class="n">table</span> <span class="o">=</span> <span class="n">catalog</span><span class="p">.</span><span class="nf">load_table</span><span class="p">(</span><span class="n">table_identifier</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Target data to write: </span><span class="si">{</span><span class="n">arrow_table</span><span class="p">.</span><span class="n">num_rows</span><span class="si">:</span><span class="p">,</span><span class="si">}</span><span class="s"> rows</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Write Arrow table to Iceberg table
</span>        <span class="n">table</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">arrow_table</span><span class="p">)</span>

        <span class="k">return</span> <span class="bp">True</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Fixed Iceberg table creation/writing error: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="kn">import</span> <span class="n">traceback</span>
        <span class="n">traceback</span><span class="p">.</span><span class="nf">print_exc</span><span class="p">()</span>
        <span class="k">return</span> <span class="bp">False</span>

<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Fixed Iceberg table creation/writing test...</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Set up DuckDB environment
</span>    <span class="k">if</span> <span class="ow">not</span> <span class="nf">setup_duckdb_environment</span><span class="p">():</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Failed to set up DuckDB environment</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="kn">import</span> <span class="n">pyiceberg</span>

        <span class="c1"># Read data
</span>        <span class="n">s3_input</span> <span class="o">=</span> <span class="nf">get_job_parameters</span><span class="p">()</span>

        <span class="n">arrow_tbl</span> <span class="o">=</span> <span class="nf">read_parquet_with_duckdb</span><span class="p">(</span><span class="n">s3_input</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Data read success: </span><span class="si">{</span><span class="n">arrow_tbl</span><span class="p">.</span><span class="n">shape</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Create and write to fixed Iceberg table
</span>        <span class="k">if</span> <span class="nf">create_and_write_iceberg_table_fixed</span><span class="p">(</span><span class="n">arrow_tbl</span><span class="p">):</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Fixed Iceberg table creation/writing fully successful!</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Fixed Iceberg table creation/writing failed</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Main error: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="kn">import</span> <span class="n">traceback</span>
        <span class="n">traceback</span><span class="p">.</span><span class="nf">print_exc</span><span class="p">()</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>

</code></pre>

</div>



<h1>
  
  
  Results
</h1>

<p>Sample data used in this demo is the commonly used NYC taxi dataset:<br>
<a href="https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page" rel="noopener noreferrer">https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page</a></p>
<h2>
  
  
  Before
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxkm9geycm6hoza3afw9n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxkm9geycm6hoza3afw9n.png" width="800" height="332"></a></p>
<h2>
  
  
  After
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwu8tbbu89si8i4r6a91d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwu8tbbu89si8i4r6a91d.png" width="800" height="337"></a></p>

<p><strong>We successfully registered data into an Iceberg table (OTF) using this approach.</strong></p>
<h1>
  
  
  Reference: Lambda implementation (for comparison)
</h1>

<p>In a previous article I implemented the same libraries directly inside Lambda. Because Lambda has a 15-minute runtime limit, large or complex workloads may time out or hit memory limits. Migrating the same library stack to Glue Python Shell can avoid those constraints.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">duckdb</span>
<span class="kn">import</span> <span class="n">pyarrow</span> <span class="k">as</span> <span class="n">pa</span>
<span class="kn">from</span> <span class="n">pyiceberg.catalog.glue</span> <span class="kn">import</span> <span class="n">GlueCatalog</span>  

<span class="k">def</span> <span class="nf">lambda_handler</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">):</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="c1"># Connect to DuckDB and set the home directory
</span>        <span class="n">duckdb_connection</span> <span class="o">=</span> <span class="n">duckdb</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">database</span><span class="o">=</span><span class="sh">'</span><span class="s">:memory:</span><span class="sh">'</span><span class="p">)</span>
        <span class="n">duckdb_connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SET home_directory=</span><span class="sh">'</span><span class="s">/tmp</span><span class="sh">'"</span><span class="p">)</span> 

        <span class="c1"># Install and load the httpfs extension
</span>        <span class="n">duckdb_connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">INSTALL httpfs;</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">duckdb_connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">LOAD httpfs;</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Load data from S3 using DuckDB
</span>        <span class="n">s3_bucket</span> <span class="o">=</span> <span class="n">event</span><span class="p">[</span><span class="sh">'</span><span class="s">Records</span><span class="sh">'</span><span class="p">][</span><span class="mi">0</span><span class="p">][</span><span class="sh">'</span><span class="s">s3</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">bucket</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">]</span>
        <span class="n">s3_object_key</span> <span class="o">=</span> <span class="n">event</span><span class="p">[</span><span class="sh">'</span><span class="s">Records</span><span class="sh">'</span><span class="p">][</span><span class="mi">0</span><span class="p">][</span><span class="sh">'</span><span class="s">s3</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">object</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">key</span><span class="sh">'</span><span class="p">]</span>

        <span class="n">s3_input_path</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">s3://</span><span class="si">{</span><span class="n">s3_bucket</span><span class="si">}</span><span class="s">/</span><span class="si">{</span><span class="n">s3_object_key</span><span class="si">}</span><span class="sh">"</span>

        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">s3_input_path: </span><span class="si">{</span><span class="n">s3_input_path</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="n">query</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"""</span><span class="s">
            SELECT * FROM read_parquet(</span><span class="sh">'</span><span class="si">{</span><span class="n">s3_input_path</span><span class="si">}</span><span class="sh">'</span><span class="s">) WHERE VendorID = 1
        </span><span class="sh">"""</span>
        <span class="c1"># Execute SQL and retrieve results as a PyArrow Table
</span>        <span class="n">result_arrow_table</span> <span class="o">=</span> <span class="n">duckdb_connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="n">query</span><span class="p">).</span><span class="nf">fetch_arrow_table</span><span class="p">()</span>

        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Number of rows retrieved: </span><span class="si">{</span><span class="n">result_arrow_table</span><span class="p">.</span><span class="n">num_rows</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Data schema: </span><span class="si">{</span><span class="n">result_arrow_table</span><span class="p">.</span><span class="n">schema</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Configure Glue Catalog (to access Iceberg table)
</span>        <span class="n">catalog</span> <span class="o">=</span> <span class="nc">GlueCatalog</span><span class="p">(</span><span class="n">region_name</span><span class="o">=</span><span class="sh">"</span><span class="s">ap-northeast-1</span><span class="sh">"</span><span class="p">,</span> <span class="n">database</span><span class="o">=</span><span class="sh">"</span><span class="s">icebergdb</span><span class="sh">"</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">my_catalog</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Adjust to your environment.
</span>
        <span class="c1"># Load the table
</span>        <span class="n">namespace</span> <span class="o">=</span> <span class="sh">"</span><span class="s">icebergdb</span><span class="sh">"</span>  <span class="c1"># Adjust to your environment.
</span>        <span class="n">table_name</span> <span class="o">=</span> <span class="sh">"</span><span class="s">yellow_tripdata</span><span class="sh">"</span>  <span class="c1"># Adjust to your environment.
</span>        <span class="n">iceberg_table</span> <span class="o">=</span> <span class="n">catalog</span><span class="p">.</span><span class="nf">load_table</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">namespace</span><span class="si">}</span><span class="s">.</span><span class="si">{</span><span class="n">table_name</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Append data to the Iceberg table in bulk
</span>        <span class="n">iceberg_table</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">result_arrow_table</span><span class="p">)</span> 

        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Data has been appended to S3 in Iceberg format.</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">An error occurred: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

</code></pre>

</div>



<h1>
  
  
  Pros &amp; Cons of the Glue Python Shell approach
</h1>

<p><strong>Pros</strong></p>

<ul>
<li>Can run for much longer than Lambda—can handle large files and long processing.</li>
<li>Serverless and elastic—no infra management.</li>
<li>Can reuse Python libraries like DuckDB and PyIceberg as-is.</li>
<li>Writing to Iceberg gives you snapshots, time travel, and other table-management features.</li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li>Startup is slower than Lambda due to loading Python libraries.</li>
<li>Glue Python Shell only supports Python (no other languages).</li>
<li>No Spark-based distributed processing—unsuitable for very large-scale or very high throughput workloads.</li>
<li>You need to manage dependency versions carefully.</li>
</ul>

<h1>
  
  
  Cost comparison vs Lambda
</h1>

<h2>
  
  
  Lambda
</h2>

<p>I compared costs under a scenario limited to Lambda’s 15-minute maximum. (Memory for Lambda was set to 2048 MB in the example.) Depending on your workload and runtime, Glue Python Shell can sometimes be cheaper than Lambda.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6wcegypoy949jg8y4ewn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6wcegypoy949jg8y4ewn.png" width="800" height="325"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwp5j9jaaw6qu9gujlpt6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwp5j9jaaw6qu9gujlpt6.png" width="800" height="228"></a></p>

<h2>
  
  
  AWS Glue Python Shell
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcyr52imbfphj48oltks8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcyr52imbfphj48oltks8.png" width="800" height="236"></a></p>

<h1>
  
  
  Conclusion
</h1>

<p>This article showed how to use AWS Glue Python Shell together with DuckDB and PyIceberg to implement ETL that writes data in Apache Iceberg format on S3. Glue Python Shell sits between Lambda and Glue Spark/EMR as a serverless execution option that supports longer runtimes without the management overhead of Spark clusters.</p>

<p>By combining DuckDB and PyIceberg you can build a flexible, lower-cost data processing pipeline—especially when Lambda’s runtime is insufficient but Glue Spark / EMR would be overkill.</p>

<p>Iceberg supports schema evolution, time travel, and other powerful table features; it’s becoming a mainstream choice for OTF workflows, so I expect its adoption to keep growing.</p>

<p>I hope this article helps anyone considering lightweight ETL or near-real-time ingestion into Iceberg tables.</p>

