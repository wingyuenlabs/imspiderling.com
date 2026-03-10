---
Title: When JS Libraries Fail at 1M Rows: Generating XLSX via DuckDB SQL
Description: 
Author: Yuki
Date: 2026-03-10T22:03:41.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>I've been building <strong><a href="https://leaprows.com/en" rel="noopener noreferrer">LeapRows</a></strong> — a browser-based CSV analysis tool that runs entirely client-side using DuckDB-WASM, with no server involved.</p>

<p>At some point I needed to add XLSX export. But every existing approach I tried fell apart at any meaningful scale.</p>

<p>I eventually landed on a solution: <strong>generate Excel-compatible XML directly via DuckDB SQL, then compress it into an XLSX file using JSZip.</strong> This post covers why I went that route and how it works.</p>

<p><em>Note: My day job is SEO, not software engineering — so please forgive any imprecise terminology. 🙏</em></p>

<h2>
  
  
  The Problem: Exporting 1 Million Rows to XLSX in the Browser
</h2>

<h3>
  
  
  DuckDB-WASM's Built-in XLSX Export Was Unreliable
</h3>

<p>My first attempt was DuckDB-WASM's native XLSX output (the <code>COPY TO</code> command via the Excel extension). It was unstable — Excel would throw <em>"The file format or file extension is not valid"</em> on some outputs and refuse to open the file.</p>

<p>I ruled it out for production use and started looking for alternatives.</p>

<h3>
  
  
  JS Libraries Ran Out of Memory
</h3>

<p>Next I tried SheetJS and ExcelJS. Both worked fine on small datasets, but memory usage exploded as file size grew, and the conversion itself became painfully slow.</p>

<p>Testing with 1 million rows, progress would reach 95% ... and then the browser would freeze for nearly 20 seconds. After the conversion finally finished, the browser stayed sluggish — and sometimes just crashed entirely.</p>

<h3>
  
  
  I Still Wanted 1 Million Rows to Work
</h3>

<p>LeapRows is built around the promise of handling 1M+ rows quickly. I didn't want to quietly cap the export at some lower limit. If Excel itself supports up to ~1,048,576 rows, I wanted LeapRows to be able to fill every one of them.</p>

<h2>
  
  
  The Insight: XLSX Is Just ZIP + XML
</h2>

<p>Something that might not be obvious: an XLSX file is actually a ZIP archive containing XML files. You can verify this yourself by renaming any <code>.xlsx</code> file to <code>.zip</code> — you'll see the contents directly.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flbsayknhxlb9nhu52tel.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flbsayknhxlb9nhu52tel.png" alt="Windows File Explorer showing the contents of a renamed XLSX file as a ZIP archive, &lt;br&gt;
revealing folders: _rels, docProps, xl, and a [Content_Types].xml file." width="636" height="154"></a></p>

<p>Those XML files contain a lot of styling information (fonts, colors, borders, conditional formatting, etc.), but the actual data lives in just one file: <code>xl/worksheets/sheet1.xml</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsoarf15722vo0fuklmok.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsoarf15722vo0fuklmok.png" alt="A code editor displaying the XML content of xl/worksheets/sheet1.xml inside an XLSX file, &lt;br&gt;
showing font styling definitions including Japanese font names like 游ゴシック." width="800" height="508"></a></p>

<p>Since LeapRows only needs to export raw data from a DuckDB table — no styling required — I realized: <em>the only file I need to generate dynamically is <code>sheet1.xml</code>. The other 4–5 files in the archive can be static strings.</em></p>
<h2>
  
  
  Generating XML Directly with DuckDB SQL
</h2>

<p>The data is already in a DuckDB table. And DuckDB SQL is extremely fast at string operations — <code>CONCAT</code>, <code>REPLACE</code>, and so on.</p>

<p>That led to the idea: <em>"What if I use a SQL query to output XML cell strings directly, then ZIP them up with JSZip?"</em></p>

<p><strong>Before:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>DuckDB → JS objects → library → XML → ZIP
</code></pre>

</div>



<p><strong>After:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>DuckDB → XML strings (via SQL) → JSZip → ZIP
</code></pre>

</div>



<p>Skipping the intermediate JS object conversion means no memory pressure from large object graphs, and no GC pauses that were causing the browser to freeze.</p>

<p>The approach uses a <code>CASE</code> expression to handle each column type appropriately — numbers pass through as-is, dates get converted to Excel serial values, and strings get XML-escaped.</p>

<h3>
  
  
  What the SQL Looks Like
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span>
    <span class="s1">'&lt;row r="'</span> <span class="o">||</span> <span class="k">CAST</span><span class="p">(</span><span class="n">rn</span> <span class="o">+</span> <span class="mi">1</span> <span class="k">AS</span> <span class="nb">VARCHAR</span><span class="p">)</span> <span class="o">||</span> <span class="s1">'"&gt;'</span>
    <span class="c1">-- String column: XML-escaped, output as inline string</span>
    <span class="o">||</span> <span class="k">CASE</span> <span class="k">WHEN</span> <span class="nv">"name"</span> <span class="k">IS</span> <span class="k">NOT</span> <span class="k">NULL</span>
       <span class="k">THEN</span> <span class="s1">'&lt;c r="A'</span> <span class="o">||</span> <span class="k">CAST</span><span class="p">(</span><span class="n">rn</span> <span class="o">+</span> <span class="mi">1</span> <span class="k">AS</span> <span class="nb">VARCHAR</span><span class="p">)</span> <span class="o">||</span> <span class="s1">'" t="inlineStr"&gt;&lt;is&gt;&lt;t&gt;'</span>
         <span class="o">||</span> <span class="k">REPLACE</span><span class="p">(</span><span class="k">REPLACE</span><span class="p">(</span><span class="k">REPLACE</span><span class="p">(</span><span class="nv">"name"</span><span class="p">,</span> <span class="s1">'&amp;'</span><span class="p">,</span> <span class="s1">'&amp;amp;'</span><span class="p">),</span> <span class="s1">'&lt;'</span><span class="p">,</span> <span class="s1">'&amp;lt;'</span><span class="p">),</span> <span class="s1">'&gt;'</span><span class="p">,</span> <span class="s1">'&amp;gt;'</span><span class="p">)</span>
         <span class="o">||</span> <span class="s1">'&lt;/t&gt;&lt;/is&gt;&lt;/c&gt;'</span>
       <span class="k">ELSE</span> <span class="s1">''</span> <span class="k">END</span>
    <span class="c1">-- Numeric column: output value directly</span>
    <span class="o">||</span> <span class="k">CASE</span> <span class="k">WHEN</span> <span class="nv">"age"</span> <span class="k">IS</span> <span class="k">NOT</span> <span class="k">NULL</span>
       <span class="k">THEN</span> <span class="s1">'&lt;c r="B'</span> <span class="o">||</span> <span class="k">CAST</span><span class="p">(</span><span class="n">rn</span> <span class="o">+</span> <span class="mi">1</span> <span class="k">AS</span> <span class="nb">VARCHAR</span><span class="p">)</span> <span class="o">||</span> <span class="s1">'"&gt;&lt;v&gt;'</span>
         <span class="o">||</span> <span class="k">CAST</span><span class="p">(</span><span class="nv">"age"</span> <span class="k">AS</span> <span class="nb">VARCHAR</span><span class="p">)</span>
         <span class="o">||</span> <span class="s1">'&lt;/v&gt;&lt;/c&gt;'</span>
       <span class="k">ELSE</span> <span class="s1">''</span> <span class="k">END</span>
    <span class="c1">-- Date column: convert to Excel serial value (days since 1899-12-30)</span>
    <span class="o">||</span> <span class="k">CASE</span> <span class="k">WHEN</span> <span class="nv">"created_at"</span> <span class="k">IS</span> <span class="k">NOT</span> <span class="k">NULL</span>
       <span class="k">THEN</span> <span class="s1">'&lt;c r="C'</span> <span class="o">||</span> <span class="k">CAST</span><span class="p">(</span><span class="n">rn</span> <span class="o">+</span> <span class="mi">1</span> <span class="k">AS</span> <span class="nb">VARCHAR</span><span class="p">)</span> <span class="o">||</span> <span class="s1">'" s="1"&gt;&lt;v&gt;'</span>
         <span class="o">||</span> <span class="k">CAST</span><span class="p">(</span><span class="n">date_diff</span><span class="p">(</span><span class="s1">'day'</span><span class="p">,</span> <span class="nb">DATE</span> <span class="s1">'1899-12-30'</span><span class="p">,</span> <span class="nv">"created_at"</span><span class="p">)</span> <span class="k">AS</span> <span class="nb">VARCHAR</span><span class="p">)</span>
         <span class="o">||</span> <span class="s1">'&lt;/v&gt;&lt;/c&gt;'</span>
       <span class="k">ELSE</span> <span class="s1">''</span> <span class="k">END</span>
    <span class="o">||</span> <span class="s1">'&lt;/row&gt;'</span> <span class="k">AS</span> <span class="n">xml_row</span>
<span class="k">FROM</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="o">*</span><span class="p">,</span> <span class="n">ROW_NUMBER</span><span class="p">()</span> <span class="n">OVER</span> <span class="p">()</span> <span class="k">AS</span> <span class="n">rn</span>
    <span class="k">FROM</span> <span class="n">my_table</span>
    <span class="k">LIMIT</span> <span class="mi">50000</span> <span class="k">OFFSET</span> <span class="mi">0</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Each row in the result set becomes one XML element:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;row</span> <span class="na">r=</span><span class="s">"2"</span><span class="nt">&gt;&lt;c</span> <span class="na">r=</span><span class="s">"A2"</span> <span class="na">t=</span><span class="s">"inlineStr"</span><span class="nt">&gt;&lt;is&gt;&lt;t&gt;</span>John Doe<span class="nt">&lt;/t&gt;&lt;/is&gt;&lt;/c&gt;&lt;c</span> <span class="na">r=</span><span class="s">"B2"</span><span class="nt">&gt;&lt;v&gt;</span>30<span class="nt">&lt;/v&gt;&lt;/c&gt;&lt;c</span> <span class="na">r=</span><span class="s">"C2"</span> <span class="na">s=</span><span class="s">"1"</span><span class="nt">&gt;&lt;v&gt;</span>45302<span class="nt">&lt;/v&gt;&lt;/c&gt;&lt;/row&gt;</span>
<span class="nt">&lt;row</span> <span class="na">r=</span><span class="s">"3"</span><span class="nt">&gt;&lt;c</span> <span class="na">r=</span><span class="s">"A3"</span> <span class="na">t=</span><span class="s">"inlineStr"</span><span class="nt">&gt;&lt;is&gt;&lt;t&gt;</span>Jane Smith<span class="nt">&lt;/t&gt;&lt;/is&gt;&lt;/c&gt;&lt;c</span> <span class="na">r=</span><span class="s">"B3"</span><span class="nt">&gt;&lt;v&gt;</span>25<span class="nt">&lt;/v&gt;&lt;/c&gt;&lt;c</span> <span class="na">r=</span><span class="s">"C3"</span> <span class="na">s=</span><span class="s">"1"</span><span class="nt">&gt;&lt;v&gt;</span>45150<span class="nt">&lt;/v&gt;&lt;/c&gt;&lt;/row&gt;</span>
</code></pre>

</div>



<p>Combine these rows with a header row and the static XML files, compress with JSZip, and you have a valid XLSX file.</p>

<h2>
  
  
  Results
</h2>

<p>With ExcelJS, exporting 1 million rows took around 20 seconds and consumed over 1GB of memory (Unfortunately I didn't capture a "before" screenshot for comparison).</p>

<p>After switching to the SQL-based XML approach:</p>

<ul>
<li>XML generation: <strong>5.5s</strong>
</li>
<li>ZIP compression: <strong>7.3s</strong>
</li>
<li>Memory usage: significantly reduced</li>
<li>Post-export browser freeze: <strong>gone</strong>
</li>
</ul>

<p>The reduction in GC-eligible objects was what solved the post-conversion sluggishness.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rc0f7bt1138gykogd61.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rc0f7bt1138gykogd61.png" alt="Processing time breakdown for 1 million row XLSX export: DuckDB Init 4ms, File Load 1ms, Table Creation 439ms, Schema Analysis 1ms, Preview Generation 4ms, Build XML 5.5s (41%), ZIP Compress 7.3s (55%) ." width="800" height="142"></a></p>

<p><em>Side note: compression could probably be moved to the WASM layer for even better performance, but that felt out of scope for now. 😇</em></p>

<h2>
  
  
  Trade-offs
</h2>

<p>This approach doesn't support cell styling — no colors, borders, or Excel-specific formatting. For LeapRows that's fine, since the goal is pure data export. But it's worth keeping in mind if you need rich formatting.</p>

<h2>
  
  
  Wrapping Up
</h2>

<p>Once I let go of the assumption that <em>"XLSX is a complex format,"</em> it became clear that for plain data export, a minimal XML structure is completely sufficient.</p>

<p>DuckDB's string processing is fast enough to make "generate XML via SQL" a genuinely practical approach — and I suspect the same idea could be applied in other creative ways beyond XLSX export.</p>

