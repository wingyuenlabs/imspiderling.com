---
Title: Daily DSA and System Design Journal - 15
Description: 
Author: I.K
Date: 2025-10-31T21:31:40.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🧩 Day 15 — Sum of Good Subsequences &amp; CDN Fundamentals
</h2>




<h3>
  
  
  🧠 DSA Problems [1 hr]
</h3>

<p><strong>Problem:</strong> <a href="https://leetcode.com/problems/sum-of-good-subsequences/" rel="noopener noreferrer">3351. Sum of Good Subsequences</a></p>

<h4>
  
  
  💡 Approach: Dynamic Programming on Value Links
</h4>

<h5>
  
  
  🧠 Intuition
</h5>

<p>We define two key state trackers:</p>

<ul>
<li>
<code>count[a]</code> — number of good subsequences ending with <code>a</code>.</li>
<li>
<code>res[a]</code> — total sum of all subsequences ending with <code>a</code>.</li>
</ul>

<p>For each number <code>a</code> in the array:</p>

<ul>
<li>We can <strong>start</strong> a new subsequence with <code>a</code>.</li>
<li>Or <strong>extend</strong> subsequences ending with <code>a - 1</code> and <code>a + 1</code>.</li>
</ul>

<p>Hence,<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>count[a] = count[a-1] + count[a+1] + 1
</code></pre>

</div>



<p>Each of these subsequences contributes additional sums based on their totals and the value <code>a</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>res[a] = res[a-1] + res[a+1] + a * (count[a-1] + count[a+1] + 1)
</code></pre>

</div>



<p>We take modulo <code>10^9 + 7</code> to keep values bounded.<br>
Finally, the total sum of all subsequences is <code>sum(res.values())</code>.</p>




<h4>
  
  
  💻 Code Implementation
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">Solution</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">sumOfGoodSubsequences</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">A</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="nb">int</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">int</span><span class="p">:</span>
        <span class="n">count</span> <span class="o">=</span> <span class="nc">Counter</span><span class="p">()</span>
        <span class="n">res</span> <span class="o">=</span> <span class="nc">Counter</span><span class="p">()</span>
        <span class="n">mod</span> <span class="o">=</span> <span class="mi">10</span> <span class="o">**</span> <span class="mi">9</span> <span class="o">+</span> <span class="mi">7</span>

        <span class="k">for</span> <span class="n">a</span> <span class="ow">in</span> <span class="n">A</span><span class="p">:</span>
            <span class="n">count</span><span class="p">[</span><span class="n">a</span><span class="p">]</span> <span class="o">+=</span> <span class="n">count</span><span class="p">[</span><span class="n">a</span> <span class="o">-</span> <span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="n">count</span><span class="p">[</span><span class="n">a</span> <span class="o">+</span> <span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="mi">1</span>
            <span class="n">count</span><span class="p">[</span><span class="n">a</span><span class="p">]</span> <span class="o">%=</span> <span class="n">mod</span>

            <span class="n">res</span><span class="p">[</span><span class="n">a</span><span class="p">]</span> <span class="o">+=</span> <span class="n">res</span><span class="p">[</span><span class="n">a</span> <span class="o">-</span> <span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="n">res</span><span class="p">[</span><span class="n">a</span> <span class="o">+</span> <span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="n">a</span> <span class="o">*</span> <span class="p">(</span><span class="n">count</span><span class="p">[</span><span class="n">a</span> <span class="o">-</span> <span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="n">count</span><span class="p">[</span><span class="n">a</span> <span class="o">+</span> <span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span>
            <span class="n">res</span><span class="p">[</span><span class="n">a</span><span class="p">]</span> <span class="o">%=</span> <span class="n">mod</span>

        <span class="k">return</span> <span class="nf">sum</span><span class="p">(</span><span class="n">res</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span> <span class="o">%</span> <span class="n">mod</span>
</code></pre>

</div>






<h4>
  
  
  ⚙️ Complexity
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Measure</th>
<th>Complexity</th>
</tr>
</thead>
<tbody>
<tr>
<td>⏱ Time</td>
<td>O(n)</td>
</tr>
<tr>
<td>💾 Space</td>
<td>O(n)</td>
</tr>
</tbody>
</table></div>




<h4>
  
  
  🧩 Key Insight
</h4>

<ul>
<li>The solution uses <strong>local adjacency relationships</strong> (<code>a-1</code>, <code>a+1</code>) to form <strong>global subsequence sums</strong>.</li>
<li>This is a clean example of <strong>DP with aggregation via counters</strong> instead of full DP arrays — space-efficient and value-oriented.</li>
<li>The transitions mimic <em>propagation</em> through neighboring states — a pattern common in DP, graph traversal, and even distributed systems.</li>
</ul>




<h3>
  
  
  🌍 SYSTEM DESIGN — Roadmap.sh [1 hr]
</h3>

<h4>
  
  
  🌐 Content Delivery Networks (CDNs)
</h4>

<p>A <strong>CDN</strong> is a globally distributed network that serves web content from servers <strong>closer to users</strong>, drastically improving latency and scalability.</p>

<p>When you load a site like <code>www.netflix.com</code>, your assets (images, CSS, JS, videos) are often fetched from a nearby CDN edge node — not from Netflix’s origin servers.</p>




<h4>
  
  
  ⚙️ How It Works
</h4>

<ol>
<li>A client requests a resource (e.g., image or video).</li>
<li>DNS resolves the domain to the <strong>nearest CDN edge server</strong>.</li>
<li>If cached, content is served immediately.</li>
<li>If not, the edge server fetches it from the <strong>origin server</strong>, caches it, and serves the user.</li>
</ol>

<p>This <strong>reduces latency</strong>, <strong>distributes load</strong>, and <strong>saves bandwidth</strong> at the origin.</p>




<h4>
  
  
  🧱 Types of CDNs
</h4>

<h5>
  
  
  🔹 Push CDN
</h5>

<ul>
<li>Content is <strong>pushed manually</strong> or via automation to CDN nodes when it changes.</li>
<li>You control cache invalidation and content updates.</li>
<li>Best for sites with <strong>low traffic</strong> or <strong>infrequent updates</strong>.</li>
<li>✅ Pros: Less redundant traffic, predictable storage.</li>
<li>⚠️ Cons: Manual management, risk of stale content.</li>
</ul>

<h5>
  
  
  🔹 Pull CDN
</h5>

<ul>
<li>CDN <strong>fetches content on demand</strong> when requested for the first time.</li>
<li>Cached locally afterward for future users.</li>
<li>Ideal for <strong>high-traffic</strong> or <strong>dynamic</strong> sites.</li>
<li>✅ Pros: Easier setup, automatic caching.</li>
<li>⚠️ Cons: First-request latency, possible redundancy if TTL is too short.</li>
</ul>




<h4>
  
  
  ⏳ TTL (Time To Live)
</h4>

<p>Each cached item has a TTL — after which it expires and must be re-fetched from the origin.<br>
Finding the right TTL is crucial for balancing <strong>freshness vs. efficiency</strong>.</p>




<h4>
  
  
  ⚖️ CDN Trade-offs
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Advantage</th>
<th>Disadvantage</th>
</tr>
</thead>
<tbody>
<tr>
<td>⚡ Faster content delivery</td>
<td>💰 Extra cost based on bandwidth usage</td>
</tr>
<tr>
<td>🧭 Global scalability</td>
<td>🕓 Risk of stale content if updates happen before TTL expires</td>
</tr>
<tr>
<td>🧱 Reduced load on origin servers</td>
<td>🔗 Requires rewriting URLs for CDN routing</td>
</tr>
</tbody>
</table></div>




<h4>
  
  
  🔍 Example Providers
</h4>

<ul>
<li>
<strong>Cloudflare</strong> — strong free tier, DDoS protection</li>
<li>
<strong>AWS CloudFront</strong> — deep AWS integration</li>
<li>
<strong>Akamai</strong>, <strong>Fastly</strong>, <strong>Google Cloud CDN</strong> — high-performance enterprise-grade CDNs</li>
</ul>




<h4>
  
  
  🧠 Reflection
</h4>

<p>Today’s DSA and System Design topics beautifully align:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>DSA Concept</th>
<th>CDN Analogy</th>
</tr>
</thead>
<tbody>
<tr>
<td>Local adjacency propagation</td>
<td>Content caching propagation across edge nodes</td>
</tr>
<tr>
<td>Dynamic aggregation via neighbors</td>
<td>Dynamic content refresh via TTL and caching rules</td>
</tr>
<tr>
<td>O(1) lookups via Counter</td>
<td>Constant-time retrieval via global CDN caches</td>
</tr>
</tbody>
</table></div>

<p>Both rely on <strong>localized computation</strong> to achieve <strong>global efficiency</strong>.</p>




<p>✅ <strong>Day 15 Summary:</strong></p>

<blockquote>
<p>From subsequences to CDNs — efficiency grows when we connect adjacent states and cache intelligently. 🚀</p>
</blockquote>

