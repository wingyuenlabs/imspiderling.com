---
Title: TypeScript Mutually Exclusive Unions: From `?: never` to `ExclusifyUnion`
Description: 
Author: YCM Jason
Date: 2025-11-06T22:02:35.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Mutually Exclusive Unions in TypeScript (and why <code>ExclusifyUnion</code> rocks)
</h1>

<p>When modelling data, we often want either one shape or another, never both at once. This is the essence of a <em>mutually exclusive union</em>. In TypeScript, that means writing unions where accessing properties becomes ergonomic <em>without</em> defensive checks everywhere.</p>

<p>This post shows a neat pattern using <code>?: never</code>, why it doesn’t scale, and how <strong>type-fest</strong>’s <code>ExclusifyUnion</code> helper elegantly solves this problem.</p>

<h2>
  
  
  The basic problem
</h2>

<p>Mutually exclusive unions help us distinguish the shape of a given object. Take the following example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Dog</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">bark</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Cat</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">meow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">vocalize</span> <span class="o">=</span> <span class="p">(</span><span class="nx">animal</span><span class="p">:</span> <span class="nx">Dog</span> <span class="o">|</span> <span class="nx">Cat</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="dl">"</span><span class="s2">bark</span><span class="dl">"</span> <span class="k">in</span> <span class="nx">animal</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">animal</span><span class="p">.</span><span class="nf">bark</span><span class="p">();</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">animal</span><span class="p">.</span><span class="nf">meow</span><span class="p">();</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Once we’ve checked that <code>bark</code> or <code>meow</code> exists on <code>animal</code>, TypeScript narrows the union and everything is fine.</p>

<p>But the ergonomics in <code>vocalize</code> aren’t ideal. Intuitively, we’d love to destructure and call whichever function exists:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">vocalize</span> <span class="o">=</span> <span class="p">({</span> <span class="nx">bark</span><span class="p">,</span> <span class="nx">meow</span> <span class="p">}:</span> <span class="nx">Dog</span> <span class="o">|</span> <span class="nx">Cat</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">//                ^     ^</span>
  <span class="c1">// Property 'meow' does not exist on type 'Dog | Cat'.</span>
  <span class="c1">// Property 'bark' does not exist on type 'Dog | Cat'.</span>
  <span class="k">return </span><span class="p">(</span><span class="nx">bark</span> <span class="o">??</span> <span class="nx">meow</span><span class="p">)();</span>
<span class="p">};</span>
</code></pre>

</div>



<p>TypeScript complains because you’re trying to access <code>bark</code> or <code>meow</code> <em>before</em> the union has been narrowed.</p>

<h2>
  
  
  The <code>?: never</code> trick
</h2>

<p>A common workaround is to add the <em>other</em> member’s exclusive properties as <code>?: never</code>. This hints to the type system that a property <em>might</em> be accessed, even if it doesn’t exist on this branch, so narrowing through destructuring becomes possible.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Dog</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">bark</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">meow</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Cat</span> <span class="o">=</span> <span class="p">{</span>
  <span class="nx">bark</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">meow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">vocalize</span> <span class="o">=</span> <span class="p">({</span> <span class="nx">bark</span><span class="p">,</span> <span class="nx">meow</span> <span class="p">}:</span> <span class="nx">Dog</span> <span class="o">|</span> <span class="nx">Cat</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span><span class="nx">bark</span> <span class="o">??</span> <span class="nx">meow</span><span class="p">)();</span>
<span class="p">};</span>

</code></pre>

</div>



<p>Now destructuring works, and TypeScript still understands that <strong>exactly one</strong> of <code>bark</code> or <code>meow</code> exists at a time (an exclusive-or).</p>

<h2>
  
  
  Scaling This Pattern
</h2>

<p>The <code>?: never</code> trick is fine for small unions, but it quickly becomes tedious when you introduce more types, each with many unique fields. Imagine adding more animals:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Dog</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">bark</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">wagTail</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">fetch</span><span class="p">:</span> <span class="p">(</span><span class="na">item</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">guard</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>

  <span class="c1">// Cat props</span>
  <span class="nl">meow</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">purr</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">hiss</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>

  <span class="c1">// Bird props</span>
  <span class="nl">chirp</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">fly</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Cat</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">meow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">purr</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">hiss</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>

  <span class="c1">// Dog props</span>
  <span class="nl">bark</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">wagTail</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">fetch</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">guard</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>

  <span class="c1">// Bird props</span>
  <span class="nl">chirp</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">fly</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Bird</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">chirp</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">fly</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>

  <span class="c1">// Dog props</span>
  <span class="nl">bark</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">wagTail</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">fetch</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">guard</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>

  <span class="c1">// Cat props</span>
  <span class="nl">meow</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">purr</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">hiss</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
<span class="p">};</span>

</code></pre>

</div>



<p>Now we’re manually repeating a growing list of <code>?: never</code> properties for every new type. As the union grows, maintaining all these exclusions becomes a nightmare of boilerplate.</p>

<h2>
  
  
  Introducing <code>ExclusifyUnion</code>
</h2>

<p>This is exactly where <a href="https://github.com/sindresorhus/type-fest" rel="noopener noreferrer">type-fest</a>’s <a href="https://github.com/sindresorhus/type-fest/pull/1278" rel="noopener noreferrer"><code>ExclusifyUnion</code></a> comes in (which I originally proposed <a href="https://github.com/sindresorhus/type-fest/issues/1275" rel="noopener noreferrer">here</a>). It automates the process of adding <code>?: never</code> based on your union type.</p>

<p>With <code>ExclusifyUnion</code>, you can keep your types clean and simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Dog</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">bark</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">wagTail</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">fetch</span><span class="p">:</span> <span class="p">(</span><span class="na">item</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">guard</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Cat</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">meow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">purr</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">hiss</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Bird</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">chirp</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">fly</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">interact</span> <span class="o">=</span> <span class="p">(</span>
  <span class="p">{</span> <span class="nx">fetch</span><span class="p">,</span> <span class="nx">chirp</span> <span class="p">}:</span> <span class="nx">ExclusifyUnion</span><span class="o">&lt;</span><span class="nx">Dog</span> <span class="o">|</span> <span class="nx">Cat</span> <span class="o">|</span> <span class="nx">Bird</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="nx">thing</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
<span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span><span class="nx">fetch</span> <span class="o">??</span> <span class="nx">chirp</span><span class="p">)?.(</span><span class="nx">thing</span><span class="p">);</span>
<span class="p">};</span>

</code></pre>

</div>



<p>That’s it. No manual exclusions, no verbosity, just clean, exclusive union types that <em>work</em>.</p>

<h2>
  
  
  A familiar example: Zod’s <code>safeParse</code>
</h2>

<p>Zod’s <code>safeParse</code> result is a mutually exclusive union shaped to be ergonomic without extra checks. Conceptually:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">SafeParseResult</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span>
  <span class="o">|</span> <span class="p">{</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">;</span> <span class="nl">data</span><span class="p">:</span> <span class="nx">T</span> <span class="p">}</span>
  <span class="o">|</span> <span class="p">{</span> <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">;</span> <span class="nl">error</span><span class="p">:</span> <span class="nb">Error</span> <span class="p">};</span>

</code></pre>

</div>



<p>In Zod, the types are defined more strictly to allow destructuring:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">ZodSafeParseSuccess</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">;</span>
  <span class="nl">data</span><span class="p">:</span> <span class="nx">T</span><span class="p">;</span>
  <span class="nl">error</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">ZodSafeParseError</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">;</span>
  <span class="nl">data</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">error</span><span class="p">:</span> <span class="nb">Error</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">ZodSafeParseResult</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span>
  <span class="o">|</span> <span class="nx">ZodSafeParseSuccess</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span>
  <span class="o">|</span> <span class="nx">ZodSafeParseError</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">;</span>

</code></pre>

</div>



<p>Which enables this pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">f</span> <span class="o">=</span> <span class="p">(</span><span class="nx">x</span><span class="p">:</span> <span class="nx">unknown</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span><span class="p">,</span> <span class="nx">error</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">safeParse</span><span class="p">(</span><span class="nx">x</span><span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">data</span><span class="p">;</span> <span class="c1">// string</span>
  <span class="p">}</span>
<span class="p">};</span>

</code></pre>

</div>



<h3>
  
  
  Key Takeaways
</h3>

<ul>
<li>
<p><strong>Mutually exclusive shapes</strong> help TypeScript distinguish object structures cleanly. For example:<br>
</p>
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Dog</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">bark</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Cat</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">meow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>


<p>These ensure that only one of the two shapes applies at any given time.</p>
</li>
<li>
<p>Adding <code>prop?: never</code> to the opposite properties gives you ergonomic access to any properties without confusing TypeScript’s type narrowing.:<br>
</p>
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Dog</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">bark</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="nl">meow</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Cat</span> <span class="o">=</span> <span class="p">{</span>
  <span class="nx">bark</span><span class="p">?:</span> <span class="nx">never</span><span class="p">;</span>
  <span class="nl">meow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</li>
<li><p>However, this approach <strong>doesn’t scale</strong> when your union includes many types or each type has many unique fields. The boilerplate grows fast.</p></li>
<li><p><strong>type-fest</strong>’s <code>ExclusifyUnion</code> solves this elegantly, combining the clarity of clean domain types with the convenience of destructuring. It automatically enforces exclusivity across union members—no more repetitive <code>?: never</code> definitions.</p></li>
</ul>

