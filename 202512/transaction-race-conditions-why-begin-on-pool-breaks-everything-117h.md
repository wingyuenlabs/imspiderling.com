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
<span class="k">async</span> <span class="kd">function</span> <span class="nf">withTransaction</span><span class="