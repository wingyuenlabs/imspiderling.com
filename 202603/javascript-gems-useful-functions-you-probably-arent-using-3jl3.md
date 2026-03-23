---
Title: JavaScript Gems: 7 Useful Functions You Should Try To Use
Description: 
Author: Mariana Kitsay
Date: 2026-03-23T21:50:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>JavaScript is full of powerful features, but usually developers tend to rely on a small, familiar subset of its capabilities. Key methods cover array &amp; string operations, object hadling, DOM interaction, and asynchronous operations.</p>

<p>Beyond the popular methods like map, filter, and reduce, there are functions and APIs that can significantly simplify the code and improve performance. Let’s explore some underrated JavaScript functions and patterns that deserve more attention.</p>

<p>1) <strong><code>Object.fromEntries()</code></strong> method transforms a list of key-value pairs into an object. It’s the reverse of <code>Object.entries()</code> and is perfect for transforming objects. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">entries</span> <span class="o">=</span> <span class="p">[[</span><span class="dl">'</span><span class="s1">name</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Alice</span><span class="dl">'</span><span class="p">],</span> <span class="p">[</span><span class="dl">'</span><span class="s1">age</span><span class="dl">'</span><span class="p">,</span> <span class="mi">25</span><span class="p">]];</span>
<span class="kd">const</span> <span class="nx">obj</span> <span class="o">=</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">fromEntries</span><span class="p">(</span><span class="nx">entries</span><span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">obj</span><span class="p">);</span> 
<span class="c1">// { name: 'Alice', age: 25 }</span>
</code></pre>

</div>



<p>Filtering object properties:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Alice</span><span class="dl">'</span><span class="p">,</span> <span class="na">age</span><span class="p">:</span> <span class="mi">25</span><span class="p">,</span> <span class="na">isUser</span><span class="p">:</span> <span class="kc">false</span> <span class="p">};</span>

<span class="kd">const</span> <span class="nx">filtered</span> <span class="o">=</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">fromEntries</span><span class="p">(</span>
  <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span><span class="nx">user</span><span class="p">).</span><span class="nf">filter</span><span class="p">(([</span><span class="nx">_</span><span class="p">,</span> <span class="nx">value</span><span class="p">])</span> <span class="o">=&gt;</span> <span class="nx">value</span> <span class="o">!==</span> <span class="kc">false</span><span class="p">)</span>
<span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">filtered</span><span class="p">);</span> 
<span class="c1">// { name: 'Alice', age: 25 }</span>
</code></pre>

</div>



<p>2) <strong><code>Array.prototype.flatMap()</code></strong> method maps and flattens an array in one step. It avoids chaining <code>.map().flat()</code> and improves readability. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">];</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">data</span><span class="p">.</span><span class="nf">flatMap</span><span class="p">(</span><span class="nx">x</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="nx">x</span><span class="p">,</span> <span class="nx">x</span> <span class="o">*</span> <span class="mi">2</span><span class="p">]);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">);</span>
<span class="c1">// [1, 2, 2, 4, 3, 6]</span>
</code></pre>

</div>



<p>3) <strong><code>Array.prototype.at()</code></strong> method accesses elements using positive or negative indices and is cleaner than <code>arr[arr.length - 1]</code>. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">arr</span> <span class="o">=</span> <span class="p">[</span><span class="mi">10</span><span class="p">,</span> <span class="mi">20</span><span class="p">,</span> <span class="mi">30</span><span class="p">];</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">arr</span><span class="p">.</span><span class="nf">at</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">));</span> 
<span class="c1">// 30</span>
</code></pre>

</div>



<p>4) <strong><code>Promise.allSettled()</code></strong> method waits for all promises to settle (either fulfilled or rejected) and prevents one failure from stopping everything. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">promises</span> <span class="o">=</span> <span class="p">[</span>
  <span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="mi">1</span><span class="p">),</span>
  <span class="nb">Promise</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error</span><span class="dl">'</span><span class="p">),</span>
  <span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="mi">3</span><span class="p">)</span>
<span class="p">];</span>

<span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">allSettled</span><span class="p">(</span><span class="nx">promises</span><span class="p">);</span>
</code></pre>

</div>



<p>5) <strong><code>String.prototype.matchAll()</code></strong> method returns all matches of a regex, including capture groups. It's more powerful than <code>match()</code> when working with regex. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">test1 test2</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">regex</span> <span class="o">=</span> <span class="sr">/test</span><span class="se">(\d)</span><span class="sr">/g</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">matches</span> <span class="o">=</span> <span class="p">[...</span><span class="nx">text</span><span class="p">.</span><span class="nf">matchAll</span><span class="p">(</span><span class="nx">regex</span><span class="p">)];</span>
</code></pre>

</div>



<p>6) <strong><code>Array.prototype.findLast()</code></strong> and <strong><code>findLastIndex()</code></strong> search arrays from the end and are cleaner than reversing arrays. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">arr</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">];</span>

<span class="kd">const</span> <span class="nx">lastEven</span> <span class="o">=</span> <span class="nx">arr</span><span class="p">.</span><span class="nf">findLast</span><span class="p">(</span><span class="nx">i</span> <span class="o">=&gt;</span> <span class="nx">i</span> <span class="o">%</span> <span class="mi">2</span> <span class="o">===</span> <span class="mi">0</span><span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">lastEven</span><span class="p">);</span> 
<span class="c1">// 4</span>
</code></pre>

</div>



<p>7) <strong><code>queueMicrotask()</code></strong> schedules a microtask to run after the current execution and is more predictable than <code>setTimeout(fn, 0)</code>. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Start</span><span class="dl">'</span><span class="p">);</span>

<span class="nf">queueMicrotask</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Microtask</span><span class="dl">'</span><span class="p">);</span>
<span class="p">});</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">End</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// Output:</span>
<span class="c1">// Start</span>
<span class="c1">// End</span>
<span class="c1">// Microtask</span>
</code></pre>

</div>



<p>Incorporating these functions into the workflow can:</p>

<ul>
<li>Reduce boilerplate code</li>
<li>Improve readability and performance</li>
<li>Make your code more powerful</li>
</ul>

<p>If you want to level up as a JavaScript developer, it’s worth occasionally exploring the “forgotten corners” of the language—you might find tools that change how you write code completely</p>

