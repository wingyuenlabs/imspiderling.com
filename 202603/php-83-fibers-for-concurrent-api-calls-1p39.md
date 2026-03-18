---
Title: PHP 8.3 Fibers for Concurrent API Calls
Description: 
Author: ahmet gedik
Date: 2026-03-18T22:00:01.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>When your application needs to call multiple external APIs — like fetching trending videos from 7 different regions — doing it sequentially is slow. PHP 8.1 introduced Fibers, and PHP 8.3 made them production-ready. Here's how I use Fibers on <a href="https://viralvidvault.com" rel="noopener noreferrer">ViralVidVault</a> to fetch data from multiple YouTube API endpoints concurrently.</p>

<h2>
  
  
  The Problem: Sequential API Calls
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// Sequential: ~7 seconds for 7 regions (1s each)</span>
<span class="k">foreach</span> <span class="p">([</span><span class="s1">'US'</span><span class="p">,</span><span class="s1">'GB'</span><span class="p">,</span><span class="s1">'PL'</span><span class="p">,</span><span class="s1">'NL'</span><span class="p">,</span><span class="s1">'SE'</span><span class="p">,</span><span class="s1">'NO'</span><span class="p">,</span><span class="s1">'AT'</span><span class="p">]</span> <span class="k">as</span> <span class="nv">$region</span><span class="p">)</span> <span class="p">{</span>
    <span class="nv">$results</span><span class="p">[</span><span class="nv">$region</span><span class="p">]</span> <span class="o">=</span> <span class="nf">fetchTrending</span><span class="p">(</span><span class="nv">$region</span><span class="p">);</span> <span class="c1">// ~1 second each</span>
<span class="p">}</span>
<span class="c1">// Total: 7 * 1s = ~7 seconds</span>
</code></pre>

</div>



<h2>
  
  
  The Solution: Fibers + curl_multi
</h2>

<p>Fibers don't make I/O faster by themselves — they provide cooperative multitasking. Combined with <code>curl_multi</code>, they let you run multiple HTTP requests in parallel:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="kd">class</span> <span class="nc">ConcurrentFetcher</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">array</span> <span class="nv">$fibers</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="k">private</span> <span class="nc">\CurlMultiHandle</span> <span class="nv">$multiHandle</span><span class="p">;</span>
    <span class="k">private</span> <span class="kt">array</span> <span class="nv">$handles</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">multiHandle</span> <span class="o">=</span> <span class="nb">curl_multi_init</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">addRequest</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$key</span><span class="p">,</span> <span class="kt">string</span> <span class="nv">$url</span><span class="p">):</span> <span class="kt">void</span>
    <span class="p">{</span>
        <span class="nv">$ch</span> <span class="o">=</span> <span class="nb">curl_init</span><span class="p">();</span>
        <span class="nb">curl_setopt_array</span><span class="p">(</span><span class="nv">$ch</span><span class="p">,</span> <span class="p">[</span>
            <span class="no">CURLOPT_URL</span> <span class="o">=&gt;</span> <span class="nv">$url</span><span class="p">,</span>
            <span class="no">CURLOPT_RETURNTRANSFER</span> <span class="o">=&gt;</span> <span class="kc">true</span><span class="p">,</span>
            <span class="no">CURLOPT_TIMEOUT</span> <span class="o">=&gt;</span> <span class="mi">10</span><span class="p">,</span>
            <span class="no">CURLOPT_HTTPHEADER</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="s1">'Accept: application/json'</span><span class="p">],</span>
        <span class="p">]);</span>

        <span class="nb">curl_multi_add_handle</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">multiHandle</span><span class="p">,</span> <span class="nv">$ch</span><span class="p">);</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">handles</span><span class="p">[</span><span class="nv">$key</span><span class="p">]</span> <span class="o">=</span> <span class="nv">$ch</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">execute</span><span class="p">():</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="nv">$results</span> <span class="o">=</span> <span class="p">[];</span>
        <span class="nv">$running</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

        <span class="c1">// Execute all requests concurrently</span>
        <span class="k">do</span> <span class="p">{</span>
            <span class="nv">$status</span> <span class="o">=</span> <span class="nb">curl_multi_exec</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">multiHandle</span><span class="p">,</span> <span class="nv">$running</span><span class="p">);</span>
            <span class="k">if</span> <span class="p">(</span><span class="nv">$running</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
                <span class="nb">curl_multi_select</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">multiHandle</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">);</span>
            <span class="p">}</span>
        <span class="p">}</span> <span class="k">while</span> <span class="p">(</span><span class="nv">$running</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nv">$status</span> <span class="o">===</span> <span class="no">CURLM_OK</span><span class="p">);</span>

        <span class="c1">// Collect results</span>
        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">handles</span> <span class="k">as</span> <span class="nv">$key</span> <span class="o">=&gt;</span> <span class="nv">$ch</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$content</span> <span class="o">=</span> <span class="nb">curl_multi_getcontent</span><span class="p">(</span><span class="nv">$ch</span><span class="p">);</span>
            <span class="nv">$httpCode</span> <span class="o">=</span> <span class="nb">curl_getinfo</span><span class="p">(</span><span class="nv">$ch</span><span class="p">,</span> <span class="no">CURLINFO_HTTP_CODE</span><span class="p">);</span>

            <span class="nv">$results</span><span class="p">[</span><span class="nv">$key</span><span class="p">]</span> <span class="o">=</span> <span class="p">[</span>
                <span class="s1">'data'</span> <span class="o">=&gt;</span> <span class="nv">$httpCode</span> <span class="o">===</span> <span class="mi">200</span> <span class="o">?</span> <span class="nb">json_decode</span><span class="p">(</span><span class="nv">$content</span><span class="p">,</span> <span class="kc">true</span><span class="p">)</span> <span class="o">:</span> <span class="kc">null</span><span class="p">,</span>
                <span class="s1">'http_code'</span> <span class="o">=&gt;</span> <span class="nv">$httpCode</span><span class="p">,</span>
                <span class="s1">'time'</span> <span class="o">=&gt;</span> <span class="nb">curl_getinfo</span><span class="p">(</span><span class="nv">$ch</span><span class="p">,</span> <span class="no">CURLINFO_TOTAL_TIME</span><span class="p">),</span>
            <span class="p">];</span>

            <span class="nb">curl_multi_remove_handle</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">multiHandle</span><span class="p">,</span> <span class="nv">$ch</span><span class="p">);</span>
            <span class="nb">curl_close</span><span class="p">(</span><span class="nv">$ch</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="nb">curl_multi_close</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">multiHandle</span><span class="p">);</span>
        <span class="k">return</span> <span class="nv">$results</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Using It for Multi-Region Fetching
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="nv">$apiKey</span> <span class="o">=</span> <span class="nv">$_ENV</span><span class="p">[</span><span class="s1">'YOUTUBE_API_KEY'</span><span class="p">];</span>
<span class="nv">$regions</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'US'</span><span class="p">,</span> <span class="s1">'GB'</span><span class="p">,</span> <span class="s1">'PL'</span><span class="p">,</span> <span class="s1">'NL'</span><span class="p">,</span> <span class="s1">'SE'</span><span class="p">,</span> <span class="s1">'NO'</span><span class="p">,</span> <span class="s1">'AT'</span><span class="p">];</span>

<span class="nv">$fetcher</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ConcurrentFetcher</span><span class="p">();</span>

<span class="k">foreach</span> <span class="p">(</span><span class="nv">$regions</span> <span class="k">as</span> <span class="nv">$region</span><span class="p">)</span> <span class="p">{</span>
    <span class="nv">$url</span> <span class="o">=</span> <span class="s1">'https://www.googleapis.com/youtube/v3/videos?'</span> <span class="mf">.</span> <span class="nb">http_build_query</span><span class="p">([</span>
        <span class="s1">'part'</span> <span class="o">=&gt;</span> <span class="s1">'snippet,statistics'</span><span class="p">,</span>
        <span class="s1">'chart'</span> <span class="o">=&gt;</span> <span class="s1">'mostPopular'</span><span class="p">,</span>
        <span class="s1">'regionCode'</span> <span class="o">=&gt;</span> <span class="nv">$region</span><span class="p">,</span>
        <span class="s1">'maxResults'</span> <span class="o">=&gt;</span> <span class="mi">25</span><span class="p">,</span>
        <span class="s1">'key'</span> <span class="o">=&gt;</span> <span class="nv">$apiKey</span><span class="p">,</span>
    <span class="p">]);</span>

    <span class="nv">$fetcher</span><span class="o">-&gt;</span><span class="nf">addRequest</span><span class="p">(</span><span class="nv">$region</span><span class="p">,</span> <span class="nv">$url</span><span class="p">);</span>
<span class="p">}</span>

<span class="nv">$startTime</span> <span class="o">=</span> <span class="nb">microtime</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
<span class="nv">$results</span> <span class="o">=</span> <span class="nv">$fetcher</span><span class="o">-&gt;</span><span class="nf">execute</span><span class="p">();</span>
<span class="nv">$elapsed</span> <span class="o">=</span> <span class="nb">round</span><span class="p">(</span><span class="nb">microtime</span><span class="p">(</span><span class="kc">true</span><span class="p">)</span> <span class="o">-</span> <span class="nv">$startTime</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>

<span class="k">echo</span> <span class="s2">"Fetched "</span> <span class="mf">.</span> <span class="nb">count</span><span class="p">(</span><span class="nv">$regions</span><span class="p">)</span> <span class="mf">.</span> <span class="s2">" regions in </span><span class="si">{</span><span class="nv">$elapsed</span><span class="si">}</span><span class="s2">s</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span>

<span class="k">foreach</span> <span class="p">(</span><span class="nv">$results</span> <span class="k">as</span> <span class="nv">$region</span> <span class="o">=&gt;</span> <span class="nv">$result</span><span class="p">)</span> <span class="p">{</span>
    <span class="nv">$count</span> <span class="o">=</span> <span class="nb">count</span><span class="p">(</span><span class="nv">$result</span><span class="p">[</span><span class="s1">'data'</span><span class="p">][</span><span class="s1">'items'</span><span class="p">]</span> <span class="o">??</span> <span class="p">[]);</span>
    <span class="k">echo</span> <span class="s2">"[</span><span class="si">{</span><span class="nv">$region</span><span class="si">}</span><span class="s2">] </span><span class="si">{</span><span class="nv">$count</span><span class="si">}</span><span class="s2"> videos (</span><span class="si">{</span><span class="nv">$result</span><span class="p">[</span><span class="s1">'time'</span><span class="p">]</span><span class="si">}</span><span class="s2">s)</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Output
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Fetched 7 regions in 1.23s
[US] 25 videos (0.89s)
[GB] 25 videos (0.95s)
[PL] 25 videos (1.02s)
[NL] 25 videos (0.91s)
[SE] 25 videos (0.98s)
[NO] 25 videos (1.12s)
[AT] 25 videos (1.01s)
</code></pre>

</div>



<p>All 7 regions fetched in 1.23 seconds instead of ~7 seconds sequentially. The total time equals the slowest individual request, not the sum.</p>

<h2>
  
  
  Adding Fibers for Structured Concurrency
</h2>

<p>For more complex scenarios where you need to process results as they arrive:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="kd">class</span> <span class="nc">FiberPool</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">array</span> <span class="nv">$fibers</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="k">private</span> <span class="kt">array</span> <span class="nv">$results</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">add</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$key</span><span class="p">,</span> <span class="kt">callable</span> <span class="nv">$task</span><span class="p">):</span> <span class="kt">void</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">fibers</span><span class="p">[</span><span class="nv">$key</span><span class="p">]</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">\Fiber</span><span class="p">(</span><span class="k">function</span> <span class="p">()</span> <span class="k">use</span> <span class="p">(</span><span class="nv">$key</span><span class="p">,</span> <span class="nv">$task</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$result</span> <span class="o">=</span> <span class="nv">$task</span><span class="p">();</span>
            <span class="nc">\Fiber</span><span class="o">::</span><span class="nf">suspend</span><span class="p">(</span><span class="nv">$result</span><span class="p">);</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">run</span><span class="p">():</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="c1">// Start all fibers</span>
        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">fibers</span> <span class="k">as</span> <span class="nv">$key</span> <span class="o">=&gt;</span> <span class="nv">$fiber</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$result</span> <span class="o">=</span> <span class="nv">$fiber</span><span class="o">-&gt;</span><span class="nf">start</span><span class="p">();</span>
            <span class="k">if</span> <span class="p">(</span><span class="nv">$fiber</span><span class="o">-&gt;</span><span class="nf">isSuspended</span><span class="p">())</span> <span class="p">{</span>
                <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">results</span><span class="p">[</span><span class="nv">$key</span><span class="p">]</span> <span class="o">=</span> <span class="nv">$result</span><span class="p">;</span>
            <span class="p">}</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">results</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="nv">$pool</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">FiberPool</span><span class="p">();</span>

<span class="nv">$pool</span><span class="o">-&gt;</span><span class="nf">add</span><span class="p">(</span><span class="s1">'trending_PL'</span><span class="p">,</span> <span class="k">fn</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">fetchAndProcess</span><span class="p">(</span><span class="s1">'PL'</span><span class="p">));</span>
<span class="nv">$pool</span><span class="o">-&gt;</span><span class="nf">add</span><span class="p">(</span><span class="s1">'trending_NL'</span><span class="p">,</span> <span class="k">fn</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">fetchAndProcess</span><span class="p">(</span><span class="s1">'NL'</span><span class="p">));</span>
<span class="nv">$pool</span><span class="o">-&gt;</span><span class="nf">add</span><span class="p">(</span><span class="s1">'trending_SE'</span><span class="p">,</span> <span class="k">fn</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">fetchAndProcess</span><span class="p">(</span><span class="s1">'SE'</span><span class="p">));</span>

<span class="nv">$results</span> <span class="o">=</span> <span class="nv">$pool</span><span class="o">-&gt;</span><span class="nf">run</span><span class="p">();</span>
</code></pre>

</div>



<h2>
  
  
  When to Use Fibers vs curl_multi
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Approach</th>
<th>Best For</th>
</tr>
</thead>
<tbody>
<tr>
<td>
<code>curl_multi</code> alone</td>
<td>Pure HTTP parallelism</td>
</tr>
<tr>
<td>Fibers + curl_multi</td>
<td>Complex async workflows</td>
</tr>
<tr>
<td>Fibers alone</td>
<td>CPU-bound task switching</td>
</tr>
</tbody>
</table></div>

<p>For <a href="https://viralvidvault.com" rel="noopener noreferrer">ViralVidVault</a>, <code>curl_multi</code> is sufficient since our concurrent work is purely HTTP-based. Fibers become valuable when you need to interleave processing between requests.</p>

<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li>
<code>curl_multi</code> gives you parallel HTTP without Fibers</li>
<li>Fibers add structured concurrency for complex workflows</li>
<li>Total time = slowest request, not sum of all requests</li>
<li>Always set <code>CURLOPT_TIMEOUT</code> to prevent hanging fibers</li>
<li>PHP 8.3 makes this pattern production-ready</li>
</ol>

<p>This concurrent fetching approach is how <a href="https://viralvidvault.com" rel="noopener noreferrer">viralvidvault.com</a> keeps its 7-region data fresh without spending 7x the cron time.</p>




<p><em>Part of the "Building ViralVidVault" series.</em></p>

