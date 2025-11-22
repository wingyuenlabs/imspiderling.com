---
Title: New User Defined Increment and decrement in C# 14
Description: 
Author: Lou Creemers
Date: 2025-11-22T21:37:51.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hi lovely readers,</p>

<p>C# 14 just arrived with .NET 10, and it brings a fun little feature that makes custom types feel even more like built in ones. You can now decide what happens when someone writes <code>myValue++</code> or <code>myValue--</code> on your type.</p>

<p>This was possible before, but only with static methods that returned a new instance. C# 14 adds a new way to do it. You can now write instance operators that update the object directly. It is simple, it is clean, and it can help performance too.</p>

<p>In this post, we are going to take a friendly look at this feature and how you can try it out. Keep in mind that you need to have .NET 1- installed in order to test this out.</p>

<h2>
  
  
  What this feature gives you
</h2>

<p>If you have ever created a numeric style type, you probably have used <code>++</code> to increment the number by one, or <code>--</code> to decrement the number by one. Now you get to decide what these postfixes do, creating more freedom.</p>

<p>C# 14 still supports the classic static operator, but it also lets you write an instance version to change it for specific classes or structs like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">void</span> <span class="k">operator</span> <span class="p">++()</span>
<span class="p">{</span>
    <span class="c1">// Update this</span>
<span class="p">}</span>
</code></pre>

</div>



<p>and the same idea works for <code>--</code>.</p>

<h2>
  
  
  A simple example
</h2>

<p>Imagine you have a small type that keeps track of a score. Maybe every time the score grows, it should go up by 2 instead of 1.</p>

<p>Here is how you can do that:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">struct</span> <span class="nc">Score</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">int</span> <span class="n">Value</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="nf">Score</span><span class="p">(</span><span class="kt">int</span> <span class="k">value</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">Value</span> <span class="p">=</span> <span class="k">value</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="k">operator</span> <span class="p">++()</span>
    <span class="p">{</span>
        <span class="n">Value</span> <span class="p">+=</span> <span class="m">2</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="k">operator</span> <span class="p">--()</span>
    <span class="p">{</span>
        <span class="n">Value</span> <span class="p">-=</span> <span class="m">2</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now you can write:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">score</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">Score</span><span class="p">(</span><span class="m">10</span><span class="p">);</span>
<span class="n">score</span><span class="p">++;</span>
<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">score</span><span class="p">.</span><span class="n">Value</span><span class="p">);</span> <span class="c1">// 12</span>
</code></pre>

</div>



<p>Nice and smooth.</p>

<h3>
  
  
  What about <code>score++</code> inside expressions
</h3>

<p>When you write <code>score++</code> alone, the instance operator works fine. But if you write something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">old</span> <span class="p">=</span> <span class="n">score</span><span class="p">++;</span>
</code></pre>

</div>



<p>the compiler needs both the old value and the new one. For that, you also need the classic static operator.</p>

<p>Here is the complete version:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">struct</span> <span class="nc">Score</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">int</span> <span class="n">Value</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="nf">Score</span><span class="p">(</span><span class="kt">int</span> <span class="k">value</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">Value</span> <span class="p">=</span> <span class="k">value</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="k">operator</span> <span class="p">++()</span>
    <span class="p">{</span>
        <span class="n">Value</span> <span class="p">+=</span> <span class="m">2</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">static</span> <span class="n">Score</span> <span class="k">operator</span> <span class="p">++(</span><span class="n">Score</span> <span class="n">s</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nf">Score</span><span class="p">(</span><span class="n">s</span><span class="p">.</span><span class="n">Value</span> <span class="p">+</span> <span class="m">2</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>With both operators in place:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">score</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">Score</span><span class="p">(</span><span class="m">10</span><span class="p">);</span>
<span class="kt">var</span> <span class="n">old</span> <span class="p">=</span> <span class="n">score</span><span class="p">++;</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">old</span><span class="p">.</span><span class="n">Value</span><span class="p">);</span>   <span class="c1">// 10</span>
<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">score</span><span class="p">.</span><span class="n">Value</span><span class="p">);</span> <span class="c1">// 12</span>
</code></pre>

</div>



<p>Prefix uses the instance operator. Postfix inside an expression uses the static one. The type now behaves just like built in ones.</p>

<h2>
  
  
  Why this is nice
</h2>

<p><strong>Better performance</strong><br>
The instance operator updates the value directly, so structs avoid extra copies.</p>

<p><strong>Cleaner code</strong><br>
Your custom type feels more natural to work with.</p>

<p><strong>Full control</strong><br>
You decide how your type increments or decrements.</p>

<h2>
  
  
  That is a wrap
</h2>

<p>I hope this post helped you understand how user defined increment and decrement operators work in C# 14. If you give them a try, I would love to hear what you come up with. You can find me on almost every social platform at <a href="https://louella.dev/socials" rel="noopener noreferrer">louella.dev/socials</a>, or you can leave a comment below.</p>

<p>See ya!</p>

