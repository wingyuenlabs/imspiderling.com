---
Title: Your-Error-Handling-is-a-Mess-and-Its-Costing-You-💸
Description: 
Author: member_875c0744
Date: 2025-10-25T22:00:27.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://github.com/hyperlane-dev/hyperlane" rel="noopener noreferrer">GitHub Home</a></p>

<h4>
  
  
  Your Error Handling is a Mess, and It's Costing You 💸
</h4>

<p>I still remember the bug that kept me up all night. A payment callback endpoint, when handling a rare, exceptional status code from a third-party payment gateway, had a <code>.catch()</code> that was accidentally omitted from a <code>Promise</code> chain. The result? No logs, no alerts, and the service itself didn't crash. It just "silently" failed. That user's order status was forever stuck on "processing," and we were completely unaware. It wasn't until a week later during a reconciliation that we discovered hundreds of these "silent orders," resulting in tens of thousands of dollars in losses. 💸</p>

<p>The lesson was painful. It made me realize that in software engineering, <strong>we probably spend less than 10% of our time on the happy path. The other 90% of the complexity comes from how to handle all sorts of expected and unexpected errors gracefully and robustly.</strong> And the quality of a framework is largely reflected in how it guides us to face this "world of errors."</p>

<p>Many frameworks, especially the "flexible" ones in dynamic languages, have a philosophy on error handling that can almost be described as "laissez-faire." They give you a thousand ways to make a mistake, but only one way to do it right, which requires extreme discipline.</p>

<h4>
  
  
  Callback Hell and Swallowed Promises: The Agony of Error Handling in JavaScript
</h4>

<p>In the world of Node.js, we've gone through a long evolution of battling with errors.</p>

<p><strong>Phase 1: Callback Hell</strong></p>

<p>The older generation of Node.js developers still remembers the fear of being dominated by the "pyramid of doom."<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">processOrder</span><span class="p">(</span><span class="nx">orderId</span><span class="p">,</span> <span class="nx">callback</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">db</span><span class="p">.</span><span class="nf">findOrder</span><span class="p">(</span><span class="nx">orderId</span><span class="p">,</span> <span class="p">(</span><span class="nx">err</span><span class="p">,</span> <span class="nx">order</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// Error handling point 1</span>
      <span class="k">return</span> <span class="nf">callback</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="nx">payment</span><span class="p">.</span><span class="nf">process</span><span class="p">(</span><span class="nx">order</span><span class="p">,</span> <span class="p">(</span><span class="nx">err</span><span class="p">,</span> <span class="nx">result</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Error handling point 2</span>
        <span class="k">return</span> <span class="nf">callback</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
      <span class="p">}</span>
      <span class="nx">inventory</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">items</span><span class="p">,</span> <span class="p">(</span><span class="nx">err</span><span class="p">,</span> <span class="nx">status</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
          <span class="c1">// Error handling point 3</span>
          <span class="k">return</span> <span class="nf">callback</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
        <span class="p">}</span>
        <span class="nf">callback</span><span class="p">(</span><span class="kc">null</span><span class="p">,</span> <span class="nx">status</span><span class="p">);</span> <span class="c1">// Success!</span>
      <span class="p">});</span>
    <span class="p">});</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This "error-first" callback style is viable in theory. But as business logic gets more complex, the code extends infinitely to the right, forming a "pyramid of death" that is difficult to maintain. You have to remember to check the <code>err</code> object in every single callback. A single oversight, and the error gets "swallowed."</p>

<p><strong>Phase 2: The Redemption of Promises and New Pitfalls</strong></p>

<p><code>Promise</code> rescued us from callback hell. We could use <code>.then()</code> and <code>.catch()</code> to build a flatter, more readable asynchronous chain.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">processOrder</span><span class="p">(</span><span class="nx">orderId</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">db</span>
    <span class="p">.</span><span class="nf">findOrder</span><span class="p">(</span><span class="nx">orderId</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">then</span><span class="p">((</span><span class="nx">order</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">payment</span><span class="p">.</span><span class="nf">process</span><span class="p">(</span><span class="nx">order</span><span class="p">))</span>
    <span class="p">.</span><span class="nf">then</span><span class="p">((</span><span class="nx">result</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">inventory</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">items</span><span class="p">))</span>
    <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">err</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// Centralized error handling point</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Order processing failed:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">err</span><span class="p">);</span>
      <span class="c1">// But here, you must remember to re-throw the error,</span>
      <span class="c1">// or the caller will think it succeeded.</span>
      <span class="k">throw</span> <span class="nx">err</span><span class="p">;</span>
    <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is much better! But new problems arose. If you forget to <code>return</code> the next <code>Promise</code> in a <code>.then()</code>, or forget to re-<code>throw</code> the error in a <code>.catch()</code>, the chain will continue to execute in a way you didn't expect. The error, once again, is "silently" swallowed.</p>

<p><strong>Phase 3: The Elegance of <code>async/await</code> and Its Final Disguise</strong></p>

<p><code>async/await</code> allows us to write asynchronous code in a seemingly synchronous way, which is a godsend.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">async</span> <span class="kd">function</span> <span class="nf">processOrder</span><span class="p">(</span><span class="nx">orderId</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">findOrder</span><span class="p">(</span><span class="nx">orderId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">payment</span><span class="p">.</span><span class="nf">process</span><span class="p">(</span><span class="nx">order</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">status</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">inventory</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">items</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">status</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Order processing failed:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">err</span><span class="p">);</span>
    <span class="k">throw</span> <span class="nx">err</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This looks almost perfect, doesn't it? But it still relies on the programmer's "conscientiousness." You have to remember to wrap all potentially failing asynchronous calls in a <code>try...catch</code> block. What if you forget to <code>await</code> a function that returns a <code>Promise</code>? The error within that function will never be caught by this <code>try...catch</code>.</p>

<p>The problem with JavaScript is that <strong>an error is a value that can be easily ignored</strong>. <code>null</code> and <code>undefined</code> can roam your code like ghosts. You need to rely on strict conventions, Linter tools, and personal discipline to ensure every error is handled correctly. And that, precisely, is unreliable.</p>

<h4>
  
  
  The <code>Result</code> Enum: When the Compiler Becomes Your Most Reliable Error-Handling Partner
</h4>

<p>Now, let's enter the world of Rust and <code>hyperlane</code>. Here, the philosophy of error handling is completely different. At the core of the Rust language, there is an enum called <code>Result&lt;T, E&gt;</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">enum</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="n">T</span><span class="p">,</span> <span class="n">E</span><span class="o">&gt;</span> <span class="p">{</span>
   <span class="nf">Ok</span><span class="p">(</span><span class="n">T</span><span class="p">),</span>  <span class="c1">// Represents success and contains a value</span>
   <span class="nf">Err</span><span class="p">(</span><span class="n">E</span><span class="p">),</span> <span class="c1">// Represents failure and contains an error</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This design is simple yet profound. It means that a function that might fail <strong>must</strong> return one of these two states. It's no longer a value that could be <code>null</code>, or a <code>Promise</code> that you need to <code>.catch()</code> somewhere else. It's a complete type that contains all possibilities.</p>

<p>Most importantly, <strong>the compiler will force you to handle the <code>Err</code> case</strong>. If you call a function that returns a <code>Result</code> but don't handle its <code>Err</code> branch, the compiler will give you a warning or even an error. You <strong>cannot</strong> "accidentally" ignore an error.</p>

<p>Let's see what the code would look like in <code>hyperlane</code>'s <code>service</code> layer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="c1">// in a service file</span>
<span class="k">pub</span> <span class="k">fn</span> <span class="nf">process_order</span><span class="p">(</span><span class="n">order_id</span><span class="p">:</span> <span class="o">&amp;</span><span class="nb">str</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="n">Status</span><span class="p">,</span> <span class="n">OrderError</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">let</span> <span class="n">order</span> <span class="o">=</span> <span class="nn">db</span><span class="p">::</span><span class="nf">find_order</span><span class="p">(</span><span class="n">order_id</span><span class="p">)</span><span class="o">?</span><span class="p">;</span> <span class="c1">// `?` operator: if it fails, return Err immediately</span>
    <span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="nn">payment</span><span class="p">::</span><span class="nf">process</span><span class="p">(</span><span class="n">order</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>
    <span class="k">let</span> <span class="n">status</span> <span class="o">=</span> <span class="nn">inventory</span><span class="p">::</span><span class="nf">update</span><span class="p">(</span><span class="n">result</span><span class="py">.items</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>
    <span class="nf">Ok</span><span class="p">(</span><span class="n">status</span><span class="p">)</span> <span class="c1">// Explicitly return success</span>
<span class="p">}</span>
</code></pre>

</div>



<p>See that <code>?</code> operator? It's the essence of Rust's error handling. It's basically saying: "Call this function. If it returns <code>Ok(value)</code>, take the <code>value</code> out and continue. If it returns <code>Err(error)</code>, immediately return that <code>Err(error)</code> from the current function."</p>

<p>This pattern transforms the logic that required <code>try...catch</code> in JavaScript into an extremely concise, clear, and compiler-guaranteed chain of calls. An error is no longer an exception to be "caught," but an expected branch in the data flow that is handled gracefully.</p>

<h4>
  
  
  <code>panic_hook</code>: The Last Line of Defense
</h4>

<p>Of course, there are always errors we can't anticipate, which are <code>panic</code>s. For example, array out of bounds, integer overflow, etc. In many frameworks, an unhandled <code>panic</code> will cause the entire thread or even the process to crash.</p>

<p>But <code>hyperlane</code> provides an elegant "last line of defense"—the <code>panic_hook</code>. We've seen it in previous articles:</p>

<p>This hook can catch any <code>panic</code> that occurs during request processing. It prevents the server from crashing directly and allows you to log detailed error information for post-mortem analysis, while returning a friendly error page to the client instead of a disconnected connection. This is an extremely responsible and robust design.</p>

<h3>
  
  
  Stop Praying for Code Without Errors, Embrace Errors from the Start
</h3>

<p>Good error handling isn't about stuffing <code>try...catch</code> blocks in every corner of your code. It's about having a system, at the language and framework level, that provides you with a mechanism to make "failure" a predictable, first-class citizen of your program's flow.</p>

<p>Rust's <code>Result</code> enum forces you to confront every possible failure, and <code>hyperlane</code>'s architecture and hook system provide you with elegant patterns for handling them. It elevates error handling from a matter of "developer discipline" to a "compiler guarantee."</p>

<p>So, if you're still struggling with messy error handling logic and fearing those "silent" failures, the problem might not be that you're not trying hard enough. It might be that the tools you've chosen didn't prioritize "robustness" from the very beginning. It's time to choose a partner that can fight alongside you to face this imperfect world.</p>

<p><a href="https://github.com/hyperlane-dev/hyperlane" rel="noopener noreferrer">GitHub Home</a></p>

