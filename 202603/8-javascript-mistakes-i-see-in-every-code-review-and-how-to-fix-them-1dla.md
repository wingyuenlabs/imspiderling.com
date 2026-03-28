---
Title: 8 JavaScript Mistakes I See in Every Code Review (And How to Fix Them)
Description: 
Author: Lucas M Dev
Date: 2026-03-28T21:51:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>After reviewing hundreds of PRs, these are the patterns that keep coming up. Let's fix them once and for all.</p>

<h2>
  
  
  1. Using == instead of ===
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - type coercion is unpredictable</span>
<span class="k">if </span><span class="p">(</span><span class="nx">user</span><span class="p">.</span><span class="nx">age</span> <span class="o">==</span> <span class="dl">"</span><span class="s2">18</span><span class="dl">"</span><span class="p">)</span> <span class="p">...</span>
<span class="k">if </span><span class="p">(</span><span class="nx">count</span> <span class="o">==</span> <span class="kc">null</span><span class="p">)</span> <span class="p">...</span>
<span class="k">if </span><span class="p">(</span><span class="mi">0</span> <span class="o">==</span> <span class="kc">false</span><span class="p">)</span> <span class="p">...</span>   <span class="c1">// true!</span>
<span class="k">if </span><span class="p">(</span><span class="dl">""</span> <span class="o">==</span> <span class="kc">false</span><span class="p">)</span> <span class="p">...</span>  <span class="c1">// true!</span>

<span class="c1">// ✅ Correct - strict equality, no surprises</span>
<span class="k">if </span><span class="p">(</span><span class="nx">user</span><span class="p">.</span><span class="nx">age</span> <span class="o">===</span> <span class="mi">18</span><span class="p">)</span> <span class="p">...</span>
<span class="k">if </span><span class="p">(</span><span class="nx">count</span> <span class="o">===</span> <span class="kc">null</span> <span class="o">||</span> <span class="nx">count</span> <span class="o">===</span> <span class="kc">undefined</span><span class="p">)</span> <span class="p">...</span>
<span class="c1">// Or better:</span>
<span class="k">if </span><span class="p">(</span><span class="nx">count</span> <span class="o">==</span> <span class="kc">null</span><span class="p">)</span> <span class="p">...</span>  <span class="c1">// Only acceptable for null/undefined check</span>
</code></pre>

</div>



<p><strong>Exception</strong>: <code>== null</code> is fine when checking for both null AND undefined.</p>

<h2>
  
  
  2. Mutating function parameters
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - mutates the caller's object</span>
<span class="kd">function</span> <span class="nf">addTimestamp</span><span class="p">(</span><span class="nx">user</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">user</span><span class="p">.</span><span class="nx">createdAt</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">();</span>
  <span class="k">return</span> <span class="nx">user</span><span class="p">;</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">admin</span> <span class="o">=</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Alice</span><span class="dl">'</span><span class="p">,</span> <span class="na">role</span><span class="p">:</span> <span class="dl">'</span><span class="s1">admin</span><span class="dl">'</span> <span class="p">};</span>
<span class="kd">const</span> <span class="nx">timestamped</span> <span class="o">=</span> <span class="nf">addTimestamp</span><span class="p">(</span><span class="nx">admin</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">admin</span><span class="p">.</span><span class="nx">createdAt</span><span class="p">);</span> <span class="c1">// Oops! admin was mutated too</span>

<span class="c1">// ✅ Correct - return a new object</span>
<span class="kd">function</span> <span class="nf">addTimestamp</span><span class="p">(</span><span class="nx">user</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span> <span class="p">...</span><span class="nx">user</span><span class="p">,</span> <span class="na">createdAt</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">()</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  3. Not handling async errors
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - crashes the whole app on network error</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchUser</span><span class="p">(</span><span class="nx">id</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`/api/users/</span><span class="p">${</span><span class="nx">id</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
<span class="p">}</span>

<span class="c1">// ✅ Correct - handle errors where they happen</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchUser</span><span class="p">(</span><span class="nx">id</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`/api/users/</span><span class="p">${</span><span class="nx">id</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`HTTP </span><span class="p">${</span><span class="nx">response</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">(),</span> <span class="na">error</span><span class="p">:</span> <span class="kc">null</span> <span class="p">};</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span> <span class="na">error</span><span class="p">:</span> <span class="nx">err</span><span class="p">.</span><span class="nx">message</span> <span class="p">};</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span><span class="p">,</span> <span class="nx">error</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchUser</span><span class="p">(</span><span class="mi">42</span><span class="p">);</span>
<span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span> <span class="cm">/* handle */</span> <span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  4. Creating functions inside loops
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - creates N closures, all referencing the same `i`</span>
<span class="kd">const</span> <span class="nx">handlers</span> <span class="o">=</span> <span class="p">[];</span>
<span class="k">for </span><span class="p">(</span><span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">5</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">handlers</span><span class="p">.</span><span class="nf">push</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">i</span><span class="p">));</span>
<span class="p">}</span>
<span class="nx">handlers</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">h</span> <span class="o">=&gt;</span> <span class="nf">h</span><span class="p">());</span> <span class="c1">// 5 5 5 5 5 — not what you want!</span>

<span class="c1">// ✅ Correct option 1 - use let (block scoping)</span>
<span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">5</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">handlers</span><span class="p">.</span><span class="nf">push</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">i</span><span class="p">));</span>
<span class="p">}</span>
<span class="c1">// 0 1 2 3 4 ✓</span>

<span class="c1">// ✅ Correct option 2 - just use map</span>
<span class="kd">const</span> <span class="nx">handlers</span> <span class="o">=</span> <span class="p">[</span><span class="mi">0</span><span class="p">,</span><span class="mi">1</span><span class="p">,</span><span class="mi">2</span><span class="p">,</span><span class="mi">3</span><span class="p">,</span><span class="mi">4</span><span class="p">].</span><span class="nf">map</span><span class="p">(</span><span class="nx">i</span> <span class="o">=&gt;</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">i</span><span class="p">));</span>
</code></pre>

</div>



<h2>
  
  
  5. Forgetting to clean up side effects
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - memory leak and stale state</span>
<span class="kd">function</span> <span class="nf">SearchComponent</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">results</span><span class="p">,</span> <span class="nx">setResults</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">([]);</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// If component unmounts before fetch completes, setState on unmounted component</span>
    <span class="nf">fetchSearch</span><span class="p">(</span><span class="nx">query</span><span class="p">).</span><span class="nf">then</span><span class="p">(</span><span class="nx">data</span> <span class="o">=&gt;</span> <span class="nf">setResults</span><span class="p">(</span><span class="nx">data</span><span class="p">));</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">query</span><span class="p">]);</span>
<span class="p">}</span>

<span class="c1">// ✅ Correct - clean up with AbortController</span>
<span class="kd">function</span> <span class="nf">SearchComponent</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">results</span><span class="p">,</span> <span class="nx">setResults</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">([]);</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">controller</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AbortController</span><span class="p">();</span>

    <span class="nf">fetchSearch</span><span class="p">(</span><span class="nx">query</span><span class="p">,</span> <span class="p">{</span> <span class="na">signal</span><span class="p">:</span> <span class="nx">controller</span><span class="p">.</span><span class="nx">signal</span> <span class="p">})</span>
      <span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">data</span> <span class="o">=&gt;</span> <span class="nf">setResults</span><span class="p">(</span><span class="nx">data</span><span class="p">))</span>
      <span class="p">.</span><span class="k">catch</span><span class="p">(</span><span class="nx">err</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">.</span><span class="nx">name</span> <span class="o">!==</span> <span class="dl">'</span><span class="s1">AbortError</span><span class="dl">'</span><span class="p">)</span> <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
      <span class="p">});</span>

    <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">controller</span><span class="p">.</span><span class="nf">abort</span><span class="p">();</span> <span class="c1">// Cleanup!</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">query</span><span class="p">]);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  6. Not using early returns
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - deeply nested, hard to read</span>
<span class="kd">function</span> <span class="nf">processOrder</span><span class="p">(</span><span class="nx">order</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">isVerified</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">total</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">return</span> <span class="nf">submitOrder</span><span class="p">(</span><span class="nx">order</span><span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// ✅ Correct - flat, readable, each failure is explicit</span>
<span class="kd">function</span> <span class="nf">processOrder</span><span class="p">(</span><span class="nx">order</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">order</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">order</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">isVerified</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">total</span> <span class="o">&lt;=</span> <span class="mi">0</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

  <span class="k">return</span> <span class="nf">submitOrder</span><span class="p">(</span><span class="nx">order</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  7. Using Array.forEach when you need a result
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - forEach doesn't return, forces mutable pattern</span>
<span class="kd">const</span> <span class="nx">doubled</span> <span class="o">=</span> <span class="p">[];</span>
<span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">].</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">n</span> <span class="o">=&gt;</span> <span class="nx">doubled</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">n</span> <span class="o">*</span> <span class="mi">2</span><span class="p">));</span>

<span class="c1">// 🚫 Also wrong - mixing concerns</span>
<span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="p">[];</span>
<span class="nx">users</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">user</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">user</span><span class="p">.</span><span class="nx">active</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">results</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span> <span class="p">...</span><span class="nx">user</span><span class="p">,</span> <span class="na">displayName</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">.</span><span class="nf">trim</span><span class="p">()</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// ✅ Correct - use the right method</span>
<span class="kd">const</span> <span class="nx">doubled</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">].</span><span class="nf">map</span><span class="p">(</span><span class="nx">n</span> <span class="o">=&gt;</span> <span class="nx">n</span> <span class="o">*</span> <span class="mi">2</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="nx">users</span>
  <span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">user</span> <span class="o">=&gt;</span> <span class="nx">user</span><span class="p">.</span><span class="nx">active</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">user</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="p">...</span><span class="nx">user</span><span class="p">,</span> <span class="na">displayName</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">.</span><span class="nf">trim</span><span class="p">()</span> <span class="p">}));</span>
</code></pre>

</div>



<h2>
  
  
  8. Not using environment checks properly
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// 🚫 Wrong - exposes sensitive info, breaks across environments</span>
<span class="kd">const</span> <span class="nx">API_URL</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">https://api.production.com</span><span class="dl">"</span><span class="p">;</span> <span class="c1">// Hardcoded!</span>
<span class="kd">const</span> <span class="nx">DEBUG</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span> <span class="c1">// Always enabled!</span>

<span class="c1">// 🚫 Also wrong - string comparison with typo risk  </span>
<span class="k">if </span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NODE_ENV</span> <span class="o">==</span> <span class="dl">"</span><span class="s2">producton</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span> <span class="p">...</span> <span class="p">}</span> <span class="c1">// Typo!</span>

<span class="c1">// ✅ Correct - centralised config</span>
<span class="c1">// config.js</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">REACT_APP_API_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">http://localhost:3000</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">isDev</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NODE_ENV</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">development</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">isProd</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NODE_ENV</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">production</span><span class="dl">'</span><span class="p">,</span>
<span class="p">};</span>

<span class="c1">// Use everywhere</span>
<span class="k">if </span><span class="p">(</span><span class="nx">config</span><span class="p">.</span><span class="nx">isDev</span><span class="p">)</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Debug info:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">data</span><span class="p">);</span>
</code></pre>

</div>






<h2>
  
  
  Quick Reference
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Anti-pattern</th>
<th>Fix</th>
</tr>
</thead>
<tbody>
<tr>
<td>
<code>==</code> loose equality</td>
<td>
<code>===</code> strict equality</td>
</tr>
<tr>
<td>Mutating params</td>
<td>Spread: <code>{ ...obj }</code>
</td>
</tr>
<tr>
<td>Unhandled async</td>
<td>try/catch + error state</td>
</tr>
<tr>
<td>var in loops</td>
<td>let or map</td>
</tr>
<tr>
<td>No cleanup in useEffect</td>
<td>return cleanup function</td>
</tr>
<tr>
<td>Deep nesting</td>
<td>Early returns</td>
</tr>
<tr>
<td>forEach with result</td>
<td>map/filter/reduce</td>
</tr>
<tr>
<td>Hardcoded config</td>
<td>Environment variables</td>
</tr>
</tbody>
</table></div>




<p><em>Did I miss any patterns you see often? Drop them in the comments!</em></p>

