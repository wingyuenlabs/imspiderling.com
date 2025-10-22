---
Title: Apache Doris 4.0: One Engine for Analytics, Full-Text Search, and Vector Search
Description: 
Author: Apache Doris
Date: 2025-10-22T22:02:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>We're thrilled to announce the official release of Apache Doris 4.0—a milestone version focused on four core enhancements: <strong>1) New AI capabilities (vector search &amp; AI functions) 2) Enhanced full-text search 3) Improved ETL/ELT processing 4) Performance optimizations (TopN lazy materialization &amp; SQL cache)</strong>. This release truly delivers a "one-engine-fits-all" analytics experience.</p>

<p>This release is the result of collaborative efforts from over 200 community members, with more than 9,000 improvements and fixes submitted. A heartfelt thank you to everyone who contributed to testing, reviewing, and refining this milestone version!</p>

<p>👉 Quick Resources:</p>

<ul>
<li>GitHub Release Page: <a href="https://github.com/apache/doris/releases" rel="noopener noreferrer">https://github.com/apache/doris/releases</a>
</li>
<li>Download Doris 4.0: <a href="https://doris.apache.org/download" rel="noopener noreferrer">https://doris.apache.org/download</a>
</li>
</ul>

<h2>
  
  
  Key Highlights at a Glance
</h2>

<ol>
<li><p><strong>AI-Ready Capabilities</strong>: Built-in vector search, AI functions, and hybrid search—no external vector database required</p></li>
<li><p><strong>Enhanced Full-Text Search</strong>: New SEARCH() function with Elasticsearch-like DSL, paired with BM25 scoring for better accuracy</p></li>
<li><p><strong>Robust ETL/ELT</strong>: Spill-to-disk mechanism automatically offloads memory to disk, ensuring stability for large-scale tasks</p></li>
<li><p><strong>Performance Leap</strong>: TopN queries up to dozens of times faster with lazy materialization; SQL cache enabled by default with 100x parsing efficiency boost</p></li>
</ol>

<h2>
  
  
  I. AI Capabilities: Dual Breakthroughs in Vector Search &amp; AI Functions
</h2>

<h3>
  
  
  A. Vector Search: Natively Integrated for Simpler Architecture
</h3>

<p>Apache Doris 4.0 introduces vector indexing, enabling the combination of vector search with native SQL analytics. This eliminates the need for external vector databases, making it ideal for AI scenarios like semantic search, intelligent recommendation, and image retrieval.</p>

<h4>
  
  
  Core Vector Search Functions
</h4>

<ul>
<li><p><code>l2_distance_approximate()</code>: HNSW index-based approximate Euclidean distance calculation (smaller value = higher similarity)</p></li>
<li><p><code>inner_product_approximate()</code>: HNSW index-based approximate inner product calculation (larger value = higher similarity)</p></li>
</ul>

<h4>
  
  
  Practical Examples
</h4>

<ol>
<li>Create a table with vector index:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">doc_store</span> <span class="p">(</span>
  <span class="n">id</span> <span class="nb">BIGINT</span><span class="p">,</span>
  <span class="n">title</span> <span class="n">STRING</span><span class="p">,</span>
  <span class="n">tags</span> <span class="n">ARRAY</span><span class="o">&lt;</span><span class="n">STRING</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="n">embedding</span> <span class="n">ARRAY</span><span class="o">&lt;</span><span class="nb">FLOAT</span><span class="o">&gt;</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
  <span class="c1">-- Vector index configuration</span>
  <span class="k">INDEX</span> <span class="n">idx_vec</span> <span class="p">(</span><span class="n">embedding</span><span class="p">)</span> <span class="k">USING</span> <span class="n">ANN</span> <span class="n">PROPERTIES</span> <span class="p">(</span>
      <span class="nv">"index_type"</span>  <span class="o">=</span> <span class="nv">"hnsw"</span><span class="p">,</span>        <span class="c1">-- Only HNSW is supported</span>
      <span class="nv">"metric_type"</span> <span class="o">=</span> <span class="nv">"l2_distance"</span><span class="p">,</span> <span class="c1">-- Metric type: l2_distance/inner_product</span>
      <span class="nv">"dim"</span>         <span class="o">=</span> <span class="nv">"768"</span><span class="p">,</span>         <span class="c1">-- Vector dimension (must match imported data)</span>
      <span class="nv">"quantizer"</span>   <span class="o">=</span> <span class="nv">"flat"</span>         <span class="c1">-- Quantization: flat/sq8/sq4</span>
  <span class="p">),</span>
  <span class="c1">-- Full-text index for hybrid search</span>
  <span class="k">INDEX</span> <span class="n">idx_title</span> <span class="p">(</span><span class="n">title</span><span class="p">)</span> <span class="k">USING</span> <span class="n">INVERTED</span> <span class="n">PROPERTIES</span> <span class="p">(</span><span class="nv">"parser"</span> <span class="o">=</span> <span class="nv">"english"</span><span class="p">)</span>
<span class="p">)</span>
<span class="n">DUPLICATE</span> <span class="k">KEY</span><span class="p">(</span><span class="n">id</span><span class="p">)</span>
<span class="n">DISTRIBUTED</span> <span class="k">BY</span> <span class="n">HASH</span><span class="p">(</span><span class="n">id</span><span class="p">)</span> <span class="n">BUCKETS</span> <span class="mi">16</span>
<span class="n">PROPERTIES</span><span class="p">(</span><span class="nv">"replication_num"</span><span class="o">=</span><span class="nv">"1"</span><span class="p">);</span>
</code></pre>

</div>



<ol>
<li>Basic TopN vector search:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">l2_distance_approximate</span><span class="p">(</span><span class="n">embedding</span><span class="p">,</span> <span class="p">[</span><span class="cm">/* Your query vector */</span><span class="p">])</span> <span class="k">AS</span> <span class="n">dist</span>
<span class="k">FROM</span> <span class="n">doc_store</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">dist</span> <span class="k">ASC</span>
<span class="k">LIMIT</span> <span class="mi">10</span><span class="p">;</span>
</code></pre>

</div>



<ol>
<li>Hybrid search (full-text filtering + vector ranking):
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">title</span><span class="p">,</span>
       <span class="n">l2_distance_approximate</span><span class="p">(</span><span class="n">embedding</span><span class="p">,</span> <span class="p">[</span><span class="cm">/* Your query vector */</span><span class="p">])</span> <span class="k">AS</span> <span class="n">dist</span>
<span class="k">FROM</span> <span class="n">doc_store</span>
<span class="k">WHERE</span> <span class="n">title</span> <span class="n">MATCH_ANY</span> <span class="s1">'music'</span>                <span class="c1">-- Full-text index filtering</span>
  <span class="k">AND</span> <span class="n">array_contains</span><span class="p">(</span><span class="n">tags</span><span class="p">,</span> <span class="s1">'recommendation'</span><span class="p">)</span> <span class="c1">-- Structured filtering</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">dist</span> <span class="k">ASC</span>
<span class="k">LIMIT</span> <span class="mi">5</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  Key Parameter Notes
</h4>

<ul>
<li><p><strong>quantizer</strong>: Quantization affects memory usage—SQ8 index is approximately 1/3 the size of FLAT index, trading minor recall for cost savings</p></li>
<li><p><strong>Pre-filtering Mechanism</strong>: First filters data via precise indexes (e.g., inverted indexes), then performs vector search on the result set to ensure recall stability</p></li>
<li><p><strong>Table Model Limitation</strong>: Currently only supports Duplicate Key table model; vector column must be <code>ARRAY&lt;FLOAT&gt; NOT NULL</code></p></li>
</ul>

<h3>
  
  
  B. AI Functions: Call LLMs Directly via SQL
</h3>

<p>Data analysts can call large language models (LLMs) through simple SQL statements—no glue code required. Supported use cases include information extraction, sentiment analysis, and text summarization. Currently integrated with locally deployed models and major LLMs like OpenAI, Anthropic, Gemini, DeepSeek, Zhipu AI, and Qwen.</p>

<h4>
  
  
  Core AI Functions List
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Function</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>AI_CLASSIFY</td>
<td>Matches text to the most relevant label from a specified set</td>
</tr>
<tr>
<td>AI_EXTRACT</td>
<td>Extracts key information from text based on specified labels</td>
</tr>
<tr>
<td>AI_SENTIMENT</td>
<td>Analyzes text sentiment (positive/negative/neutral/mixed)</td>
</tr>
<tr>
<td>AI_SUMMARIZE</td>
<td>Generates a concise summary of the text</td>
</tr>
<tr>
<td>AI_FILTER</td>
<td>Validates text correctness and returns a boolean value</td>
</tr>
<tr>
<td>AI_TRANSLATE</td>
<td>Translates text to a specified language</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  Practical Example: Resume-Job Matching
</h4>

<p>Use AI_FILTER for semantic matching between resumes and job requirements:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="c1">-- 1. Create candidate and job tables</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">candidate_profiles</span> <span class="p">(</span>
    <span class="n">candidate_id</span> <span class="nb">INT</span><span class="p">,</span>
    <span class="n">name</span>         <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">50</span><span class="p">),</span>
    <span class="n">self_intro</span>   <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">500</span><span class="p">)</span>
<span class="p">)</span>
<span class="n">DUPLICATE</span> <span class="k">KEY</span><span class="p">(</span><span class="n">candidate_id</span><span class="p">)</span>
<span class="n">DISTRIBUTED</span> <span class="k">BY</span> <span class="n">HASH</span><span class="p">(</span><span class="n">candidate_id</span><span class="p">)</span> <span class="n">BUCKETS</span> <span class="mi">1</span>
<span class="n">PROPERTIES</span> <span class="p">(</span><span class="nv">"replication_num"</span> <span class="o">=</span> <span class="nv">"1"</span><span class="p">);</span>

<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">job_requirements</span> <span class="p">(</span>
    <span class="n">job_id</span>   <span class="nb">INT</span><span class="p">,</span>
    <span class="n">title</span>    <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">100</span><span class="p">),</span>
    <span class="n">jd_text</span>  <span class="nb">VARCHAR</span><span class="p">(</span><span class="mi">500</span><span class="p">)</span>
<span class="p">)</span>
<span class="n">DUPLICATE</span> <span class="k">KEY</span><span class="p">(</span><span class="n">job_id</span><span class="p">)</span>
<span class="n">DISTRIBUTED</span> <span class="k">BY</span> <span class="n">HASH</span><span class="p">(</span><span class="n">job_id</span><span class="p">)</span> <span class="n">BUCKETS</span> <span class="mi">1</span>
<span class="n">PROPERTIES</span> <span class="p">(</span><span class="nv">"replication_num"</span> <span class="o">=</span> <span class="nv">"1"</span><span class="p">);</span>

<span class="c1">-- 2. Insert test data</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">candidate_profiles</span> <span class="k">VALUES</span>
<span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s1">'Alice'</span><span class="p">,</span> <span class="s1">'I am a senior backend engineer with 7 years of experience in Java, Spring Cloud and high-concurrency systems.'</span><span class="p">),</span>
<span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s1">'Bob'</span><span class="p">,</span>   <span class="s1">'Frontend developer focusing on React, TypeScript and performance optimization for e-commerce sites.'</span><span class="p">),</span>
<span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">'Cathy'</span><span class="p">,</span> <span class="s1">'Data scientist specializing in NLP, large language models and recommendation systems.'</span><span class="p">);</span>

<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">job_requirements</span> <span class="k">VALUES</span>
<span class="p">(</span><span class="mi">101</span><span class="p">,</span> <span class="s1">'Backend Engineer'</span><span class="p">,</span> <span class="s1">'Looking for a senior backend engineer with deep Java expertise and experience designing distributed systems.'</span><span class="p">),</span>
<span class="p">(</span><span class="mi">102</span><span class="p">,</span> <span class="s1">'ML Engineer'</span><span class="p">,</span>      <span class="s1">'Seeking a data scientist or ML engineer familiar with NLP and large language models.'</span><span class="p">);</span>

<span class="c1">-- 3. Semantic matching query</span>
<span class="k">SELECT</span>
    <span class="k">c</span><span class="p">.</span><span class="n">candidate_id</span><span class="p">,</span> <span class="k">c</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
    <span class="n">j</span><span class="p">.</span><span class="n">job_id</span><span class="p">,</span> <span class="n">j</span><span class="p">.</span><span class="n">title</span>
<span class="k">FROM</span> <span class="n">candidate_profiles</span> <span class="k">AS</span> <span class="k">c</span>
<span class="k">JOIN</span> <span class="n">job_requirements</span> <span class="k">AS</span> <span class="n">j</span>
<span class="k">WHERE</span> <span class="n">AI_FILTER</span><span class="p">(</span><span class="n">CONCAT</span><span class="p">(</span><span class="s1">'Does the following candidate self-introduction match the job description?'</span><span class="p">,</span> 
                <span class="s1">'Job: '</span><span class="p">,</span> <span class="n">j</span><span class="p">.</span><span class="n">jd_text</span><span class="p">,</span> <span class="s1">' Candidate: '</span><span class="p">,</span> <span class="k">c</span><span class="p">.</span><span class="n">self_intro</span><span class="p">));</span>
</code></pre>

</div>



<p>Query Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
+--------------+-------+--------+------------------+
| candidate_id | name  | job_id | title            |
+--------------+-------+--------+------------------+
|            3 | Cathy |    102 | ML Engineer      |
|            1 | Alice |    101 | Backend Engineer |
+--------------+-------+--------+------------------+
</code></pre>

</div>



<h2>
  
  
  II. Enhanced Full-Text Search: More Flexible &amp; Accurate
</h2>

<p>Apache Doris 4.0 redefines full-text search capabilities with Elasticsearch-like lightweight DSL, paired with the BM25 scoring algorithm. This balances performance and accuracy, making it perfect for hybrid search scenarios.</p>

<h3>
  
  
  A. New SEARCH() Function: Unified Search Entry
</h3>

<p>Consolidates complex text search operators with syntax similar to Elasticsearch Query String, reducing migration costs and supporting multi-condition index pushdown for better performance.</p>

<h4>
  
  
  Core Syntax Examples
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="c1">-- 1. Term query</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">docs</span> <span class="k">WHERE</span> <span class="k">search</span><span class="p">(</span><span class="s1">'title:apache'</span><span class="p">);</span>

<span class="c1">-- 2. Multi-value matching (ANY matches any, ALL matches all)</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">docs</span> <span class="k">WHERE</span> <span class="k">search</span><span class="p">(</span><span class="s1">'tags:ANY(java python golang)'</span><span class="p">);</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">docs</span> <span class="k">WHERE</span> <span class="k">search</span><span class="p">(</span><span class="s1">'tags:ALL(machine learning)'</span><span class="p">);</span>

<span class="c1">-- 3. Multi-field boolean combination</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">docs</span>
<span class="k">WHERE</span> <span class="k">search</span><span class="p">(</span><span class="s1">'(title:Doris OR content:database) AND NOT category:archived'</span><span class="p">);</span>

<span class="c1">-- 4. Combine with structured filtering</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">docs</span>
<span class="k">WHERE</span> <span class="k">search</span><span class="p">(</span><span class="s1">'title:apache'</span><span class="p">)</span> <span class="k">AND</span> <span class="n">publish_date</span> <span class="o">&gt;=</span> <span class="s1">'2025-01-01'</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  B. BM25 Scoring Algorithm: Improved Relevance
</h3>

<p>Replaces the traditional TF-IDF algorithm, dynamically adjusting term frequency weights based on document length. This significantly improves accuracy for long-text and multi-field search scenarios.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Sort by relevance score</span>
<span class="k">SELECT</span> <span class="o">*</span><span class="p">,</span> <span class="n">score</span><span class="p">()</span> <span class="k">as</span> <span class="n">score</span> 
<span class="k">FROM</span> <span class="n">search_demo</span> 
<span class="k">WHERE</span> <span class="n">content</span> <span class="n">MATCH_ANY</span> <span class="s1">'search query'</span> 
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">score</span> <span class="k">DESC</span> 
<span class="k">LIMIT</span> <span class="mi">10</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  C. Enhanced Tokenization Capabilities
</h3>

<p>New multilingual tokenizers and custom tokenization pipelines meet diverse text processing needs.</p>

<h4>
  
  
  1. New Built-in Tokenizers
</h4>

<ul>
<li><p><strong>ICU Tokenizer</strong>: Adapts to multilingual mixed documents, supporting complex writing systems like Arabic and Thai</p></li>
<li><p><strong>Basic Tokenizer</strong>: Lightweight and high-performance, suitable for simple scenarios like log processing<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- ICU Tokenizer example (multilingual mix)</span>
<span class="k">SELECT</span> <span class="n">TOKENIZE</span><span class="p">(</span><span class="s1">'مرحبا بالعالم Hello 世界'</span><span class="p">,</span> <span class="s1">'"parser"="icu"'</span><span class="p">);</span>
<span class="c1">-- Result: ["مرحبا", "بالعالم", "Hello", "世界"]</span>

<span class="c1">-- Basic Tokenizer example (Chinese character splitting)</span>
<span class="k">SELECT</span> <span class="n">TOKENIZE</span><span class="p">(</span><span class="s1">'你好世界'</span><span class="p">,</span> <span class="s1">'"parser"="basic"'</span><span class="p">);</span>
<span class="c1">-- Result: ["你", "好", "世", "界"]</span>
</code></pre>

</div>



<h4>
  
  
  2. Custom Tokenization Pipelines
</h4>

<p>Build personalized tokenization logic through chained configuration of char filters, tokenizers, and token filters. Reusable components reduce maintenance costs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Example 1: Split identifiers by . and _</span>
<span class="c1">-- 1) Create token filter</span>
<span class="k">CREATE</span> <span class="n">INVERTED</span> <span class="k">INDEX</span> <span class="n">TOKEN_FILTER</span> <span class="n">IF</span> <span class="k">NOT</span> <span class="k">EXISTS</span> <span class="n">complex_word_splitter</span>
<span class="n">PROPERTIES</span>
<span class="p">(</span>
    <span class="nv">"type"</span> <span class="o">=</span> <span class="nv">"word_delimiter"</span><span class="p">,</span>
    <span class="nv">"type_table"</span> <span class="o">=</span> <span class="nv">"[. =&gt; SUBWORD_DELIM], [_ =&gt; SUBWORD_DELIM]"</span>
<span class="p">);</span>

<span class="c1">-- 2) Create custom analyzer</span>
<span class="k">CREATE</span> <span class="n">INVERTED</span> <span class="k">INDEX</span> <span class="n">ANALYZER</span> <span class="n">IF</span> <span class="k">NOT</span> <span class="k">EXISTS</span> <span class="n">complex_identifier_analyzer</span>
<span class="n">PROPERTIES</span>
<span class="p">(</span>
    <span class="nv">"tokenizer"</span> <span class="o">=</span> <span class="nv">"standard"</span><span class="p">,</span>
    <span class="nv">"token_filter"</span> <span class="o">=</span> <span class="nv">"complex_word_splitter, lowercase"</span>
<span class="p">);</span>

<span class="c1">-- Test tokenization effect</span>
<span class="k">SELECT</span> <span class="n">TOKENIZE</span><span class="p">(</span><span class="s1">'apy217.39_202501260000026_526'</span><span class="p">,</span> <span class="s1">'"analyzer"="complex_identifier_analyzer"'</span><span class="p">);</span>
<span class="c1">-- Result: ["apy", "217", "39", "202501260000026", "526"]</span>
</code></pre>

</div>



<h2>
  
  
  III. Robust ETL/ELT Processing: Spill-to-Disk Ensures Large-Scale Stability
</h2>

<p>The new Spill-to-Disk mechanism automatically offloads intermediate data to disk when a computation task exceeds memory limits. This prevents OOM errors and supports core operators like Hash Join, Aggregation, Sort, and CTE.</p>

<h3>
  
  
  Core Configurations
</h3>

<h4>
  
  
  1. BE Configuration (be.conf)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight ini"><code>
<span class="c"># Spill data storage path (defaults to data storage path)
</span><span class="py">spill_storage_root_path</span><span class="p">=</span><span class="s">/mnt/disk1/spill;/mnt/disk2/spill</span>
<span class="c"># Spill disk space limit (supports percentage or specific size, default 20%)
</span><span class="py">spill_storage_limit</span><span class="p">=</span><span class="s">100%</span>
</code></pre>

</div>



<h4>
  
  
  2. FE Session Variables
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Enable spill-to-disk</span>
<span class="k">set</span> <span class="n">enable_spill</span><span class="o">=</span><span class="k">true</span><span class="p">;</span>
<span class="c1">-- Single query memory limit</span>
<span class="k">set</span> <span class="n">exec_mem_limit</span> <span class="o">=</span> <span class="mi">10</span><span class="k">g</span><span class="p">;</span>
<span class="c1">-- Extend query timeout (spill scenarios may take longer)</span>
<span class="k">set</span> <span class="n">query_timeout</span> <span class="o">=</span> <span class="mi">3600</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  Execution Monitoring
</h3>

<p>Monitor spill execution via audit logs, Query Profile, and system tables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Query spill statistics from system table</span>
<span class="k">SELECT</span> <span class="n">BE_ID</span><span class="p">,</span> <span class="n">QUERY_ID</span><span class="p">,</span> <span class="n">SPILL_WRITE_BYTES_TO_LOCAL_STORAGE</span><span class="p">,</span> <span class="n">SPILL_READ_BYTES_FROM_LOCAL_STORAGE</span>
<span class="k">FROM</span> <span class="n">information_schema</span><span class="p">.</span><span class="n">backend_active_tasks</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  Performance Validation
</h3>

<p>Tested with TPC-DS 10TB dataset (3 BE nodes, 16-core CPU/64GB RAM per node, 1:52 memory-to-data ratio). All 99 queries completed successfully with a total execution time of 28,102.386 seconds—validating stability in large-scale scenarios.</p>

<h2>
  
  
  IV. Data Quality Assurance: End-to-End Accuracy &amp; Control
</h2>

<p>Apache Doris 4.0 comprehensively standardizes function behaviors, establishing an end-to-end validation mechanism from data ingestion to computation. Key optimizations focus on CAST function, floating-point calculation, and date function scenarios.</p>

<h3>
  
  
  CAST Function: Three Modes for Diverse Scenarios
</h3>

<p>CAST behavior is standardized via BNF syntax, supporting strict mode, non-strict mode, and TRY_CAST function to meet different data reliability requirements.</p>

<ul>
<li><p><strong>Strict Mode</strong>: Strictly validates against BNF rules; invalid data triggers errors (ideal for high-precision scenarios like financial reconciliation)</p></li>
<li><p><strong>Non-Strict Mode</strong>: Converts invalid data to NULL to ensure task continuity (suitable for log processing)</p></li>
<li><p><strong>TRY_CAST Function</strong>: Controls conversion logic at the individual function level; returns NULL on failure<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- TRY_CAST example</span>
<span class="k">SELECT</span> <span class="n">TRY_CAST</span><span class="p">(</span><span class="s1">'123'</span> <span class="k">AS</span> <span class="nb">INT</span><span class="p">)</span> <span class="k">AS</span> <span class="k">valid</span><span class="p">,</span> <span class="n">TRY_CAST</span><span class="p">(</span><span class="s1">'abc'</span> <span class="k">AS</span> <span class="nb">INT</span><span class="p">)</span> <span class="k">AS</span> <span class="n">invalid</span><span class="p">;</span>
<span class="c1">-- Result: valid=123, invalid=NULL</span>
</code></pre>

</div>



<h2>
  
  
  V. Performance Optimizations: Dozens of Times Faster in Key Scenarios
</h2>

<h3>
  
  
  1. TopN Queries: Lazy Materialization Breaks Bottlenecks
</h3>

<p>For high-frequency <code>SELECT * FROM table ORDER BY col LIMIT N</code> scenarios, execution is split into two phases:</p>

<ol>
<li><p>Only reads sort columns and row identifiers to quickly filter target rows matching LIMIT N</p></li>
<li><p>Precisely reads full-column data for target rows using row identifiers</p></li>
</ol>

<p>Now supports multi-table join and external table query scenarios, delivering <strong>dozens of times faster</strong> performance in wide-table, small-LIMIT scenarios.</p>

<h3>
  
  
  2. SQL Cache: Enabled by Default with 100x Parsing Boost
</h3>

<p>After resolving cache correctness issues related to permission changes, session variables, and non-deterministic functions, SQL cache is now enabled by default. For complex view queries with 163 joins and 17 unions, parsing time dropped from 400ms to 2ms—a <strong>100x efficiency improvement</strong>.</p>

<h3>
  
  
  3. JSONB Optimization: Decimal Support + 30% Performance Boost
</h3>

<ul>
<li><p>Added Decimal type support to avoid precision loss in high-precision numeric conversion</p></li>
<li><p>30%+ performance improvement for functions like json_extract, accelerating semi-structured data processing</p></li>
</ul>

<h2>
  
  
  VI. More User-Friendly Resource Management
</h2>

<p>Unified CPU/memory soft/hard limit configuration for Workload Groups, supporting both limits in the same group. Integrates with Spill-to-Disk for dynamic resource scheduling.</p>

<h4>
  
  
  Typical Scenario: Departmental Resource Isolation
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>-- Sales Department (high-priority CPU-intensive)
MIN_CPU_PERCENT=40%  -- Guarantees 40% CPU during contention
MAX_MEMORY_PERCENT=60% -- 60% memory upper limit

-- Marketing Department (low-priority CPU-intensive)
MAX_CPU_PERCENT=30%  -- 30% CPU upper limit to avoid resource preemption
MIN_MEMORY_PERCENT=20% -- 20% basic memory guarantee
</code></pre>

</div>



<h2>
  
  
  Conclusion: The Transformation Brought by Apache Doris 4.0
</h2>

<p>Apache Doris 4.0 achieves a core breakthrough in "AI readiness" by deeply integrating vector search, AI functions, and full-text search—breaking the scenario boundaries of traditional analytics engines. Whether for traditional BI scenarios like real-time dashboards and user behavior analysis, or AI-driven scenarios like document search and intelligent recommendation, a single engine can provide efficient support.</p>

<p>Download the latest version today and visit the <a href="https://doris.apache.org/docs/dev/" rel="noopener noreferrer">official documentation</a> for detailed upgrade guides. We sincerely invite more developers to join the Doris community to drive technological innovation and build a more powerful analytics ecosystem together!</p>

