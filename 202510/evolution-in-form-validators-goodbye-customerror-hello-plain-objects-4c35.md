---
Title: Evolution in Form Validators: Goodbye customError, Hello Plain Objects
Description: 
Author: Damian Sire (Dev)
Date: 2025-10-30T21:42:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>Form management in Angular, especially with the arrival of <strong>Signal-based forms</strong>, is constantly evolving to improve ergonomics and the overall <strong>developer experience (DX)</strong>.</p>

<p>A subtle but incredibly welcome change is the simplification in how we define <strong>custom validation errors</strong>.<br><br>
What previously required a utility function like <code>customError</code> to wrap our error object, now allows us to return a <strong>plain JavaScript object (POJO)</strong> directly.</p>

<p>This change reduces boilerplate, simplifies the API, and makes creating validators much more intuitive.</p>

<blockquote>
<p>💡 <strong>Note:</strong> The <code>form()</code> and <code>validate()</code> APIs discussed here are part of the new Signal-based forms system.<br><br>
You can explore the implementation details in the Angular PR:<br><br>
👉 <a href="https://github.com/angular/angular/pull/64339/files" rel="noopener noreferrer">PR #64339 — Simplify Custom Errors</a></p>
</blockquote>


<h2>
  
  
  🔑 The Key Change: Before and After
</h2>

<p>Let's look at a direct comparison — the core of this improvement.</p>
<h3>
  
  
  Before 👎
</h3>

<p>Previously, to indicate a custom validation error, we often needed to import and use a utility function to <em>wrap</em> our error and ensure it was correctly typed and recognized by the forms system.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">customError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">some-forms-library</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">cat</span> <span class="o">=</span> <span class="nf">signal</span><span class="p">(</span><span class="dl">'</span><span class="s1">meow</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">f</span> <span class="o">=</span> <span class="nf">form</span><span class="p">(</span>
  <span class="nx">cat</span><span class="p">,</span>
  <span class="p">(</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">validate</span><span class="p">(</span><span class="nx">p</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// We needed the customError wrapper</span>
      <span class="k">if </span><span class="p">(</span><span class="nf">p</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">meow</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="nf">customError</span><span class="p">({</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">i am a custom error</span><span class="dl">'</span> <span class="p">});</span>
      <span class="p">}</span>
      <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
    <span class="p">});</span>
  <span class="p">});</span>
</code></pre>

</div>



<h1>
  
  
  🧩 Simplified Validator API in Angular Signal Forms
</h1>

<p>With the new simplified API, the validation engine is smart enough to understand a simple object as an error response.<br><br>
If the validator function returns an object, it's an error.<br><br>
If it returns <code>null</code> or <code>undefined</code>, it's valid.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">cat</span> <span class="o">=</span> <span class="nf">signal</span><span class="p">(</span><span class="dl">'</span><span class="s1">meow</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">f</span> <span class="o">=</span> <span class="nf">form</span><span class="p">(</span>
  <span class="nx">cat</span><span class="p">,</span>
  <span class="p">(</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">validate</span><span class="p">(</span><span class="nx">p</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// We simply return the error object</span>
      <span class="k">if </span><span class="p">(</span><span class="nf">p</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">meow</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">iAmACustomError</span><span class="dl">'</span> <span class="p">};</span> <span class="c1">// Much cleaner!</span>
      <span class="p">}</span>
      <span class="k">return</span> <span class="kc">null</span><span class="p">;</span> <span class="c1">// Valid</span>
    <span class="p">});</span>
  <span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  💡 Why Does This Change Matter?
</h2>

<h3>
  
  
  🚫 Less Boilerplate
</h3>

<p>You no longer need to import <code>customError</code> in every validator file.</p>

<h3>
  
  
  🧠 More Intuitive API
</h3>

<p>The flow feels natural — <em>"Is there an error? If so, return the error object."</em></p>

<h3>
  
  
  🧪 Easier Testing
</h3>

<p>Mocking or stubbing a validator is trivial when it just returns an object.</p>

<h3>
  
  
  ⚙️ Consistency
</h3>

<p>It aligns with how synchronous validators work in traditional reactive forms (<code>Validators.required</code> returns <code>{ required: true }</code>).</p>




<h2>
  
  
  🎯 Final Thoughts
</h2>

<p>While this might seem like a small change, it’s a perfect example of Angular’s ongoing effort to make APIs simpler, more consistent, and more developer-friendly.</p>

<p>Cleaner validators mean fewer imports, less friction, and a smoother DX overall — especially as Signal-based forms continue to mature.</p>

