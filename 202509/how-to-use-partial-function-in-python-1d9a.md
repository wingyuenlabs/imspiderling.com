---
Title: How to Use Partial Function in Python
Description: 
Author: Ankit malik
Date: 2025-09-16T21:16:44.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>In Python, functions are first-class objects, which means they can be passed around, modified, or even treated as variables. The <code>functools</code> module provides a tool called <strong>partial function</strong>, which allows you to fix a certain number of arguments of a function and generate a new function with fewer parameters. This is especially useful when working with repetitive function calls where many arguments remain the same.</p>

<h3>
  
  
  Use Cases
</h3>

<p>Partial functions are commonly used in scenarios where:</p>

<ul>
<li>
<strong>Pre-filling function parameters:</strong> You want to reuse a function with some arguments fixed, avoiding repeated input.</li>
<li>
<strong>Simplifying callbacks:</strong> Passing simplified functions to APIs, GUIs, or event-driven frameworks.</li>
<li>
<strong>Improving code readability:</strong> Defining specialized versions of general-purpose functions, such as formatting or mathematical calculations.</li>
<li>
<strong>Data processing pipelines:</strong> When mapping or filtering with functions that require multiple arguments, a partial function can simplify the structure.</li>
</ul>

<h3>
  
  
  Code Example with Explanation
</h3>

<p>Let’s explore an example.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">functools</span> <span class="kn">import</span> <span class="n">partial</span>

<span class="c1"># A normal function that multiplies two numbers
</span><span class="k">def</span> <span class="nf">multiply</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">x</span> <span class="o">*</span> <span class="n">y</span>

<span class="c1"># Creating a partial function to always multiply with 2
</span><span class="n">double</span> <span class="o">=</span> <span class="nf">partial</span><span class="p">(</span><span class="n">multiply</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>

<span class="c1"># Creating another partial function to always multiply with 5
</span><span class="n">five_times</span> <span class="o">=</span> <span class="nf">partial</span><span class="p">(</span><span class="n">multiply</span><span class="p">,</span> <span class="mi">5</span><span class="p">)</span>

<span class="c1"># Using the partial functions
</span><span class="nf">print</span><span class="p">(</span><span class="nf">double</span><span class="p">(</span><span class="mi">10</span><span class="p">))</span>       <span class="c1"># Output: 20 (2 * 10)
</span><span class="nf">print</span><span class="p">(</span><span class="nf">five_times</span><span class="p">(</span><span class="mi">4</span><span class="p">))</span>    <span class="c1"># Output: 20 (5 * 4)
</span></code></pre>

</div>



<p><strong>Explanation:</strong></p>

<ul>
<li>The function <code>multiply(x, y)</code> takes two arguments.
</li>
<li>By applying <code>partial(multiply, 2)</code>, we fix <code>x = 2</code>. The new function <code>double(y)</code> only needs one argument.
</li>
<li>Similarly, <code>five_times(y)</code> will always multiply numbers by 5.
</li>
<li>This approach eliminates rewriting separate functions for these common operations.</li>
</ul>

<p>Another real-world example with formatting:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">functools</span> <span class="kn">import</span> <span class="n">partial</span>

<span class="k">def</span> <span class="nf">format_text</span><span class="p">(</span><span class="n">text</span><span class="p">,</span> <span class="n">prefix</span><span class="p">,</span> <span class="n">suffix</span><span class="p">):</span>
    <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">prefix</span><span class="si">}{</span><span class="n">text</span><span class="si">}{</span><span class="n">suffix</span><span class="si">}</span><span class="sh">"</span>

<span class="c1"># Create specific formatters using partial functions
</span><span class="n">add_brackets</span> <span class="o">=</span> <span class="nf">partial</span><span class="p">(</span><span class="n">format_text</span><span class="p">,</span> <span class="n">prefix</span><span class="o">=</span><span class="sh">"</span><span class="s">[</span><span class="sh">"</span><span class="p">,</span> <span class="n">suffix</span><span class="o">=</span><span class="sh">"</span><span class="s">]</span><span class="sh">"</span><span class="p">)</span>
<span class="n">add_quotes</span> <span class="o">=</span> <span class="nf">partial</span><span class="p">(</span><span class="n">format_text</span><span class="p">,</span> <span class="n">prefix</span><span class="o">=</span><span class="sh">'"'</span><span class="p">,</span> <span class="n">suffix</span><span class="o">=</span><span class="sh">'"'</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="nf">add_brackets</span><span class="p">(</span><span class="sh">"</span><span class="s">Python</span><span class="sh">"</span><span class="p">))</span>   <span class="c1"># Output: [Python]
</span><span class="nf">print</span><span class="p">(</span><span class="nf">add_quotes</span><span class="p">(</span><span class="sh">"</span><span class="s">Hello</span><span class="sh">"</span><span class="p">))</span>      <span class="c1"># Output: "Hello"
</span></code></pre>

</div>



<p>This technique is particularly handy when creating specialized behaviors without redefining functions.</p>

<h3>
  
  
  Conclusion
</h3>

<p>Partial functions in Python, provided by the <code>functools</code> module, are a clean and efficient way to reduce code duplication. By fixing certain arguments of a function, they let you create customized versions of existing functions for specific use cases. Whether for callbacks, formatting, or data processing, partial functions help write cleaner and more reusable code.  </p>

