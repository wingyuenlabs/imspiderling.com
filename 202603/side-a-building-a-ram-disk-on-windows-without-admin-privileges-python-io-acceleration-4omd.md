---
Title: [Side A] Building a RAM Disk on Windows without Admin Privileges — Python I/O Acceleration Techniques
Description: 
Author: D
Date: 2026-03-25T22:11:19.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>From the Author:</strong><br>
<strong>D-MemFS</strong> was featured in <a href="https://www.pythonweekly.com/p/python-weekly-issue-737-march-19-2026" rel="noopener noreferrer"><strong>Python Weekly Issue #737</strong></a> (March 19, 2026) under <em>Interesting Projects, Tools and Libraries</em>. Being picked up by one of the most widely-read Python newsletters confirmed that in-memory I/O bottlenecks and memory management are truly universal challenges for developers everywhere. This series is my response to that interest.</p>
</blockquote>

<h3>
  
  
  🧭 About this Series: The Two Sides of Development
</h3>

<p>To provide a complete picture of this project, I’ve split each update into two perspectives:</p>

<ul>
<li>
<strong>Side A (Practical / from Qiita):</strong> Implementation details, benchmarks, and technical solutions.</li>
<li>
<strong>Side B (Philosophy / from Zenn):</strong> The development war stories, AI-collaboration, and design decisions.</li>
</ul>

<p><em>Looking for the other side of this chapter?</em></p>

<ul>
<li>
<strong>[Side B] Breaking Free from Vibe Coding Fatigue: A Practical Record of Building an OSS with 'Spec-First AI Development'</strong>: <a href="https://dev.to/d_9d93cd53/side-b-breaking-free-from-vibe-coding-fatigue-a-practical-record-of-building-an-oss-with-1pl9">https://dev.to/d_9d93cd53/side-b-breaking-free-from-vibe-coding-fatigue-a-practical-record-of-building-an-oss-with-1pl9</a>
</li>
</ul>




<h2>
  
  
  Introduction
</h2>

<p>When you want to use a RAM disk on Windows, tools like ImDisk Toolkit or OSFMount are usually the first candidates that come to mind. However, they share a common restriction—they require <strong>administrative privileges</strong>, or a paid license. Often, you can't use them in CI environments or on shared development machines.</p>

<p>But if you slightly change your perspective, another approach emerges.</p>

<p><strong>Isn't the memory of the Python process itself a RAM disk from the perspective of the Python code?</strong></p>

<p>In this article, I will introduce how to leverage <strong>D-MemFS</strong> as a "Python-exclusive software RAM disk" that requires neither drivers nor administrative privileges.</p>

<h2>
  
  
  The Conclusion First: Comparison of Methods
</h2>

<p>Let's first summarize "which method you should choose."</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Comparison Item</th>
<th>
<code>tempfile</code> on SSD</th>
<th>RAM Disk (ImDisk, etc.)</th>
<th><strong>D-MemFS</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Admin Privileges</td>
<td>Not Required</td>
<td><strong>Required</strong></td>
<td>Not Required</td>
</tr>
<tr>
<td>External Tools</td>
<td>Not Required</td>
<td>Required</td>
<td>Not Required (pip only)</td>
</tr>
<tr>
<td>Volatile (Auto-delete)</td>
<td>△ Manual Delete</td>
<td>△ Lost on Reboot</td>
<td>✅ Auto-collected by GC</td>
</tr>
<tr>
<td>Cross-Platform</td>
<td>✅</td>
<td>❌ Windows Only</td>
<td>✅ Win/Mac/Linux</td>
</tr>
<tr>
<td>Access from Other Processes</td>
<td>✅</td>
<td>✅</td>
<td>❌ Python Internal Only</td>
</tr>
<tr>
<td>Memory Limit Management</td>
<td>❌</td>
<td>❌</td>
<td>✅ Hard Quotas + Memory Guard</td>
</tr>
<tr>
<td>Sequential I/O</td>
<td>~1.9 GB/s</td>
<td>~2.0 GB/s</td>
<td>~1.9 GB/s</td>
</tr>
<tr>
<td>Random Access I/O</td>
<td>~1.4 GB/s</td>
<td>~1.3 GB/s</td>
<td><strong>~1.4 GB/s</strong></td>
</tr>
<tr>
<td>Usage in CI Environments</td>
<td>✅</td>
<td>❌ Permission Wall</td>
<td>✅</td>
</tr>
</tbody>
</table></div>

<p><strong>Conclusion</strong>: If it's pure Python processing without the need to call external commands, D-MemFS is the easiest and fastest choice.</p>

<blockquote>
<p>💡 <strong>v0.3.0 New Feature: Memory Guard</strong><br>
While Hard Quotas manage the "budget within the virtual FS," the <strong>Memory Guard</strong> introduced in v0.3.0 checks the host machine's <strong>remaining physical memory</strong>. If there isn't enough memory, it rejects the write beforehand. Even if you set a quota of 4 GiB, it's meaningless if the machine only has 2 GiB free—Memory Guard prevents this issue. Details will be covered in <a href="https://dev.to/d_9d93cd53/d-memfs-03a-hard-quota-oom-defense">Side A - Part 3: Design Philosophy of Hard Quotas</a>.</p>
</blockquote>

<h2>
  
  
  Premise: Why Do We Want a RAM Disk?
</h2>

<p>The primary uses for a RAM disk are twofold:</p>

<ol>
<li>
<strong>High-speed temporary file processing</strong> — To speed up processing by eliminating disk I/O latency.</li>
<li>
<strong>Avoiding writes to disk</strong> — To protect SSD write endurance, and to avoid leaving sensitive data behind.</li>
</ol>

<p>Both of these are very common requirements in Python testing, CI, and data processing pipelines.</p>

<h2>
  
  
  D-MemFS: Python-Exclusive Software RAM Disk
</h2>

<p><strong>D-MemFS</strong> (<code>dmemfs</code>) is a pure Python in-memory file system library.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>D-MemFS
</code></pre>

</div>



<ul>
<li>Standard library only (Zero external dependencies)</li>
<li>No administrative privileges required</li>
<li>Runs on Windows / macOS / Linux</li>
<li>Automatically deleted when the process terminates (volatile)</li>
</ul>

<h2>
  
  
  Replacing tempfile with D-MemFS
</h2>

<p>The most typical pattern is replacing <code>tempfile.TemporaryDirectory()</code>.</p>

<h3>
  
  
  Before Replacement (Writes to disk occur)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">tempfile</span>
<span class="kn">import</span> <span class="n">os</span>

<span class="k">def</span> <span class="nf">process_data</span><span class="p">(</span><span class="n">raw</span><span class="p">:</span> <span class="nb">bytes</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">bytes</span><span class="p">:</span>
    <span class="k">with</span> <span class="n">tempfile</span><span class="p">.</span><span class="nc">TemporaryDirectory</span><span class="p">()</span> <span class="k">as</span> <span class="n">tmpdir</span><span class="p">:</span>
        <span class="n">input_path</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">tmpdir</span><span class="p">,</span> <span class="sh">"</span><span class="s">input.bin</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">output_path</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">tmpdir</span><span class="p">,</span> <span class="sh">"</span><span class="s">output.bin</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">input_path</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">raw</span><span class="p">)</span>

        <span class="c1"># Some processing (an example assuming calling an external command)
</span>        <span class="n">data</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="n">input_path</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">).</span><span class="nf">read</span><span class="p">()</span>
        <span class="n">result</span> <span class="o">=</span> <span class="nf">bytes</span><span class="p">(</span><span class="n">b</span> <span class="o">^</span> <span class="mh">0xFF</span> <span class="k">for</span> <span class="n">b</span> <span class="ow">in</span> <span class="n">data</span><span class="p">)</span>  <span class="c1"># Dummy conversion
</span>
        <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">output_path</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">result</span><span class="p">)</span>

        <span class="k">return</span> <span class="nf">open</span><span class="p">(</span><span class="n">output_path</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">).</span><span class="nf">read</span><span class="p">()</span>
</code></pre>

</div>



<h3>
  
  
  After Replacement (Completely contained in memory)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">MemoryFileSystem</span>

<span class="k">def</span> <span class="nf">process_data</span><span class="p">(</span><span class="n">raw</span><span class="p">:</span> <span class="nb">bytes</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">bytes</span><span class="p">:</span>
    <span class="n">mfs</span> <span class="o">=</span> <span class="nc">MemoryFileSystem</span><span class="p">()</span>
    <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/tmp</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/tmp/input.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">raw</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/tmp/input.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">data</span> <span class="o">=</span> <span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>

    <span class="n">result</span> <span class="o">=</span> <span class="nf">bytes</span><span class="p">(</span><span class="n">b</span> <span class="o">^</span> <span class="mh">0xFF</span> <span class="k">for</span> <span class="n">b</span> <span class="ow">in</span> <span class="n">data</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/tmp/output.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">result</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/tmp/output.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>
</code></pre>

</div>



<p>Access to the disk becomes zero. When <code>mfs</code> goes out of scope, it is collected by the GC, so no clean-up is necessary.</p>

<h3>
  
  
  Automatic Release via GC
</h3>

<p><code>MemoryFileSystem</code> is collected by the GC when it exits the scope. You can use it similarly to <code>TemporaryDirectory</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">MemoryFileSystem</span>

<span class="k">def</span> <span class="nf">process</span><span class="p">():</span>
    <span class="n">mfs</span> <span class="o">=</span> <span class="nc">MemoryFileSystem</span><span class="p">(</span><span class="n">max_quota</span><span class="o">=</span><span class="mi">64</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>
    <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/work</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/work/data.csv</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="sa">b</span><span class="sh">"</span><span class="s">id,value</span><span class="se">\n</span><span class="s">1,100</span><span class="se">\n</span><span class="s">2,200</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/work/data.csv</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">().</span><span class="nf">decode</span><span class="p">())</span>

<span class="nf">process</span><span class="p">()</span>
<span class="c1"># When exiting the function, mfs is collected by GC, and all contents are wiped.
</span></code></pre>

</div>



<h2>
  
  
  Benchmarks
</h2>

<p>The following are benchmark results comparing D-MemFS, <code>tempfile</code> on a RAM Disk, and <code>tempfile</code> on an SSD (repeat=5, warmup=1).</p>

<blockquote>
<p><strong>Measurement Environment</strong>: Windows, X:\TEMP (RAM Disk) / C:\TempX (SSD)<br>
Throughput values are calculated from 512 MiB of write + read.</p>
</blockquote>

<h3>
  
  
  Mass read/write of small files (300 files × 4 KiB)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Time (ms)</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>io.BytesIO</code></td>
<td>6</td>
</tr>
<tr>
<td>D-MemFS</td>
<td>51</td>
</tr>
<tr>
<td>tempfile (on RAM Disk)</td>
<td>207</td>
</tr>
<tr>
<td>tempfile (on SSD)</td>
<td>267</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Stream read/write (16 MiB, 64 KiB chunks)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Time (ms)</th>
</tr>
</thead>
<tbody>
<tr>
<td>tempfile (on RAM Disk)</td>
<td>20</td>
</tr>
<tr>
<td>tempfile (on SSD)</td>
<td>21</td>
</tr>
<tr>
<td><code>io.BytesIO</code></td>
<td>62</td>
</tr>
<tr>
<td>D-MemFS</td>
<td>81</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Random Access (16 MiB)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Time (ms)</th>
</tr>
</thead>
<tbody>
<tr>
<td>D-MemFS</td>
<td><strong>34</strong></td>
</tr>
<tr>
<td>tempfile (on SSD)</td>
<td>35</td>
</tr>
<tr>
<td>tempfile (on RAM Disk)</td>
<td>37</td>
</tr>
<tr>
<td><code>io.BytesIO</code></td>
<td>82</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Large Capacity Stream read/write (512 MiB, 1 MiB chunks)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Time (ms)</th>
</tr>
</thead>
<tbody>
<tr>
<td>D-MemFS</td>
<td><strong>529</strong></td>
</tr>
<tr>
<td>tempfile (on RAM Disk)</td>
<td>514</td>
</tr>
<tr>
<td>tempfile (on SSD)</td>
<td>541</td>
</tr>
<tr>
<td><code>io.BytesIO</code></td>
<td>2 258</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Random reading of numerous files (10,000 files × 4 KiB)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Time (ms)</th>
</tr>
</thead>
<tbody>
<tr>
<td>D-MemFS</td>
<td><strong>1 280</strong></td>
</tr>
<tr>
<td>tempfile (on RAM Disk)</td>
<td>6 310</td>
</tr>
<tr>
<td>tempfile (on SSD)</td>
<td>8 601</td>
</tr>
</tbody>
</table></div>

<p><strong>Key Points</strong>:</p>

<ul>
<li>
<strong>For small and numerous files, D-MemFS is overwhelmingly faster.</strong> Because the file open/close overhead is practically zero, it's 4 times faster with 300 files and <strong>over 5 times faster</strong> for random reads of 10,000 files.</li>
<li>
<strong>For large capacity streams (512 MiB+, large chunks), D-MemFS and tempfile are equivalent.</strong> Because the memory bandwidth becomes the bottleneck, the difference in storage location hardly shows.</li>
<li>
<strong>For small to medium capacity streams around 16 MiB, tempfile is faster.</strong> The per-chunk overhead is relatively higher for D-MemFS, so the finer the chunk size, the wider the gap.</li>
<li>
<code>io.BytesIO</code> is the fastest for single-stream usage, but lacks file management features (paths, directories, quotas).</li>
</ul>

<h2>
  
  
  Using in Asynchronous Code (AsyncMemoryFileSystem)
</h2>

<p>For <code>asyncio</code>-based code, use <code>AsyncMemoryFileSystem</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">AsyncMemoryFileSystem</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">mfs</span> <span class="o">=</span> <span class="nc">AsyncMemoryFileSystem</span><span class="p">(</span><span class="n">max_quota</span><span class="o">=</span><span class="mi">32</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>
    <span class="k">await</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/data</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Asynchronous write
</span>    <span class="k">async</span> <span class="k">with</span> <span class="k">await</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/data/result.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">await</span> <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="sa">b</span><span class="sh">"</span><span class="s">async result data</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Asynchronous read
</span>    <span class="k">async</span> <span class="k">with</span> <span class="k">await</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/data/result.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">content</span> <span class="o">=</span> <span class="k">await</span> <span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>
        <span class="nf">print</span><span class="p">(</span><span class="n">content</span><span class="p">)</span>

<span class="n">asyncio</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="nf">main</span><span class="p">())</span>
</code></pre>

</div>



<p>Internally, it incorporates exclusive control based on <code>asyncio.Lock</code>, making it safe to access simultaneously from multiple coroutines.</p>

<h3>
  
  
  Example: Parallel Downloads and Immediate Processing
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">import</span> <span class="n">aiohttp</span>
<span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">AsyncMemoryFileSystem</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">fetch_and_process</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="n">url</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">mfs</span><span class="p">,</span> <span class="n">path</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="k">async</span> <span class="k">with</span> <span class="n">session</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span> <span class="k">as</span> <span class="n">resp</span><span class="p">:</span>
        <span class="n">data</span> <span class="o">=</span> <span class="k">await</span> <span class="n">resp</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>
    <span class="k">async</span> <span class="k">with</span> <span class="k">await</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">path</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">await</span> <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">urls</span> <span class="o">=</span> <span class="p">[</span>
        <span class="p">(</span><span class="sh">"</span><span class="s">https://example.com/a.json</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">/cache/a.json</span><span class="sh">"</span><span class="p">),</span>
        <span class="p">(</span><span class="sh">"</span><span class="s">https://example.com/b.json</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">/cache/b.json</span><span class="sh">"</span><span class="p">),</span>
        <span class="p">(</span><span class="sh">"</span><span class="s">https://example.com/c.json</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">/cache/c.json</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">]</span>

    <span class="n">mfs</span> <span class="o">=</span> <span class="nc">AsyncMemoryFileSystem</span><span class="p">(</span><span class="n">max_quota</span><span class="o">=</span><span class="mi">64</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>
    <span class="k">await</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/cache</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">async</span> <span class="k">with</span> <span class="n">aiohttp</span><span class="p">.</span><span class="nc">ClientSession</span><span class="p">()</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
        <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">gather</span><span class="p">(</span><span class="o">*</span><span class="p">[</span>
            <span class="nf">fetch_and_process</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="n">url</span><span class="p">,</span> <span class="n">mfs</span><span class="p">,</span> <span class="n">path</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">url</span><span class="p">,</span> <span class="n">path</span> <span class="ow">in</span> <span class="n">urls</span>
        <span class="p">])</span>

    <span class="c1"># Process all files while they are readily available in memory
</span>    <span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">path</span> <span class="ow">in</span> <span class="n">urls</span><span class="p">:</span>
        <span class="k">async</span> <span class="k">with</span> <span class="k">await</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">path</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="n">data</span> <span class="o">=</span> <span class="k">await</span> <span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">path</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">data</span><span class="p">)</span><span class="si">}</span><span class="s"> bytes</span><span class="sh">"</span><span class="p">)</span>

<span class="n">asyncio</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="nf">main</span><span class="p">())</span>
</code></pre>

</div>



<h2>
  
  
  Limitations
</h2>

<p>While D-MemFS is extremely useful as a software RAM disk, it has fundamental limitations compared to a real RAM disk.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Limitation</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Restricted to within the Python Process</strong></td>
<td>It cannot be accessed from other processes. It cannot be used for external commands or subprocess.</td>
</tr>
<tr>
<td><strong>Volatile</strong></td>
<td>When the process ends, all contents disappear (it does not persist until explicitly written out).</td>
</tr>
<tr>
<td><strong>Cannot be Mounted</strong></td>
<td>It cannot be exposed to the OS as a drive letter or mount point.</td>
</tr>
<tr>
<td><strong>Not compatible with <code>os.PathLike</code> (Intentional)</strong></td>
<td>A design decision to prevent confusing virtual paths with host paths.</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Use Cases
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Use Case</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Testing</strong></td>
<td>For tests involving the file system using <code>pytest</code>. As a replacement for the <code>tmp_path</code> fixture.</td>
</tr>
<tr>
<td><strong>CI</strong></td>
<td>Speeding up builds on GitHub Actions or Azure Pipelines by reducing disk I/O.</td>
</tr>
<tr>
<td><strong>Data Processing Pipelines</strong></td>
<td>Handling all intermediate files entirely in memory.</td>
</tr>
<tr>
<td><strong>Archive Extraction/Processing</strong></td>
<td>Extracting zip/tar archives into memory and processing the internal files without writing to disk.</td>
</tr>
<tr>
<td><strong>Dealing with Sensitive Data</strong></td>
<td>Temporary processing where you don't want to leave any footprints on disk.</td>
</tr>
<tr>
<td><strong>Mocking without External Dependencies</strong></td>
<td>Intercepting file system access during testing.</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Extracting Archives into Memory
</h2>

<p>This is an example of extracting a zip file without writing it to disk and processing the internal files.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">io</span>
<span class="kn">import</span> <span class="n">zipfile</span>
<span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">MemoryFileSystem</span>

<span class="k">def</span> <span class="nf">process_zip</span><span class="p">(</span><span class="n">zip_bytes</span><span class="p">:</span> <span class="nb">bytes</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">int</span><span class="p">]:</span>
    <span class="sh">"""</span><span class="s">Extract contents of the zip to MFS in memory, returning the size of each file.</span><span class="sh">"""</span>
    <span class="n">mfs</span> <span class="o">=</span> <span class="nc">MemoryFileSystem</span><span class="p">(</span><span class="n">max_quota</span><span class="o">=</span><span class="mi">256</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>
    <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/extracted</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">zipfile</span><span class="p">.</span><span class="nc">ZipFile</span><span class="p">(</span><span class="n">io</span><span class="p">.</span><span class="nc">BytesIO</span><span class="p">(</span><span class="n">zip_bytes</span><span class="p">))</span> <span class="k">as</span> <span class="n">zf</span><span class="p">:</span>
        <span class="k">for</span> <span class="n">name</span> <span class="ow">in</span> <span class="n">zf</span><span class="p">.</span><span class="nf">namelist</span><span class="p">():</span>
            <span class="n">data</span> <span class="o">=</span> <span class="n">zf</span><span class="p">.</span><span class="nf">read</span><span class="p">(</span><span class="n">name</span><span class="p">)</span>
            <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">/extracted/</span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
                <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>

    <span class="c1"># Process files on MFS (here returning a list of sizes)
</span>    <span class="n">result</span> <span class="o">=</span> <span class="p">{}</span>
    <span class="k">for</span> <span class="n">entry</span> <span class="ow">in</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">listdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/extracted</span><span class="sh">"</span><span class="p">):</span>
        <span class="n">path</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">/extracted/</span><span class="si">{</span><span class="n">entry</span><span class="si">}</span><span class="sh">"</span>
        <span class="n">result</span><span class="p">[</span><span class="n">entry</span><span class="p">]</span> <span class="o">=</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">stat</span><span class="p">(</span><span class="n">path</span><span class="p">)[</span><span class="sh">"</span><span class="s">size</span><span class="sh">"</span><span class="p">]</span>
    <span class="k">return</span> <span class="n">result</span>
</code></pre>

</div>



<p>You can do similar things with tar.gz.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">io</span>
<span class="kn">import</span> <span class="n">tarfile</span>
<span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">MemoryFileSystem</span>

<span class="k">def</span> <span class="nf">process_targz</span><span class="p">(</span><span class="n">targz_bytes</span><span class="p">:</span> <span class="nb">bytes</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">]:</span>
    <span class="sh">"""</span><span class="s">Extract tar.gz into MFS in memory and return the file list.</span><span class="sh">"""</span>
    <span class="n">mfs</span> <span class="o">=</span> <span class="nc">MemoryFileSystem</span><span class="p">(</span><span class="n">max_quota</span><span class="o">=</span><span class="mi">512</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>
    <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/extracted</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">tarfile</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">fileobj</span><span class="o">=</span><span class="n">io</span><span class="p">.</span><span class="nc">BytesIO</span><span class="p">(</span><span class="n">targz_bytes</span><span class="p">),</span> <span class="n">mode</span><span class="o">=</span><span class="sh">"</span><span class="s">r:gz</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">tf</span><span class="p">:</span>
        <span class="k">for</span> <span class="n">member</span> <span class="ow">in</span> <span class="n">tf</span><span class="p">.</span><span class="nf">getmembers</span><span class="p">():</span>
            <span class="k">if</span> <span class="n">member</span><span class="p">.</span><span class="nf">isfile</span><span class="p">():</span>
                <span class="n">f</span> <span class="o">=</span> <span class="n">tf</span><span class="p">.</span><span class="nf">extractfile</span><span class="p">(</span><span class="n">member</span><span class="p">)</span>
                <span class="k">if</span> <span class="n">f</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
                    <span class="c1"># Reproduce directory hierarchy
</span>                    <span class="n">parts</span> <span class="o">=</span> <span class="n">member</span><span class="p">.</span><span class="n">name</span><span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="sh">"</span><span class="s">/</span><span class="sh">"</span><span class="p">)</span>
                    <span class="k">for</span> <span class="n">depth</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nf">len</span><span class="p">(</span><span class="n">parts</span><span class="p">)):</span>
                        <span class="n">d</span> <span class="o">=</span> <span class="sh">"</span><span class="s">/extracted/</span><span class="sh">"</span> <span class="o">+</span> <span class="sh">"</span><span class="s">/</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">parts</span><span class="p">[:</span><span class="n">depth</span><span class="p">])</span>
                        <span class="k">try</span><span class="p">:</span>
                            <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="n">d</span><span class="p">)</span>
                        <span class="k">except</span> <span class="nb">FileExistsError</span><span class="p">:</span>
                            <span class="k">pass</span>
                    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">/extracted/</span><span class="si">{</span><span class="n">member</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">out</span><span class="p">:</span>
                        <span class="n">out</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">())</span>

    <span class="k">return</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">listdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/extracted</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>The key point is that <strong>writing to disk is zero</strong>. Compared to the traditional pattern of extracting downloaded archives to a temporary directory before processing, you won't wear down the write endurance of your SSD.</p>

<h2>
  
  
  Using as a pytest Fixture
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pytest</span>
<span class="kn">from</span> <span class="n">dmemfs</span> <span class="kn">import</span> <span class="n">MemoryFileSystem</span>

<span class="nd">@pytest.fixture</span>
<span class="k">def</span> <span class="nf">mfs</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Provides an independent in-memory FS for each test.</span><span class="sh">"""</span>
    <span class="k">return</span> <span class="nc">MemoryFileSystem</span><span class="p">(</span><span class="n">max_quota</span><span class="o">=</span><span class="mi">32</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">test_write_and_read</span><span class="p">(</span><span class="n">mfs</span><span class="p">):</span>
    <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/work</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/work/test.txt</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="sa">b</span><span class="sh">"</span><span class="s">test data</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/work/test.txt</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">rb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">assert</span> <span class="n">f</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span> <span class="o">==</span> <span class="sa">b</span><span class="sh">"</span><span class="s">test data</span><span class="sh">"</span>

<span class="k">def</span> <span class="nf">test_quota_is_isolated</span><span class="p">(</span><span class="n">mfs</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Each test has an independent quota.</span><span class="sh">"""</span>
    <span class="n">mfs</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="sh">"</span><span class="s">/data</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">with</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">/data/file.bin</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">wb</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="sa">b</span><span class="sh">"</span><span class="s">x</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">1024</span><span class="p">)</span>
    <span class="n">st</span> <span class="o">=</span> <span class="n">mfs</span><span class="p">.</span><span class="nf">stat</span><span class="p">(</span><span class="sh">"</span><span class="s">/data/file.bin</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">assert</span> <span class="n">st</span><span class="p">[</span><span class="sh">"</span><span class="s">size</span><span class="sh">"</span><span class="p">]</span> <span class="o">==</span> <span class="mi">1024</span>
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>The problem of needing administrative privileges for RAM disks on Windows can be completely avoided within Python code by using D-MemFS. Just by running <code>pip install D-MemFS</code> and replacing <code>TemporaryDirectory</code>, you can gain a dramatic speed improvement over using tempfile on an SSD.</p>

<p>For pure Python processing that doesn't rely on external commands, D-MemFS is the easiest and fastest choice.</p>

<ul>
<li>GitHub: <a href="https://github.com/nightmarewalker/D-MemFS" rel="noopener noreferrer">https://github.com/nightmarewalker/D-MemFS</a>
</li>
<li>PyPI: <a href="https://pypi.org/project/D-MemFS/" rel="noopener noreferrer">https://pypi.org/project/D-MemFS/</a>
</li>
</ul>




<h3>
  
  
  🔗 Links &amp; Resources
</h3>

<ul>
<li>
<strong>GitHub</strong>: <a href="https://github.com/nightmarewalker/D-MemFS" rel="noopener noreferrer">https://github.com/nightmarewalker/D-MemFS</a>
</li>
<li>
<strong>Original Japanese Article</strong>: <a href="https://qiita.com/_D_/items/f0017799e1e7e0626983" rel="noopener noreferrer">管理者権限不要でWindowsにRAMディスクを構築する ― Python環境のI/O高速化手法</a>
</li>
</ul>

<p><strong>If you find this project interesting, a ⭐ on GitHub would be the best way to support my work!</strong></p>

