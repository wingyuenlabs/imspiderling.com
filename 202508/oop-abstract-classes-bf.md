---
Title: OOP - Abstract Classes
Description: What are abstract classes?


If you've been using or learning OOP for some time, you would've heard the words abstract classes being thrown about. I have found that those who are self-taught generally...
Author: Carl Beattie
Date: 2025-08-13T00:28:05.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What are abstract classes?
</h2>

<p>If you've been using or learning OOP for some time, you would've heard the words abstract classes being thrown about. I have found that those who are self-taught generally miss or skip the sections that focus on all aspects of abstraction. I think this is mainly down to its use case shining when working on large codebases and gigantic teams.</p>

<p>Unlike regular classes, abstract classes can't be instantiated, although they are allowed to contain concrete methods and properties. Classes that inherit from an abstract class must implement all abstract members.</p>

<p>Something to consider is that you don't always need abstract classes; sometimes, interfaces will be the better option. The general rule of thumb is that if in your abstract class you've just created has no state, it should be an interface. C# has a special <code>abstract</code> keyword for creating abstract classes.</p>

<p>Let's look at a quick example.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">abstract</span> <span class="k">class</span> <span class="nc">Animal</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">bool</span> <span class="n">isHungry</span> <span class="p">=</span> <span class="k">false</span><span class="p">;</span>

    <span class="k">public</span> <span class="k">abstract</span> <span class="k">void</span> <span class="nf">Eat</span><span class="p">();</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">Cow</span> <span class="p">:</span> <span class="n">Animal</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">Eat</span><span class="p">()</span>
    <span class="p">{</span>
       <span class="c1">// Implement how the Cow will eat its food. </span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>I am creating these articles for my Advanced OOP roadmap for GameDev, which you can visit <a href="https://roadmap.sh/r/oop-hiwo8" rel="noopener noreferrer">here</a> over on roadmap.sh</p>
</blockquote>

