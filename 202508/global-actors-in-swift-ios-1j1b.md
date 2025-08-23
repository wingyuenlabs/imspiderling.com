---
Title: Global Actors in Swift iOS
Description: 
Author: ArshTechPro
Date: 2025-08-23T21:49:55.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  How Actors Work
</h3>

<p>An actor processes one request at a time in a serialized manner. When you call an actor's method from outside its isolation context, you must use <code>await</code>, which creates a suspension point where your code waits for its turn to execute.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">actor</span> <span class="kt">DownloadManager</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">activeDownloads</span><span class="p">:</span> <span class="p">[</span><span class="kt">URL</span><span class="p">:</span> <span class="kt">Progress</span><span class="p">]</span> <span class="o">=</span> <span class="p">[:]</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">completedFiles</span><span class="p">:</span> <span class="p">[</span><span class="kt">URL</span><span class="p">:</span> <span class="kt">Data</span><span class="p">]</span> <span class="o">=</span> <span class="p">[:]</span>

    <span class="kd">func</span> <span class="nf">startDownload</span><span class="p">(</span><span class="n">from</span> <span class="nv">url</span><span class="p">:</span> <span class="kt">URL</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">String</span> <span class="p">{</span>
        <span class="k">if</span> <span class="n">activeDownloads</span><span class="p">[</span><span class="n">url</span><span class="p">]</span> <span class="o">!=</span> <span class="kc">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="s">"Download already in progress"</span>
        <span class="p">}</span>

        <span class="k">let</span> <span class="nv">progress</span> <span class="o">=</span> <span class="kt">Progress</span><span class="p">()</span>
        <span class="n">activeDownloads</span><span class="p">[</span><span class="n">url</span><span class="p">]</span> <span class="o">=</span> <span class="n">progress</span>
        <span class="k">return</span> <span class="s">"Download started"</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">completeDownload</span><span class="p">(</span><span class="nv">url</span><span class="p">:</span> <span class="kt">URL</span><span class="p">,</span> <span class="nv">data</span><span class="p">:</span> <span class="kt">Data</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">activeDownloads</span><span class="o">.</span><span class="nf">removeValue</span><span class="p">(</span><span class="nv">forKey</span><span class="p">:</span> <span class="n">url</span><span class="p">)</span>
        <span class="n">completedFiles</span><span class="p">[</span><span class="n">url</span><span class="p">]</span> <span class="o">=</span> <span class="n">data</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">getCompletedData</span><span class="p">(</span><span class="k">for</span> <span class="nv">url</span><span class="p">:</span> <span class="kt">URL</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">Data</span><span class="p">?</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">completedFiles</span><span class="p">[</span><span class="n">url</span><span class="p">]</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage requires await</span>
<span class="kd">func</span> <span class="nf">handleDownload</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
    <span class="k">let</span> <span class="nv">manager</span> <span class="o">=</span> <span class="kt">DownloadManager</span><span class="p">()</span>
    <span class="k">let</span> <span class="nv">status</span> <span class="o">=</span> <span class="k">await</span> <span class="n">manager</span><span class="o">.</span><span class="nf">startDownload</span><span class="p">(</span><span class="nv">from</span><span class="p">:</span> <span class="kt">URL</span><span class="p">(</span><span class="nv">string</span><span class="p">:</span> <span class="s">"https://example.com/file"</span><span class="p">)</span><span class="o">!</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">status</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>For details about MainActor refer below article</strong><br>
<a href="https://dev.to/arshtechpro/understanding-mainactor-in-swift-when-and-how-to-use-it-4ii4">https://dev.to/arshtechpro/understanding-mainactor-in-swift-when-and-how-to-use-it-4ii4</a></p>
<h2>
  
  
  What are Global Actors?
</h2>

<p>Global actors are a powerful Swift concurrency feature that extend the actor model to provide app-wide synchronization domains. While regular actors protect individual instances, global actors ensure that multiple pieces of code across different types and modules execute on the same serialized executor.</p>

<p>Think of a global actor as a synchronization coordinator that manages access to shared resources across your entire application. The most familiar example is <code>@MainActor</code>, which ensures code runs on the main thread.</p>
<h2>
  
  
  Why Do We Need Global Actors?
</h2>

<p>Global actors solve specific synchronization challenges:</p>

<ol>
<li>
<strong>Cross-Type Coordination</strong>: When multiple classes need to work with the same shared resource</li>
<li>
<strong>Thread Affinity</strong>: Ensuring certain code always runs on specific threads (like UI on main thread)</li>
<li>
<strong>Domain Isolation</strong>: Keeping different subsystems (networking, database, analytics) properly synchronized</li>
<li>
<strong>Compile-Time Safety</strong>: Moving thread-safety from runtime checks to compile-time guarantees</li>
</ol>
<h2>
  
  
  Creating a Global Actor
</h2>

<p>To create a global actor, you need:</p>

<ol>
<li>The <code>@globalActor</code> attribute</li>
<li>A shared static instance</li>
<li>The actor keyword</li>
</ol>

<p>Here's the basic structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@globalActor</span>
<span class="kd">actor</span> <span class="kt">MyCustomActor</span> <span class="p">{</span>
    <span class="kd">static</span> <span class="k">let</span> <span class="nv">shared</span> <span class="o">=</span> <span class="kt">MyCustomActor</span><span class="p">()</span>
    <span class="kd">private</span> <span class="nf">init</span><span class="p">()</span> <span class="p">{}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  How to Use Global Actors
</h2>

<p>Global actors can be applied at three levels:</p>

<h3>
  
  
  1. Class-Level Application
</h3>

<p>When you mark an entire class with a global actor, all its properties and methods become part of that actor's domain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@globalActor</span>
<span class="kd">actor</span> <span class="kt">DatabaseActor</span> <span class="p">{</span>
    <span class="kd">static</span> <span class="k">let</span> <span class="nv">shared</span> <span class="o">=</span> <span class="kt">DatabaseActor</span><span class="p">()</span>
    <span class="kd">private</span> <span class="nf">init</span><span class="p">()</span> <span class="p">{}</span>
<span class="p">}</span>

<span class="kd">@DatabaseActor</span>
<span class="kd">class</span> <span class="kt">DatabaseManager</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">cache</span><span class="p">:</span> <span class="p">[</span><span class="kt">String</span><span class="p">:</span> <span class="kt">Any</span><span class="p">]</span> <span class="o">=</span> <span class="p">[:]</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">transactionCount</span> <span class="o">=</span> <span class="mi">0</span>

    <span class="kd">func</span> <span class="nf">save</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="kt">String</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="kt">Any</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">cache</span><span class="p">[</span><span class="n">key</span><span class="p">]</span> <span class="o">=</span> <span class="n">value</span>
        <span class="n">transactionCount</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="nf">print</span><span class="p">(</span><span class="s">"Saved: </span><span class="se">\(</span><span class="n">key</span><span class="se">)</span><span class="s"> - Total transactions: </span><span class="se">\(</span><span class="n">transactionCount</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">retrieve</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">Any</span><span class="p">?</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">cache</span><span class="p">[</span><span class="n">key</span><span class="p">]</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">clearCache</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cache</span><span class="o">.</span><span class="nf">removeAll</span><span class="p">()</span>
        <span class="nf">print</span><span class="p">(</span><span class="s">"Cache cleared"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="kd">class</span> <span class="kt">ViewController</span><span class="p">:</span> <span class="kt">UIViewController</span> <span class="p">{</span>
    <span class="k">let</span> <span class="nv">database</span> <span class="o">=</span> <span class="kt">DatabaseManager</span><span class="p">()</span>

    <span class="kd">func</span> <span class="nf">saveUserData</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
        <span class="c1">// Must use await - accessing DatabaseActor from outside</span>
        <span class="k">await</span> <span class="n">database</span><span class="o">.</span><span class="nf">save</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="s">"username"</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="s">"John"</span><span class="p">)</span>
        <span class="k">await</span> <span class="n">database</span><span class="o">.</span><span class="nf">save</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="s">"lastLogin"</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="kt">Date</span><span class="p">())</span>

        <span class="c1">// All these calls are synchronized - no race conditions</span>
        <span class="k">if</span> <span class="k">let</span> <span class="nv">username</span> <span class="o">=</span> <span class="k">await</span> <span class="n">database</span><span class="o">.</span><span class="nf">retrieve</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="s">"username"</span><span class="p">)</span> <span class="p">{</span>
            <span class="nf">print</span><span class="p">(</span><span class="s">"Retrieved: </span><span class="se">\(</span><span class="n">username</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Method-Level Application
</h3>

<p>You can mark specific methods to run on a global actor while keeping the rest of the class unaffected:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">class</span> <span class="kt">DataService</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">localCache</span><span class="p">:</span> <span class="p">[</span><span class="kt">String</span><span class="p">:</span> <span class="kt">String</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="c1">// Regular method - runs on any thread</span>
    <span class="kd">func</span> <span class="nf">processData</span><span class="p">(</span><span class="n">_</span> <span class="nv">input</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">String</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">input</span><span class="o">.</span><span class="nf">uppercased</span><span class="p">()</span>
    <span class="p">}</span>

    <span class="c1">// This method runs on DatabaseActor</span>
    <span class="kd">@DatabaseActor</span>
    <span class="kd">func</span> <span class="nf">syncToDatabase</span><span class="p">(</span><span class="n">_</span> <span class="nv">data</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="p">{</span>
        <span class="nf">print</span><span class="p">(</span><span class="s">"Syncing to database: </span><span class="se">\(</span><span class="n">data</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
        <span class="c1">// This is synchronized with all other DatabaseActor code</span>
    <span class="p">}</span>

    <span class="c1">// This method runs on MainActor</span>
    <span class="kd">@MainActor</span>
    <span class="kd">func</span> <span class="nf">updateUI</span><span class="p">(</span><span class="n">with</span> <span class="nv">message</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Safe to update UI here</span>
        <span class="kt">NotificationCenter</span><span class="o">.</span><span class="k">default</span><span class="o">.</span><span class="nf">post</span><span class="p">(</span>
            <span class="nv">name</span><span class="p">:</span> <span class="o">.</span><span class="n">dataUpdated</span><span class="p">,</span>
            <span class="nv">object</span><span class="p">:</span> <span class="n">message</span>
        <span class="p">)</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">performCompleteSync</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
        <span class="k">let</span> <span class="nv">processed</span> <span class="o">=</span> <span class="nf">processData</span><span class="p">(</span><span class="s">"hello world"</span><span class="p">)</span>
        <span class="k">await</span> <span class="nf">syncToDatabase</span><span class="p">(</span><span class="n">processed</span><span class="p">)</span>
        <span class="k">await</span> <span class="nf">updateUI</span><span class="p">(</span><span class="nv">with</span><span class="p">:</span> <span class="s">"Sync complete"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  3. Property-Level Application
</h3>

<p>Individual properties can be bound to global actors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">class</span> <span class="kt">AppSettings</span> <span class="p">{</span>
    <span class="kd">@DatabaseActor</span> <span class="k">var</span> <span class="nv">userData</span><span class="p">:</span> <span class="p">[</span><span class="kt">String</span><span class="p">:</span> <span class="kt">Any</span><span class="p">]</span> <span class="o">=</span> <span class="p">[:]</span>  <span class="c1">// Bound to DatabaseActor</span>
    <span class="kd">@MainActor</span> <span class="k">var</span> <span class="nv">currentTheme</span><span class="p">:</span> <span class="kt">String</span> <span class="o">=</span> <span class="s">"light"</span>     <span class="c1">// Bound to MainActor</span>
    <span class="k">var</span> <span class="nv">cacheSize</span><span class="p">:</span> <span class="kt">Int</span> <span class="o">=</span> <span class="mi">100</span>                          <span class="c1">// Not actor-bound</span>

    <span class="kd">func</span> <span class="nf">updateSettings</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
        <span class="c1">// Need await for DatabaseActor property</span>
        <span class="k">await</span> <span class="kt">DatabaseActor</span><span class="o">.</span><span class="n">run</span> <span class="p">{</span>
            <span class="n">userData</span><span class="p">[</span><span class="s">"lastUpdate"</span><span class="p">]</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>
        <span class="p">}</span>

        <span class="c1">// Need await for MainActor property</span>
        <span class="k">await</span> <span class="kt">MainActor</span><span class="o">.</span><span class="n">run</span> <span class="p">{</span>
            <span class="n">currentTheme</span> <span class="o">=</span> <span class="s">"dark"</span>
        <span class="p">}</span>

        <span class="c1">// No await needed for regular property</span>
        <span class="n">cacheSize</span> <span class="o">=</span> <span class="mi">200</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Running Code on Global Actors
</h2>

<p>You can explicitly run code on a global actor using the <code>run</code> method:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="c1">// Run a closure on MainActor</span>
<span class="k">await</span> <span class="kt">MainActor</span><span class="o">.</span><span class="n">run</span> <span class="p">{</span>
    <span class="c1">// Update UI safely</span>
    <span class="n">myLabel</span><span class="o">.</span><span class="n">text</span> <span class="o">=</span> <span class="s">"Updated"</span>
<span class="p">}</span>

<span class="c1">// Run on your custom actor</span>
<span class="k">await</span> <span class="kt">DatabaseActor</span><span class="o">.</span><span class="n">run</span> <span class="p">{</span>
    <span class="c1">// This code runs on DatabaseActor</span>
    <span class="nf">print</span><span class="p">(</span><span class="s">"Running database operation"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Nonisolated and Global Actors
</h2>

<p>You can opt specific members out of global actor isolation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@DatabaseActor</span>
<span class="kd">class</span> <span class="kt">DataStore</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">records</span><span class="p">:</span> <span class="p">[</span><span class="kt">String</span><span class="p">:</span> <span class="kt">Any</span><span class="p">]</span> <span class="o">=</span> <span class="p">[:]</span>
    <span class="k">let</span> <span class="nv">storeId</span> <span class="o">=</span> <span class="kt">UUID</span><span class="p">()</span>  <span class="c1">// Immutable - safe to access</span>

    <span class="c1">// This property doesn't need synchronization</span>
    <span class="kd">nonisolated</span> <span class="k">var</span> <span class="nv">debugDescription</span><span class="p">:</span> <span class="kt">String</span> <span class="p">{</span>
        <span class="k">return</span> <span class="s">"DataStore: </span><span class="se">\(</span><span class="n">storeId</span><span class="se">)</span><span class="s">"</span>
    <span class="p">}</span>

    <span class="c1">// This method can be called without await</span>
    <span class="kd">nonisolated</span> <span class="kd">func</span> <span class="nf">validateKey</span><span class="p">(</span><span class="n">_</span> <span class="nv">key</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">Bool</span> <span class="p">{</span>
        <span class="k">return</span> <span class="o">!</span><span class="n">key</span><span class="o">.</span><span class="n">isEmpty</span> <span class="o">&amp;&amp;</span> <span class="n">key</span><span class="o">.</span><span class="n">count</span> <span class="o">&lt;</span> <span class="mi">100</span>
    <span class="p">}</span>

    <span class="c1">// This needs synchronization - accesses mutable state</span>
    <span class="kd">func</span> <span class="nf">addRecord</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="kt">String</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="kt">Any</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">records</span><span class="p">[</span><span class="n">key</span><span class="p">]</span> <span class="o">=</span> <span class="n">value</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="k">let</span> <span class="nv">store</span> <span class="o">=</span> <span class="kt">DataStore</span><span class="p">()</span>
<span class="nf">print</span><span class="p">(</span><span class="n">store</span><span class="o">.</span><span class="n">debugDescription</span><span class="p">)</span>  <span class="c1">// No await needed</span>
<span class="k">let</span> <span class="nv">isValid</span> <span class="o">=</span> <span class="n">store</span><span class="o">.</span><span class="nf">validateKey</span><span class="p">(</span><span class="s">"myKey"</span><span class="p">)</span>  <span class="c1">// No await needed</span>
<span class="k">await</span> <span class="n">store</span><span class="o">.</span><span class="nf">addRecord</span><span class="p">(</span><span class="nv">key</span><span class="p">:</span> <span class="s">"myKey"</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="s">"data"</span><span class="p">)</span>  <span class="c1">// Await required</span>
</code></pre>

</div>



<h2>
  
  
  Best Practices
</h2>

<ol>
<li><p><strong>Use Meaningful Names</strong>: Name your global actors based on their purpose (DatabaseActor, NetworkActor, etc.)</p></li>
<li><p><strong>Keep Global Actors Focused</strong>: Each global actor should have a single, clear responsibility</p></li>
<li><p><strong>Minimize Cross-Actor Communication</strong>: Frequent switching between actors impacts performance</p></li>
<li><p><strong>Use @MainActor for UI</strong>: Always use @MainActor for UI updates rather than creating custom UI actors</p></li>
<li><p><strong>Consider Performance</strong>: Global actors serialize access - only use when synchronization is needed</p></li>
</ol>

<h2>
  
  
  Common Pitfalls to Avoid
</h2>

<ol>
<li><p><strong>Over-using Global Actors</strong>: Don't mark everything with a global actor - only use when you need synchronization</p></li>
<li><p><strong>Blocking Operations</strong>: Avoid long-running synchronous operations in global actors as they block other operations</p></li>
<li><p><strong>Circular Dependencies</strong>: Be careful when global actors call each other - this can lead to deadlocks</p></li>
</ol>

<h2>
  
  
  Summary
</h2>

<p>Global actors are a powerful tool for managing synchronization across your entire application. By understanding global actors, you can write safer concurrent code.</p>

