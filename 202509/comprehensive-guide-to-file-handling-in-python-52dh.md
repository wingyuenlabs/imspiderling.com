---
Title: Comprehensive Guide to File Handling in Python
Description: 
Author: Tapoban Ray
Date: 2025-09-03T21:23:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>File handling is the process of storing, reading, and writing data to and from files permanently on a storage device, rather than just in a computer's memory.</p>

<h3>
  
  
  In this tutorial, you are going to learn the following,
</h3>

<ol>
<li>Types of files that can be handled in Python.</li>
<li>The modes of opening a file.</li>
<li>Reading from a file.</li>
<li>Writing to a file.</li>
<li>Using <code>tell()</code> and <code>seek()</code> functions.</li>
</ol>




<h2>
  
  
  Types of data files:
</h2>

<ol>
<li>
<strong>Text Files</strong>: Stores data in the same format as typed.

<ol>
<li>
<strong>Regular Text Files</strong> (<em>.txt)</em>
</li>
<li>
<strong>Delimited Text File.</strong>

<ol>
<li>
<strong>Tab-Separated Values files</strong> (<em>.txt or .csv</em>)</li>
<li>
<strong>Comma-Separated Values files</strong> (<em>.csv</em>)</li>
</ol>


</li>

</ol>

</li>

<li>

<strong>Binary Files</strong>: Stores information in the form of a stream of bytes. Not a human-readable format.</li>

</ol>

<h2>
  
  
  Opening and Closing files in Python
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">myfile.txt</span><span class="sh">"</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">)</span>
<span class="n">fileObj</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>

<span class="c1"># -------------------------- OR --------------------------- 
</span>
<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">myfile.csv</span><span class="sh">"</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">fileObj</span><span class="p">:</span>
    <span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>
    <span class="c1"># Closes file automatically once you get out of this indentation
</span></code></pre>

</div>



<p><strong>When using filenames with path, you remember the following:</strong></p>

<ul>
<li>Always make the file path string a raw string.
Using a raw string helps us in ensuring there is no special meaning attached to any character of our file path string.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">C:\Temp\data1.txt</span><span class="sh">"</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">)</span> <span class="c1"># Use prefix 'r'
</span>
<span class="c1"># -------------------------- OR --------------------------- 
</span>
<span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">C:</span><span class="se">\\</span><span class="s">Temp</span><span class="se">\\</span><span class="s">data1.csv</span><span class="sh">"</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">)</span> <span class="c1"># Or use double slashes
</span></code></pre>

</div>



<h2>
  
  
  Common modes of opening a file:
</h2>

<p>A <strong>file-mode</strong> governs the type of operations possible in the opened file.</p>

<div class="table-wrapper-paragraph"><table>
  <tr>
    <th>Text File Mode</th>
    <th>Binary File Mode</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>'r'</td>
    <td>'rb'</td>
    <td>READ ONLY.<br>File must exist already, or Python raises I/O error.</td>
  </tr>
  <tr>
    <td>'w'</td>
    <td>'wb'</td>
    <td>WRITE ONLY.<br>File is created if it does not already exist. If the file already exists with data, Python overwrites it.</td>
  </tr>
<tr>
    <td>'a'</td>
    <td>'ab'</td>
    <td>APPEND.<br>File is created if it does not already exist.<br>If the file exists with data, Python retains the data, and new data will be appended to the end.</td>
  </tr>
<tr>
    <td>'r+'</td>
    <td>'rb+' or 'r+b'</td>
    <td>READ AND WRITE.<br>Properties of READ ONLY mode are applied.<br>Both read and write operations can take place.</td>
  </tr>
<tr>
    <td>'w+'</td>
    <td>'wb+' or 'w+b'</td>
    <td>WRITE AND READ<br>Properties of WRITE ONLY mode are applied.<br>Both read and write operations can take place.</td>
  </tr>
<tr>
    <td>'a+'</td>
    <td>'ab+' or 'a+b </td>
    <td>APPEND AND READ<br>File must exist already, or Python raises I/O error.</td> 
</tr>
</table></div>




<h2>
  
  
  Reading and Writing files in Python
</h2>

<h4>
  
  
  Writing Text files:
</h4>

<p><strong><code>write(str)</code> method</strong> -&gt; Writes a string of characters<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">fileObj</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="sh">"</span><span class="s">Hello World</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><strong><code>writelines([str])</code> method</strong> -&gt; Writes a list of lines.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">fileObj</span><span class="p">.</span><span class="nf">writelines</span><span class="p">([</span><span class="sh">"</span><span class="s">Hello 1</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">World here</span><span class="sh">"</span><span class="p">])</span>
</code></pre>

</div>



<h4>
  
  
  Reading Text files:
</h4>

<ol>
<li>
<strong><code>read(n)</code> method</strong> -&gt; Returns a Python String
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>
<span class="c1"># -------------------------- OR --------------------------- 
</span><span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">read</span><span class="p">(</span><span class="n">n</span><span class="p">)</span> <span class="c1"># where n is the number of bytes to read.
</span></code></pre>

</div>



<ol>
<li>
<strong><code>readline(n)</code> method</strong> -&gt; Returns a line of string ending with '\n'
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">readline</span><span class="p">()</span>
<span class="c1"># -------------------------- OR --------------------------- 
</span><span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">readline</span><span class="p">(</span><span class="n">n</span><span class="p">)</span> <span class="c1"># where n is the number of lines to read.
</span></code></pre>

</div>



<ol>
<li>
<strong><code>readlines()</code> method</strong> -&gt; Returns a list of lines.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">readlines</span><span class="p">()</span>
<span class="c1"># -------------------------- OR --------------------------- 
</span><span class="n">data</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">readlines</span><span class="p">()</span> 
</code></pre>

</div>



<h4>
  
  
  Reading CSV files:
</h4>

<p><em>Opening CSV files with</em> <code>newline=""</code> <em>ensures that no translation of EndOfLine(EOL) character takes place.</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">csv</span>

<span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">data.csv</span><span class="sh">"</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">""</span><span class="p">)</span>
<span class="n">cReader</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nf">reader</span><span class="p">(</span><span class="n">fileObj</span><span class="p">)</span>
<span class="k">for</span> <span class="n">val</span> <span class="ow">in</span> <span class="n">cReader</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">val</span><span class="p">)</span>
</code></pre>

</div>



<h4>
  
  
  Writing CSV files:
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">csv</span>

<span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">data.csv</span><span class="sh">"</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">w</span><span class="sh">"</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">""</span><span class="p">)</span>
<span class="n">cWriter</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nf">writer</span><span class="p">(</span><span class="n">fileObj</span><span class="p">)</span>
<span class="n">cWriter</span><span class="p">.</span><span class="nf">writerow</span><span class="p">([</span><span class="sh">"</span><span class="s">Roll No</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Name</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Marks</span><span class="sh">"</span><span class="p">])</span> <span class="c1"># Writes only one row
</span>
<span class="c1"># Writes multiple rows at once.
</span><span class="n">cWriter</span><span class="p">.</span><span class="nf">writerows</span><span class="p">(</span>
    <span class="p">[</span>
        <span class="p">[</span><span class="sh">"</span><span class="s">01</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Raj</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">50</span><span class="sh">"</span><span class="p">],</span>
        <span class="p">[</span><span class="sh">"</span><span class="s">02</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Meera</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">46</span><span class="sh">"</span><span class="p">],</span>
        <span class="p">[</span><span class="sh">"</span><span class="s">03</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Anil</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">48</span><span class="sh">"</span><span class="p">],</span>
    <span class="p">]</span>
<span class="p">)</span>
<span class="n">fileObj</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
</code></pre>

</div>



<h4>
  
  
  Writing Binary files:
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pickle</span>

<span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">data.dat</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span>
<span class="n">data</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">Roll No</span><span class="sh">"</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="sh">"</span><span class="s">Name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">John</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Marks</span><span class="sh">"</span><span class="p">:</span> <span class="mi">50</span><span class="p">}</span>
<span class="n">pickle</span><span class="p">.</span><span class="nf">dump</span><span class="p">(</span><span class="n">data</span><span class="p">,</span> <span class="n">fileObj</span><span class="p">)</span>
<span class="n">fileObj</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
</code></pre>

</div>



<h4>
  
  
  Reading Binary files:
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pickle</span>

<span class="n">fileObj</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">data.dat</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span>
<span class="k">try</span><span class="p">:</span> 
    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">data</span> <span class="o">=</span> <span class="n">pickle</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="n">fileObj</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>

<span class="k">except</span> <span class="nb">EOFError</span><span class="p">:</span>
    <span class="n">fileObj</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
</code></pre>

</div>



<h2>
  
  
  The <code>tell()</code> function
</h2>

<p>Returns the current position of the file pointer in the file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">pos</span> <span class="o">=</span> <span class="n">fileObj</span><span class="p">.</span><span class="nf">tell</span><span class="p">()</span>
</code></pre>

</div>



<h2>
  
  
  The <code>seek()</code> function
</h2>

<p>Changes the position of the file-pointer by placing the file-pointer at the specified position of the opened file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">fileObj</span><span class="p">.</span><span class="nf">seek</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="n">mode</span><span class="p">)</span>
</code></pre>

</div>



<blockquote>
<p><strong>mode can be 0, 1, or 2.</strong></p>

<ul>
<li>0 -&gt; Move the file-pointer with respect to the <strong>beginning</strong> of the file. (DEFAULT MODE)</li>
<li>1 -&gt; Move the file-pointer with respect to the <strong>current position</strong> of the file.</li>
<li>2 -&gt; 
Move the file-pointer with respect to the <strong>end</strong> of the file.</li>
</ul>
</blockquote>

<p><strong><code>seek()</code> and <code>tell()</code> functions are mostly used when editing a file.</strong></p>

