---
Title: Python by Structure - Class-Based Decorators That Remember
Description: 
Author: Aaron Rose
Date: 2025-11-21T20:49:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>Timothy was reviewing a performance monitoring system when he stopped on an unfamiliar pattern. "Margaret, this decorator isn't a function - it's a class. I've only ever seen decorators written as functions."</p>

<p>Margaret looked over. "Class-based decorators are powerful when you need your decorator to maintain state. What are you looking at?"<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">CallCounter</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">A decorator class to count function calls.</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">func</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">func</span> <span class="o">=</span> <span class="n">func</span>
        <span class="n">self</span><span class="p">.</span><span class="n">count</span> <span class="o">=</span> <span class="mi">0</span>

    <span class="k">def</span> <span class="nf">__call__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">count</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">-&gt; Calling </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">func</span><span class="p">.</span><span class="n">__name__</span><span class="si">}</span><span class="s">. Call count: </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">count</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">func</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>

<span class="nd">@CallCounter</span>
<span class="k">def</span> <span class="nf">process_data</span><span class="p">(</span><span class="n">data</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Processes the input data.</span><span class="sh">"""</span>
    <span class="k">return</span> <span class="n">data</span><span class="p">.</span><span class="nf">upper</span><span class="p">()</span>

<span class="nf">print</span><span class="p">(</span><span class="nf">process_data</span><span class="p">(</span><span class="sh">"</span><span class="s">hello</span><span class="sh">"</span><span class="p">))</span>
<span class="nf">print</span><span class="p">(</span><span class="nf">process_data</span><span class="p">(</span><span class="sh">"</span><span class="s">world</span><span class="sh">"</span><span class="p">))</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Total calls: </span><span class="si">{</span><span class="n">process_data</span><span class="p">.</span><span class="n">count</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>"See how <code>CallCounter</code> is a class, not a function?" Timothy asked. "How does that even work as a decorator?"</p>

<h2>
  
  
  The Problem: Decorators That Need Memory
</h2>

<p>"Function decorators work fine for most cases," Margaret said. "But what if you need your decorator to track something across multiple calls? Functions don't naturally hold state between invocations."</p>

<p>She pointed at the structure:</p>

<p><strong>Tree View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class CallCounter
    'A decorator class to count function calls.'
    __init__(self, func)
        self.func = func
        self.count = 0
    __call__(self)
        self.count += 1
        print(f'-&gt; Calling {self.func.__name__}. Call count: {self.count}')
        Return self.func(*args, **kwargs)

@CallCounter
process_data(data)
    'Processes the input data.'
    Return data.upper()

print(process_data('hello'))
print(process_data('world'))
print(f'Total calls: {process_data.count}')
</code></pre>

</div>



<p><strong>English View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Class CallCounter:
  Evaluate 'A decorator class to count function calls.'.
  Function __init__(self, func):
    Set self.func to func.
    Set self.count to 0.
  Function __call__(self):
    self.count += 1.
    Evaluate print(f'-&gt; Calling {self.func.__name__}. Call count: {self.count}').
    Return self.func(*args, **kwargs).

Decorator @CallCounter
Function process_data(data):
  Evaluate 'Processes the input data.'.
  Return data.upper().

Evaluate print(process_data('hello')).
Evaluate print(process_data('world')).
Evaluate print(f'Total calls: {process_data.count}').
</code></pre>

</div>



<h2>
  
  
  How Class Decorators Work
</h2>

<p>"Look at the structure," Margaret said. "When Python sees <code>@CallCounter</code> above <code>process_data</code>, it does the same thing as with function decorators: <code>process_data = CallCounter(process_data)</code>."</p>

<p>Timothy traced through it. "So <code>CallCounter.__init__</code> receives the original <code>process_data</code> function and stores it in <code>self.func</code>?"</p>

<p>"Exactly. And it initializes <code>self.count</code> to zero. Now <code>process_data</code> isn't a function anymore - it's an instance of <code>CallCounter</code>."</p>

<p>"But how do we call it? We're calling <code>process_data('hello')</code> like it's still a function."</p>

<p>Margaret pointed to <code>__call__</code>. "The magic method <code>__call__</code> makes any object callable. When you write <code>process_data('hello')</code>, Python sees that <code>process_data</code> is a <code>CallCounter</code> instance and calls its <code>__call__</code> method."</p>

<p>Timothy worked through the execution:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>-&gt; Calling process_data. Call count: 1
HELLO
-&gt; Calling process_data. Call count: 2
WORLD
Total calls: 2
</code></pre>

</div>



<p>"So each time we call <code>process_data</code>, the <code>__call__</code> method runs, increments <code>self.count</code>, and then calls the original function stored in <code>self.func</code>?"</p>

<p>"Precisely. The class instance persists between calls, so <code>self.count</code> accumulates. You can't do that easily with a function decorator."</p>

<h2>
  
  
  When to Use Class-Based Decorators
</h2>

<p>"So when would I choose a class over a function for my decorator?" Timothy asked.</p>

<p>"Use a class-based decorator when you need to track state across multiple calls, store configuration that might change, implement multiple related decorators that share behavior, or need the decorator itself to have methods you can call later."</p>

<p>Timothy nodded. "The structure makes it clear - <code>__init__</code> sets up the decorator, <code>__call__</code> handles each invocation. The class holds everything together."</p>

<p>"That's the pattern," Margaret confirmed. "Class-based decorators give you an object with state, methods, and attributes. Function decorators are simpler for stateless transformations. Choose the right tool for your needs."</p>




<p><strong>Analyze Python structure yourself:</strong> Download the <a href="https://www.pacificw.com" rel="noopener noreferrer">Python Structure Viewer</a> - a free tool that shows code structure in tree and plain English views. Works offline, no installation required.</p>




<p><em>Aaron Rose is a software engineer and technology writer at <a href="https://www.tech-reader.blog" rel="noopener noreferrer">tech-reader.blog</a> and the author of <a href="https://amazon.com/author/aaron.rose" rel="noopener noreferrer">Think Like a Genius</a>.</em></p>

