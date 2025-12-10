---
Title: You Need Your Code to Be More Reliable Than People Using It
Description: 
Author: Abdulkabir Musa
Date: 2025-12-10T21:35:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>There's a harsh truth about software development that many engineers learn the hard way: users will break your application in ways you never imagined possible. They'll enter dates in the wrong format, upload 50GB files to your profile picture field, click submit 47 times in rapid succession, and somehow manage to get your application into states you didn't even know existed.</p>

<p>The knee-jerk reaction is often to blame the user. "They should have known better!" But here's the thing: <strong>your code needs to be more reliable than the people using it.</strong></p>

<h2>
  
  
  The Reality of User Behavior
</h2>

<p>Users don't read documentation. They don't carefully review error messages. They're distracted, in a hurry, or simply don't have the technical background to understand what your application expects. And that's perfectly fine—it's not their job to accommodate your code. It's your code's job to accommodate them.</p>

<p>Consider these real-world scenarios:</p>

<ul>
<li>A user copies text from a PDF that includes invisible Unicode characters</li>
<li>Someone's internet connection drops mid-transaction</li>
<li>A user navigates away from the page while your async operation is still running</li>
<li>Someone clicks "back" after submitting a form</li>
<li>A user leaves their session open for three days straight</li>
<li>Multiple tabs of your application are open simultaneously</li>
</ul>

<p>If any of these scenarios can break your application or corrupt data, that's not a user problem—that's a code reliability problem.</p>

<h2>
  
  
  What Reliability Actually Means
</h2>

<p>Reliable code isn't just code that works under ideal conditions. It's code that:</p>

<ol>
<li>
<strong>Validates rigorously</strong> - Never trust input, even from your own frontend</li>
<li>
<strong>Fails gracefully</strong> - When things go wrong, degrade functionality rather than crash</li>
<li>
<strong>Handles edge cases</strong> - The 99% case is important, but the 1% case is where bugs live</li>
<li>
<strong>Maintains data integrity</strong> - No matter what users do, your data stays consistent</li>
<li>
<strong>Recovers automatically</strong> - When possible, fix issues without user intervention</li>
</ol>

<h2>
  
  
  The Cost of Unreliable Code
</h2>

<p>I once worked on a financial application where a race condition allowed users to double-submit wire transfers. The bug was rare—it required clicking submit twice within a 200ms window. "Users won't do that," we thought. But with thousands of users, "rare" happened daily. Each incident required manual intervention, customer service calls, and potential financial liability.</p>

<p>The fix took two hours to implement: disable the button on first click and add idempotency keys. The cost of not implementing it from the start? Hundreds of hours of support time and damaged customer trust.</p>

<h2>
  
  
  Strategies for Building Reliable Code
</h2>

<h3>
  
  
  1. Input Validation Everywhere
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Bad: Trust the frontend</span>
<span class="kd">function</span> <span class="nf">createUser</span><span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">database</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nf">insert</span><span class="p">(</span><span class="nx">data</span><span class="p">);</span>
<span class="p">}</span>

<span class="c1">// Good: Validate everything</span>
<span class="kd">function</span> <span class="nf">createUser</span><span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">validated</span> <span class="o">=</span> <span class="nx">userSchema</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">data</span><span class="p">);</span> <span class="c1">// Throws on invalid data</span>
  <span class="kd">const</span> <span class="nx">sanitized</span> <span class="o">=</span> <span class="nf">sanitizeInput</span><span class="p">(</span><span class="nx">validated</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">database</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nf">insert</span><span class="p">(</span><span class="nx">sanitized</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Never assume input is clean, even from your own UI. Browsers can be manipulated, APIs can be called directly, and middleware can fail.</p>

<h3>
  
  
  2. Idempotency for All State Changes
</h3>

<p>Every operation that changes state should be idempotent—running it multiple times should produce the same result as running it once.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Bad: Can create duplicates
</span><span class="k">def</span> <span class="nf">submit_order</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">items</span><span class="p">):</span>
    <span class="n">order</span> <span class="o">=</span> <span class="n">Order</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span><span class="n">user_id</span><span class="o">=</span><span class="n">user_id</span><span class="p">,</span> <span class="n">items</span><span class="o">=</span><span class="n">items</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">order</span>

<span class="c1"># Good: Uses idempotency key
</span><span class="k">def</span> <span class="nf">submit_order</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">items</span><span class="p">,</span> <span class="n">idempotency_key</span><span class="p">):</span>
    <span class="n">existing</span> <span class="o">=</span> <span class="n">Order</span><span class="p">.</span><span class="nf">find_by_idempotency_key</span><span class="p">(</span><span class="n">idempotency_key</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">existing</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">existing</span>

    <span class="n">order</span> <span class="o">=</span> <span class="n">Order</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
        <span class="n">user_id</span><span class="o">=</span><span class="n">user_id</span><span class="p">,</span> 
        <span class="n">items</span><span class="o">=</span><span class="n">items</span><span class="p">,</span>
        <span class="n">idempotency_key</span><span class="o">=</span><span class="n">idempotency_key</span>
    <span class="p">)</span>
    <span class="k">return</span> <span class="n">order</span>
</code></pre>

</div>



<h3>
  
  
  3. Defensive Database Operations
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Bad: Assumes the record exists</span>
<span class="k">UPDATE</span> <span class="n">users</span> <span class="k">SET</span> <span class="n">balance</span> <span class="o">=</span> <span class="n">balance</span> <span class="o">-</span> <span class="mi">100</span> <span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="o">?</span><span class="p">;</span>

<span class="c1">-- Good: Ensures constraints are maintained</span>
<span class="k">UPDATE</span> <span class="n">users</span> 
<span class="k">SET</span> <span class="n">balance</span> <span class="o">=</span> <span class="n">balance</span> <span class="o">-</span> <span class="mi">100</span> 
<span class="k">WHERE</span> <span class="n">id</span> <span class="o">=</span> <span class="o">?</span> <span class="k">AND</span> <span class="n">balance</span> <span class="o">&gt;=</span> <span class="mi">100</span><span class="p">;</span>

<span class="c1">-- Then check affected rows to ensure it succeeded</span>
</code></pre>

</div>



<h3>
  
  
  4. Timeouts and Circuit Breakers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Bad: Wait forever for a response</span>
<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">externalAPI</span><span class="p">);</span>

<span class="c1">// Good: Fail fast with timeout</span>
<span class="kd">const</span> <span class="nx">controller</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AbortController</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">timeout</span> <span class="o">=</span> <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">controller</span><span class="p">.</span><span class="nf">abort</span><span class="p">(),</span> <span class="mi">5000</span><span class="p">);</span>

<span class="k">try</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">externalAPI</span><span class="p">,</span> <span class="p">{</span> <span class="na">signal</span><span class="p">:</span> <span class="nx">controller</span><span class="p">.</span><span class="nx">signal</span> <span class="p">});</span>
  <span class="k">return</span> <span class="nx">response</span><span class="p">;</span>
<span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">AbortError</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Handle timeout gracefully</span>
    <span class="k">return</span> <span class="nf">fallbackResponse</span><span class="p">();</span>
  <span class="p">}</span>
  <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
<span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
  <span class="nf">clearTimeout</span><span class="p">(</span><span class="nx">timeout</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  5. Rate Limiting and Resource Protection
</h3>

<p>Users will accidentally (or intentionally) hammer your endpoints. Your code needs to protect itself:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">functools</span> <span class="kn">import</span> <span class="n">wraps</span>
<span class="kn">from</span> <span class="n">flask</span> <span class="kn">import</span> <span class="n">request</span>
<span class="kn">import</span> <span class="n">time</span>

<span class="k">def</span> <span class="nf">rate_limit</span><span class="p">(</span><span class="n">max_calls</span><span class="p">,</span> <span class="n">time_window</span><span class="p">):</span>
    <span class="n">calls</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">decorator</span><span class="p">(</span><span class="n">f</span><span class="p">):</span>
        <span class="nd">@wraps</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>
        <span class="k">def</span> <span class="nf">wrapped</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
            <span class="n">user_id</span> <span class="o">=</span> <span class="nf">get_current_user_id</span><span class="p">()</span>
            <span class="n">now</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span>

            <span class="c1"># Clean old entries
</span>            <span class="n">calls</span><span class="p">[</span><span class="n">user_id</span><span class="p">]</span> <span class="o">=</span> <span class="p">[</span><span class="n">t</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">calls</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="p">[])</span> 
                             <span class="k">if</span> <span class="n">now</span> <span class="o">-</span> <span class="n">t</span> <span class="o">&lt;</span> <span class="n">time_window</span><span class="p">]</span>

            <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">calls</span><span class="p">[</span><span class="n">user_id</span><span class="p">])</span> <span class="o">&gt;=</span> <span class="n">max_calls</span><span class="p">:</span>
                <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Rate limit exceeded</span><span class="sh">"</span><span class="p">},</span> <span class="mi">429</span>

            <span class="n">calls</span><span class="p">[</span><span class="n">user_id</span><span class="p">].</span><span class="nf">append</span><span class="p">(</span><span class="n">now</span><span class="p">)</span>
            <span class="k">return</span> <span class="nf">f</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">wrapped</span>
    <span class="k">return</span> <span class="n">decorator</span>
</code></pre>

</div>



<h2>
  
  
  The Mindset Shift
</h2>

<p>Building reliable code requires a mindset shift from "this should work" to "how could this fail?" Start asking questions like:</p>

<ul>
<li>What happens if this takes 10 seconds instead of 100ms?</li>
<li>What if this function is called with null?</li>
<li>What if two users do this at the exact same time?</li>
<li>What if the network fails halfway through?</li>
<li>What if this external service is down?</li>
<li>What if someone sends me a 100MB string?</li>
</ul>

<h2>
  
  
  Testing for Reliability
</h2>

<p>Unit tests are great, but they rarely catch reliability issues. You need:</p>

<ul>
<li>
<strong>Chaos testing</strong> - Randomly kill processes, simulate network failures</li>
<li>
<strong>Load testing</strong> - See what breaks under pressure</li>
<li>
<strong>Fuzzing</strong> - Send random garbage input and see what happens</li>
<li>
<strong>Concurrent testing</strong> - Run multiple instances simultaneously</li>
<li>
<strong>Time-travel testing</strong> - Test with system clocks set to edge cases</li>
</ul>

<h2>
  
  
  The Payoff
</h2>

<p>Yes, building reliable code takes more time upfront. You'll write more validation logic, more error handling, more defensive checks. But the payoff is enormous:</p>

<ul>
<li>Fewer production incidents and 3 AM wake-up calls</li>
<li>Reduced support burden</li>
<li>Increased user trust</li>
<li>Lower maintenance costs</li>
<li>Better sleep at night</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>Your users will make mistakes. They'll encounter network issues, browser quirks, and timing problems. They'll use your application in ways you never anticipated. That's not a bug in your users—it's the reality of building software for humans.</p>

<p>The question isn't whether users will do unexpected things. The question is: when they do, will your code handle it gracefully, or will everything fall apart?</p>

<p>Make your code more reliable than the people using it. Your future self (and your on-call rotation) will thank you.</p>




<p><em>What reliability lessons have you learned the hard way? Share your war stories in the comments below.</em></p>

