---
Title: Instance Actors in Swift: Part 3 of Actor Series
Description: 
Author: ArshTechPro
Date: 2025-09-05T21:15:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>In our previous articles, we explored <a href="https://dev.to/arshtechpro/understanding-mainactor-in-swift-when-and-how-to-use-it-4ii4">@MainActor</a> for main thread operations and <a href="https://dev.to/arshtechpro/global-actors-in-swift-ios-1j1b">Global Actors</a> for app-wide synchronization domains. Now, let's complete our actor series by diving deep into <strong>Instance Actors</strong> - the most fundamental and flexible type of actor in Swift's concurrency model.</p>

<h2>
  
  
  What are Instance Actors?
</h2>

<p>Instance actors, also known as regular actors, are individual actor instances that each maintain their own isolated execution context. Unlike global actors that provide shared synchronization domains, each instance actor creates its own protective boundary around its mutable state.</p>

<p>Think of an instance actor as a personal bodyguard for your data. Each actor instance has its own dedicated security guard that ensures only one operation can access the actor's internal state at a time, preventing data races and ensuring thread safety.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">actor</span> <span class="kt">Counter</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">value</span> <span class="o">=</span> <span class="mi">0</span>

    <span class="kd">func</span> <span class="nf">increment</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="kt">Int</span> <span class="p">{</span>
        <span class="n">value</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="k">return</span> <span class="n">value</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">decrement</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="kt">Int</span> <span class="p">{</span>
        <span class="n">value</span> <span class="o">-=</span> <span class="mi">1</span>
        <span class="k">return</span> <span class="n">value</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">getValue</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="kt">Int</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">value</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In this example, each <code>Counter</code> instance has its own isolated execution context. If you create multiple <code>Counter</code> instances, they operate independently without interfering with each other.</p>

<h2>
  
  
  Why Use Instance Actors?
</h2>

<p>Instance actors solve specific concurrency challenges:</p>

<ul>
<li>
<strong>Data Isolation</strong>: Each instance protects its own mutable state independently</li>
<li>
<strong>Granular Control</strong>: Fine-grained synchronization for specific objects or resources</li>
<li>
<strong>Scalable Concurrency</strong>: Multiple instances can run concurrently without blocking each other</li>
<li>
<strong>Resource Management</strong>: Perfect for managing individual resources like file handlers, network connections, or cache entries</li>
<li>
<strong>State Machines</strong>: Ideal for implementing thread-safe state machines and object lifecycle management</li>
</ul>

<h2>
  
  
  How Instance Actors Work
</h2>

<p>Instance actors process requests one at a time in a serialized manner for each specific instance. When you call an actor's method from outside its isolation context, you must use <code>await</code>, which creates a suspension point where your code waits for its turn to execute on that particular actor.</p>

<h3>
  
  
  Example Bank Account Manager
</h3>

<p>This example shows how instance actors protect individual account data from race conditions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">actor</span> <span class="kt">BankAccount</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">let</span> <span class="nv">accountNumber</span><span class="p">:</span> <span class="kt">String</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">balance</span><span class="p">:</span> <span class="kt">Double</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">transactionHistory</span><span class="p">:</span> <span class="p">[</span><span class="kt">Transaction</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="nf">init</span><span class="p">(</span><span class="nv">accountNumber</span><span class="p">:</span> <span class="kt">String</span><span class="p">,</span> <span class="nv">initialBalance</span><span class="p">:</span> <span class="kt">Double</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">self</span><span class="o">.</span><span class="n">accountNumber</span> <span class="o">=</span> <span class="n">accountNumber</span>
        <span class="k">self</span><span class="o">.</span><span class="n">balance</span> <span class="o">=</span> <span class="n">initialBalance</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">deposit</span><span class="p">(</span><span class="n">_</span> <span class="nv">amount</span><span class="p">:</span> <span class="kt">Double</span><span class="p">)</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="kt">Bool</span> <span class="p">{</span>
        <span class="k">guard</span> <span class="n">amount</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="k">else</span> <span class="p">{</span> <span class="k">return</span> <span class="kc">false</span> <span class="p">}</span>

        <span class="c1">// Simulate processing delay</span>
        <span class="k">try</span><span class="p">?</span> <span class="k">await</span> <span class="kt">Task</span><span class="o">.</span><span class="nf">sleep</span><span class="p">(</span><span class="nv">nanoseconds</span><span class="p">:</span> <span class="mi">100_000_000</span><span class="p">)</span>

        <span class="n">balance</span> <span class="o">+=</span> <span class="n">amount</span>
        <span class="n">transactionHistory</span><span class="o">.</span><span class="nf">append</span><span class="p">(</span><span class="kt">Transaction</span><span class="p">(</span><span class="nv">type</span><span class="p">:</span> <span class="o">.</span><span class="n">deposit</span><span class="p">,</span> <span class="nv">amount</span><span class="p">:</span> <span class="n">amount</span><span class="p">))</span>

        <span class="nf">print</span><span class="p">(</span><span class="s">" Deposited $</span><span class="se">\(</span><span class="n">amount</span><span class="se">)</span><span class="s">. New balance: $</span><span class="se">\(</span><span class="n">balance</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="kc">true</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">withdraw</span><span class="p">(</span><span class="n">_</span> <span class="nv">amount</span><span class="p">:</span> <span class="kt">Double</span><span class="p">)</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="kt">Bool</span> <span class="p">{</span>
        <span class="k">guard</span> <span class="n">amount</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="n">balance</span> <span class="o">&gt;=</span> <span class="n">amount</span> <span class="k">else</span> <span class="p">{</span>
            <span class="nf">print</span><span class="p">(</span><span class="s">"Insufficient funds for withdrawal of $</span><span class="se">\(</span><span class="n">amount</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
            <span class="k">return</span> <span class="kc">false</span>
        <span class="p">}</span>

        <span class="c1">// Simulate processing delay</span>
        <span class="k">try</span><span class="p">?</span> <span class="k">await</span> <span class="kt">Task</span><span class="o">.</span><span class="nf">sleep</span><span class="p">(</span><span class="nv">nanoseconds</span><span class="p">:</span> <span class="mi">100_000_000</span><span class="p">)</span>

        <span class="n">balance</span> <span class="o">-=</span> <span class="n">amount</span>
        <span class="n">transactionHistory</span><span class="o">.</span><span class="nf">append</span><span class="p">(</span><span class="kt">Transaction</span><span class="p">(</span><span class="nv">type</span><span class="p">:</span> <span class="o">.</span><span class="n">withdrawal</span><span class="p">,</span> <span class="nv">amount</span><span class="p">:</span> <span class="n">amount</span><span class="p">))</span>

        <span class="nf">print</span><span class="p">(</span><span class="s">"Withdrew $</span><span class="se">\(</span><span class="n">amount</span><span class="se">)</span><span class="s">. New balance: $</span><span class="se">\(</span><span class="n">balance</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="kc">true</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">getBalance</span><span class="p">()</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="kt">Double</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">balance</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">getTransactionHistory</span><span class="p">()</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="p">[</span><span class="kt">Transaction</span><span class="p">]</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">transactionHistory</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="kd">struct</span> <span class="kt">Transaction</span> <span class="p">{</span>
    <span class="kd">enum</span> <span class="kt">TransactionType</span> <span class="p">{</span>
        <span class="k">case</span> <span class="n">deposit</span>
        <span class="k">case</span> <span class="n">withdrawal</span>
    <span class="p">}</span>

    <span class="k">let</span> <span class="nv">type</span><span class="p">:</span> <span class="kt">TransactionType</span>
    <span class="k">let</span> <span class="nv">amount</span><span class="p">:</span> <span class="kt">Double</span>
    <span class="k">let</span> <span class="nv">timestamp</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>
<span class="p">}</span>

<span class="c1">// Real-world usage demonstrating thread safety</span>
<span class="kd">func</span> <span class="nf">bankAccountExample</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
    <span class="k">let</span> <span class="nv">account</span> <span class="o">=</span> <span class="kt">BankAccount</span><span class="p">(</span><span class="nv">accountNumber</span><span class="p">:</span> <span class="s">"12345"</span><span class="p">,</span> <span class="nv">initialBalance</span><span class="p">:</span> <span class="mf">1000.0</span><span class="p">)</span>

    <span class="c1">// Multiple concurrent operations - all safely serialized by the actor</span>
    <span class="k">async</span> <span class="k">let</span> <span class="nv">deposit1</span><span class="p">:</span> <span class="kt">Bool</span> <span class="o">=</span> <span class="n">account</span><span class="o">.</span><span class="nf">deposit</span><span class="p">(</span><span class="mf">200.0</span><span class="p">)</span>
    <span class="k">async</span> <span class="k">let</span> <span class="nv">withdrawal1</span><span class="p">:</span> <span class="kt">Bool</span> <span class="o">=</span> <span class="n">account</span><span class="o">.</span><span class="nf">withdraw</span><span class="p">(</span><span class="mf">150.0</span><span class="p">)</span>
    <span class="k">async</span> <span class="k">let</span> <span class="nv">deposit2</span><span class="p">:</span> <span class="kt">Bool</span> <span class="o">=</span> <span class="n">account</span><span class="o">.</span><span class="nf">deposit</span><span class="p">(</span><span class="mf">300.0</span><span class="p">)</span>
    <span class="k">async</span> <span class="k">let</span> <span class="nv">withdrawal2</span><span class="p">:</span> <span class="kt">Bool</span> <span class="o">=</span> <span class="n">account</span><span class="o">.</span><span class="nf">withdraw</span><span class="p">(</span><span class="mf">50.0</span><span class="p">)</span>

    <span class="k">let</span> <span class="nv">results</span> <span class="o">=</span> <span class="k">await</span> <span class="p">[</span><span class="n">deposit1</span><span class="p">,</span> <span class="n">withdrawal1</span><span class="p">,</span> <span class="n">deposit2</span><span class="p">,</span> <span class="n">withdrawal2</span><span class="p">]</span>
    <span class="k">let</span> <span class="nv">finalBalance</span> <span class="o">=</span> <span class="k">await</span> <span class="n">account</span><span class="o">.</span><span class="nf">getBalance</span><span class="p">()</span>

    <span class="nf">print</span><span class="p">(</span><span class="s">"Final balance: $</span><span class="se">\(</span><span class="n">finalBalance</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="s">"All operations completed: </span><span class="se">\(</span><span class="n">results</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This example demonstrates the core strength of instance actors:</p>

<ol>
<li>
<strong>Individual Protection</strong>: Each bank account actor instance protects its own state</li>
<li>
<strong>Independent Operation</strong>: Multiple account instances work concurrently without blocking each other
</li>
<li>
<strong>Thread Safety</strong>: No risk of data corruption from concurrent access</li>
<li>
<strong>Natural Boundaries</strong>: Each actor represents a logical unit (individual bank account)</li>
<li>
<strong>Scalability</strong>: You can create as many instances as needed without performance degradation</li>
</ol>

<h2>
  
  
  When to Use Instance Actors
</h2>

<p>Instance actors are perfect when you need:</p>

<ul>
<li>
<strong>Individual Resource Management</strong>: Each instance manages its own dedicated resource (file, connection, cache)</li>
<li>
<strong>State Isolation</strong>: Different instances need to maintain separate state without interference</li>
<li>
<strong>Concurrent Processing</strong>: Multiple similar operations need to run simultaneously without blocking</li>
<li>
<strong>Task-Based Operations</strong>: Each actor represents an individual task or workflow</li>
<li>
<strong>Scalable Architecture</strong>: You need to create many independent units that work concurrently</li>
</ul>

<h2>
  
  
  Best Practices
</h2>

<h3>
  
  
  Keep Actor Operations Focused
</h3>

<p>Design actors around single responsibilities and keep their interfaces clean:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="c1">// Good: Focused responsibility</span>
<span class="kd">actor</span> <span class="kt">TokenManager</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">accessToken</span><span class="p">:</span> <span class="kt">String</span><span class="p">?</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">refreshToken</span><span class="p">:</span> <span class="kt">String</span><span class="p">?</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">expiryDate</span><span class="p">:</span> <span class="kt">Date</span><span class="p">?</span>

    <span class="kd">func</span> <span class="nf">setTokens</span><span class="p">(</span><span class="nv">access</span><span class="p">:</span> <span class="kt">String</span><span class="p">,</span> <span class="nv">refresh</span><span class="p">:</span> <span class="kt">String</span><span class="p">,</span> <span class="nv">expiresIn</span><span class="p">:</span> <span class="kt">TimeInterval</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">accessToken</span> <span class="o">=</span> <span class="n">access</span>
        <span class="n">refreshToken</span> <span class="o">=</span> <span class="n">refresh</span>
        <span class="n">expiryDate</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span><span class="o">.</span><span class="nf">addingTimeInterval</span><span class="p">(</span><span class="n">expiresIn</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">getValidAccessToken</span><span class="p">()</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="kt">String</span><span class="p">?</span> <span class="p">{</span>
        <span class="k">guard</span> <span class="k">let</span> <span class="nv">token</span> <span class="o">=</span> <span class="n">accessToken</span><span class="p">,</span>
              <span class="k">let</span> <span class="nv">expiry</span> <span class="o">=</span> <span class="n">expiryDate</span><span class="p">,</span>
              <span class="n">expiry</span> <span class="o">&gt;</span> <span class="kt">Date</span><span class="p">()</span> <span class="k">else</span> <span class="p">{</span>
            <span class="k">return</span> <span class="kc">nil</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="n">token</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">clearTokens</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">accessToken</span> <span class="o">=</span> <span class="kc">nil</span>
        <span class="n">refreshToken</span> <span class="o">=</span> <span class="kc">nil</span>
        <span class="n">expiryDate</span> <span class="o">=</span> <span class="kc">nil</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Minimize Cross-Actor Communication
</h3>

<p>Avoid frequent communication between different actors as it can impact performance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="c1">// Better: Batch operations to minimize actor switching</span>
<span class="kd">actor</span> <span class="kt">DataProcessor</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="k">var</span> <span class="nv">processedItems</span><span class="p">:</span> <span class="p">[</span><span class="kt">String</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="kd">func</span> <span class="nf">processItems</span><span class="p">(</span><span class="n">_</span> <span class="nv">items</span><span class="p">:</span> <span class="p">[</span><span class="kt">String</span><span class="p">])</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="p">[</span><span class="kt">String</span><span class="p">]</span> <span class="p">{</span>
        <span class="k">let</span> <span class="nv">processed</span> <span class="o">=</span> <span class="n">items</span><span class="o">.</span><span class="n">map</span> <span class="p">{</span> <span class="n">item</span> <span class="k">in</span>
            <span class="k">return</span> <span class="s">"processed-</span><span class="se">\(</span><span class="n">item</span><span class="se">)</span><span class="s">"</span>
        <span class="p">}</span>
        <span class="n">processedItems</span><span class="o">.</span><span class="nf">append</span><span class="p">(</span><span class="nv">contentsOf</span><span class="p">:</span> <span class="n">processed</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">processed</span>
    <span class="p">}</span>

    <span class="kd">func</span> <span class="nf">getProcessedCount</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="kt">Int</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">processedItems</span><span class="o">.</span><span class="n">count</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Summary
</h2>

<p>Instance actors are the foundation of Swift's actor concurrency model. They provide:</p>

<ul>
<li>
<strong>Individual Isolation</strong>: Each instance protects its own state independently</li>
<li>
<strong>Scalable Concurrency</strong>: Multiple instances can work simultaneously without interference
</li>
<li>
<strong>Fine-Grained Control</strong>: Perfect for managing specific resources or implementing state machines</li>
<li>
<strong>Thread Safety</strong>: Automatic protection against data races and concurrent access issues</li>
</ul>

<p>Together with <a href="https://dev.to/arshtechpro/understanding-mainactor-in-swift-when-and-how-to-use-it-4ii4">@MainActor</a> and <a href="https://dev.to/arshtechpro/global-actors-in-swift-ios-1j1b">Global Actors</a>, instance actors complete Swift's comprehensive actor concurrency system. Each type serves its purpose:</p>

<ul>
<li>
<strong>@MainActor</strong>: For UI updates and main thread operations</li>
<li>
<strong>Global Actors</strong>: For app-wide shared synchronization domains
</li>
<li>
<strong>Instance Actors</strong>: For individual, independent state protection</li>
</ul>

<p>This completes our 3-part actor series! You now have the knowledge to choose the right actor type for any concurrency challenge in your Swift applications.</p>

