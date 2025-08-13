---
Title: OOP Abstraction
Description: What is abstraction?


I'll keep it fairly short. Abstraction focuses on simplifying complex systems by modeling classes based on essential characteristics, hiding unnecessary details from the user.
S...
Author: Carl Beattie
Date: 2025-08-13T00:09:31.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What is abstraction?
</h2>

<p>I'll keep it fairly short. Abstraction focuses on simplifying complex systems by modeling classes based on essential characteristics, hiding unnecessary details from the user.</p>

<p>Someone who buys a car does not necessarily need to know how to fix it; that will be abstracted to a mechanic. The mechanic is not required to know how to design the car; that's abstracted to the engineers.</p>

<p>Let's look at an example (Inheritance understanding is assumed)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Vehicle</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">float</span> <span class="n">speed</span> <span class="p">=</span> <span class="m">0f</span><span class="p">;</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">Accelerate</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="c1">// Physics calculations to accelerate the vehicle</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">Decelerate</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="c1">// Physics calculations to decelerate the vehicle</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">Car</span> <span class="p">:</span> <span class="n">Vehicle</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">int</span> <span class="n">doors</span><span class="p">;</span>
    <span class="k">private</span> <span class="n">Color</span> <span class="n">color</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">Car</span><span class="p">(</span><span class="kt">int</span> <span class="n">doors</span><span class="p">,</span> <span class="n">Color</span> <span class="n">color</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="n">doors</span> <span class="p">=</span> <span class="n">doors</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="n">color</span> <span class="p">=</span> <span class="n">color</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">SomeCarModel</span> <span class="p">:</span> <span class="n">Car</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">string</span> <span class="n">name</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">SomeCarModel</span><span class="p">(</span><span class="kt">string</span> <span class="n">name</span><span class="p">,</span> <span class="kt">int</span> <span class="n">doors</span><span class="p">,</span> <span class="n">Color</span> <span class="n">color</span><span class="p">)</span> <span class="p">:</span> <span class="k">base</span><span class="p">(</span><span class="n">doors</span><span class="p">,</span> <span class="n">color</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="n">name</span> <span class="p">=</span> <span class="n">name</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">CarSpawner</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="nf">CarSpawner</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="n">SomeCarModel</span> <span class="n">someCarModel</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">SomeCarModel</span><span class="p">(</span><span class="s">"someCar"</span><span class="p">,</span> <span class="m">4</span><span class="p">,</span> <span class="k">new</span> <span class="nf">Color</span><span class="p">(</span><span class="m">100</span><span class="p">,</span> <span class="m">100</span><span class="p">,</span> <span class="m">100</span><span class="p">));</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">userInput</span><span class="p">.</span><span class="n">w</span><span class="p">)</span>
        <span class="p">{</span>
          <span class="n">someCarModel</span><span class="p">.</span><span class="nf">Accelerate</span><span class="p">();</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now, someone can add a new car without worrying about creating the logic to make the car move; we have <strong>abstracted</strong> that away.</p>

<p>Other abstraction topics we will focus on in the future are:</p>

<ul>
<li>Data Abstraction</li>
<li>Control Abstraction</li>
<li>Abstraction Levels</li>
<li>Abstract Classes</li>
</ul>

<blockquote>
<p>I am creating these articles for my Advanced OOP roadmap for GameDev, which you can visit <a href="https://roadmap.sh/r/oop-hiwo8" rel="noopener noreferrer">here</a> over on roadmap.sh</p>
</blockquote>

