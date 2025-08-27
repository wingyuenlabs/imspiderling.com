---
Title: PostgreSQL JSONB Size Limits to Prevent TOAST Slicing
Description: 
Author: Franck Pachot
Date: 2025-08-27T22:01:54.000Z
Robots: noindex,nofollow
Template: index
---
<p>In my previous post <a href="https://dev.to/mongodb/embedding-into-jsonb-still-feels-like-a-join-for-large-documents-3nd0">Embedding Into JSONB Still Feels Like a JOIN for Large Documents</a>, I examined the behavior of large JSONB documents in PostgreSQL when they exceed 2 KB. This limitation can compromise the data locality objectives inherent in the document model. However, it's worth noting that this limit can be increased up to 32 KB.</p>

<h2>
  
  
  Details about TOAST thresholds
</h2>

<p>PostgreSQL stores table rows in fixed-size pages (BLCKSZ), defaulting to 8 KB but configurable up to 32 KB at compile time. If a row value exceeds the TOAST_TUPLE_THRESHOLD, about a quarter of the page size (≈ 2 KB for 8 KB pages), the TOAST mechanism activates. PostgreSQL first attempts to reduce the row size to the TOAST_TUPLE_TARGET (defaulting to the threshold, but tunable per table with <code>ALTER TABLE … SET (toast_tuple_target = …)</code>) by compressing or relocating large TOASTable columns.</p>

<p>If the value remains too large, it is written to a separate TOAST table in chunks (up to TOAST_MAX_CHUNK_SIZE, also about a quarter of the page size), with each chunk stored as a separate row and retrieved via an index on (chunk_id, chunk_seq).</p>

<p>TOAST_TUPLE_THRESHOLD and TOAST_TUPLE_TARGET can be adjusted at compile time, while TOAST_MAX_CHUNK_SIZE and page size changes require initdb for a new database.</p>

<p>Storage strategies (PLAIN, MAIN, EXTERNAL, EXTENDED) and compression methods (pglz, lz4, etc.) are configurable in SQL. For large JSONB fields, exceeding the threshold incurs the cost of compression and indexed fetching, akin to an implicit join.</p>

<p>To fit the largest document possible in PostgreSQL, we must use the largest block size available, which is <a href="https://github.com/postgres/postgres/blob/REL_18_BETA3/src/include/pg_config.h.in#L31" rel="noopener noreferrer">32KB</a>. This requires compiling PostgreSQL and creating a new database. While this is not feasible in a managed service environment, we can perform these actions in a lab setting.</p>

<h2>
  
  
  32KB BLKSZ PostgreSQL lab
</h2>

<p>I used the official <a href="https://github.com/docker-library/postgres.git" rel="noopener noreferrer">Dockerfile</a> and added <code>--with-blocksize=32</code> to the configure command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code><span class="gh">diff --git a/18/alpine3.22/Dockerfile b/18/alpine3.22/Dockerfile
index 0a8c650..8ed3d26 100644
</span><span class="gd">--- a/18/alpine3.22/Dockerfile
</span><span class="gi">+++ b/18/alpine3.22/Dockerfile
</span><span class="p">@@ -148,6 +148,7 @@</span> RUN set -eux; \
                --with-llvm \
                --with-lz4 \
                --with-zstd \
<span class="gi">+               --with-blocksize=32 \
</span>        ; \
        make -j "$(nproc)" world-bin; \
        make install-world-bin; \
</code></pre>

</div>



<p>I built the image and started a container:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>docker build <span class="nt">-t</span> pg18-blksz32 <span class="nb">.</span>

<span class="nb">sudo </span>docker <span class="nb">rm</span> <span class="nt">-f</span> pg18-blksz32
<span class="nb">sudo </span>docker <span class="nb">exec</span> <span class="nt">-it</span> <span class="nt">-e</span> <span class="nv">PGUSER</span><span class="o">=</span>postgres <span class="si">$(</span>
 <span class="nb">sudo </span>docker run <span class="nt">-d</span> <span class="nt">--name</span> pg18-blksz32 <span class="nt">-e</span> <span class="nv">POSTGRES_PASSWORD</span><span class="o">=</span>x pg18-blksz32
<span class="nb">sleep </span>5
<span class="si">)</span> psql
</code></pre>

</div>



<h2>
  
  
  increase toast_tuple_target
</h2>

<p>I created the same table as in the <a href="https://dev.to/mongodb/embedding-into-jsonb-still-feels-like-a-join-for-large-documents-3nd0">previous post</a> with only one difference: I set the TOAST target to the maximum, the block size minus the tuple header (24 bytes), and TOAST header (8 bytes), at the column level:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">orders</span> <span class="k">SET</span> <span class="p">(</span><span class="n">toast_tuple_target</span> <span class="o">=</span> <span class="mi">32736</span><span class="p">);</span>
</code></pre>

</div>



<p>Another solution could have been disabling external storage at the table level, unless it doesn't fit in the block.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">orders</span> <span class="k">ALTER</span> <span class="k">COLUMN</span> <span class="n">items</span> <span class="k">SET</span> <span class="k">STORAGE</span> <span class="n">MAIN</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  Explain (analyze, buffers, serialize)
</h2>

<p>After setting the document with the UPDATE statement of the previous post, I checked the number of pages read to get one document:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">postgres</span><span class="o">=#</span> <span class="k">explain</span> <span class="p">(</span><span class="k">analyze</span><span class="p">,</span> <span class="n">buffers</span><span class="p">,</span> <span class="n">serialize</span> <span class="nb">text</span><span class="p">,</span> <span class="k">verbose</span><span class="p">,</span> <span class="n">costs</span> <span class="k">off</span><span class="p">)</span>
<span class="k">select</span> <span class="o">*</span>
 <span class="k">from</span> <span class="n">orders</span>
 <span class="k">where</span> <span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span>
<span class="p">;</span>
                                     <span class="n">QUERY</span> <span class="n">PLAN</span>
<span class="c1">-------------------------------------------------------------------------------------</span>
 <span class="n">Bitmap</span> <span class="n">Heap</span> <span class="n">Scan</span> <span class="k">on</span> <span class="k">public</span><span class="p">.</span><span class="n">orders</span> <span class="p">(</span><span class="n">actual</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">047</span><span class="p">..</span><span class="mi">0</span><span class="p">.</span><span class="mi">048</span> <span class="k">rows</span><span class="o">=</span><span class="mi">1</span><span class="p">.</span><span class="mi">00</span> <span class="n">loops</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
   <span class="k">Output</span><span class="p">:</span> <span class="n">ord_id</span><span class="p">,</span> <span class="n">ord_dat</span><span class="p">,</span> <span class="n">items</span>
   <span class="k">Recheck</span> <span class="n">Cond</span><span class="p">:</span> <span class="p">(</span><span class="n">orders</span><span class="p">.</span><span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span><span class="p">)</span>
   <span class="n">Heap</span> <span class="n">Blocks</span><span class="p">:</span> <span class="n">exact</span><span class="o">=</span><span class="mi">2</span>
   <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">4</span>
   <span class="o">-&gt;</span>  <span class="n">Bitmap</span> <span class="k">Index</span> <span class="n">Scan</span> <span class="k">on</span> <span class="n">orders_pkey</span> <span class="p">(</span><span class="n">actual</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">034</span><span class="p">..</span><span class="mi">0</span><span class="p">.</span><span class="mi">035</span> <span class="k">rows</span><span class="o">=</span><span class="mi">2</span><span class="p">.</span><span class="mi">00</span> <span class="n">loops</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
         <span class="k">Index</span> <span class="n">Cond</span><span class="p">:</span> <span class="p">(</span><span class="n">orders</span><span class="p">.</span><span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span><span class="p">)</span>
         <span class="k">Index</span> <span class="n">Searches</span><span class="p">:</span> <span class="mi">1</span>
         <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">2</span>
 <span class="n">Planning</span> <span class="nb">Time</span><span class="p">:</span> <span class="mi">0</span><span class="p">.</span><span class="mi">067</span> <span class="n">ms</span>
 <span class="n">Serialization</span><span class="p">:</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">036</span> <span class="n">ms</span>  <span class="k">output</span><span class="o">=</span><span class="mi">20</span><span class="n">kB</span>  <span class="n">format</span><span class="o">=</span><span class="nb">text</span>
 <span class="n">Execution</span> <span class="nb">Time</span><span class="p">:</span> <span class="mi">0</span><span class="p">.</span><span class="mi">169</span> <span class="n">ms</span>
</code></pre>

</div>



<p>As before, scanning the index requires 3 pages and 1 page for the heap table. Because the document fits entirely within this 32 KB page, no additional buffers are needed during the <code>Serialization</code> phase, unlike the six buffer hits required in the database with an 8 KB block size.</p>

<h2>
  
  
  Larger documents
</h2>

<p>If the document size increases above the block size, it will be TOASTed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code>
<span class="n">postgres</span><span class="o">=#</span> <span class="k">update</span> <span class="n">orders</span> 
 <span class="k">set</span> <span class="n">items</span> <span class="o">=</span> <span class="n">items</span> <span class="o">||</span> <span class="n">items</span> 
 <span class="k">where</span> <span class="n">ord_id</span><span class="o">=</span><span class="mi">42</span>
<span class="p">;</span>
<span class="k">UPDATE</span> <span class="mi">1</span>

<span class="n">postgres</span><span class="o">=#</span> <span class="k">SELECT</span> <span class="n">o</span><span class="p">.</span><span class="n">ord_id</span><span class="p">,</span> <span class="n">o</span><span class="p">.</span><span class="n">ord_dat</span><span class="p">,</span> <span class="n">t</span><span class="p">.</span><span class="n">chunk_id</span><span class="p">,</span> <span class="n">t</span><span class="p">.</span><span class="n">chunk_seq</span><span class="p">,</span> <span class="n">t</span><span class="p">.</span><span class="n">ctid</span><span class="p">,</span>
       <span class="n">pg_size_pretty</span><span class="p">(</span><span class="k">length</span><span class="p">(</span><span class="n">t</span><span class="p">.</span><span class="n">chunk_data</span><span class="p">)::</span><span class="nb">bigint</span><span class="p">)</span> <span class="k">AS</span> <span class="n">chunk_size</span>
<span class="k">FROM</span> <span class="n">orders</span> <span class="n">o</span>
<span class="k">JOIN</span> <span class="n">pg_toast</span><span class="p">.</span><span class="n">pg_toast_16384</span> <span class="n">t</span>
  <span class="k">ON</span> <span class="n">t</span><span class="p">.</span><span class="n">chunk_id</span> <span class="o">=</span> <span class="n">pg_column_toast_chunk_id</span><span class="p">(</span><span class="n">o</span><span class="p">.</span><span class="n">items</span><span class="p">)</span>
<span class="k">WHERE</span> <span class="n">o</span><span class="p">.</span><span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">t</span><span class="p">.</span><span class="n">chunk_seq</span>
<span class="p">;</span>
 <span class="n">ord_id</span> <span class="o">|</span>            <span class="n">ord_dat</span>            <span class="o">|</span> <span class="n">chunk_id</span> <span class="o">|</span> <span class="n">chunk_seq</span> <span class="o">|</span> <span class="n">ctid</span>  <span class="o">|</span> <span class="n">chunk_size</span>
<span class="c1">--------+-------------------------------+----------+-----------+-------+------------</span>
     <span class="mi">42</span> <span class="o">|</span> <span class="mi">2025</span><span class="o">-</span><span class="mi">08</span><span class="o">-</span><span class="mi">27</span> <span class="mi">17</span><span class="p">:</span><span class="mi">21</span><span class="p">:</span><span class="mi">30</span><span class="p">.</span><span class="mi">677401</span><span class="o">+</span><span class="mi">00</span> <span class="o">|</span>   <span class="mi">116406</span> <span class="o">|</span>         <span class="mi">0</span> <span class="o">|</span> <span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="mi">1</span><span class="p">)</span> <span class="o">|</span> <span class="mi">8140</span> <span class="n">bytes</span>
     <span class="mi">42</span> <span class="o">|</span> <span class="mi">2025</span><span class="o">-</span><span class="mi">08</span><span class="o">-</span><span class="mi">27</span> <span class="mi">17</span><span class="p">:</span><span class="mi">21</span><span class="p">:</span><span class="mi">30</span><span class="p">.</span><span class="mi">677401</span><span class="o">+</span><span class="mi">00</span> <span class="o">|</span>   <span class="mi">116406</span> <span class="o">|</span>         <span class="mi">1</span> <span class="o">|</span> <span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="mi">2</span><span class="p">)</span> <span class="o">|</span> <span class="mi">8140</span> <span class="n">bytes</span>
     <span class="mi">42</span> <span class="o">|</span> <span class="mi">2025</span><span class="o">-</span><span class="mi">08</span><span class="o">-</span><span class="mi">27</span> <span class="mi">17</span><span class="p">:</span><span class="mi">21</span><span class="p">:</span><span class="mi">30</span><span class="p">.</span><span class="mi">677401</span><span class="o">+</span><span class="mi">00</span> <span class="o">|</span>   <span class="mi">116406</span> <span class="o">|</span>         <span class="mi">2</span> <span class="o">|</span> <span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="mi">3</span><span class="p">)</span> <span class="o">|</span> <span class="mi">8140</span> <span class="n">bytes</span>
     <span class="mi">42</span> <span class="o">|</span> <span class="mi">2025</span><span class="o">-</span><span class="mi">08</span><span class="o">-</span><span class="mi">27</span> <span class="mi">17</span><span class="p">:</span><span class="mi">21</span><span class="p">:</span><span class="mi">30</span><span class="p">.</span><span class="mi">677401</span><span class="o">+</span><span class="mi">00</span> <span class="o">|</span>   <span class="mi">116406</span> <span class="o">|</span>         <span class="mi">3</span> <span class="o">|</span> <span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="mi">4</span><span class="p">)</span> <span class="o">|</span> <span class="mi">8140</span> <span class="n">bytes</span>
     <span class="mi">42</span> <span class="o">|</span> <span class="mi">2025</span><span class="o">-</span><span class="mi">08</span><span class="o">-</span><span class="mi">27</span> <span class="mi">17</span><span class="p">:</span><span class="mi">21</span><span class="p">:</span><span class="mi">30</span><span class="p">.</span><span class="mi">677401</span><span class="o">+</span><span class="mi">00</span> <span class="o">|</span>   <span class="mi">116406</span> <span class="o">|</span>         <span class="mi">4</span> <span class="o">|</span> <span class="p">(</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">)</span> <span class="o">|</span> <span class="mi">7524</span> <span class="n">bytes</span>
<span class="p">(</span><span class="mi">5</span> <span class="k">rows</span><span class="p">)</span>
</code></pre>

</div>



<p>When stored inline, the document was 20K (<code>output=20kB</code>) and was stored as a single datum. Now that it has doubled to 40 KB, it is stored into five chunks because the maximum TOAST chunk size is 1/4th of the block size. That doesn't matter because they are stored on the same page, or contiguous pages, as indicated by the CTID. We have doubled the size, so it makes sense to read two heap blocks instead of one. However, those are in addition to the tuple, and the main problem is the overhead of the TOAST index traversal to find those pages. There are 6 pages read in total:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">postgres</span><span class="o">=#</span> <span class="k">explain</span> <span class="p">(</span><span class="k">analyze</span><span class="p">,</span> <span class="n">buffers</span><span class="p">,</span> <span class="n">serialize</span> <span class="nb">text</span><span class="p">,</span> <span class="k">verbose</span><span class="p">,</span> <span class="n">costs</span> <span class="k">off</span><span class="p">)</span>
<span class="k">select</span> <span class="o">*</span>
 <span class="k">from</span> <span class="n">orders</span>
 <span class="k">where</span> <span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span>
<span class="p">;</span>
                                     <span class="n">QUERY</span> <span class="n">PLAN</span>
<span class="c1">-------------------------------------------------------------------------------------</span>
 <span class="n">Bitmap</span> <span class="n">Heap</span> <span class="n">Scan</span> <span class="k">on</span> <span class="k">public</span><span class="p">.</span><span class="n">orders</span> <span class="p">(</span><span class="n">actual</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">028</span><span class="p">..</span><span class="mi">0</span><span class="p">.</span><span class="mi">028</span> <span class="k">rows</span><span class="o">=</span><span class="mi">1</span><span class="p">.</span><span class="mi">00</span> <span class="n">loops</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
   <span class="k">Output</span><span class="p">:</span> <span class="n">ord_id</span><span class="p">,</span> <span class="n">ord_dat</span><span class="p">,</span> <span class="n">items</span>
   <span class="k">Recheck</span> <span class="n">Cond</span><span class="p">:</span> <span class="p">(</span><span class="n">orders</span><span class="p">.</span><span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span><span class="p">)</span>
   <span class="n">Heap</span> <span class="n">Blocks</span><span class="p">:</span> <span class="n">exact</span><span class="o">=</span><span class="mi">1</span>
   <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">3</span>
   <span class="o">-&gt;</span>  <span class="n">Bitmap</span> <span class="k">Index</span> <span class="n">Scan</span> <span class="k">on</span> <span class="n">orders_pkey</span> <span class="p">(</span><span class="n">actual</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">016</span><span class="p">..</span><span class="mi">0</span><span class="p">.</span><span class="mi">016</span> <span class="k">rows</span><span class="o">=</span><span class="mi">1</span><span class="p">.</span><span class="mi">00</span> <span class="n">loops</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
         <span class="k">Index</span> <span class="n">Cond</span><span class="p">:</span> <span class="p">(</span><span class="n">orders</span><span class="p">.</span><span class="n">ord_id</span> <span class="o">=</span> <span class="mi">42</span><span class="p">)</span>
         <span class="k">Index</span> <span class="n">Searches</span><span class="p">:</span> <span class="mi">1</span>
         <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">2</span>
 <span class="n">Planning</span> <span class="nb">Time</span><span class="p">:</span> <span class="mi">0</span><span class="p">.</span><span class="mi">109</span> <span class="n">ms</span>
 <span class="n">Serialization</span><span class="p">:</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">143</span> <span class="n">ms</span>  <span class="k">output</span><span class="o">=</span><span class="mi">40</span><span class="n">kB</span>  <span class="n">format</span><span class="o">=</span><span class="nb">text</span>
   <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">3</span>
 <span class="n">Execution</span> <span class="nb">Time</span><span class="p">:</span> <span class="mi">0</span><span class="p">.</span><span class="mi">238</span> <span class="n">ms</span>
</code></pre>

</div>



<p>Compared to the previous example on a 8 KB block size database, there are less pages read in total because the indexes benefit from the large block size and necessitate less branch levels:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">postgres</span><span class="o">=#</span> <span class="k">create</span> <span class="n">extension</span> <span class="n">if</span> <span class="k">not</span> <span class="k">exists</span> <span class="n">pageinspect</span><span class="p">;</span>
<span class="k">CREATE</span> <span class="n">EXTENSION</span>

<span class="n">postgres</span><span class="o">=#</span> <span class="k">select</span> <span class="n">btpo_level</span> <span class="k">from</span> <span class="n">bt_page_stats</span><span class="p">(</span><span class="s1">'public.orders_pkey'</span><span class="p">,</span> <span class="p">(</span>
 <span class="k">select</span> <span class="n">root</span> <span class="k">from</span> <span class="n">bt_metap</span><span class="p">(</span><span class="s1">'public.orders_pkey'</span><span class="p">))</span>
<span class="p">);</span>

 <span class="n">btpo_level</span>
<span class="c1">------------</span>
          <span class="mi">1</span>
<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>

<span class="n">postgres</span><span class="o">=#</span> <span class="k">select</span> <span class="n">btpo_level</span> <span class="k">from</span> <span class="n">bt_page_stats</span><span class="p">(</span><span class="s1">'pg_toast.pg_toast_16384_index'</span><span class="p">,</span> <span class="p">(</span>
 <span class="k">select</span> <span class="n">root</span> <span class="k">from</span> <span class="n">bt_metap</span><span class="p">(</span><span class="s1">'pg_toast.pg_toast_16384_index'</span><span class="p">))</span>
<span class="p">);</span>

 <span class="n">btpo_level</span>
<span class="c1">------------</span>
          <span class="mi">1</span>
<span class="p">(</span><span class="mi">1</span> <span class="k">row</span><span class="p">)</span>
</code></pre>

</div>



<p>The main advantage of a document database is to offer a data model that natively matches application objects, without the complexity of object–relational mapping (ORM), and to preserve that logical model down to the physical layout. This reduces random I/O, improves cache efficiency, enables transparent sharding, and increases transactional scalability.</p>

<h2>
  
  
  Conclusion
</h2>

<p>In MongoDB, documents are stored in a single block, with size ranging from 32 KB to 16 MB (the maximum BSON size), which ensures strong data locality. Each document typically corresponds to a domain-driven design (DDD) aggregate and can embed large fields, including text, arrays, vectors, extended references, and one-to-many relationships. Common document sizes ranging from 32 KB to 256 KB align with WiredTiger’s minimum I/O size, cloud storage maximum I/O size, and modern CPU cache sizes.</p>

<p>JSONB in PostgreSQL allows some locality for small documents. However, when a document exceeds about 2 KB, it moves to TOAST storage, breaking locality and adding overhead due to an extra index traversal. Increasing the page size to 32 KB allows larger documents to remain in-line, but this still falls short of the sizes typical in document databases. Moreover, achieving these sizes often requires non‑default builds and self-managed deployments, which are usually unavailable in managed services.</p>

<p>PostgreSQL’s TOAST mechanism allows for the storage of very large JSONB values, but it comes with a performance trade-off, impacting locality and access speed in favor of accommodating oversized data in a fixed block size storage engine. In contrast, MongoDB is designed to maintain whole-document locality up to a significantly larger size limit, and preserves the logical model of an application in its physical storage. While both databases can store JSON, they fulfill different purposes: MongoDB is optimized for an application-centric document model preserved down to the storage layer, whereas PostgreSQL is tailored for a database-centric normalized model, enhancing it with JSONB either for small additional attributes or for storing large documents that are accessed infrequently.</p>

