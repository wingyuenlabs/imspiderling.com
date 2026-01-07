---
Title: Building a Fast File Transfer Tool, Part 2: Beating rsync by 58% with kTLS
Description: 
Author: Vincent Du
Date: 2026-01-07T21:19:00.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building a Fast File Transfer Tool, Part 2: Beating rsync by 58% with kTLS
</h1>

<p>In <a href="https://dev.to/vincentdu2021/building-a-file-copier-4x-faster-than-cp-using-iouring-4b5n">Part 1</a>, I built <strong>uring-sync</strong>—a file copier that's 4.2x faster than <code>cp</code> for local copies using io_uring. Now I've added <strong>network transfer</strong> with kernel TLS encryption, achieving <strong>58% faster transfers than rsync</strong>.</p>

<h2>
  
  
  The Problem: SSH is the Bottleneck
</h2>

<p>When transferring ML datasets between machines, rsync over SSH is the go-to tool:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>rsync <span class="nt">-az</span> /data/ml_dataset user@server:/backup/
</code></pre>

</div>



<p>It works, but it's slow. For a 9.7GB dataset (100K files), rsync took <strong>390 seconds</strong>—a throughput of just 25 MB/s.</p>

<p>The bottleneck isn't the network. It's <strong>encryption in userspace</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  File   │────▶│ rsync   │────▶│  SSH    │────▶│ Network │
│  Read   │     │ (delta) │     │ encrypt │     │  Send   │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                                     │
                              Context switches,
                              userspace copies,
                              CPU-bound AES
</code></pre>

</div>



<p>Every byte passes through the SSH process, which encrypts it using OpenSSL in userspace. This involves:</p>

<ul>
<li>Multiple context switches between kernel and userspace</li>
<li>Copying data between kernel buffers and userspace buffers</li>
<li>CPU time for AES encryption (even with AES-NI)</li>
</ul>

<h2>
  
  
  The Solution: kTLS (Kernel TLS)
</h2>

<p>Linux 4.13+ supports <strong>kTLS</strong>—TLS encryption handled directly in the kernel. Once you set up the TLS session, the kernel encrypts data as it flows through the socket.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────┐     ┌─────────┐     ┌──────────────────┐
│  File   │────▶│  read   │────▶│ Socket (kTLS)    │
│         │     │         │     │ encrypt + send   │
└─────────┘     └─────────┘     └──────────────────┘
                                        │
                                 One kernel operation,
                                 no userspace copies,
                                 AES-NI in kernel
</code></pre>

</div>



<p>Benefits:</p>

<ol>
<li>
<strong>No userspace encryption process</strong> - kernel handles it directly</li>
<li>
<strong>Fewer copies</strong> - data doesn't bounce through userspace</li>
<li>
<strong>AES-NI in kernel</strong> - hardware acceleration without context switches</li>
</ol>

<h2>
  
  
  Implementation
</h2>

<p>Setting up kTLS requires:</p>

<ol>
<li>
<strong>TLS handshake</strong> - Exchange keys (we use a pre-shared secret + HKDF)</li>
<li>
<strong>Configure kernel</strong> - <code>setsockopt(SOL_TLS, TLS_TX, ...)</code> with cipher keys</li>
<li>
<strong>Send data</strong> - Regular <code>send()</code> calls, kernel encrypts automatically
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// After deriving keys from shared secret...</span>
<span class="k">struct</span> <span class="nc">tls12_crypto_info_aes_gcm_128</span> <span class="n">crypto_info</span> <span class="o">=</span> <span class="p">{</span>
    <span class="p">.</span><span class="n">info</span><span class="p">.</span><span class="n">version</span> <span class="o">=</span> <span class="n">TLS_1_2_VERSION</span><span class="p">,</span>
    <span class="p">.</span><span class="n">info</span><span class="p">.</span><span class="n">cipher_type</span> <span class="o">=</span> <span class="n">TLS_CIPHER_AES_GCM_128</span><span class="p">,</span>
<span class="p">};</span>
<span class="n">memcpy</span><span class="p">(</span><span class="n">crypto_info</span><span class="p">.</span><span class="n">key</span><span class="p">,</span> <span class="n">key</span><span class="p">,</span> <span class="mi">16</span><span class="p">);</span>
<span class="n">memcpy</span><span class="p">(</span><span class="n">crypto_info</span><span class="p">.</span><span class="n">iv</span><span class="p">,</span> <span class="n">iv</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>
<span class="n">memcpy</span><span class="p">(</span><span class="n">crypto_info</span><span class="p">.</span><span class="n">salt</span><span class="p">,</span> <span class="n">salt</span><span class="p">,</span> <span class="mi">4</span><span class="p">);</span>

<span class="n">setsockopt</span><span class="p">(</span><span class="n">sock</span><span class="p">,</span> <span class="n">SOL_TLS</span><span class="p">,</span> <span class="n">TLS_TX</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">crypto_info</span><span class="p">,</span> <span class="k">sizeof</span><span class="p">(</span><span class="n">crypto_info</span><span class="p">));</span>
<span class="c1">// Now all send() calls are automatically encrypted!</span>
</code></pre>

</div>



<h2>
  
  
  Benchmark Results
</h2>

<p>Testing on real network: Laptop → GCP VM (public internet)</p>

<h3>
  
  
  The Headline Number
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Dataset</th>
<th>uring-sync + kTLS</th>
<th>rsync (SSH)</th>
<th>Improvement</th>
</tr>
</thead>
<tbody>
<tr>
<td>ml_small (60MB, 10K files)</td>
<td>2.98s</td>
<td>2.63s</td>
<td>~equal</td>
</tr>
<tr>
<td>ml_large (589MB, 100K files)</td>
<td><strong>16.4s</strong></td>
<td>24.8s</td>
<td><strong>34% faster</strong></td>
</tr>
<tr>
<td>ml_images (9.7GB, 100K files)</td>
<td><strong>165s</strong></td>
<td>390s</td>
<td><strong>58% faster</strong></td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  The Pattern
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Data size:    60MB  →  589MB  →   9.7GB
Improvement:   0%   →   34%   →    58%
</code></pre>

</div>



<p><strong>The larger the transfer, the bigger the kTLS advantage.</strong></p>

<p>Why? Per-connection overhead (handshake, key derivation) is amortized over more data. And SSH's userspace encryption overhead grows linearly with data size.</p>

<h3>
  
  
  Throughput Comparison
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Throughput</th>
<th>CPU Usage</th>
</tr>
</thead>
<tbody>
<tr>
<td>rsync (SSH)</td>
<td>25 MB/s</td>
<td>High (userspace encryption)</td>
</tr>
<tr>
<td>uring-sync + kTLS</td>
<td><strong>60 MB/s</strong></td>
<td>Low (kernel encryption)</td>
</tr>
</tbody>
</table></div>

<p>kTLS achieves <strong>2.4x the throughput</strong> of rsync while using less CPU.</p>

<h2>
  
  
  Why Not Zero-Copy Splice?
</h2>

<p>In theory, kTLS supports splice() for true zero-copy transfers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>File → Pipe → kTLS Socket (no userspace copies!)
</code></pre>

</div>



<p>I implemented this and expected it to be fastest. Instead, it was <strong>2.9x slower</strong>.</p>

<h3>
  
  
  The Investigation
</h3>

<p>Using strace, I found the problem:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>splice(file→pipe):   27μs    ← instant
splice(pipe→socket): 33ms    ← 1000x slower!
</code></pre>

</div>



<p>The <code>splice(pipe → kTLS socket)</code> call <strong>blocks</strong> waiting for TCP ACKs. The kernel can't buffer aggressively like it does with regular <code>send()</code> calls.</p>

<h3>
  
  
  The Lesson
</h3>

<p>Zero-copy isn't always faster. For many-file workloads:</p>

<ul>
<li>
<strong>read/send</strong>: Kernel manages buffering efficiently</li>
<li>
<strong>splice</strong>: Blocks on each chunk, killing throughput</li>
</ul>

<p>Splice might help for single huge files, but for ML datasets (many small files), stick with read/send.</p>

<h2>
  
  
  When to Use This
</h2>

<p><strong>Use kTLS file transfer when:</strong></p>

<ul>
<li>Transferring large datasets (&gt;500MB)</li>
<li>Network has bandwidth to spare</li>
<li>You control both endpoints</li>
<li>Security is required (not just over VPN)</li>
</ul>

<p><strong>Stick with rsync when:</strong></p>

<ul>
<li>You need delta sync (only changed bytes)</li>
<li>Destination already has partial data</li>
<li>SSH infrastructure is already in place</li>
<li>Simplicity matters more than speed</li>
</ul>

<h2>
  
  
  The Protocol
</h2>

<p>Our wire protocol is minimal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>HELLO (secret hash) ──────────────────▶ Verify
                    ◀────────────────── HELLO_OK (+ enable kTLS)

FILE_HDR (path, size, mode) ──────────▶ Create file
FILE_DATA (chunks) ────────────────────▶ Write data
FILE_END ──────────────────────────────▶ Close file

(repeat for all files)

ALL_DONE ──────────────────────────────▶ Complete
</code></pre>

</div>



<p>No delta encoding, no checksums (kTLS provides integrity via GCM). Just raw file transfer with authentication and encryption.</p>

<h2>
  
  
  Code
</h2>

<p>Usage:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Receiver (on remote host)</span>
uring-sync recv /backup <span class="nt">--listen</span> 9999 <span class="nt">--secret</span> mykey <span class="nt">--tls</span>

<span class="c"># Sender (on local host)</span>
uring-sync send /data remote-host:9999 <span class="nt">--secret</span> mykey <span class="nt">--tls</span>
</code></pre>

</div>



<p>The implementation uses:</p>

<ul>
<li>HKDF for key derivation from shared secret</li>
<li>AES-128-GCM via kTLS</li>
<li>Simple TCP protocol (no HTTP, no gRPC)</li>
</ul>

<p>Full source: <a href="https://github.com/VincentDu2021/uring_sync" rel="noopener noreferrer">github.com/VincentDu2021/uring_sync</a></p>

<h2>
  
  
  Conclusion
</h2>

<p>By moving encryption from userspace SSH to kernel kTLS, we achieved:</p>

<ul>
<li>
<strong>58% faster</strong> than rsync for large transfers</li>
<li>
<strong>2.4x throughput</strong> (60 MB/s vs 25 MB/s)</li>
<li>
<strong>Lower CPU usage</strong> (kernel AES-NI vs userspace OpenSSL)</li>
</ul>

<p>The key insight: for bulk data transfer, SSH's flexibility is overhead. A purpose-built tool with kernel encryption wins.</p>




<h2>
  
  
  Appendix: Full Benchmark Data
</h2>

<h3>
  
  
  Test Environment
</h3>

<ul>
<li>Sender: Ubuntu laptop, local NVMe</li>
<li>Receiver: GCP VM (us-central1-a)</li>
<li>Network: Public internet</li>
<li>All tests with cold cache (<code>echo 3 &gt; /proc/sys/vm/drop_caches</code>)</li>
</ul>

<h3>
  
  
  Raw Results
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Dataset</th>
<th>Files</th>
<th>Size</th>
<th>kTLS Time</th>
<th>kTLS Speed</th>
<th>rsync Time</th>
<th>rsync Speed</th>
</tr>
</thead>
<tbody>
<tr>
<td>ml_small</td>
<td>10K</td>
<td>60MB</td>
<td>2.98s</td>
<td>20 MB/s</td>
<td>2.63s</td>
<td>23 MB/s</td>
</tr>
<tr>
<td>ml_large</td>
<td>100K</td>
<td>589MB</td>
<td>16.4s</td>
<td>36 MB/s</td>
<td>24.8s</td>
<td>24 MB/s</td>
</tr>
<tr>
<td>ml_images</td>
<td>100K</td>
<td>9.7GB</td>
<td>165s</td>
<td>60 MB/s</td>
<td>390s</td>
<td>25 MB/s</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Splice Investigation (ml_images)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Mode</th>
<th>Time</th>
<th>Speed</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>Plaintext + read/send</td>
<td>146s</td>
<td>68 MB/s</td>
<td>Fastest (no encryption)</td>
</tr>
<tr>
<td>Plaintext + splice</td>
<td>157s</td>
<td>63 MB/s</td>
<td>+8% overhead</td>
</tr>
<tr>
<td>kTLS + read/send</td>
<td>165s</td>
<td>60 MB/s</td>
<td>+13% (encryption cost)</td>
</tr>
<tr>
<td>kTLS + splice</td>
<td>428s</td>
<td>23 MB/s</td>
<td>2.9x slower (broken)</td>
</tr>
</tbody>
</table></div>




<p><em>Benchmarks run January 2026. Your mileage may vary depending on network conditions and hardware.</em></p>




<p><strong>Tags:</strong> #linux #ktls #tls #rsync #performance #networking #encryption</p>

