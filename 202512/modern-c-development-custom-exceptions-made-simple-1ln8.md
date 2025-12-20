---
Title: Modern C# Development: Custom Exceptions Made Simple
Description: 
Author: Lou Creemers
Date: 2025-12-20T21:00:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hi lovely readers,</p>

<p>Errors happen. Sometimes they are simple, and sometimes they are very specific to your domain. C# gives us many built in exceptions, but they do not always tell the full story. That is where custom exceptions come in.</p>

<p>In this post, we are going to look at how to create your own custom exceptions in C#. We will keep it simple, practical, and easy to use in real projects.</p>

<h2>
  
  
  Why custom exceptions exist
</h2>

<p>Imagine you are building an ordering system. A user tries to place an order, but the business rules say no. You could throw something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">throw</span> <span class="k">new</span> <span class="nf">InvalidOperationException</span><span class="p">(</span><span class="s">"User cannot place this order"</span><span class="p">);</span>
</code></pre>

</div>



<p>This works, but it does not say much. Later on, when you read logs or handle errors, you have to rely on the message string.</p>

<p>Now compare it to this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">throw</span> <span class="k">new</span> <span class="nf">OrderNotAllowedException</span><span class="p">(</span><span class="n">userId</span><span class="p">);</span>
</code></pre>

</div>



<p>This tells a clear story. Something specific went wrong, and it belongs to your domain.</p>

<h2>
  
  
  Creating a basic custom exception
</h2>

<p>A custom exception is just a class that inherits from <code>Exception</code>.</p>

<p>Here is the smallest useful version:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">OrderNotAllowedException</span> <span class="p">:</span> <span class="n">Exception</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="nf">OrderNotAllowedException</span><span class="p">(</span><span class="kt">string</span> <span class="n">message</span><span class="p">)</span>
        <span class="p">:</span> <span class="k">base</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
    <span class="p">{</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>That is all you need to get started.</p>

<h2>
  
  
  Adding useful information
</h2>

<p>The real power of custom exceptions comes from adding context. Instead of putting everything into the message, you can add properties.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">OrderNotAllowedException</span> <span class="p">:</span> <span class="n">Exception</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">Guid</span> <span class="n">UserId</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="nf">OrderNotAllowedException</span><span class="p">(</span><span class="n">Guid</span> <span class="n">userId</span><span class="p">)</span>
        <span class="p">:</span> <span class="k">base</span><span class="p">(</span><span class="s">$"User </span><span class="p">{</span><span class="n">userId</span><span class="p">}</span><span class="s"> is not allowed to place an order"</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">UserId</span> <span class="p">=</span> <span class="n">userId</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now your logs and error handlers can access <code>UserId</code> directly. No string parsing needed. This makes debugging much easier.</p>

<h2>
  
  
  Do you need serialization constructors
</h2>

<p>In older .NET versions (.NET Framework 4.x and earlier), you often saw extra constructors for serialization. If you are using a modern version of .NET, a simple exception class like the ones above is enough.</p>

<h2>
  
  
  When not to create custom exceptions
</h2>

<p>Custom exceptions are useful, but they are easy to overuse.</p>

<p>Try to avoid these situations:</p>

<ul>
<li>Using exceptions for normal control flow
</li>
<li>Creating very generic custom exceptions that add no value
</li>
<li>Exposing internal exception details from public APIs
</li>
</ul>

<p>If a built in exception already describes the problem well, use it.</p>

<h2>
  
  
  A quick checklist
</h2>

<p>Before creating a custom exception, ask yourself:</p>

<ul>
<li>Does this represent a real exceptional case
</li>
<li>Does it add meaning beyond a message string
</li>
<li>Does the name clearly describe the problem
</li>
</ul>

<p>If the answer is yes, a custom exception is a good fit.</p>

<h2>
  
  
  That is a wrap
</h2>

<p>Custom exceptions are a small feature, but they can make your code much clearer and easier to maintain. Used well, they tell a story about what went wrong and why.</p>

<p>I hope this post gave you a quick introduction to creating your own exceptions in C#. If you have thoughts or questions, feel free to leave a comment or find me on <a href="https://louella.dev/socials" rel="noopener noreferrer">my socials</a>.</p>

<p>See ya!</p>

