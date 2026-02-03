---
Title: React Compiler: Stop Manually Optimizing Your React Apps
Description: 
Author: Yuvaraj
Date: 2026-02-03T21:47:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>During our team KATA session, a colleague asked a question that I bet you've thought about it too:</p>

<blockquote>
<p><em>"If React already knows to only render the elements that changed, why do we need to optimize anything manually?"</em></p>
</blockquote>

<p>It was a brilliant question. The answer reveals a major pain point we’ve lived with for years—and let's see how React compiler addresses few areas. </p>

<p>Let’s take a journey through the evolution of React optimization, using a simple analogy: <strong>The Restaurant Kitchen</strong>.</p>

<h2>
  
  
  🍝 The Restaurant Kitchen: How React <em>Actually</em> Works
</h2>

<p>Imagine your App is a kitchen.</p>

<ul>
<li>
<strong>Head Chef (Parent Component):</strong> Manages the kitchen.</li>
<li>
<strong>Line cooks (Child Components):</strong> Handle specific stations.</li>
</ul>

<p>In a standard React app, every time the Head Chef changes something—even just restocking the salt—they ring a giant bell. <strong>Every single cook stops and redoes their work</strong>, even if their specific station didn't change.</p>

<p>This is React’s default behavior: <strong>When a parent re-renders, all children re-render.</strong></p>

<p>For years, to stop this waste, we had to write additional code to give instruction(hooks) to react's optimisation technique. Let’s look at how a single component evolved from "without hooks(instructions to compiler)" to "With hooks(instructions to react optimisation technique)" to "React compiler code automatically optimises it."</p>




<h2>
  
  
  The Evolution of a Component
</h2>

<p>Let's look at a <code>RestaurantMenu</code> that does three things:</p>

<ol>
<li>Holds a list of dishes.</li>
<li>Filters them (an expensive calculation).</li>
<li>Renders a list of items (child components).</li>
</ol>

<h3>
  
  
  Phase 1: The Code (Clean but Slow)
</h3>

<p>Here is the code most beginners write. It looks clean, but it has hidden performance traps.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// A simple child component</span>
<span class="kd">const</span> <span class="nx">DishList</span> <span class="o">=</span> <span class="p">({</span> <span class="nx">dishes</span><span class="p">,</span> <span class="nx">onOrder</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">🍝 Rendering DishList (Child)</span><span class="dl">"</span><span class="p">);</span> <span class="c1">// &lt;--- Watch this log!</span>
  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="p">{</span><span class="cm">/* items... */</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span><span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">RestaurantMenu</span><span class="p">({</span> <span class="nx">allDishes</span><span class="p">,</span> <span class="nx">theme</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">category</span><span class="p">,</span> <span class="nx">setCategory</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">pasta</span><span class="dl">'</span><span class="p">);</span>

  <span class="c1">// ⚠️ PROBLEM 1: Expensive Calculation runs every render</span>
  <span class="kd">const</span> <span class="nx">filteredDishes</span> <span class="o">=</span> <span class="nx">allDishes</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">dish</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">🧮 Filtering... (Slow Math)</span><span class="dl">"</span><span class="p">);</span> 
    <span class="k">return</span> <span class="nx">dish</span><span class="p">.</span><span class="nx">category</span> <span class="o">===</span> <span class="nx">category</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">handleOrder</span> <span class="o">=</span> <span class="p">(</span><span class="nx">dish</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Ordered:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">dish</span><span class="p">);</span>
  <span class="p">};</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span> <span class="nx">className</span><span class="o">=</span><span class="p">{</span><span class="nx">theme</span><span class="p">}</span><span class="o">&gt;</span>
      <span class="p">{</span><span class="cm">/* Clicking this causes a re-render */</span><span class="p">}</span>
      <span class="o">&lt;</span><span class="nx">button</span> <span class="nx">onClick</span><span class="o">=</span><span class="p">{()</span> <span class="o">=&gt;</span> <span class="nf">setCategory</span><span class="p">(</span><span class="dl">'</span><span class="s1">salad</span><span class="dl">'</span><span class="p">)}</span><span class="o">&gt;</span><span class="nx">Switch</span> <span class="nx">Category</span><span class="o">&lt;</span><span class="sr">/button</span><span class="err">&gt;
</span>
      <span class="p">{</span><span class="cm">/* ⚠️ PROBLEM 2: Inline Arrow Function */</span><span class="p">}</span>
      <span class="p">{</span><span class="cm">/* Writing (dish) =&gt; handleOrder(dish) creates a BRAND NEW function 
          in memory every single time this component renders. 
          This forces DishList to re-render. */</span><span class="p">}</span>
      <span class="o">&lt;</span><span class="nx">DishList</span> 
        <span class="nx">dishes</span><span class="o">=</span><span class="p">{</span><span class="nx">filteredDishes</span><span class="p">}</span> 
        <span class="nx">onOrder</span><span class="o">=</span><span class="p">{(</span><span class="nx">dish</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">handleOrder</span><span class="p">(</span><span class="nx">dish</span><span class="p">)}</span> 
      <span class="sr">/</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>What happens in the Console?</strong><br>
Even if the parent re-renders for a minor reason (or if we click the button), everything runs again.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🖥️ CONSOLE OUTPUT:
---------------------------------------------
🧮 Filtering... (Slow Math)
🍝 Rendering DishList (Child)

</code></pre>

</div>



<p><em>(Every single interaction triggers these logs. Wasteful!)</em></p>




<h3>
  
  
  Phase 2: The Solution with hooks(addition instructions)
</h3>

<p>To fix this in React, we had to introduce "Hooks." We wrap in <code>useMemo</code>, <code>useCallback</code>, and <code>memo</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useState</span><span class="p">,</span> <span class="nx">useMemo</span><span class="p">,</span> <span class="nx">useCallback</span><span class="p">,</span> <span class="nx">memo</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Solution A: Wrap child in memo to prevent useless re-renders</span>
<span class="kd">const</span> <span class="nx">DishList</span> <span class="o">=</span> <span class="nf">memo</span><span class="p">(({</span> <span class="nx">dishes</span><span class="p">,</span> <span class="nx">onOrder</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">🍝 Rendering DishList (Child)</span><span class="dl">"</span><span class="p">);</span>
  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="p">{</span><span class="cm">/* items... */</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span><span class="p">});</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">RestaurantMenu</span><span class="p">({</span> <span class="nx">allDishes</span><span class="p">,</span> <span class="nx">theme</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">category</span><span class="p">,</span> <span class="nx">setCategory</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">pasta</span><span class="dl">'</span><span class="p">);</span>

  <span class="c1">// Solution B: Cache calculation with useMemo</span>
  <span class="kd">const</span> <span class="nx">filteredDishes</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">🧮 Filtering... (Slow Math)</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">allDishes</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">dish</span> <span class="o">=&gt;</span> <span class="nx">dish</span><span class="p">.</span><span class="nx">category</span> <span class="o">===</span> <span class="nx">category</span><span class="p">);</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">allDishes</span><span class="p">,</span> <span class="nx">category</span><span class="p">]);</span> 

  <span class="c1">// Solution C: Freeze function with useCallback</span>
  <span class="kd">const</span> <span class="nx">handleOrder</span> <span class="o">=</span> <span class="nf">useCallback</span><span class="p">((</span><span class="nx">dish</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Ordered:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">dish</span><span class="p">);</span>
  <span class="p">},</span> <span class="p">[]);</span> 

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span> <span class="nx">className</span><span class="o">=</span><span class="p">{</span><span class="nx">theme</span><span class="p">}</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">button</span> <span class="nx">onClick</span><span class="o">=</span><span class="p">{()</span> <span class="o">=&gt;</span> <span class="nf">setCategory</span><span class="p">(</span><span class="dl">'</span><span class="s1">salad</span><span class="dl">'</span><span class="p">)}</span><span class="o">&gt;</span><span class="nx">Switch</span> <span class="nx">Category</span><span class="o">&lt;</span><span class="sr">/button</span><span class="err">&gt;
</span>
      <span class="p">{</span><span class="cm">/* ⚠️ THE TRAP: We CANNOT use an inline arrow here! 
          If we wrote: onOrder={(dish) =&gt; handleOrder(dish)}
          It would BREAK the optimization because the arrow wrapper 
          is a new reference. We are FORCED to pass the function directly. */</span><span class="p">}</span>
      <span class="o">&lt;</span><span class="nx">DishList</span> 
        <span class="nx">dishes</span><span class="o">=</span><span class="p">{</span><span class="nx">filteredDishes</span><span class="p">}</span> 
        <span class="nx">onOrder</span><span class="o">=</span><span class="p">{</span><span class="nx">handleOrder</span><span class="p">}</span> 
      <span class="sr">/</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>

</code></pre>

</div>



<p><strong>What happens in the Console now?</strong><br>
If the parent re-renders (for example, if <code>theme</code> changes but <code>category</code> stays the same), the console stays silent.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🖥️ CONSOLE OUTPUT:
---------------------------------------------
(Silent. No logs appear.)

</code></pre>

</div>



<p><em>(Performance is achieved, but the code is hard to read because of hooks syntax)</em></p>

<p><em>What happens, If your colleague changes onOrder={handleOrder} to onOrder={() =&gt; handleOrder()}</em>, the optimization breaks silently, the arrow function () =&gt; handleOrder() creates a new function every time the component renders</p>




<h3>
  
  
  Phase 3: The React Compiler Solution (without additional code)
</h3>

<p>This is the magic of React compiler. You go back to writing the code from <strong>Phase 1</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// No useMemo. No useCallback. No memo.</span>
<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">RestaurantMenu</span><span class="p">({</span> <span class="nx">allDishes</span><span class="p">,</span> <span class="nx">theme</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">category</span><span class="p">,</span> <span class="nx">setCategory</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">pasta</span><span class="dl">'</span><span class="p">);</span>

  <span class="c1">// The Compiler AUTOMATICALLY memoizes this</span>
  <span class="kd">const</span> <span class="nx">filteredDishes</span> <span class="o">=</span> <span class="nx">allDishes</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">dish</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">🧮 Filtering... (Slow Math)</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">dish</span><span class="p">.</span><span class="nx">category</span> <span class="o">===</span> <span class="nx">category</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="c1">// The Compiler AUTOMATICALLY stabilizes this function</span>
  <span class="kd">const</span> <span class="nx">handleOrder</span> <span class="o">=</span> <span class="p">(</span><span class="nx">dish</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Ordered:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">dish</span><span class="p">);</span>
  <span class="p">};</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span> <span class="nx">className</span><span class="o">=</span><span class="p">{</span><span class="nx">theme</span><span class="p">}</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">button</span> <span class="nx">onClick</span><span class="o">=</span><span class="p">{()</span> <span class="o">=&gt;</span> <span class="nf">setCategory</span><span class="p">(</span><span class="dl">'</span><span class="s1">salad</span><span class="dl">'</span><span class="p">)}</span><span class="o">&gt;</span><span class="nx">Switch</span> <span class="nx">Category</span><span class="o">&lt;</span><span class="sr">/button</span><span class="err">&gt;
</span>      <span class="p">{</span><span class="cm">/* ✅ COMPILER MAGIC: We can use an inline arrow again! 
          The compiler is smart enough to "memoize" this arrow function 
          wrapper automatically. It sees that 'handleOrder' is stable, 
          so it makes this arrow stable too. */</span><span class="p">}</span>
      <span class="o">&lt;</span><span class="nx">DishList</span> <span class="nx">dishes</span><span class="o">=</span><span class="p">{</span><span class="nx">filteredDishes</span><span class="p">}</span> <span class="nx">onOrder</span><span class="o">=</span><span class="p">{(</span><span class="nx">dish</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">handleOrder</span><span class="p">(</span><span class="nx">dish</span><span class="p">)}</span> <span class="sr">/</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>

</code></pre>

</div>



<p><strong>What happens in the Console?</strong><br>
Even though we deleted all the hooks, the result is identical to Phase 2.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🖥️ CONSOLE OUTPUT:
---------------------------------------------
(Silent. No logs appear.)

</code></pre>

</div>



<p><strong>What just happened?</strong><br>
The React Compiler analyzed your code at build time. It understands data flow better than we do.</p>

<ul>
<li>It sees <code>filteredDishes</code> only changes when <code>category</code> changes.</li>
<li>It sees you wrapped handleOrder in an arrow function (dish) =&gt; handleOrder(dish).</li>
<li>It automatically caches that arrow function wrapper so it remains the exact same reference across renders.</li>
<li>It effectively generates the optimized code from <strong>Phase 2</strong> for you, behind the scenes.</li>
</ul>




<h2>
  
  
  The Philosophy Shift
</h2>

<p>For years, We had to manually tell the framework: <em>"Remember this variable! Freeze this function!"</em></p>

<p><strong>React compiler address this problem!.</strong><br>
React now assumes the burden of optimization. It allows us to stop worrying about render cycles and dependency arrays, and start focusing on what actually matters: <strong>shipping features.</strong></p>

<h3>
  
  
  What Now?
</h3>

<p>The best part is that React Compiler is <strong>backward compatible (React v17, v18 as well)</strong>. You don't have to rewrite your codebase. You can enable it, and it will optimize your "plain" components while leaving your existing hooks.</p>




<p><em>Thanks for reading! This is my first post on Dev.to, and I wrote it to help solidify my own understanding of the Compiler. I’d love your feedback—did the restaurant analogy make sense to you? Let me know in the comments!</em></p>

