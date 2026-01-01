---
Title: Transaction Race Conditions: Why BEGIN on Pool Breaks Everything
Description: 
Author: Ofri Peretz
Date: 2025-12-31T21:38:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>This code looks correct. It passes all tests. It works in development.</p>

<p>In production with 100 concurrent users, it corrupts data.</p>

<h2>
  
  
  The Bug
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ❌ Dangerous: Transaction on pool</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">transferFunds</span><span class="p">(</span><span class="k">from</span><span class="p">,</span> <span class="nx">to</span><span class="p">,</span> <span class="nx">amount</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">BEGIN</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">UPDATE accounts SET balance = balance - $1 WHERE id = $2</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span>
    <span class="nx">amount</span><span class="p">,</span>
    <span class="k">from</span><span class="p">,</span>
  <span class="p">]);</span>
  <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">UPDATE accounts SET balance = balance + $1 WHERE id = $2</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span>
    <span class="nx">amount</span><span class="p">,</span>
    <span class="nx">to</span><span class="p">,</span>
  <span class="p">]);</span>
  <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">COMMIT</span><span class="dl">'</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Why It Fails
</h2>

<p>A PostgreSQL <strong>pool</strong> is a set of client connections. Each <code>pool.query()</code> can use a <strong>different client</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Request 1: pool.query('BEGIN')     → Client A
Request 1: pool.query('UPDATE...')  → Client B (different!)
Request 2: pool.query('BEGIN')     → Client A (reused!)
</code></pre>

</div>



<p>Your transaction is now spread across multiple clients. Your data is now inconsistent.</p>

<h2>
  
  
  The Correct Pattern
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ✅ Safe: Get dedicated client, use it for entire transaction</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">transferFunds</span><span class="p">(</span><span class="k">from</span><span class="p">,</span> <span class="nx">to</span><span class="p">,</span> <span class="nx">amount</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">BEGIN</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span>
      <span class="dl">'</span><span class="s1">UPDATE accounts SET balance = balance - $1 WHERE id = $2</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">amount</span><span class="p">,</span> <span class="k">from</span><span class="p">],</span>
    <span class="p">);</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span>
      <span class="dl">'</span><span class="s1">UPDATE accounts SET balance = balance + $1 WHERE id = $2</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">[</span><span class="nx">amount</span><span class="p">,</span> <span class="nx">to</span><span class="p">],</span>
    <span class="p">);</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">COMMIT</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">ROLLBACK</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">throw</span> <span class="nx">e</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
    <span class="nx">client</span><span class="p">.</span><span class="nf">release</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Same client</strong> for <code>BEGIN</code>, all queries, and <code>COMMIT</code>. Transaction integrity guaranteed.</p>

<h2>
  
  
  The Rule
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ❌ pool.query('BEGIN')      → Error</span>
<span class="c1">// ❌ pool.query('COMMIT')     → Error</span>
<span class="c1">// ❌ pool.query('ROLLBACK')   → Error</span>
<span class="c1">// ❌ pool.query('SAVEPOINT')  → Error</span>

<span class="c1">// ✅ client.query('BEGIN')    → OK</span>
<span class="c1">// ✅ pool.query('SELECT...')  → OK (no transaction)</span>
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



<p>The <code>no-transaction-on-pool</code> rule catches every case:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>src/transfer.ts
  3:9  error  🔒 CWE-362 | Transaction <span class="nb">command </span>on pool - use pool.connect<span class="o">()</span> <span class="k">for </span>transactions
               Fix: const client <span class="o">=</span> await pool.connect<span class="o">()</span><span class="p">;</span> client.query<span class="o">(</span><span class="s1">'BEGIN'</span><span class="o">)</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  Helper Function Pattern
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ✅ Reusable transaction wrapper</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">withTransaction</span><span class="p">(</span><span class="nx">callback</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">BEGIN</span><span class="dl">'</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">callback</span><span class="p">(</span><span class="nx">client</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">COMMIT</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">result</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">ROLLBACK</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">throw</span> <span class="nx">e</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
    <span class="nx">client</span><span class="p">.</span><span class="nf">release</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="k">await</span> <span class="nf">withTransaction</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">client</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">UPDATE accounts SET...</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span><span class="nx">amount</span><span class="p">,</span> <span class="k">from</span><span class="p">]);</span>
  <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">UPDATE accounts SET...</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span><span class="nx">amount</span><span class="p">,</span> <span class="nx">to</span><span class="p">]);</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  When To Use What
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Scenario</th>
<th>Use</th>
</tr>
</thead>
<tbody>
<tr>
<td>Single query</td>
<td><code>pool.query()</code></td>
</tr>
<tr>
<td>Multiple independent queries</td>
<td><code>pool.query()</code></td>
</tr>
<tr>
<td>Transaction (BEGIN/COMMIT)</td>
<td>
<code>pool.connect()</code> → <code>client.query()</code>
</td>
</tr>
<tr>
<td>Long-running session</td>
<td>
<code>pool.connect()</code> → <code>client.query()</code>
</td>
</tr>
</tbody>
</table></div>

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



<p>Don't let race conditions corrupt your data.</p>




<p>📦 <a href="https://www.npmjs.com/package/eslint-plugin-pg" rel="noopener noreferrer">npm: eslint-plugin-pg</a><br>
📖 <a href="https://github.com/ofri-peretz/eslint/blob/main/packages/eslint-plugin-pg/docs/rules/no-transaction-on-pool.md" rel="noopener noreferrer">Rule docs: no-transaction-on-pool</a></p>

<p><a href="https://github.com/ofri-peretz/eslint" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">⭐ Star on GitHub</a>
</p>




<p>🚀 <strong>Follow me for more security articles &amp; updates:</strong><br>
<a href="https://github.com/ofri-peretz" rel="noopener noreferrer">GitHub</a> | <a href="https://linkedin.com/in/ofri-peretz" rel="noopener noreferrer">LinkedIn</a> | <a href="https://dev.to/ofri-peretz">Dev.to</a></p>

