---
Title: Resilience of MongoDB's WiredTiger Storage Engine to Disk Failure Compared to PostgreSQL and Oracle
Description: 
Author: Franck Pachot
Date: 2025-09-08T21:50:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>There have been <a href="https://github.com/dcramer/mangodb" rel="noopener noreferrer">jokes</a> that have contributed to persistent myths about MongoDB's durability. The authors of those myths ignore that MongoDB's storage engine is among the most robust in the industry, and it's easy to demonstrate. MongoDB uses WiredTiger (created by the same author as Berkeley DB), which provides block corruption protection stronger than that of many other databases. In this article I'll show how to reproduce a simple write loss, in a lab, and see how the database detects it to avoid returning corrupt data.</p>

<h2>
  
  
  PostgreSQL
</h2>

<p>To expose the issue when a database doesn't detect lost writes, I chose PostgreSQL for this demonstration. As of version 18, PostgreSQL enables checksums by default. I'm testing it with the Release Candidate in a docker lab:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">--rm</span> <span class="nt">-it</span> <span class="nt">--cap-add</span><span class="o">=</span>SYS_PTRACE postgres:18rc1 bash

<span class="c"># Install some utilities</span>
apt update <span class="nt">-y</span> <span class="o">&amp;&amp;</span> apt <span class="nb">install</span> <span class="nt">-y</span> strace

<span class="c"># Start PostgreSQL</span>
<span class="nv">POSTGRES_PASSWORD</span><span class="o">=</span>x <span class="se">\</span>
strace <span class="nt">-fy</span> <span class="nt">-e</span> <span class="nv">trace</span><span class="o">=</span>pread64,pwrite64 <span class="se">\</span>
docker-entrypoint.sh postgres &amp;

<span class="c"># Connect to PostgreSQL</span>
psql <span class="nt">-U</span> postgres
</code></pre>

</div>



<p>I've started PostgreSQL, tracing the read and write calls with <code>strace</code>.</p>

<p>I check that block checksum is enabled:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">postgres</span><span class="o">=#</span> <span class="k">show</span> <span class="n">data_checksums</span><span class="p">;</span>
 <span class="n">data_checksums</span>
<span class="c1">----------------</span>
 <span class="k">on</span>
<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>
</code></pre>

</div>



<p>I create a demo table and insert random data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">create</span> <span class="k">table</span> <span class="n">demo</span> <span class="p">(</span><span class="n">k</span> <span class="nb">int</span> <span class="k">primary</span> <span class="k">key</span><span class="p">,</span> <span class="n">v</span> <span class="nb">text</span><span class="p">);</span>
<span class="k">copy</span> <span class="n">demo</span> <span class="k">from</span> <span class="n">program</span> <span class="err">$$</span>
<span class="n">cat</span> <span class="o">/</span><span class="n">dev</span><span class="o">/</span><span class="n">urandom</span> <span class="o">|</span>
 <span class="n">base64</span> <span class="o">|</span>
 <span class="n">head</span> <span class="o">-</span><span class="mi">10000</span> <span class="o">|</span>
 <span class="n">awk</span> <span class="s1">'{print NR"</span><span class="se">\t</span><span class="s1">"$0}'</span> 
<span class="err">$$</span> <span class="p">;</span>
<span class="k">vacuum</span> <span class="n">demo</span><span class="p">;</span>
<span class="k">checkpoint</span><span class="p">;</span>
<span class="k">create</span> <span class="n">extension</span> <span class="n">pg_buffercache</span><span class="p">;</span>
<span class="k">select</span> <span class="k">distinct</span> <span class="n">pg_buffercache_evict</span><span class="p">(</span><span class="n">bufferid</span><span class="p">)</span> <span class="k">from</span> <span class="n">pg_buffercache</span><span class="p">;</span>
</code></pre>

</div>



<p>I triggered a checkpoint to write to disk and flushed the shared buffers, so that I can see the read IO for the next query:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">set</span> <span class="n">enable_seqscan</span> <span class="k">to</span> <span class="k">false</span><span class="p">;</span>
<span class="k">select</span> <span class="k">distinct</span> <span class="n">pg_buffercache_evict</span><span class="p">(</span><span class="n">bufferid</span><span class="p">)</span> <span class="k">from</span> <span class="n">pg_buffercache</span><span class="p">;</span>
<span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>

<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0</span><span class="nv">@j</span><span class="se">\2</span><span class="nv">25</span><span class="se">\1</span><span class="nv">R</span><span class="se">\2</span><span class="nv">32</span><span class="se">\0\0\2</span><span class="nv">10</span><span class="se">\0</span><span class="nv">8</span><span class="se">\3</span><span class="nv">6</span><span class="se">\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">50</span><span class="se">\2</span><span class="nv">37</span><span class="se">\2</span><span class="nv">0</span><span class="se">\0\3</span><span class="nv">30</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">24576</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">43</span><span class="se">\2</span><span class="nv">00</span><span class="se">\1</span><span class="nv">mj</span><span class="se">\0\0\3</span><span class="nv">24</span><span class="se">\5\0\t\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">40</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">32768</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">58</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\1</span><span class="nv">0</span><span class="se">\2</span><span class="nv">32</span><span class="se">\2</span><span class="nv">32</span><span class="se">\1\3</span><span class="nv">02</span><span class="se">\2</span><span class="nv">60</span><span class="se">\4\0</span><span class="nv">000</span><span class="se">\1</span><span class="nv">`</span><span class="se">\1\0</span><span class="nv"> </span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\2</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv"> </span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">114688</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>

  <span class="n">k</span>  <span class="o">|</span>                                      <span class="n">v</span>
<span class="c1">-----+------------------------------------------------------------------------------</span>
 <span class="mi">999</span> <span class="o">|</span> <span class="n">MQIEZSsmBjk7MRtgIZLL</span><span class="o">/</span><span class="n">MqABsjhuMR6I4LtayWfR2764PdB</span><span class="o">+</span><span class="n">AcQt2saRtFXkgBUGCLKzM8SBmKX</span>
<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>
</code></pre>

</div>



<p>This has read two pages from the index, and then the page that contains the row I'm querying. This page is in <code>base/5/16388</code> at offset <code>114688</code>.</p>

<p>I use <code>dd</code> to save the content of this page, then update the row 999, trigger a checkpoint and flush the buffer cache:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="err">\</span><span class="o">!</span> <span class="n">dd</span> <span class="n">if</span><span class="o">=/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span> <span class="k">of</span><span class="o">=</span><span class="n">block1</span><span class="p">.</span><span class="n">tmp</span> <span class="n">bs</span><span class="o">=</span><span class="mi">1</span> <span class="n">skip</span><span class="o">=</span><span class="mi">114688</span> <span class="k">count</span><span class="o">=</span><span class="mi">8192</span>

<span class="k">update</span> <span class="n">demo</span> <span class="k">set</span> <span class="n">v</span><span class="o">=</span><span class="s1">'xxxxxx'</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>
<span class="k">checkpoint</span><span class="p">;</span>
<span class="k">select</span> <span class="k">distinct</span> <span class="n">pg_buffercache_evict</span><span class="p">(</span><span class="n">bufferid</span><span class="p">)</span> <span class="k">from</span> <span class="n">pg_buffercache</span><span class="p">;</span>
</code></pre>

</div>



<p>I query the row again, and it shows the updated value:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">set</span> <span class="n">enable_seqscan</span> <span class="k">to</span> <span class="k">false</span><span class="p">;</span>
<span class="k">select</span> <span class="k">distinct</span> <span class="n">pg_buffercache_evict</span><span class="p">(</span><span class="n">bufferid</span><span class="p">)</span> <span class="k">from</span> <span class="n">pg_buffercache</span><span class="p">;</span>
<span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>

<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0</span><span class="nv">@j</span><span class="se">\2</span><span class="nv">25</span><span class="se">\1</span><span class="nv">R</span><span class="se">\2</span><span class="nv">32</span><span class="se">\0\0\2</span><span class="nv">10</span><span class="se">\0</span><span class="nv">8</span><span class="se">\3</span><span class="nv">6</span><span class="se">\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">50</span><span class="se">\2</span><span class="nv">37</span><span class="se">\2</span><span class="nv">0</span><span class="se">\0\3</span><span class="nv">30</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">24576</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">43</span><span class="se">\2</span><span class="nv">00</span><span class="se">\1</span><span class="nv">mj</span><span class="se">\0\0\3</span><span class="nv">24</span><span class="se">\5\0\t\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">40</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">32768</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">58</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\1</span><span class="nv">0m</span><span class="se">\2</span><span class="nv">61</span><span class="se">\1</span><span class="nv">K~</span><span class="se">\0\0</span><span class="nv">8</span><span class="se">\1\2</span><span class="nv">00</span><span class="se">\1\0</span><span class="nv"> </span><span class="se">\4</span><span class="nv"> </span><span class="se">\2\3\0\0\2</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv"> </span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">114688</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>

  <span class="n">k</span>  <span class="o">|</span>   <span class="n">v</span>
<span class="c1">-----+--------</span>
 <span class="mi">999</span> <span class="o">|</span> <span class="n">xxxxxx</span>
<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>
</code></pre>

</div>



<p>It reads the same index pages, but the leaf points to another table page, at offset <code>114688</code>, that holds the new value.</p>

<p>To simulate disk corruption, I copy the previous block to this new location, and query again:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="err">\</span><span class="o">!</span> <span class="n">dd</span> <span class="k">of</span><span class="o">=/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span> <span class="n">if</span><span class="o">=</span><span class="n">block1</span><span class="p">.</span><span class="n">tmp</span> <span class="n">bs</span><span class="o">=</span><span class="mi">1</span> <span class="n">seek</span><span class="o">=</span><span class="mi">114688</span> <span class="n">conv</span><span class="o">=</span><span class="n">notrunc</span>

<span class="k">set</span> <span class="n">enable_seqscan</span> <span class="k">to</span> <span class="k">false</span><span class="p">;</span>
<span class="k">select</span> <span class="k">distinct</span> <span class="n">pg_buffercache_evict</span><span class="p">(</span><span class="n">bufferid</span><span class="p">)</span> <span class="k">from</span> <span class="n">pg_buffercache</span><span class="p">;</span>
<span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>

<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0</span><span class="nv">@j</span><span class="se">\2</span><span class="nv">25</span><span class="se">\1</span><span class="nv">R</span><span class="se">\2</span><span class="nv">32</span><span class="se">\0\0\2</span><span class="nv">10</span><span class="se">\0</span><span class="nv">8</span><span class="se">\3</span><span class="nv">6</span><span class="se">\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">50</span><span class="se">\2</span><span class="nv">37</span><span class="se">\2</span><span class="nv">0</span><span class="se">\0\3</span><span class="nv">30</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">24576</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">43</span><span class="se">\2</span><span class="nv">00</span><span class="se">\1</span><span class="nv">mj</span><span class="se">\0\0\3</span><span class="nv">24</span><span class="se">\5\0\t\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">40</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">32768</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">58</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\1</span><span class="nv">0</span><span class="se">\2</span><span class="nv">32</span><span class="se">\2</span><span class="nv">32</span><span class="se">\1\3</span><span class="nv">02</span><span class="se">\2</span><span class="nv">60</span><span class="se">\4\0</span><span class="nv">000</span><span class="se">\1</span><span class="nv">`</span><span class="se">\1\0</span><span class="nv"> </span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\2</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv"> </span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">114688</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
  <span class="n">k</span>  <span class="o">|</span>                                      <span class="n">v</span>
<span class="c1">-----+------------------------------------------------------------------------------</span>
 <span class="mi">999</span> <span class="o">|</span> <span class="n">MQIEZSsmBjk7MRtgIZLL</span><span class="o">/</span><span class="n">MqABsjhuMR6I4LtayWfR2764PdB</span><span class="o">+</span><span class="n">AcQt2saRtFXkgBUGCLKzM8SBmKX</span>
<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>
</code></pre>

</div>



<p>There's no error because this new block has a correct checksum, as it is a valid block, just not at its right place. And it holds the right structure, as it comes from a block of the same table. However, it shows a row that should not be there. This is an error that can happen with a failure in the storage that does not write a block at the right place.</p>

<p>Checksum is still useful if the corruption is not aligned with well formatted blocks. For example, I'm replacing the first half of the page with the second part of the page:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">checkpoint</span><span class="p">;</span>
<span class="err">\</span><span class="o">!</span> <span class="n">dd</span> <span class="k">of</span><span class="o">=/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span> <span class="n">if</span><span class="o">=</span><span class="n">block1</span><span class="p">.</span><span class="n">tmp</span> <span class="n">bs</span><span class="o">=</span><span class="mi">1</span> <span class="n">seek</span><span class="o">=</span><span class="mi">118784</span> <span class="k">count</span><span class="o">=</span><span class="mi">4096</span> <span class="n">conv</span><span class="o">=</span><span class="n">notrunc</span>

<span class="k">set</span> <span class="n">enable_seqscan</span> <span class="k">to</span> <span class="k">false</span><span class="p">;</span>
<span class="k">select</span> <span class="k">distinct</span> <span class="n">pg_buffercache_evict</span><span class="p">(</span><span class="n">bufferid</span><span class="p">)</span> <span class="k">from</span> <span class="n">pg_buffercache</span><span class="p">;</span>
<span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>

<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0</span><span class="nv">@j</span><span class="se">\2</span><span class="nv">25</span><span class="se">\1</span><span class="nv">R</span><span class="se">\2</span><span class="nv">32</span><span class="se">\0\0\2</span><span class="nv">10</span><span class="se">\0</span><span class="nv">8</span><span class="se">\3</span><span class="nv">6</span><span class="se">\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">50</span><span class="se">\2</span><span class="nv">37</span><span class="se">\2</span><span class="nv">0</span><span class="se">\0\3</span><span class="nv">30</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">24576</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">59</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16394</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">43</span><span class="se">\2</span><span class="nv">00</span><span class="se">\1</span><span class="nv">mj</span><span class="se">\0\0\3</span><span class="nv">24</span><span class="se">\5\0\t\3</span><span class="nv">60</span><span class="se">\3</span><span class="nv">7</span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\3</span><span class="nv">40</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0\3</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37 </span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">32768</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="p">[</span><span class="n">pid</span>   <span class="mi">161</span><span class="p">]</span> <span class="n">pread64</span><span class="p">(</span><span class="mi">58</span><span class="o">&lt;/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="mi">18</span><span class="o">/</span><span class="n">docker</span><span class="o">/</span><span class="n">base</span><span class="o">/</span><span class="mi">5</span><span class="o">/</span><span class="mi">16388</span><span class="o">&gt;</span><span class="p">,</span> <span class="nv">"</span><span class="se">\0\0\0\0\1</span><span class="nv">0</span><span class="se">\2</span><span class="nv">32</span><span class="se">\2</span><span class="nv">32</span><span class="se">\1\3</span><span class="nv">02</span><span class="se">\2</span><span class="nv">60</span><span class="se">\4\0</span><span class="nv">000</span><span class="se">\1</span><span class="nv">`</span><span class="se">\1\0</span><span class="nv"> </span><span class="se">\4</span><span class="nv"> </span><span class="se">\0\0\0\0\2</span><span class="nv">20</span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv"> </span><span class="se">\2</span><span class="nv">37</span><span class="se">\3</span><span class="nv">22</span><span class="se">\0</span><span class="nv">"</span><span class="p">...,</span> <span class="mi">8192</span><span class="p">,</span> <span class="mi">114688</span><span class="p">)</span> <span class="o">=</span> <span class="mi">8192</span>
<span class="mi">2025</span><span class="o">-</span><span class="mi">09</span><span class="o">-</span><span class="mi">08</span> <span class="mi">17</span><span class="p">:</span><span class="mi">58</span><span class="p">:</span><span class="mi">41</span><span class="p">.</span><span class="mi">876</span> <span class="n">UTC</span> <span class="p">[</span><span class="mi">161</span><span class="p">]</span> <span class="n">LOG</span><span class="p">:</span>  <span class="n">page</span> <span class="n">verification</span> <span class="n">failed</span><span class="p">,</span> <span class="n">calculated</span> <span class="n">checksum</span> <span class="mi">20176</span> <span class="n">but</span> <span class="n">expected</span> <span class="mi">45250</span>
<span class="mi">2025</span><span class="o">-</span><span class="mi">09</span><span class="o">-</span><span class="mi">08</span> <span class="mi">17</span><span class="p">:</span><span class="mi">58</span><span class="p">:</span><span class="mi">41</span><span class="p">.</span><span class="mi">876</span> <span class="n">UTC</span> <span class="p">[</span><span class="mi">161</span><span class="p">]</span> <span class="k">STATEMENT</span><span class="p">:</span>  <span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>
<span class="mi">2025</span><span class="o">-</span><span class="mi">09</span><span class="o">-</span><span class="mi">08</span> <span class="mi">17</span><span class="p">:</span><span class="mi">58</span><span class="p">:</span><span class="mi">41</span><span class="p">.</span><span class="mi">876</span> <span class="n">UTC</span> <span class="p">[</span><span class="mi">161</span><span class="p">]</span> <span class="n">LOG</span><span class="p">:</span>  <span class="n">invalid</span> <span class="n">page</span> <span class="k">in</span> <span class="n">block</span> <span class="mi">14</span> <span class="k">of</span> <span class="n">relation</span> <span class="nv">"base/5/16388"</span>
<span class="mi">2025</span><span class="o">-</span><span class="mi">09</span><span class="o">-</span><span class="mi">08</span> <span class="mi">17</span><span class="p">:</span><span class="mi">58</span><span class="p">:</span><span class="mi">41</span><span class="p">.</span><span class="mi">876</span> <span class="n">UTC</span> <span class="p">[</span><span class="mi">161</span><span class="p">]</span> <span class="k">STATEMENT</span><span class="p">:</span>  <span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>
<span class="mi">2025</span><span class="o">-</span><span class="mi">09</span><span class="o">-</span><span class="mi">08</span> <span class="mi">17</span><span class="p">:</span><span class="mi">58</span><span class="p">:</span><span class="mi">41</span><span class="p">.</span><span class="mi">876</span> <span class="n">UTC</span> <span class="p">[</span><span class="mi">161</span><span class="p">]</span> <span class="n">ERROR</span><span class="p">:</span>  <span class="n">invalid</span> <span class="n">page</span> <span class="k">in</span> <span class="n">block</span> <span class="mi">14</span> <span class="k">of</span> <span class="n">relation</span> <span class="nv">"base/5/16388"</span>
<span class="mi">2025</span><span class="o">-</span><span class="mi">09</span><span class="o">-</span><span class="mi">08</span> <span class="mi">17</span><span class="p">:</span><span class="mi">58</span><span class="p">:</span><span class="mi">41</span><span class="p">.</span><span class="mi">876</span> <span class="n">UTC</span> <span class="p">[</span><span class="mi">161</span><span class="p">]</span> <span class="k">STATEMENT</span><span class="p">:</span>  <span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span> <span class="k">where</span> <span class="n">k</span><span class="o">=</span><span class="mi">999</span><span class="p">;</span>
<span class="n">ERROR</span><span class="p">:</span>  <span class="n">invalid</span> <span class="n">page</span> <span class="k">in</span> <span class="n">block</span> <span class="mi">14</span> <span class="k">of</span> <span class="n">relation</span> <span class="nv">"base/5/16388"</span>
</code></pre>

</div>



<p>Here the checksum calculated is not correct and an error has been raised. PostgreSQL checksums can detect some block corruption, but it is still possible that a bug or a malicious user that has access to the filesystem can change the data without being detected.</p>

<h2>
  
  
  Oracle Database
</h2>

<p>To detect lost writes like the one I simulated above, Oracle Database compares the block checksum with the standby databases, as there is a low chance that the corruption happened in both environments. I've demonstrated this in the past with a similar demo: <a href="https://franckpachot.github.io/pages/2013-2018/18c-new-lost-write-protection" rel="noopener noreferrer">18c new Lost Write Protection</a></p>

<h2>
  
  
  WiredTiger (MongoDB storage engine)
</h2>

<p>MongoDB employs the WiredTiger storage engine, which is designed to prevent lost writes and detect disk failures that might return the wrong page. To achieve this, WiredTiger stores a checksum alongside each page address within the pointers between the BTree pages, in an <a href="https://source.wiredtiger.com/develop/arch-glossary.html" rel="noopener noreferrer">address cookie</a>:</p>

<blockquote>
<p><strong>Address cookie</strong>: an opaque set of bytes returned by the block manager to reference a block in a Btree file, it includes an offset, size, checksum, and object id.</p>
</blockquote>

<p>In my lab, I first start a MongoDB container and compile <code>wt</code>, a command-line utility that allows direct interaction with WiredTiger files. This tool enables me to examine the storage engine without relying on the MongoDB query layer, and I'll use it in this series of blog posts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">--rm</span> <span class="nt">-it</span> <span class="nt">--cap-add</span><span class="o">=</span>SYS_PTRACE mongo bash
<span class="c"># install required packages</span>
apt-get update <span class="o">&amp;&amp;</span> apt-get <span class="nb">install</span> <span class="nt">-y</span> git xxd strace curl jq python3 python3-dev python3-pip python3-venv build-essential cmake gcc g++ libstdc++-12-dev libtool autoconf automake swig liblz4-dev zlib1g-dev libmemkind-dev libsnappy-dev libsodium-dev libzstd-dev
<span class="c"># get latest WiredTiger</span>
curl <span class="nt">-L</span> <span class="si">$(</span>curl <span class="nt">-s</span> https://api.github.com/repos/wiredtiger/wiredtiger/releases/latest | jq <span class="nt">-r</span> <span class="s1">'.tarball_url'</span><span class="si">)</span> <span class="nt">-o</span> wiredtiger.tar.gz
<span class="c"># Compile</span>
<span class="nb">mkdir</span> /wiredtiger <span class="o">&amp;&amp;</span> <span class="nb">tar</span> <span class="nt">-xzf</span> wiredtiger.tar.gz <span class="nt">--strip-components</span><span class="o">=</span>1 <span class="nt">-C</span> /wiredtiger <span class="p">;</span> <span class="nb">cd</span> /wiredtiger
<span class="nb">mkdir </span>build <span class="o">&amp;&amp;</span> cmake <span class="nt">-S</span> /wiredtiger <span class="nt">-B</span> /wiredtiger/build <span class="se">\</span>
        <span class="nt">-DHAVE_BUILTIN_EXTENSION_SNAPPY</span><span class="o">=</span>1 <span class="se">\</span>
        <span class="nt">-DCMAKE_BUILD_TYPE</span><span class="o">=</span>Release <span class="se">\</span>
        <span class="nt">-DENABLE_WERROR</span><span class="o">=</span>0 <span class="se">\</span>
        <span class="nt">-DENABLE_QPL</span><span class="o">=</span>0 <span class="se">\</span>
        <span class="nt">-DCMAKE_C_FLAGS</span><span class="o">=</span><span class="s2">"-O0 -Wno-error -Wno-format-overflow -Wno-error=array-bounds -Wno-error=format-overflow -Wno-error=nonnull"</span> <span class="se">\</span>
        <span class="nt">-DPYTHON_EXECUTABLE</span><span class="o">=</span><span class="si">$(</span>which python3<span class="si">)</span>
cmake <span class="nt">--build</span> /wiredtiger/build
<span class="nb">ln</span> <span class="nt">-s</span> /wiredtiger/build/wt /usr/local/bin/wt
</code></pre>

</div>



<p>It takes some time, but the <a href="https://source.wiredtiger.com/develop/command_line.html" rel="noopener noreferrer">wt</a> command line utility will make the investigation easier. That's an advantage of MongoDB pluggable storage—you can examine it in layers.</p>

<p>I create a demo table and insert ten thousand records:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>root@7d6d105a1663:/tmp# wt create table:demo

root@7d6d105a1663:/tmp# <span class="nb">ls</span> <span class="nt">-alrt</span>

total 68
drwxr-xr-x. 1 root root  4096 Sep  8 19:21 ..
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root    21 Sep  8 19:35 WiredTiger.lock
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root    50 Sep  8 19:35 WiredTiger
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root   299 Sep  8 19:35 WiredTiger.basecfg
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root  4096 Sep  8 19:35 demo.wt
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root  4096 Sep  8 19:35 WiredTigerHS.wt
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root  1475 Sep  8 19:35 WiredTiger.turtle
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 root root 32768 Sep  8 19:35 WiredTiger.wt
drwxrwxrwt. 1 root root  4096 Sep  8 19:35 <span class="nb">.</span>

root@7d6d105a1663:/tmp# wt list

colgroup:demo
file:demo.wt
table:demo

<span class="nb">cat</span> /dev/urandom |
 <span class="nb">base64</span> |
 <span class="nb">head</span> <span class="nt">-10000</span> |
 <span class="nb">awk</span> <span class="s1">'{print "i",NR,$0}'</span> |
wt dump <span class="nt">-e</span> table:demo

...
Inserted key <span class="s1">'9997'</span> and value <span class="s1">'ILZeUq/u/ErLB/i7LOUb4nwYP6D535trb8Mt3vcJXXRAqLeAiYIHn5bEWs1buflmiZMYd3rMMvhh'</span><span class="nb">.</span>
Inserted key <span class="s1">'9998'</span> and value <span class="s1">'y+b0eTV/4Ao12qRqtHhgP2xGUr+C9ZOfvOG3ZwbdDNXvpnbM1/laoJ9Yzyt6cbLJOR6jdQktpgFM'</span><span class="nb">.</span>
Inserted key <span class="s1">'9999'</span> and value <span class="s1">'cJY9uWtopqFOuZjggkZDWVZEEdygpMLyL7LscqehnKoVY7BrmTh4ZzyTLrZ1glROwLtZYbvLbu5c'</span><span class="nb">.</span>
Inserted key <span class="s1">'10000'</span> and value <span class="s1">'QYwyjaRxa9Q+5dvzwQtvv2QE/uS/vhRPCVsQ6p7re/L2yDrVRxyqkvSyMHeRCzMIsIovrCUJpPXI'</span><span class="nb">.</span>
</code></pre>

</div>



<p>I read record 9999 with <code>wt</code> and use <code>strace</code> to see the read calls:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>strace <span class="nt">-yy</span> <span class="nt">-e</span> <span class="nv">trace</span><span class="o">=</span>pread64 <span class="nt">-xs</span> 36 wt dump <span class="nt">-e</span> table:demo <span class="o">&lt;&lt;&lt;</span><span class="s2">"s 9999"</span>

...
pread64<span class="o">(</span>6&lt;/tmp/demo.wt&gt;, <span class="s2">"</span><span class="se">\x</span><span class="s2">00...</span><span class="se">\x</span><span class="s2">1f</span><span class="se">\x</span><span class="s2">63</span><span class="se">\x</span><span class="s2">77</span><span class="se">\x</span><span class="s2">41"</span>..., 28672, 806912<span class="o">)</span> <span class="o">=</span> 28672
9999
cJY9uWtopqFOuZjggkZDWVZEEdygpMLyL7LscqehnKoVY7BrmTh4ZzyTLrZ1glROwLtZYbvLbu5c
...
</code></pre>

</div>



<p>This record is at offset 806912 in a 28672 bytes block. I 'save' this block with <code>dd</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">dd </span><span class="k">if</span><span class="o">=</span>demo.wt <span class="nv">of</span><span class="o">=</span>block1.tmp <span class="nv">bs</span><span class="o">=</span>1 <span class="nv">skip</span><span class="o">=</span>806912 <span class="nv">count</span><span class="o">=</span>28672

28672+0 records <span class="k">in
</span>28672+0 records out
28672 bytes <span class="o">(</span>29 kB, 28 KiB<span class="o">)</span> copied, 0.0680832 s, 421 kB/s
</code></pre>

</div>



<p>I update this record to "xxxxxx" and trace the write calls:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>strace <span class="nt">-yy</span> <span class="nt">-e</span> <span class="nv">trace</span><span class="o">=</span>pwrite64 <span class="nt">-xs</span> 36 wt dump <span class="nt">-e</span> table:demo <span class="o">&lt;&lt;&lt;</span><span class="s2">"u 9999 xxxxxx"</span>

...
Updated key <span class="s1">'9999'</span> to value <span class="s1">'xxxxxx'</span><span class="nb">.</span>
pwrite64<span class="o">(</span>6&lt;/tmp/demo.wt&gt;, <span class="s2">"</span><span class="se">\x</span><span class="s2">00...</span><span class="se">\x</span><span class="s2">02</span><span class="se">\x</span><span class="s2">b1</span><span class="se">\x</span><span class="s2">1f</span><span class="se">\x</span><span class="s2">f5"</span>..., 28672, 847872<span class="o">)</span> <span class="o">=</span> 28672
pwrite64<span class="o">(</span>6&lt;/tmp/demo.wt&gt;, <span class="s2">"</span><span class="se">\x</span><span class="s2">00...</span><span class="se">\x</span><span class="s2">4d</span><span class="se">\x</span><span class="s2">19</span><span class="se">\x</span><span class="s2">14</span><span class="se">\x</span><span class="s2">4e"</span>..., 4096, 876544<span class="o">)</span> <span class="o">=</span> 4096
pwrite64<span class="o">(</span>6&lt;/tmp/demo.wt&gt;, <span class="s2">"</span><span class="se">\x</span><span class="s2">00...</span><span class="se">\x</span><span class="s2">4f</span><span class="se">\x</span><span class="s2">74</span><span class="se">\x</span><span class="s2">db</span><span class="se">\x</span><span class="s2">1e"</span>..., 4096, 880640<span class="o">)</span> <span class="o">=</span> 4096
pwrite64<span class="o">(</span>6&lt;/tmp/demo.wt&gt;, <span class="s2">"</span><span class="se">\x</span><span class="s2">00...</span><span class="se">\x</span><span class="s2">21</span><span class="se">\x</span><span class="s2">cc</span><span class="se">\x</span><span class="s2">25</span><span class="se">\x</span><span class="s2">06"</span>..., 4096, 884736<span class="o">)</span> <span class="o">=</span> 4096
...
</code></pre>

</div>



<p>This writes a new block (WiredTiger do not write in-place, which helps to avoid corruption) of 28672 bytes, and updates the BTree branches.</p>

<p>I can read this new value:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>strace <span class="nt">-yy</span> <span class="nt">-e</span> <span class="nv">trace</span><span class="o">=</span>pread64  <span class="nt">-xs</span> 36 wt dump <span class="nt">-e</span> table:demo <span class="o">&lt;&lt;&lt;</span><span class="s2">"s 9999"</span>

...
pread64<span class="o">(</span>6&lt;/tmp/demo.wt&gt;, <span class="s2">"</span><span class="se">\x</span><span class="s2">00...</span><span class="se">\x</span><span class="s2">02</span><span class="se">\x</span><span class="s2">b1</span><span class="se">\x</span><span class="s2">1f</span><span class="se">\x</span><span class="s2">f5"</span>..., 28672, 847872<span class="o">)</span> <span class="o">=</span> 28672
9999
xxxxxx
...
</code></pre>

</div>



<p>To simulate disk corruption, I do the same as I did on PostgreSQL: replace the current block with the old one. I save the current block before overwriting it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">dd </span><span class="k">if</span><span class="o">=</span>demo.wt <span class="nv">of</span><span class="o">=</span>block2.tmp <span class="nv">bs</span><span class="o">=</span>1 <span class="nv">skip</span><span class="o">=</span>847872 <span class="nv">count</span><span class="o">=</span>28672

28672+0 records <span class="k">in
</span>28672+0 records out
28672 bytes <span class="o">(</span>29 kB, 28 KiB<span class="o">)</span> copied, 0.0688249 s, 417 kB/s

<span class="nb">dd </span><span class="nv">of</span><span class="o">=</span>demo.wt <span class="k">if</span><span class="o">=</span>block1.tmp <span class="nv">bs</span><span class="o">=</span>1 <span class="nv">seek</span><span class="o">=</span>847872 <span class="nv">conv</span><span class="o">=</span>notrunc

28672+0 records <span class="k">in
</span>28672+0 records out
28672 bytes <span class="o">(</span>29 kB, 28 KiB<span class="o">)</span> copied, 0.0666375 s, 430 kB/s
</code></pre>

</div>



<p>If I try to read the record in this block, the corruption is detected:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>strace -yy -e trace=pread64  -xs 36 wt dump -e table:demo &lt;&lt;&lt;"s 9999"

...
pread64(6&lt;/tmp/demo.wt&gt;, "\x00...\x1f\x63\x77\x41"..., 28672, 847872) = 28672
[1757361305:392519][8246:0x7fe9a087e740], wt, file:demo.wt, WT_SESSION.open_cursor: [WT_VERB_DEFAULT][ERROR]: __wti_block_read_off, 279: demo.wt: potential hardware corruption,
read checksum error for 28672B block at offset 847872: block header
checksum of 0x4177631f doesn't match expected checksum of 0xf51fb102
[1757361305:392904][8246:0x7fe9a087e740], wt, file:demo.wt,
WT_SESSION.open_cursor: [WT_VERB_DEFAULT][ERROR]: __bm_corrupt_dump, 86: {0: 847872, 28672, 0xf51fb102}: (chunk 1 of 28): 00 00 00 00 00
00 00 00 3a 00 00 00 00 00 00 00 40 6e 00 00 a8 02 00 00 07 04 00 01 00 70 00 00 1f 63 77 41 01 00 00 00 11 39 36 39 33 80 8c 43 38 30 58 51 66 64 
...
</code></pre>

</div>



<p>Even if the block checksum is correct for the block itself, it was detected that the checksum of the block, <code>0x4177631f</code>, which is visible as <code>1f 63 77 41</code> in the hexadecimal dump, or <code>\x1f\x63\x77\x41</code> in the read trace, is different from the expected <code>0xf51fb102</code> from the address cookie.</p>

<p><code>0xf51fb102</code> was visible as <code>\x02\xb1\x1f\xf5</code> in the write call of the update, and is visible as <code>02 b1 1f f5</code> in the block that I've saved before overwriting it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>root@7d6d105a1663:/tmp# xxd <span class="nt">-l</span> 36 block2.tmp
00000000: 0000 0000 0000 0000 5c00 0000 0000 0000  ........<span class="se">\.</span>......
00000010: f96d 0000 a802 0000 0704 0001 0070 0000  .m...........p..
00000020: 02b1 1ff5
</code></pre>

</div>



<p>Even with access to the files, it would be extremely difficult to corrupt the data in an undetected way because any change must update the checksum, and the checksum is referenced in all address cookies in other blocks. Block corruption is highly unlikely as the blocks are not updated in place, and failure to write blocks would break the pointers.</p>

<h2>
  
  
  Conclusion
</h2>

<p>PostgreSQL requires you to enable checksums to detect data corruption. This detects when a page’s checksum does not match its content. However, if the system erroneously writes a different, but valid, block from the same table in place of the intended one, or misses a write and the previous version of the block remains, PostgreSQL cannot identify this issue. As a result, some disk failures may escape detection and return wrong results.</p>

<p>Oracle Database stores blocks with checksums and can enable checking them on read. With a Data Guard standby and some network overhead, the database can transmit checksums over the network to verify data integrity when reading.</p>

<p>MongoDB <a href="https://www.mongodb.com/docs/manual/core/wiredtiger/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=wiredtiger-checksum" rel="noopener noreferrer">WiredTiger</a> enables checksums by default and can detect the wrong blocks without the need to contact replicas. It embeds the expected checksum inside the BTree address cookie so that every internal BTree pointer to a leaf page includes the checksum for the referenced page. If an obsolete or different page is swapped in, any mismatch will be detected because the pointer's checksum won’t match. WiredTiger uses copy-on-write, not in-place overwrites, further reducing the risk of corruption.</p>

<p>Here is a description of WiredTiger by Keith Bostic:<br>
<iframe width="710" height="399" src="https://www.youtube.com/embed/GkgDDs9EJUw?start=2474">
</iframe>
</p>

