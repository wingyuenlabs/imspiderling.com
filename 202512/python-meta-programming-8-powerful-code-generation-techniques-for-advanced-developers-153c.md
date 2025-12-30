---
Title: Python Meta-Programming: 8 Powerful Code Generation Techniques for Advanced Developers
Description: 
Author: Nithin Bharadwaj
Date: 2025-12-29T21:23:38.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>As a best-selling author, I invite you to explore my books on <a href="https://www.amazon.com/stores/Aarav-Joshi/author/B0DQYNVXZ7?ref=ap_rdr&amp;isDramIntegrated=true&amp;shoppingPortalEnabled=true&amp;ccs_id=738636bd-0ca1-4d7b-8efa-481bfc222571" rel="noopener noreferrer">Amazon</a>. Don't forget to follow me on <a href="https://medium.com/@aarav-joshi" rel="noopener noreferrer">Medium</a> and show your support. Thank you! Your support means the world! </p>
</blockquote>

<p>Let's talk about code that writes code. It sounds like something from science fiction, but in Python, it's a practical set of tools you can use today. I'm going to show you eight methods that let your programs examine, modify, and even create other pieces of code. This isn't about being clever for the sake of it. It's about solving real problems, like removing tedious repetition, building adaptable systems, and giving your programs a new level of flexibility.</p>

<p>Think about a time you've written the same pattern of code over and over. Maybe it's checking inputs, logging function calls, or managing resources. Meta-programming offers a way out of that loop. It helps you build the tools that write the boring parts for you, so you can focus on the unique logic of your application.</p>

<p>We'll start with one of the most common and useful entry points.</p>

<p>Decorators are like wrappers you can put around your functions. They let you add behavior before and after the function runs without touching the function's own code. It's perfect for tasks you want to apply to many different functions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">time</span>
<span class="kn">from</span> <span class="n">functools</span> <span class="kn">import</span> <span class="n">wraps</span>

<span class="k">def</span> <span class="nf">timing_decorator</span><span class="p">(</span><span class="n">func</span><span class="p">):</span>
    <span class="nd">@wraps</span><span class="p">(</span><span class="n">func</span><span class="p">)</span>
    <span class="k">def</span> <span class="nf">wrapper</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="n">start_time</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span>
        <span class="n">result</span> <span class="o">=</span> <span class="nf">func</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="n">end_time</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">func</span><span class="p">.</span><span class="n">__name__</span><span class="si">}</span><span class="s"> executed in </span><span class="si">{</span><span class="n">end_time</span> <span class="o">-</span> <span class="n">start_time</span><span class="si">:</span><span class="p">.</span><span class="mi">4</span><span class="n">f</span><span class="si">}</span><span class="s"> seconds</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">result</span>
    <span class="k">return</span> <span class="n">wrapper</span>

<span class="k">def</span> <span class="nf">retry_decorator</span><span class="p">(</span><span class="n">max_attempts</span><span class="o">=</span><span class="mi">3</span><span class="p">,</span> <span class="n">delay</span><span class="o">=</span><span class="mi">1</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">decorator</span><span class="p">(</span><span class="n">func</span><span class="p">):</span>
        <span class="nd">@wraps</span><span class="p">(</span><span class="n">func</span><span class="p">)</span>
        <span class="k">def</span> <span class="nf">wrapper</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
            <span class="n">last_exception</span> <span class="o">=</span> <span class="bp">None</span>
            <span class="k">for</span> <span class="n">attempt</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">max_attempts</span><span class="p">):</span>
                <span class="k">try</span><span class="p">:</span>
                    <span class="k">return</span> <span class="nf">func</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
                <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
                    <span class="n">last_exception</span> <span class="o">=</span> <span class="n">e</span>
                    <span class="k">if</span> <span class="n">attempt</span> <span class="o">&lt;</span> <span class="n">max_attempts</span> <span class="o">-</span> <span class="mi">1</span><span class="p">:</span>
                        <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">delay</span><span class="p">)</span>
            <span class="k">raise</span> <span class="n">last_exception</span>
        <span class="k">return</span> <span class="n">wrapper</span>
    <span class="k">return</span> <span class="n">decorator</span>

<span class="nd">@timing_decorator</span>
<span class="nd">@retry_decorator</span><span class="p">(</span><span class="n">max_attempts</span><span class="o">=</span><span class="mi">3</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">api_call</span><span class="p">(</span><span class="n">endpoint</span><span class="p">):</span>
    <span class="c1"># Simulate an unreliable API
</span>    <span class="k">if</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span> <span class="o">%</span> <span class="mi">2</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">:</span>
        <span class="k">raise</span> <span class="nc">ConnectionError</span><span class="p">(</span><span class="sh">"</span><span class="s">Temporary failure</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Data from </span><span class="si">{</span><span class="n">endpoint</span><span class="si">}</span><span class="sh">"</span>

<span class="n">result</span> <span class="o">=</span> <span class="nf">api_call</span><span class="p">(</span><span class="sh">"</span><span class="s">/users</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Result: </span><span class="si">{</span><span class="n">result</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>Here, <code>api_call</code> gets two new behaviors. The <code>retry_decorator</code> will try it up to three times if it fails. The <code>timing_decorator</code> then records how long the whole process took. You can stack these, and more importantly, you can apply them to any other function with a simple <code>@</code> symbol. I use this pattern constantly for adding logging, permission checks, or caching.</p>

<p>While decorators change functions and classes after they're made, sometimes you need to change the blueprint itself. That's where metaclasses come in. A metaclass is like a template for creating classes. It lets you intervene at the moment a class is defined.</p>

<p>Imagine you need a guarantee that only one instance of a particular class ever exists—a Singleton. A metaclass can enforce this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">SingletonMeta</span><span class="p">(</span><span class="nb">type</span><span class="p">):</span>
    <span class="n">_instances</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">__call__</span><span class="p">(</span><span class="n">cls</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">cls</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instances</span><span class="p">:</span>
            <span class="n">cls</span><span class="p">.</span><span class="n">_instances</span><span class="p">[</span><span class="n">cls</span><span class="p">]</span> <span class="o">=</span> <span class="nf">super</span><span class="p">().</span><span class="nf">__call__</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instances</span><span class="p">[</span><span class="n">cls</span><span class="p">]</span>

<span class="k">class</span> <span class="nc">DatabaseConnection</span><span class="p">(</span><span class="n">metaclass</span><span class="o">=</span><span class="n">SingletonMeta</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">connection_string</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">connection_string</span> <span class="o">=</span> <span class="n">connection_string</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Connecting to </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">connection_string</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Both variables point to the exact same object
</span><span class="n">db1</span> <span class="o">=</span> <span class="nc">DatabaseConnection</span><span class="p">(</span><span class="sh">"</span><span class="s">postgresql://localhost/db</span><span class="sh">"</span><span class="p">)</span>
<span class="n">db2</span> <span class="o">=</span> <span class="nc">DatabaseConnection</span><span class="p">(</span><span class="sh">"</span><span class="s">postgresql://localhost/db</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Same instance? </span><span class="si">{</span><span class="n">db1</span> <span class="ow">is</span> <span class="n">db2</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Output: True
</span></code></pre>

</div>



<p>The <code>SingletonMeta</code> metaclass stores instances in a dictionary. When you try to call <code>DatabaseConnection()</code> to create an object, the metaclass's <code>__call__</code> method checks if one already exists. If it does, it returns the old one. The class itself doesn't contain this logic; the metaclass manages it behind the scenes.</p>

<p>You can also use metaclasses for automatic setup. Let's say you want all classes with certain attributes to get automatic validation methods.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ValidationMeta</span><span class="p">(</span><span class="nb">type</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__new__</span><span class="p">(</span><span class="n">cls</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">bases</span><span class="p">,</span> <span class="n">attrs</span><span class="p">):</span>
        <span class="k">for</span> <span class="n">attr_name</span><span class="p">,</span> <span class="n">value</span> <span class="ow">in</span> <span class="n">attrs</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
            <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">value</span><span class="p">,</span> <span class="n">ValidatedAttribute</span><span class="p">):</span>
                <span class="n">attrs</span><span class="p">[</span><span class="sa">f</span><span class="sh">'</span><span class="s">_validate_</span><span class="si">{</span><span class="n">attr_name</span><span class="si">}</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">value</span><span class="p">.</span><span class="nf">create_validator</span><span class="p">()</span>
        <span class="k">return</span> <span class="nf">super</span><span class="p">().</span><span class="nf">__new__</span><span class="p">(</span><span class="n">cls</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">bases</span><span class="p">,</span> <span class="n">attrs</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">ValidatedAttribute</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">type_hint</span><span class="p">,</span> <span class="n">validator</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">type_hint</span> <span class="o">=</span> <span class="n">type_hint</span>
        <span class="n">self</span><span class="p">.</span><span class="n">validator</span> <span class="o">=</span> <span class="n">validator</span>

    <span class="k">def</span> <span class="nf">create_validator</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="k">def</span> <span class="nf">validator</span><span class="p">(</span><span class="n">value</span><span class="p">):</span>
            <span class="k">if</span> <span class="ow">not</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">value</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">type_hint</span><span class="p">):</span>
                <span class="k">raise</span> <span class="nc">TypeError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Expected </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">type_hint</span><span class="si">}</span><span class="s">, got </span><span class="si">{</span><span class="nf">type</span><span class="p">(</span><span class="n">value</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">validator</span><span class="p">:</span>
                <span class="n">self</span><span class="p">.</span><span class="nf">validator</span><span class="p">(</span><span class="n">value</span><span class="p">)</span>
            <span class="k">return</span> <span class="bp">True</span>
        <span class="k">return</span> <span class="n">validator</span>

<span class="k">class</span> <span class="nc">Product</span><span class="p">(</span><span class="n">metaclass</span><span class="o">=</span><span class="n">ValidationMeta</span><span class="p">):</span>
    <span class="n">name</span> <span class="o">=</span> <span class="nc">ValidatedAttribute</span><span class="p">(</span><span class="nb">str</span><span class="p">,</span> <span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="nf">len</span><span class="p">(</span><span class="n">x</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span>
    <span class="n">price</span> <span class="o">=</span> <span class="nc">ValidatedAttribute</span><span class="p">(</span><span class="nb">float</span><span class="p">,</span> <span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span> <span class="o">&gt;=</span> <span class="mi">0</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">price</span><span class="p">):</span>
        <span class="c1"># Use the auto-generated validation methods
</span>        <span class="nf">getattr</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="sa">f</span><span class="sh">'</span><span class="s">_validate_name</span><span class="sh">'</span><span class="p">)(</span><span class="n">name</span><span class="p">)</span>
        <span class="nf">getattr</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="sa">f</span><span class="sh">'</span><span class="s">_validate_price</span><span class="sh">'</span><span class="p">)(</span><span class="n">price</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="n">name</span>
        <span class="n">self</span><span class="p">.</span><span class="n">price</span> <span class="o">=</span> <span class="n">price</span>

<span class="c1"># This will fail because the name is empty and the price is negative
</span><span class="k">try</span><span class="p">:</span>
    <span class="n">product</span> <span class="o">=</span> <span class="nc">Product</span><span class="p">(</span><span class="sh">""</span><span class="p">,</span> <span class="o">-</span><span class="mi">10</span><span class="p">)</span>
<span class="nf">except </span><span class="p">(</span><span class="nb">TypeError</span><span class="p">,</span> <span class="nb">ValueError</span><span class="p">)</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Validation worked: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>When Python creates the <code>Product</code> class, the <code>ValidationMeta</code> metaclass runs. It looks for <code>ValidatedAttribute</code> objects and, for each one, adds a corresponding <code>_validate_...</code> method to the class. The <code>Product.__init__</code> method can then use these methods. This keeps the validation rules declared right next to the attribute, clean and self-contained.</p>

<p>What happens when you try to access an attribute that doesn't exist? Normally, Python raises an <code>AttributeError</code>. But you can change that. The <code>__getattr__</code> method is your custom handler for missing attributes.</p>

<p>This is incredibly useful for creating proxy objects or lazy-loading systems. A proxy object stands in for another object, controlling access to it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">LazyLoader</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_loaded_data</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">__getattr__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">name</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">name</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">_loaded_data</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Loading </span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="s"> from database...</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">self</span><span class="p">.</span><span class="n">_loaded_data</span><span class="p">[</span><span class="n">name</span><span class="p">]</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_load_from_source</span><span class="p">(</span><span class="n">name</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">_loaded_data</span><span class="p">[</span><span class="n">name</span><span class="p">]</span>

    <span class="k">def</span> <span class="nf">_load_from_source</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">key</span><span class="p">):</span>
        <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mf">0.5</span><span class="p">)</span>  <span class="c1"># Simulate slow database fetch
</span>        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Data for </span><span class="si">{</span><span class="n">key</span><span class="si">}</span><span class="sh">"</span>

<span class="n">loader</span> <span class="o">=</span> <span class="nc">LazyLoader</span><span class="p">()</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">First access: </span><span class="si">{</span><span class="n">loader</span><span class="p">.</span><span class="n">user_profile</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Triggers loading
</span><span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Second access: </span><span class="si">{</span><span class="n">loader</span><span class="p">.</span><span class="n">user_profile</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span> <span class="c1"># Returns cached data
</span></code></pre>

</div>



<p>The first time you ask for <code>loader.user_profile</code>, Python can't find that attribute. So it calls <code>__getattr__('user_profile')</code>. Our method sees it's not in the cache, simulates a slow load from a database, stores the result, and returns it. The next time you ask, the data is already in the cache and is returned instantly. This pattern is great for expensive resources.</p>

<p>Let's look at a configuration proxy example.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ConfigurationProxy</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">config_dict</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_config</span> <span class="o">=</span> <span class="n">config_dict</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_accessed_keys</span> <span class="o">=</span> <span class="nf">set</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">__getattr__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">name</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">name</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">_config</span><span class="p">:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">_accessed_keys</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="n">name</span><span class="p">)</span>
            <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">_config</span><span class="p">[</span><span class="n">name</span><span class="p">]</span>
        <span class="k">raise</span> <span class="nc">AttributeError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">No configuration for </span><span class="sh">'</span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="sh">'"</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_accessed_keys</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">_accessed_keys</span><span class="p">)</span>

<span class="n">config</span> <span class="o">=</span> <span class="nc">ConfigurationProxy</span><span class="p">({</span>
    <span class="sh">'</span><span class="s">database_host</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">localhost</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">database_port</span><span class="sh">'</span><span class="p">:</span> <span class="mi">5432</span>
<span class="p">})</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Host: </span><span class="si">{</span><span class="n">config</span><span class="p">.</span><span class="n">database_host</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Accessed keys: </span><span class="si">{</span><span class="n">config</span><span class="p">.</span><span class="nf">get_accessed_keys</span><span class="p">()</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="c1"># Trying config.some_unknown_key would raise an AttributeError
</span></code></pre>

</div>



<p>Here, the proxy lets you access configuration values with dot notation (<code>config.database_host</code>), while secretly tracking which keys were used. This can help with auditing or detecting unused settings.</p>

<p>This technique is where things get powerful. Instead of just modifying existing code, you can build new code as text or as an Abstract Syntax Tree (AST) and then execute it. The <code>ast</code> module lets you construct code programmatically.</p>

<p>Why would you do this? Imagine building a custom query language, or generating a batch of similar functions from a configuration file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">ast</span>
<span class="kn">import</span> <span class="n">inspect</span>

<span class="k">class</span> <span class="nc">CodeGenerator</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">create_function</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="n">params</span><span class="p">,</span> <span class="n">body_statements</span><span class="p">):</span>
        <span class="n">args</span> <span class="o">=</span> <span class="n">ast</span><span class="p">.</span><span class="nf">arguments</span><span class="p">(</span>
            <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="n">ast</span><span class="p">.</span><span class="nf">arg</span><span class="p">(</span><span class="n">arg</span><span class="o">=</span><span class="n">param</span><span class="p">)</span> <span class="k">for</span> <span class="n">param</span> <span class="ow">in</span> <span class="n">params</span><span class="p">],</span>
            <span class="n">kwonlyargs</span><span class="o">=</span><span class="p">[],</span> <span class="n">defaults</span><span class="o">=</span><span class="p">[]</span>
        <span class="p">)</span>

        <span class="n">body</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">for</span> <span class="n">stmt</span> <span class="ow">in</span> <span class="n">body_statements</span><span class="p">:</span>
            <span class="n">body</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">stmt</span><span class="p">)</span>

        <span class="n">func_def</span> <span class="o">=</span> <span class="n">ast</span><span class="p">.</span><span class="nc">FunctionDef</span><span class="p">(</span>
            <span class="n">name</span><span class="o">=</span><span class="n">name</span><span class="p">,</span>
            <span class="n">args</span><span class="o">=</span><span class="n">args</span><span class="p">,</span>
            <span class="n">body</span><span class="o">=</span><span class="n">body</span><span class="p">,</span>
            <span class="n">decorator_list</span><span class="o">=</span><span class="p">[]</span>
        <span class="p">)</span>

        <span class="n">module</span> <span class="o">=</span> <span class="n">ast</span><span class="p">.</span><span class="nc">Module</span><span class="p">(</span><span class="n">body</span><span class="o">=</span><span class="p">[</span><span class="n">func_def</span><span class="p">],</span> <span class="n">type_ignores</span><span class="o">=</span><span class="p">[])</span>
        <span class="n">ast</span><span class="p">.</span><span class="nf">fix_missing_locations</span><span class="p">(</span><span class="n">module</span><span class="p">)</span>

        <span class="n">code</span> <span class="o">=</span> <span class="nf">compile</span><span class="p">(</span><span class="n">module</span><span class="p">,</span> <span class="n">filename</span><span class="o">=</span><span class="sh">'</span><span class="s">&lt;generated&gt;</span><span class="sh">'</span><span class="p">,</span> <span class="n">mode</span><span class="o">=</span><span class="sh">'</span><span class="s">exec</span><span class="sh">'</span><span class="p">)</span>
        <span class="nf">exec</span><span class="p">(</span><span class="n">code</span><span class="p">,</span> <span class="nf">globals</span><span class="p">())</span>

        <span class="k">return</span> <span class="nf">locals</span><span class="p">()[</span><span class="n">name</span><span class="p">]</span>

<span class="n">generator</span> <span class="o">=</span> <span class="nc">CodeGenerator</span><span class="p">()</span>

<span class="c1"># Let's generate a simple 'add' function
</span><span class="n">add_body</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">ast</span><span class="p">.</span><span class="nc">Return</span><span class="p">(</span><span class="n">ast</span><span class="p">.</span><span class="nc">BinOp</span><span class="p">(</span>
        <span class="n">left</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Name</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="sh">'</span><span class="s">a</span><span class="sh">'</span><span class="p">,</span> <span class="n">ctx</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Load</span><span class="p">()),</span>
        <span class="n">op</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Add</span><span class="p">(),</span>
        <span class="n">right</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Name</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="sh">'</span><span class="s">b</span><span class="sh">'</span><span class="p">,</span> <span class="n">ctx</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Load</span><span class="p">())</span>
    <span class="p">))</span>
<span class="p">]</span>

<span class="n">generated_add</span> <span class="o">=</span> <span class="n">generator</span><span class="p">.</span><span class="nf">create_function</span><span class="p">(</span><span class="sh">'</span><span class="s">generated_add</span><span class="sh">'</span><span class="p">,</span> <span class="p">[</span><span class="sh">'</span><span class="s">a</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">b</span><span class="sh">'</span><span class="p">],</span> <span class="n">add_body</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">5 + 3 = </span><span class="si">{</span><span class="nf">generated_add</span><span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>We built the AST nodes for a function that takes two arguments (<code>a</code> and <code>b</code>) and returns their sum. We then compiled and executed that AST to create a real, working function in memory.</p>

<p>You can generate more complex logic too.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Generate a 'classify' function: return 'positive' if x &gt; 0, else 'non-positive'
</span><span class="n">if_body</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">ast</span><span class="p">.</span><span class="nc">If</span><span class="p">(</span>
        <span class="n">test</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Compare</span><span class="p">(</span>
            <span class="n">left</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Name</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="sh">'</span><span class="s">x</span><span class="sh">'</span><span class="p">,</span> <span class="n">ctx</span><span class="o">=</span><span class="n">ast</span><span class="p">.</span><span class="nc">Load</span><span class="p">()),</span>
            <span class="n">ops</span><span class="o">=</span><span class="p">[</span><span class="n">ast</span><span class="p">.</span><span class="nc">Gt</span><span class="p">()],</span>
            <span class="n">comparators</span><span class="o">=</span><span class="p">[</span><span class="n">ast</span><span class="p">.</span><span class="nc">Constant</span><span class="p">(</span><span class="n">value</span><span class="o">=</span><span class="mi">0</span><span class="p">)]</span>
        <span class="p">),</span>
        <span class="n">body</span><span class="o">=</span><span class="p">[</span><span class="n">ast</span><span class="p">.</span><span class="nc">Return</span><span class="p">(</span><span class="n">ast</span><span class="p">.</span><span class="nc">Constant</span><span class="p">(</span><span class="n">value</span><span class="o">=</span><span class="sh">'</span><span class="s">positive</span><span class="sh">'</span><span class="p">))],</span>
        <span class="n">orelse</span><span class="o">=</span><span class="p">[</span><span class="n">ast</span><span class="p">.</span><span class="nc">Return</span><span class="p">(</span><span class="n">ast</span><span class="p">.</span><span class="nc">Constant</span><span class="p">(</span><span class="n">value</span><span class="o">=</span><span class="sh">'</span><span class="s">non-positive</span><span class="sh">'</span><span class="p">))]</span>
    <span class="p">)</span>
<span class="p">]</span>

<span class="n">classify</span> <span class="o">=</span> <span class="n">generator</span><span class="p">.</span><span class="nf">create_function</span><span class="p">(</span><span class="sh">'</span><span class="s">classify_number</span><span class="sh">'</span><span class="p">,</span> <span class="p">[</span><span class="sh">'</span><span class="s">x</span><span class="sh">'</span><span class="p">],</span> <span class="n">if_body</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Classify 10: </span><span class="si">{</span><span class="nf">classify</span><span class="p">(</span><span class="mi">10</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Classify -5: </span><span class="si">{</span><span class="nf">classify</span><span class="p">(</span><span class="o">-</span><span class="mi">5</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>This approach is fundamental to how many frameworks work. They take your high-level declarations and generate the specific Python code needed to execute them.</p>

<p>Sometimes you don't know what code you'll need until the program is running. Dynamic imports let you load modules based on conditions, user input, or configuration files. This is the foundation of plugin architectures.</p>

<p>The <code>importlib</code> module is your tool for this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">importlib</span>
<span class="kn">import</span> <span class="n">sys</span>
<span class="kn">from</span> <span class="n">pathlib</span> <span class="kn">import</span> <span class="n">Path</span>

<span class="k">class</span> <span class="nc">PluginLoader</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">plugin_directory</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">plugin_directory</span> <span class="o">=</span> <span class="nc">Path</span><span class="p">(</span><span class="n">plugin_directory</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">plugins</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">discover_plugins</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="k">for</span> <span class="n">module_file</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">plugin_directory</span><span class="p">.</span><span class="nf">glob</span><span class="p">(</span><span class="sh">"</span><span class="s">*.py</span><span class="sh">"</span><span class="p">):</span>
            <span class="k">if</span> <span class="n">module_file</span><span class="p">.</span><span class="n">stem</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__init__</span><span class="sh">"</span><span class="p">:</span>
                <span class="k">continue</span>

            <span class="n">module_name</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">plugins.</span><span class="si">{</span><span class="n">module_file</span><span class="p">.</span><span class="n">stem</span><span class="si">}</span><span class="sh">"</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="n">spec</span> <span class="o">=</span> <span class="n">importlib</span><span class="p">.</span><span class="n">util</span><span class="p">.</span><span class="nf">spec_from_file_location</span><span class="p">(</span><span class="n">module_name</span><span class="p">,</span> <span class="n">module_file</span><span class="p">)</span>
                <span class="n">module</span> <span class="o">=</span> <span class="n">importlib</span><span class="p">.</span><span class="n">util</span><span class="p">.</span><span class="nf">module_from_spec</span><span class="p">(</span><span class="n">spec</span><span class="p">)</span>
                <span class="n">spec</span><span class="p">.</span><span class="n">loader</span><span class="p">.</span><span class="nf">exec_module</span><span class="p">(</span><span class="n">module</span><span class="p">)</span>

                <span class="k">for</span> <span class="n">attr_name</span> <span class="ow">in</span> <span class="nf">dir</span><span class="p">(</span><span class="n">module</span><span class="p">):</span>
                    <span class="n">attr</span> <span class="o">=</span> <span class="nf">getattr</span><span class="p">(</span><span class="n">module</span><span class="p">,</span> <span class="n">attr_name</span><span class="p">)</span>
                    <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">attr</span><span class="p">,</span> <span class="nb">type</span><span class="p">)</span> <span class="ow">and</span> <span class="nf">hasattr</span><span class="p">(</span><span class="n">attr</span><span class="p">,</span> <span class="sh">'</span><span class="s">is_plugin</span><span class="sh">'</span><span class="p">):</span>
                        <span class="n">self</span><span class="p">.</span><span class="n">plugins</span><span class="p">[</span><span class="n">attr_name</span><span class="p">]</span> <span class="o">=</span> <span class="n">attr</span>
                        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Loaded plugin: </span><span class="si">{</span><span class="n">attr_name</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

            <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
                <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Failed to load </span><span class="si">{</span><span class="n">module_file</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_plugin</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">name</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="n">plugin_class</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">plugins</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">name</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">plugin_class</span><span class="p">:</span>
            <span class="k">return</span> <span class="nf">plugin_class</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="k">raise</span> <span class="nc">ValueError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Unknown plugin: </span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Example of a plugin base class you might define
</span><span class="k">class</span> <span class="nc">BasePlugin</span><span class="p">:</span>
    <span class="n">is_plugin</span> <span class="o">=</span> <span class="bp">True</span>
    <span class="k">def</span> <span class="nf">execute</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">data</span><span class="p">):</span>
        <span class="k">raise</span> <span class="nb">NotImplementedError</span>

<span class="c1"># Simulate finding a plugin file
</span><span class="n">loader</span> <span class="o">=</span> <span class="nc">PluginLoader</span><span class="p">(</span><span class="sh">"</span><span class="s">/tmp/my_plugins</span><span class="sh">"</span><span class="p">)</span>
<span class="n">loader</span><span class="p">.</span><span class="nf">discover_plugins</span><span class="p">()</span>
</code></pre>

</div>



<p>This loader scans a directory for Python files, imports each one, and looks for classes marked as plugins. Your main application can then instantiate and use these plugins without ever having hardcoded their names. It makes your system extensible by others.</p>

<p>You can also import by string.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">DynamicImporter</span><span class="p">:</span>
    <span class="nd">@staticmethod</span>
    <span class="k">def</span> <span class="nf">import_from_string</span><span class="p">(</span><span class="n">import_path</span><span class="p">):</span>
        <span class="n">module_path</span><span class="p">,</span> <span class="n">class_name</span> <span class="o">=</span> <span class="n">import_path</span><span class="p">.</span><span class="nf">rsplit</span><span class="p">(</span><span class="sh">'</span><span class="s">.</span><span class="sh">'</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
        <span class="n">module</span> <span class="o">=</span> <span class="n">importlib</span><span class="p">.</span><span class="nf">import_module</span><span class="p">(</span><span class="n">module_path</span><span class="p">)</span>
        <span class="k">return</span> <span class="nf">getattr</span><span class="p">(</span><span class="n">module</span><span class="p">,</span> <span class="n">class_name</span><span class="p">)</span>

<span class="c1"># Use a string to import the 'dumps' function from the 'json' module
</span><span class="n">json_dumper</span> <span class="o">=</span> <span class="n">DynamicImporter</span><span class="p">.</span><span class="nf">import_from_string</span><span class="p">(</span><span class="sh">"</span><span class="s">json.dumps</span><span class="sh">"</span><span class="p">)</span>
<span class="n">data</span> <span class="o">=</span> <span class="nf">json_dumper</span><span class="p">({</span><span class="sh">"</span><span class="s">key</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">value</span><span class="sh">"</span><span class="p">})</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Dynamically imported function result: </span><span class="si">{</span><span class="n">data</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>This is how many web frameworks convert string-based settings like <code>'path.to.MyViewClass'</code> into actual Python classes.</p>

<p>Descriptors are objects that manage the access (get, set, delete) of another attribute. The built-in <code>@property</code> decorator is actually a descriptor. You can create your own to add custom logic whenever an attribute is accessed or changed.</p>

<p>A classic use case is validated or cached attributes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ValidatedDescriptor</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">validator</span><span class="o">=</span><span class="bp">None</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">validator</span> <span class="o">=</span> <span class="n">validator</span>
        <span class="n">self</span><span class="p">.</span><span class="n">default</span> <span class="o">=</span> <span class="n">default</span>
        <span class="n">self</span><span class="p">.</span><span class="n">data</span> <span class="o">=</span> <span class="p">{}</span>  <span class="c1"># Stores data per instance
</span>
    <span class="k">def</span> <span class="nf">__get__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">,</span> <span class="n">objtype</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">obj</span> <span class="ow">is</span> <span class="bp">None</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">self</span>
        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nf">id</span><span class="p">(</span><span class="n">obj</span><span class="p">),</span> <span class="n">self</span><span class="p">.</span><span class="n">default</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">__set__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">,</span> <span class="n">value</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">validator</span> <span class="ow">and</span> <span class="ow">not</span> <span class="n">self</span><span class="p">.</span><span class="nf">validator</span><span class="p">(</span><span class="n">value</span><span class="p">):</span>
            <span class="k">raise</span> <span class="nc">ValueError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Invalid value: </span><span class="si">{</span><span class="n">value</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">[</span><span class="nf">id</span><span class="p">(</span><span class="n">obj</span><span class="p">)]</span> <span class="o">=</span> <span class="n">value</span>

    <span class="k">def</span> <span class="nf">__delete__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">):</span>
        <span class="k">if</span> <span class="nf">id</span><span class="p">(</span><span class="n">obj</span><span class="p">)</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">:</span>
            <span class="k">del</span> <span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">[</span><span class="nf">id</span><span class="p">(</span><span class="n">obj</span><span class="p">)]</span>

<span class="k">class</span> <span class="nc">Configuration</span><span class="p">:</span>
    <span class="n">port</span> <span class="o">=</span> <span class="nc">ValidatedDescriptor</span><span class="p">(</span>
        <span class="n">validator</span><span class="o">=</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="mi">1</span> <span class="o">&lt;=</span> <span class="n">x</span> <span class="o">&lt;=</span> <span class="mi">65535</span><span class="p">,</span>
        <span class="n">default</span><span class="o">=</span><span class="mi">8080</span>
    <span class="p">)</span>

<span class="n">config</span> <span class="o">=</span> <span class="nc">Configuration</span><span class="p">()</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Default port: </span><span class="si">{</span><span class="n">config</span><span class="p">.</span><span class="n">port</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># 8080
</span><span class="n">config</span><span class="p">.</span><span class="n">port</span> <span class="o">=</span> <span class="mi">3000</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Set port: </span><span class="si">{</span><span class="n">config</span><span class="p">.</span><span class="n">port</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>      <span class="c1"># 3000
</span>
<span class="k">try</span><span class="p">:</span>
    <span class="n">config</span><span class="p">.</span><span class="n">port</span> <span class="o">=</span> <span class="mi">70000</span>  <span class="c1"># Invalid, will raise ValueError
</span><span class="k">except</span> <span class="nb">ValueError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Caught error: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>The <code>ValidatedDescriptor</code> object is assigned as a class attribute <code>port</code>. When you do <code>config.port = 3000</code>, it's not a simple assignment. Python calls the descriptor's <code>__set__</code> method, which runs the validator. The value is then stored in the descriptor's <code>data</code> dictionary, keyed by the instance's id. This keeps the data for each instance separate.</p>

<p>Let's build a read-only cached property.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">CachedProperty</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">func</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">func</span> <span class="o">=</span> <span class="n">func</span>
        <span class="n">self</span><span class="p">.</span><span class="n">cache</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">__get__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">,</span> <span class="n">objtype</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">obj</span> <span class="ow">is</span> <span class="bp">None</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">self</span>
        <span class="n">cache_key</span> <span class="o">=</span> <span class="nf">id</span><span class="p">(</span><span class="n">obj</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">cache_key</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">cache</span><span class="p">:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">cache</span><span class="p">[</span><span class="n">cache_key</span><span class="p">]</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">func</span><span class="p">(</span><span class="n">obj</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">cache</span><span class="p">[</span><span class="n">cache_key</span><span class="p">]</span>

    <span class="k">def</span> <span class="nf">__set__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">,</span> <span class="n">value</span><span class="p">):</span>
        <span class="k">raise</span> <span class="nc">AttributeError</span><span class="p">(</span><span class="sh">"</span><span class="s">CachedProperty is read-only</span><span class="sh">"</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">DataProcessor</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">data</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">data</span> <span class="o">=</span> <span class="n">data</span>

    <span class="nd">@CachedProperty</span>
    <span class="k">def</span> <span class="nf">expensive_summary</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Performing heavy calculation...</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>  <span class="c1"># Simulate hard work
</span>        <span class="k">return</span> <span class="nf">sum</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">)</span> <span class="o">/</span> <span class="nf">len</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">)</span>

<span class="n">processor</span> <span class="o">=</span> <span class="nc">DataProcessor</span><span class="p">([</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">])</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">First call: </span><span class="si">{</span><span class="n">processor</span><span class="p">.</span><span class="n">expensive_summary</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Calculates
</span><span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Second call: </span><span class="si">{</span><span class="n">processor</span><span class="p">.</span><span class="n">expensive_summary</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span> <span class="c1"># Returns cached value
</span></code></pre>

</div>



<p>The <code>@CachedProperty</code> decorator wraps the method. The first time you access <code>expensive_summary</code>, the descriptor's <code>__get__</code> runs the original function and saves the result. Every subsequent access returns the cached value instantly. It's a clean way to optimize expensive operations.</p>

<p>Context managers, defined by the <code>with</code> statement, give you a reliable way to set up and tear down resources. You know them from working with files (<code>with open('file.txt') as f:</code>). You can build your own for any paired operations.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">contextlib</span> <span class="kn">import</span> <span class="n">contextmanager</span><span class="p">,</span> <span class="n">AbstractContextManager</span>
<span class="kn">import</span> <span class="n">tempfile</span>
<span class="kn">import</span> <span class="n">os</span>

<span class="k">class</span> <span class="nc">DatabaseTransaction</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">connection</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">connection</span> <span class="o">=</span> <span class="n">connection</span>

    <span class="k">def</span> <span class="nf">__enter__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Starting transaction</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">BEGIN</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">self</span>

    <span class="k">def</span> <span class="nf">__exit__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">exc_type</span><span class="p">,</span> <span class="n">exc_val</span><span class="p">,</span> <span class="n">exc_tb</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">exc_type</span> <span class="ow">is</span> <span class="bp">None</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Committing transaction</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">self</span><span class="p">.</span><span class="n">connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">COMMIT</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Rolling back transaction</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">self</span><span class="p">.</span><span class="n">connection</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">ROLLBACK</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="bp">False</span>  <span class="c1"># Let any exception propagate
</span>
<span class="k">class</span> <span class="nc">MockConnection</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">execute</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">query</span><span class="p">):</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Executing: </span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="n">conn</span> <span class="o">=</span> <span class="nc">MockConnection</span><span class="p">()</span>
<span class="k">try</span><span class="p">:</span>
    <span class="k">with</span> <span class="nc">DatabaseTransaction</span><span class="p">(</span><span class="n">conn</span><span class="p">):</span>
        <span class="n">conn</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">INSERT INTO users VALUES (</span><span class="sh">'</span><span class="s">Alice</span><span class="sh">'</span><span class="s">)</span><span class="sh">"</span><span class="p">)</span>
        <span class="c1"># Uncomment next line to simulate a failure and trigger rollback
</span>        <span class="c1"># raise ValueError("Oops!")
</span><span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">An error occurred, but the transaction was rolled back.</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>The <code>__enter__</code> method sets up the transaction. The <code>__exit__</code> method is your cleanup hook. It receives information about any exception that happened inside the <code>with</code> block. If there was no exception (<code>exc_type is None</code>), it commits. If there was, it rolls back. The <code>return False</code> tells Python to not suppress the exception.</p>

<p>The <code>contextlib</code> module makes it even easier for functions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@contextmanager</span>
<span class="k">def</span> <span class="nf">temporary_workspace</span><span class="p">(</span><span class="n">cleanup</span><span class="o">=</span><span class="bp">True</span><span class="p">):</span>
    <span class="n">workspace</span> <span class="o">=</span> <span class="n">tempfile</span><span class="p">.</span><span class="nf">mkdtemp</span><span class="p">()</span>
    <span class="n">original_cwd</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="nf">getcwd</span><span class="p">()</span>
    <span class="n">os</span><span class="p">.</span><span class="nf">chdir</span><span class="p">(</span><span class="n">workspace</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Working in </span><span class="si">{</span><span class="n">workspace</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="k">yield</span> <span class="n">workspace</span>  <span class="c1"># This is where the 'with' block runs
</span>    <span class="k">finally</span><span class="p">:</span>
        <span class="n">os</span><span class="p">.</span><span class="nf">chdir</span><span class="p">(</span><span class="n">original_cwd</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">cleanup</span><span class="p">:</span>
            <span class="kn">import</span> <span class="n">shutil</span>
            <span class="n">shutil</span><span class="p">.</span><span class="nf">rmtree</span><span class="p">(</span><span class="n">workspace</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Cleaned up </span><span class="si">{</span><span class="n">workspace</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">with</span> <span class="nf">temporary_workspace</span><span class="p">()</span> <span class="k">as</span> <span class="n">ws</span><span class="p">:</span>
    <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">my_file.txt</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">w</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="sh">"</span><span class="s">test</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Created file in: </span><span class="si">{</span><span class="n">os</span><span class="p">.</span><span class="nf">listdir</span><span class="p">(</span><span class="n">ws</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="c1"># The directory is now gone
</span></code></pre>

</div>



<p>The <code>@contextmanager</code> decorator lets you write a generator function. Everything before the <code>yield</code> is <code>__enter__</code>. The <code>yield</code>ed value becomes the <code>as</code> variable. Everything after, inside the <code>finally</code> block, is <code>__exit__</code>. It's a very concise pattern.</p>

<p>This is an advanced area, but it reveals how Python works under the hood. The <code>dis</code> module lets you see the bytecode—the low-level instructions the Python interpreter executes. You can even create or modify it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">dis</span>
<span class="kn">import</span> <span class="n">types</span>

<span class="k">def</span> <span class="nf">simple_function</span><span class="p">(</span><span class="n">x</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">x</span> <span class="o">*</span> <span class="mi">2</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Bytecode for simple_function:</span><span class="sh">"</span><span class="p">)</span>
<span class="n">dis</span><span class="p">.</span><span class="nf">dis</span><span class="p">(</span><span class="n">simple_function</span><span class="p">)</span>
</code></pre>

</div>



<p>This will print out the step-by-step instructions. It can be helpful for debugging performance issues or understanding a tricky piece of code.</p>

<p>You can go further and create a function from raw bytecode.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">create_function_from_code</span><span class="p">():</span>
    <span class="c1"># Bytecode for: def add(a, b): return a + b
</span>    <span class="n">bytecode</span> <span class="o">=</span> <span class="nf">bytes</span><span class="p">([</span><span class="mi">124</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">124</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">23</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">83</span><span class="p">,</span> <span class="mi">0</span><span class="p">])</span>

    <span class="n">code_obj</span> <span class="o">=</span> <span class="n">types</span><span class="p">.</span><span class="nc">CodeType</span><span class="p">(</span>
        <span class="mi">2</span><span class="p">,</span>  <span class="c1"># argcount
</span>        <span class="mi">0</span><span class="p">,</span>  <span class="c1"># posonlyargcount
</span>        <span class="mi">2</span><span class="p">,</span>  <span class="c1"># kwonlyargcount? Actually nlocals
</span>        <span class="mi">2</span><span class="p">,</span>  <span class="c1"># nlocals
</span>        <span class="mi">1</span><span class="p">,</span>  <span class="c1"># stacksize
</span>        <span class="mi">67</span><span class="p">,</span> <span class="c1"># flags
</span>        <span class="n">bytecode</span><span class="p">,</span>
        <span class="p">(</span><span class="bp">None</span><span class="p">,),</span>        <span class="c1"># constants
</span>        <span class="p">(</span><span class="sh">'</span><span class="s">a</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">b</span><span class="sh">'</span><span class="p">),</span>     <span class="c1"># varnames
</span>        <span class="p">(),</span> <span class="p">(),</span>         <span class="c1"># freevars, cellvars
</span>        <span class="sh">'</span><span class="s">&lt;generated&gt;</span><span class="sh">'</span><span class="p">,</span>  <span class="c1"># filename
</span>        <span class="sh">'</span><span class="s">add</span><span class="sh">'</span><span class="p">,</span>          <span class="c1"># name
</span>        <span class="mi">1</span><span class="p">,</span>              <span class="c1"># firstlineno
</span>        <span class="sa">b</span><span class="sh">''</span>             <span class="c1"># linetable
</span>    <span class="p">)</span>

    <span class="k">return</span> <span class="n">types</span><span class="p">.</span><span class="nc">FunctionType</span><span class="p">(</span><span class="n">code_obj</span><span class="p">,</span> <span class="p">{})</span>

<span class="n">my_add</span> <span class="o">=</span> <span class="nf">create_function_from_code</span><span class="p">()</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Generated add function: </span><span class="si">{</span><span class="nf">my_add</span><span class="p">(</span><span class="mi">7</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>This is complex, but it shows the fundamental building blocks of a function. Some optimization and obfuscation tools work at this level.</p>

<p>You've seen eight different angles on meta-programming. They range from the everyday utility of decorators to the deep mechanics of bytecode. The goal isn't to use them all in every project. The goal is to know they exist.</p>

<p>When you face a problem involving repetitive structure, think of decorators or code generation. When you need runtime adaptability, think of dynamic imports or <code>__getattr__</code>. When you need to enforce rules across many classes, consider descriptors or metaclasses.</p>

<h2>
  
  
  These techniques are the foundation of major frameworks like Django, SQLAlchemy, and Pytest. They let those libraries provide a clean, high-level interface while generating the complex code required to make it work. By understanding these tools, you move from just using Python to shaping it, allowing you to build systems that are more expressive, maintainable, and powerful.
</h2>

<p>📘 <strong>Checkout my <a href="https://youtu.be/WpR6F4ky4uM" rel="noopener noreferrer">latest ebook</a> for free on my channel!</strong><br><br>
Be sure to <strong>like</strong>, <strong>share</strong>, <strong>comment</strong>, and <strong>subscribe</strong> to the channel!</p>




<h2>
  
  
  101 Books
</h2>

<p><strong>101 Books</strong> is an AI-driven publishing company co-founded by author <strong>Aarav Joshi</strong>. By leveraging advanced AI technology, we keep our publishing costs incredibly low—some books are priced as low as <strong>$4</strong>—making quality knowledge accessible to everyone.</p>

<p>Check out our book <strong><a href="https://www.amazon.com/dp/B0DQQF9K3Z" rel="noopener noreferrer">Golang Clean Code</a></strong> available on Amazon. </p>

<p>Stay tuned for updates and exciting news. When shopping for books, search for <strong>Aarav Joshi</strong> to find more of our titles. Use the provided link to enjoy <strong>special discounts</strong>!</p>

<h2>
  
  
  Our Creations
</h2>

<p>Be sure to check out our creations:</p>

<p><strong><a href="https://www.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central</a></strong> | <strong><a href="https://spanish.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central Spanish</a></strong> | <strong><a href="https://german.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central German</a></strong> | <strong><a href="https://smartliving.investorcentral.co.uk/" rel="noopener noreferrer">Smart Living</a></strong> | <strong><a href="https://epochsandechoes.com/" rel="noopener noreferrer">Epochs &amp; Echoes</a></strong> | <strong><a href="https://www.puzzlingmysteries.com/" rel="noopener noreferrer">Puzzling Mysteries</a></strong> | <strong><a href="http://hindutva.epochsandechoes.com/" rel="noopener noreferrer">Hindutva</a></strong> | <strong><a href="https://elitedev.in/" rel="noopener noreferrer">Elite Dev</a></strong> | <strong><a href="https://java.elitedev.in/" rel="noopener noreferrer">Java Elite Dev</a></strong> | <strong><a href="https://golang.elitedev.in/" rel="noopener noreferrer">Golang Elite Dev</a></strong> | <strong><a href="https://python.elitedev.in/" rel="noopener noreferrer">Python Elite Dev</a></strong> | <strong><a href="https://js.elitedev.in/" rel="noopener noreferrer">JS Elite Dev</a></strong> | <strong><a href="https://jsschools.com/" rel="noopener noreferrer">JS Schools</a></strong></p>




<h3>
  
  
  We are on Medium
</h3>

<p><strong><a href="https://techkoalainsights.com/" rel="noopener noreferrer">Tech Koala Insights</a></strong> | <strong><a href="https://world.epochsandechoes.com/" rel="noopener noreferrer">Epochs &amp; Echoes World</a></strong> | <strong><a href="https://medium.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central Medium</a></strong> | <strong><a href="https://medium.com/puzzling-mysteries" rel="noopener noreferrer">Puzzling Mysteries Medium</a></strong> | <strong><a href="https://science.epochsandechoes.com/" rel="noopener noreferrer">Science &amp; Epochs Medium</a></strong> | <strong><a href="https://modernhindutva.substack.com/" rel="noopener noreferrer">Modern Hindutva</a></strong></p>

