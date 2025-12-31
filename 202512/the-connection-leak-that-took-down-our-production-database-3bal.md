---
Title: The Connection Leak That Took Down Our Production Database
Description: 
Author: Ofri Peretz
Date: 2025-12-31T21:35:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>It was 3 AM. PagerDuty woke me up. Our API was returning 500 errors.</p>

<p>The database was fine. CPU was fine. Memory was fine. But every query was timing out.</p>

<h2>
  
  
  The Problem
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FATAL: too many connections for role "app_user"
</code></pre>

</div>



<p>We had exhausted our 100-connection limit. But our traffic was normal. Where were all the connections going?</p>

<h2>
  
  
  The Leak
</h2>

<p>After hours of debugging, we found it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ❌ The connection leak hiding in our codebase</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">getUserOrders</span><span class="p">(</span><span class="nx">userId</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">orders</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">SELECT * FROM orders WHERE user_id = $1</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span>
    <span class="nx">userId</span><span class="p">,</span>
  <span class="p">]);</span>
  <span class="k">return</span> <span class="nx">orders</span><span class="p">.</span><span class="nx">rows</span><span class="p">;</span>
  <span class="c1">// Where's client.release()? 🤔</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Every call leaked a connection. With 50 requests/minute, we exhausted the pool in 2 minutes.</p>

<h2>
  
  
  Why This Happens
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Scenario</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td>Forgot <code>release()</code> entirely</td>
<td>Connection never returned</td>
</tr>
<tr>
<td>Early return before <code>release()</code>
</td>
<td>Connection leaked</td>
</tr>
<tr>
<td>Exception thrown</td>
<td>
<code>finally</code> block missing</td>
</tr>
<tr>
<td>Async error</td>
<td>Unhandled rejection, no cleanup</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  The Correct Pattern
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ✅ Always release in finally block</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">getUserOrders</span><span class="p">(</span><span class="nx">userId</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">orders</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span>
      <span class="dl">'</span><span class="s1">SELECT * FROM orders WHERE user_id = $1</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">userId</span><span class="p">],</span>
    <span class="p">);</span>
    <span class="k">return</span> <span class="nx">orders</span><span class="p">.</span><span class="nx">rows</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
    <span class="nx">client</span><span class="p">.</span><span class="nf">release</span><span class="p">();</span> <span class="c1">// Always executes</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Or even better—don't use <code>connect()</code> at all for simple queries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ✅ Best pattern: use pool.query() directly</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">getUserOrders</span><span class="p">(</span><span class="nx">userId</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">orders</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">SELECT * FROM orders WHERE user_id = $1</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span>
    <span class="nx">userId</span><span class="p">,</span>
  <span class="p">]);</span>
  <span class="k">return</span> <span class="nx">orders</span><span class="p">.</span><span class="nx">rows</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Let ESLint Catch This
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> eslint-plugin-pg
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">pg</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">eslint-plugin-pg</span><span class="dl">'</span><span class="p">;</span>
<span class="k">export</span> <span class="k">default</span> <span class="p">[</span><span class="nx">pg</span><span class="p">.</span><span class="nx">configs</span><span class="p">.</span><span class="nx">recommended</span><span class="p">];</span>
</code></pre>

</div>



<p>Now every missing release is caught:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>src/orders.ts
  3:17  error  🔒 CWE-772 | Missing client.release<span class="o">()</span> detected
               Fix: Add client.release<span class="o">()</span> <span class="k">in </span>finally block or use pool.query<span class="o">()</span> <span class="k">for </span>simple queries
</code></pre>

</div>



<h2>
  
  
  The Rule: <code>no-missing-client-release</code>
</h2>

<p>This rule tracks:</p>

<ul>
<li>Every <code>pool.connect()</code> call</li>
<li>Every code path through the function</li>
<li>Whether <code>client.release()</code> is called on all paths</li>
<li>Whether it's in a <code>finally</code> block (recommended)</li>
</ul>

<h2>
  
  
  Production Impact
</h2>

<p>After deploying this rule:</p>

<ul>
<li>
<strong>0 connection leaks</strong> in 6 months</li>
<li>
<strong>No more 3 AM pages</strong> for connection exhaustion</li>
<li>
<strong>CI catches issues</strong> before they reach staging</li>
</ul>

<h2>
  
  
  Quick Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> eslint-plugin-pg
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">pg</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">eslint-plugin-pg</span><span class="dl">'</span><span class="p">;</span>
<span class="k">export</span> <span class="k">default</span> <span class="p">[</span><span class="nx">pg</span><span class="p">.</span><span class="nx">configs</span><span class="p">.</span><span class="nx">recommended</span><span class="p">];</span>
</code></pre>

</div>



<p>Don't wait for the 3 AM wake-up call.</p>




<p>📦 <a href="https://www.npmjs.com/package/eslint-plugin-pg" rel="noopener noreferrer">npm: eslint-plugin-pg</a><br>
📖 <a href="https://github.com/ofri-peretz/eslint/blob/main/packages/eslint-plugin-pg/docs/rules/no-missing-client-release.md" rel="noopener noreferrer">Rule docs: no-missing-client-release</a></p>

<p><a href="https://github.com/ofri-peretz/eslint" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">⭐ Star on GitHub</a>
</p>




<p>🚀 <strong>Follow me for more security articles &amp; updates:</strong><br>
<a href="https://github.com/ofri-peretz" rel="noopener noreferrer">GitHub</a> | <a href="https://linkedin.com/in/ofri-peretz" rel="noopener noreferrer">LinkedIn</a> | <a href="https://dev.to/ofri-peretz">Dev.to</a></p>

