---
Title: How to Extract Tables from PDFs Using Python (Without Losing Your Mind)
Description: 
Author: Burhanettin Nacar
Date: 2026-01-13T21:55:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you've ever tried to extract data from a PDF, you know the pain. What looks like a simple table on screen is actually a chaotic mess of positioned text elements in the file.</p>

<p>A friend of mine was struggling with extracting data from PDFs for his project, so I built a PDF extraction API to help him. Along the way, I learned way more about PDF internals than I expected. Here's a practical breakdown.</p>

<h2>
  
  
  The Problem: PDFs Don't Have "Tables"
</h2>

<p>Open any PDF with tabular data. It looks organized, right? Rows, columns, headers.</p>

<p>Now look at what's actually in the file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>draw "Product" at position (50, 100)
draw "Price" at position (200, 100)
draw "Widget" at position (50, 120)
draw "$99" at position (200, 120)
</code></pre>

</div>



<p>There's no table structure. No rows. No columns. Just text floating at coordinates.</p>

<p>Your job is to reconstruct the logical structure from spatial positions.</p>

<h2>
  
  
  Approach 1: PyMuPDF (Basic Text Extraction)
</h2>

<p>For simple text extraction, PyMuPDF (also called <code>fitz</code>) is fast and reliable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">fitz</span>

<span class="k">def</span> <span class="nf">extract_text</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">):</span>
    <span class="n">doc</span> <span class="o">=</span> <span class="n">fitz</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">)</span>
    <span class="n">text</span> <span class="o">=</span> <span class="sh">""</span>
    <span class="k">for</span> <span class="n">page</span> <span class="ow">in</span> <span class="n">doc</span><span class="p">:</span>
        <span class="n">text</span> <span class="o">+=</span> <span class="n">page</span><span class="p">.</span><span class="nf">get_text</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">text</span>
</code></pre>

</div>



<p><strong>Pros:</strong> Fast, handles most PDFs<br><br>
<strong>Cons:</strong> Tables come out as jumbled text</p>

<p>Output from a table:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Product Price Quantity
Widget $99 10
Gadget $149 5
</code></pre>

</div>



<p>Not useful if you need structured data.</p>

<h2>
  
  
  Approach 2: pdfplumber (Table Detection)
</h2>

<p>pdfplumber is specifically designed for table extraction:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pdfplumber</span>

<span class="k">def</span> <span class="nf">extract_tables</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">):</span>
    <span class="n">tables</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">with</span> <span class="n">pdfplumber</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">)</span> <span class="k">as</span> <span class="n">pdf</span><span class="p">:</span>
        <span class="k">for</span> <span class="n">page</span> <span class="ow">in</span> <span class="n">pdf</span><span class="p">.</span><span class="n">pages</span><span class="p">:</span>
            <span class="n">page_tables</span> <span class="o">=</span> <span class="n">page</span><span class="p">.</span><span class="nf">extract_tables</span><span class="p">()</span>
            <span class="n">tables</span><span class="p">.</span><span class="nf">extend</span><span class="p">(</span><span class="n">page_tables</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">tables</span>
</code></pre>

</div>



<p><strong>Pros:</strong> Detects table boundaries automatically<br><br>
<strong>Cons:</strong> Struggles with complex layouts, merged cells</p>

<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="p">[</span>
    <span class="p">[[</span><span class="sh">'</span><span class="s">Product</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Price</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Quantity</span><span class="sh">'</span><span class="p">],</span>
     <span class="p">[</span><span class="sh">'</span><span class="s">Widget</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">$99</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">10</span><span class="sh">'</span><span class="p">],</span>
     <span class="p">[</span><span class="sh">'</span><span class="s">Gadget</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">$149</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">5</span><span class="sh">'</span><span class="p">]]</span>
<span class="p">]</span>
</code></pre>

</div>



<p>Much better! But still needs post-processing.</p>

<h2>
  
  
  Approach 3: Combining Both
</h2>

<p>The best results come from combining approaches:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">fitz</span>
<span class="kn">import</span> <span class="n">pdfplumber</span>

<span class="k">def</span> <span class="nf">smart_extract</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">):</span>
    <span class="c1"># First, check if PDF has selectable text
</span>    <span class="n">doc</span> <span class="o">=</span> <span class="n">fitz</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">)</span>
    <span class="n">first_page_text</span> <span class="o">=</span> <span class="n">doc</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nf">get_text</span><span class="p">().</span><span class="nf">strip</span><span class="p">()</span>

    <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">first_page_text</span><span class="p">)</span> <span class="o">&lt;</span> <span class="mi">50</span><span class="p">:</span>
        <span class="c1"># Likely a scanned PDF - needs OCR
</span>        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Scanned PDF detected, OCR required</span><span class="sh">"</span><span class="p">}</span>

    <span class="c1"># Extract tables with pdfplumber
</span>    <span class="n">tables</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">with</span> <span class="n">pdfplumber</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">pdf_path</span><span class="p">)</span> <span class="k">as</span> <span class="n">pdf</span><span class="p">:</span>
        <span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="n">page</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="n">pdf</span><span class="p">.</span><span class="n">pages</span><span class="p">):</span>
            <span class="k">for</span> <span class="n">table</span> <span class="ow">in</span> <span class="n">page</span><span class="p">.</span><span class="nf">extract_tables</span><span class="p">():</span>
                <span class="k">if</span> <span class="n">table</span> <span class="ow">and</span> <span class="nf">len</span><span class="p">(</span><span class="n">table</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">:</span>
                    <span class="n">headers</span> <span class="o">=</span> <span class="n">table</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>
                    <span class="n">rows</span> <span class="o">=</span> <span class="n">table</span><span class="p">[</span><span class="mi">1</span><span class="p">:]</span>
                    <span class="n">tables</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                        <span class="sh">"</span><span class="s">page</span><span class="sh">"</span><span class="p">:</span> <span class="n">i</span> <span class="o">+</span> <span class="mi">1</span><span class="p">,</span>
                        <span class="sh">"</span><span class="s">headers</span><span class="sh">"</span><span class="p">:</span> <span class="n">headers</span><span class="p">,</span>
                        <span class="sh">"</span><span class="s">rows</span><span class="sh">"</span><span class="p">:</span> <span class="n">rows</span>
                    <span class="p">})</span>

    <span class="c1"># Extract remaining text with PyMuPDF
</span>    <span class="n">full_text</span> <span class="o">=</span> <span class="sh">""</span>
    <span class="k">for</span> <span class="n">page</span> <span class="ow">in</span> <span class="n">doc</span><span class="p">:</span>
        <span class="n">full_text</span> <span class="o">+=</span> <span class="n">page</span><span class="p">.</span><span class="nf">get_text</span><span class="p">()</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">tables</span><span class="sh">"</span><span class="p">:</span> <span class="n">tables</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="n">full_text</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">page_count</span><span class="sh">"</span><span class="p">:</span> <span class="nf">len</span><span class="p">(</span><span class="n">doc</span><span class="p">)</span>
    <span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  The Hard Parts Nobody Tells You About
</h2>

<h3>
  
  
  1. Table boundaries are ambiguous
</h3>

<p>Is this one table or two?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Name     | Email
---------|------------------
John     | john@example.com

Department | Budget
-----------|--------
Sales      | $50,000
</code></pre>

</div>



<p>Humans see two tables. Algorithms often merge them.</p>

<h3>
  
  
  2. Headers aren't always on top
</h3>

<p>Some invoices put totals at the bottom. Some have headers on the left side. Some have no headers at all.</p>

<h3>
  
  
  3. Multi-page tables
</h3>

<p>When a table spans pages, you need to:</p>

<ul>
<li>Detect it's a continuation (no headers on page 2)</li>
<li>Merge rows correctly</li>
<li>Handle page breaks mid-row</li>
</ul>

<h3>
  
  
  4. Currency and number parsing
</h3>

<p>"$1,234.56" vs "1.234,56 EUR" vs "JPY 1234"</p>

<p>Different locales, different formats. Don't assume.</p>

<h2>
  
  
  A Better Way: Use an API
</h2>

<p>After building this for my friend, I packaged it into an API so others don't have to:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"https://pdfpull-895295000838.europe-west1.run.app/api/v1/extract/tables"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-API-Key: sk_demo_123456789"</span> <span class="se">\</span>
  <span class="nt">-F</span> <span class="s2">"file=@invoice.pdf"</span>
</code></pre>

</div>



<p>Response:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"tables"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"page_number"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="nl">"headers"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Product"</span><span class="p">,</span><span class="w"> </span><span class="s2">"Price"</span><span class="p">,</span><span class="w"> </span><span class="s2">"Qty"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"rows"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">[</span><span class="s2">"Widget"</span><span class="p">,</span><span class="w"> </span><span class="s2">"$99"</span><span class="p">,</span><span class="w"> </span><span class="s2">"10"</span><span class="p">],</span><span class="w">
        </span><span class="p">[</span><span class="s2">"Gadget"</span><span class="p">,</span><span class="w"> </span><span class="s2">"$149"</span><span class="p">,</span><span class="w"> </span><span class="s2">"5"</span><span class="p">]</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"table_count"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>It also has smart parsers for invoices and resumes that extract specific fields:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"https://pdfpull-895295000838.europe-west1.run.app/api/v1/parse/invoice"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-API-Key: sk_demo_123456789"</span> <span class="se">\</span>
  <span class="nt">-F</span> <span class="s2">"file=@invoice.pdf"</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"vendor_name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ACME Corporation"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"invoice_number"</span><span class="p">:</span><span class="w"> </span><span class="s2">"INV-2024-0042"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"invoice_date"</span><span class="p">:</span><span class="w"> </span><span class="s2">"January 15, 2024"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"total_amount"</span><span class="p">:</span><span class="w"> </span><span class="mf">1250.00</span><span class="p">,</span><span class="w">
  </span><span class="nl">"currency"</span><span class="p">:</span><span class="w"> </span><span class="s2">"USD"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"line_items"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Widget"</span><span class="p">,</span><span class="w"> </span><span class="nl">"quantity"</span><span class="p">:</span><span class="w"> </span><span class="mi">10</span><span class="p">,</span><span class="w"> </span><span class="nl">"amount"</span><span class="p">:</span><span class="w"> </span><span class="mf">990.00</span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Gadget"</span><span class="p">,</span><span class="w"> </span><span class="nl">"quantity"</span><span class="p">:</span><span class="w"> </span><span class="mi">5</span><span class="p">,</span><span class="w"> </span><span class="nl">"amount"</span><span class="p">:</span><span class="w"> </span><span class="mf">260.00</span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"confidence"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.91</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Conclusion
</h2>

<p>PDF extraction is harder than it looks. If you're building something that only occasionally needs PDF parsing, use a library. If you're doing it at scale, consider an API that handles the edge cases for you.</p>

<p>If you need consistent results across lots of PDFs, an API can save you time. For one-off jobs, a library is usually enough.</p>




<p>Building this in public. Follow along on Twitter: <a href="https://twitter.com/uppnrise" rel="noopener noreferrer">@uppnrise</a></p>

