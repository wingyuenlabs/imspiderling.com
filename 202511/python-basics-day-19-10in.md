---
Title: Python basics - Day 19
Description: 
Author: Sabin Sim
Date: 2025-11-03T21:44:08.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <strong>Day 19 – Modules &amp; Packages (import, from, <strong>main</strong>)</strong>
</h2>

<p><strong>Project:</strong> Build a modular “Math &amp; Greeting Toolkit” that imports and reuses your own functions.</p>




<p><strong>01. Learning Goal</strong></p>

<p>By the end of this lesson, you will be able to:</p>

<ul>
<li>Understand what <strong>modules</strong> and <strong>packages</strong> are
</li>
<li>Import both built-in and custom Python modules
</li>
<li>Use <strong>from-import</strong>, <strong>aliasing</strong>, and <strong><strong>main</strong></strong> effectively
</li>
<li>Organize your code for <strong>reusability and clarity</strong>
</li>
</ul>




<p><strong>02. Problem Scenario</strong></p>

<p>As your projects grow, you’ll want to organize your code into smaller, reusable files.<br><br>
Instead of putting everything into one Python file, you’ll learn to use <strong>modules</strong> and <strong>packages</strong> to make your project scalable and clean.</p>



<p><strong>03. Step 1 – What is a Module?</strong></p>

<p>A <strong>module</strong> is simply a Python file (<code>.py</code>) that contains functions, classes, or variables.<br><br>
You can import it into another file to reuse its code.</p>

<p>Example: using Python’s built-in <code>math</code> module<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">math</span>

<span class="nf">print</span><span class="p">(</span><span class="n">math</span><span class="p">.</span><span class="nf">sqrt</span><span class="p">(</span><span class="mi">16</span><span class="p">))</span>   <span class="c1"># 4.0
</span><span class="nf">print</span><span class="p">(</span><span class="n">math</span><span class="p">.</span><span class="n">pi</span><span class="p">)</span>         <span class="c1"># 3.141592...
</span></code></pre>

</div>






<p><strong>04. Step 2 – Different Ways to Import</strong></p>

<p>You can import modules in multiple ways depending on your use case.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Import the entire module
</span><span class="kn">import</span> <span class="n">math</span>
<span class="nf">print</span><span class="p">(</span><span class="n">math</span><span class="p">.</span><span class="nf">sqrt</span><span class="p">(</span><span class="mi">25</span><span class="p">))</span>

<span class="c1"># Import specific functions
</span><span class="kn">from</span> <span class="n">math</span> <span class="kn">import</span> <span class="n">sqrt</span><span class="p">,</span> <span class="n">pi</span>
<span class="nf">print</span><span class="p">(</span><span class="nf">sqrt</span><span class="p">(</span><span class="mi">25</span><span class="p">),</span> <span class="n">pi</span><span class="p">)</span>

<span class="c1"># Use an alias for convenience
</span><span class="kn">import</span> <span class="n">math</span> <span class="k">as</span> <span class="n">m</span>
<span class="nf">print</span><span class="p">(</span><span class="n">m</span><span class="p">.</span><span class="nf">sqrt</span><span class="p">(</span><span class="mi">36</span><span class="p">))</span>
</code></pre>

</div>






<p><strong>05. Step 3 – Creating Your Own Module</strong></p>

<p>You can create your own Python file and reuse it as a module.</p>

<p><strong>calc.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">add</span><span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">a</span> <span class="o">+</span> <span class="n">b</span>

<span class="k">def</span> <span class="nf">sub</span><span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">a</span> <span class="o">-</span> <span class="n">b</span>
</code></pre>

</div>



<p><strong>main.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">calc</span>

<span class="nf">print</span><span class="p">(</span><span class="n">calc</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="mi">5</span><span class="p">))</span>   <span class="c1"># 8
</span><span class="nf">print</span><span class="p">(</span><span class="n">calc</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span> <span class="mi">7</span><span class="p">))</span>  <span class="c1"># 3
</span></code></pre>

</div>



<p>When you import <code>calc</code>, Python automatically runs that file once and gives access to its functions.</p>




<p><strong>06. Step 4 – What is a Package?</strong></p>

<p>A <strong>package</strong> is a folder that contains multiple modules.<br>
It must include an <code>__init__.py</code> file to be recognized as a package.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>my_package/
    __init__.py
    math_ops.py
    string_ops.py
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">my_package</span> <span class="kn">import</span> <span class="n">math_ops</span>
<span class="nf">print</span><span class="p">(</span><span class="n">math_ops</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">))</span>
</code></pre>

</div>



<p>Packages make it easy to group related modules together for large-scale projects.</p>




<p><strong>07. Step 5 – The <code>__name__</code> and <code>__main__</code> Check</strong></p>

<p>Every Python file can act as either:</p>

<ol>
<li>
<strong>A script</strong> (run directly)</li>
<li>
<strong>A module</strong> (imported into another file)</li>
</ol>

<p>You can distinguish them using <code>if __name__ == "__main__":</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># test.py
</span><span class="k">def</span> <span class="nf">hello</span><span class="p">():</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Hello!</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">hello</span><span class="p">()</span>
</code></pre>

</div>



<p>When executed directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>python test.py
Hello!
</code></pre>

</div>



<p>When imported:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">test</span>
<span class="c1"># Does not print automatically
</span></code></pre>

</div>



<p>This allows you to include test code or demos that run <strong>only when executed directly</strong>.</p>




<p><strong>08. Step 6 – Installing External Libraries</strong></p>

<p>You can install third-party libraries using <strong>pip</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>requests
</code></pre>

</div>



<p>Then, use it in your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>

<span class="n">res</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">https://api.github.com</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">res</span><span class="p">.</span><span class="n">status_code</span><span class="p">)</span>
</code></pre>

</div>






<p><strong>09. Step 7 – Practice Examples</strong></p>

<p><strong>Example 1: Standard Module</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">random</span>

<span class="nf">print</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">))</span>   <span class="c1"># Roll a dice
</span></code></pre>

</div>






<p><strong>Example 2: Custom Module</strong></p>

<p><strong>greet.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">say_hello</span><span class="p">(</span><span class="n">name</span><span class="p">):</span>
    <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Hello, </span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="s">!</span><span class="sh">"</span>
</code></pre>

</div>



<p><strong>main.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">greet</span>
<span class="nf">print</span><span class="p">(</span><span class="n">greet</span><span class="p">.</span><span class="nf">say_hello</span><span class="p">(</span><span class="sh">"</span><span class="s">Sabin</span><span class="sh">"</span><span class="p">))</span>
</code></pre>

</div>






<p><strong>10. Step 8 – Mini Project: Math &amp; Greeting Toolkit</strong></p>

<p>Let’s build a simple package that combines math and greeting modules.</p>

<p><strong>project structure</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>toolkit/
    __init__.py
    math_tools.py
    greet_tools.py
main.py
</code></pre>

</div>



<p><strong>math_tools.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">add</span><span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">a</span> <span class="o">+</span> <span class="n">b</span>

<span class="k">def</span> <span class="nf">multiply</span><span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">a</span> <span class="o">*</span> <span class="n">b</span>
</code></pre>

</div>



<p><strong>greet_tools.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">greet</span><span class="p">(</span><span class="n">name</span><span class="p">):</span>
    <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Welcome, </span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="s">!</span><span class="sh">"</span>
</code></pre>

</div>



<p><strong>main.py</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">toolkit</span> <span class="kn">import</span> <span class="n">math_tools</span><span class="p">,</span> <span class="n">greet_tools</span>

<span class="nf">print</span><span class="p">(</span><span class="n">math_tools</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span> <span class="mi">5</span><span class="p">))</span>
<span class="nf">print</span><span class="p">(</span><span class="n">greet_tools</span><span class="p">.</span><span class="nf">greet</span><span class="p">(</span><span class="sh">"</span><span class="s">Sabin</span><span class="sh">"</span><span class="p">))</span>
</code></pre>

</div>



<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>15
Welcome, Sabin!
</code></pre>

</div>






<p><strong>11. Reflection</strong></p>

<p>You have learned how to:</p>

<ul>
<li>Import built-in and custom <strong>modules</strong>
</li>
<li>Organize multiple files into <strong>packages</strong>
</li>
<li>Use <code>__main__</code> to separate test code from imports</li>
<li>Build your own <strong>Math &amp; Greeting Toolkit</strong> for reusable code</li>
</ul>

<p>Next → <strong>Day 20 – Python Standard Library</strong><br>
Explore the most useful built-in modules like <code>math</code>, <code>random</code>, <code>datetime</code>, and <code>os</code>.</p>

