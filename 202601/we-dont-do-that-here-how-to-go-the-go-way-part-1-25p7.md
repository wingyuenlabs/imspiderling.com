---
Title: We don't do that here: How to Go the Go way - Part 1
Description: 
Author: medunes
Date: 2026-01-03T21:42:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you are like me transitioning to Go from another tech-stack or ecosystem, you might have found yourself unarmed from some of your most powerful tactics and weapons you gathered throughout your career. In simpler words, you would hit a wall on which is written: "we don't do this here :)."</p>

<p>Some of the best practices you learned in <code>Java/Spring</code> <code>OOP</code> or <code>PHP/Symfony</code> would be considered anti-patterns and red flags in Go. What to do? Not many options, indeed. You only have to "go the Go way."</p>

<p>Here we explain a common confusion for the newcomer: the straightforward dependency injection is not (from what I could research at least) often the very welcome within the idiomatic Go republic.</p>

<p><strong>Disclaimer</strong>: There might be no single place to dictate to you "we don't do this here," but based on a couple of widely used open-source projects, language authors and maintainers, and famous blog posts you might conclude this.</p>

<h3>
  
  
  How they do it the Go way?
</h3>

<p>Have you ever came across this code style, and had couple question marks itching your brain for a while?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">UserAggregator</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">profileFetcher</span> <span class="n">Fetcher</span>
    <span class="n">ordersFetcher</span>  <span class="n">Fetcher</span>
    <span class="n">timeout</span>        <span class="n">time</span><span class="o">.</span><span class="n">Duration</span>
    <span class="n">logger</span>         <span class="o">*</span><span class="n">slog</span><span class="o">.</span><span class="n">Logger</span>
<span class="p">}</span>

<span class="k">type</span> <span class="n">Option</span> <span class="k">func</span><span class="p">(</span><span class="o">*</span><span class="n">UserAggregator</span><span class="p">)</span>

<span class="k">func</span> <span class="n">WithTimeout</span><span class="p">(</span><span class="n">d</span> <span class="n">time</span><span class="o">.</span><span class="n">Duration</span><span class="p">)</span> <span class="n">Option</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">func</span><span class="p">(</span><span class="n">u</span> <span class="o">*</span><span class="n">UserAggregator</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">u</span><span class="o">.</span><span class="n">timeout</span> <span class="o">=</span> <span class="n">d</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">WithLogger</span><span class="p">(</span><span class="n">l</span> <span class="o">*</span><span class="n">slog</span><span class="o">.</span><span class="n">Logger</span><span class="p">)</span> <span class="n">Option</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">func</span><span class="p">(</span><span class="n">u</span> <span class="o">*</span><span class="n">UserAggregator</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">u</span><span class="o">.</span><span class="n">logger</span> <span class="o">=</span> <span class="n">l</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">NewUserAggregator</span><span class="p">(</span><span class="n">p</span> <span class="n">Fetcher</span><span class="p">,</span> <span class="n">o</span> <span class="n">Fetcher</span><span class="p">,</span> <span class="n">opts</span> <span class="o">...</span><span class="n">Option</span><span class="p">)</span> <span class="o">*</span><span class="n">UserAggregator</span> <span class="p">{</span>
    <span class="c">// Default configuration</span>
    <span class="n">agg</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">UserAggregator</span><span class="p">{</span>
        <span class="n">profileFetcher</span><span class="o">:</span> <span class="n">p</span><span class="p">,</span>
        <span class="n">ordersFetcher</span><span class="o">:</span>  <span class="n">o</span><span class="p">,</span>
        <span class="n">timeout</span><span class="o">:</span>        <span class="m">5</span> <span class="o">*</span> <span class="n">time</span><span class="o">.</span><span class="n">Second</span><span class="p">,</span>
        <span class="n">logger</span><span class="o">:</span>         <span class="n">slog</span><span class="o">.</span><span class="n">Default</span><span class="p">(),</span>
    <span class="p">}</span>

    <span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">opt</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">opts</span> <span class="p">{</span>
        <span class="n">opt</span><span class="p">(</span><span class="n">agg</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">agg</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I assume you got that moment of confusion: "Isn't this just a Builder or Setters with extra steps?"</p>

<p>You are spotting a pattern correctly. The code snippet above is known as the <strong>Functional Options Pattern</strong> (popularized by Dave Cheney and <a href="https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html" rel="noopener noreferrer">Rob Pike</a>).</p>

<p>To answer your question: <strong>Yes, this is analogous to the Builder pattern or Setter injection in <code>OOP</code>, but with a twist regarding mutability and safety.</strong></p>

<p>In a typical <code>OOP</code> container (such as <code>Spring Boot</code> or <code>Symfony</code>), you often have two phases:</p>

<ol>
<li>
<strong>Instantiation:</strong> The object is created (often with a no-arg constructor).</li>
<li>
<strong>Hydration (Setters):</strong> The container calls <code>setLogger()</code>, <code>setTimeout()</code>, etc.</li>
</ol>

<p><strong>The Go "Anti-Pattern" concern here is state validity.</strong><br>
If you use setters (<code>u.SetLogger(l)</code>), you introduce a period of time where the object exists but is incomplete (e.g., the logger is nil). You also make the object mutable—anyone can change the logger halfway through the program's lifecycle.</p>

<p>The Functional Options pattern allows you to simulate a constructor that accepts optional parameters while ensuring that <strong>once the function returns, the object is immutable, fully configured, and ready to use.</strong></p>
<h3>
  
  
  The Alternatives: Why not <code>New(a, b, c)</code> or <code>New(Config)</code>?
</h3>

<p>Have you just wondered why Go ditches the explicit configuration object. Actually, Go doesn't ditch it entirely (you will see <code>New(Config)</code> in standard libraries), but Functional Options are preferred for libraries for at least the following two reasons:</p>
<h4>
  
  
  1. The "Massive Constructor" Problem
</h4>

<p><code>New(logger, timeout, db, cache, metrics, ...)</code><br>
This is brittle. If you add a new dependency, you break every caller in your code-base.</p>
<h4>
  
  
  2. The <code>New(Config)</code> Problem
</h4>

<p>Passing a struct, like <code>New(Config{Timeout: 10s})</code>, seems clean, but it has hidden edges:</p>

<ul>
<li><p><strong>Ambiguity of Zero Values:</strong> If you pass <code>Config{Timeout: 0}</code>, does that mean "No Timeout" or "Default Timeout"? In Go, 0 is the default value for integers, It's hard to distinguish "I forgot to set this" from "I explicitly want 0". There isn't the luxury of the "nullable" parameter as you'd face in PHP as <code>?int</code></p></li>
<li><p><strong>Boilerplate:</strong> If you only want to change <em>one</em> setting, you still have to construct the struct, potentially dealing with pointers to optional fields to distinguish <code>nil</code> from <code>0</code>.</p></li>
</ul>
<h4>
  
  
  The Java/OOP Equivalent (The Builder Pattern)
</h4>

<p>In Java/Spring, to achieve the same "safety" (immutability + optional parameters), you wouldn't typically use raw Setters; you would use a <strong>Builder</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// Usage</span>
<span class="nc">UserAggregator</span> <span class="n">u</span> <span class="o">=</span> <span class="nc">UserAggregator</span><span class="o">.</span><span class="na">builder</span><span class="o">()</span>
    <span class="o">.</span><span class="na">timeout</span><span class="o">(</span><span class="nc">Duration</span><span class="o">.</span><span class="na">ofSeconds</span><span class="o">(</span><span class="mi">10</span><span class="o">))</span>
    <span class="o">.</span><span class="na">logger</span><span class="o">(</span><span class="n">myLogger</span><span class="o">)</span>
    <span class="o">.</span><span class="na">build</span><span class="o">();</span>

<span class="c1">// Inside the class</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">UserAggregator</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">Duration</span> <span class="n">timeout</span><span class="o">;</span> <span class="c1">// final = immutable, like Go's approach</span>
    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">Logger</span> <span class="n">logger</span><span class="o">;</span>

    <span class="kd">private</span> <span class="nf">UserAggregator</span><span class="o">(</span><span class="nc">Builder</span> <span class="n">b</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">timeout</span> <span class="o">=</span> <span class="n">b</span><span class="o">.</span><span class="na">timeout</span> <span class="o">!=</span> <span class="kc">null</span> <span class="o">?</span> <span class="n">b</span><span class="o">.</span><span class="na">timeout</span> <span class="o">:</span> <span class="nc">Duration</span><span class="o">.</span><span class="na">ofSeconds</span><span class="o">(</span><span class="mi">5</span><span class="o">);</span> <span class="c1">// Default handling</span>
        <span class="k">this</span><span class="o">.</span><span class="na">logger</span> <span class="o">=</span> <span class="n">b</span><span class="o">.</span><span class="na">logger</span><span class="o">;</span>
    <span class="o">}</span>
    <span class="c1">// ...</span>
<span class="o">}</span>

</code></pre>

</div>



<p><em>Comparison:</em> Go achieves this using functions as first-class citizens, whereas Java requires a separate inner class (<code>Builder</code>) to hold the temporary state.</p>

<h4>
  
  
  The PHP/Symfony Equivalent (Options Resolver)
</h4>

<p>In PHP, specifically Symfony, this is often handled via the <code>OptionsResolver</code> component because PHP lacks named arguments (historically) and strict typing (historically).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// Usage</span>
<span class="nv">$aggregator</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">UserAggregator</span><span class="p">([</span>
    <span class="s1">'timeout'</span> <span class="o">=&gt;</span> <span class="mi">10</span><span class="p">,</span>
    <span class="s1">'logger'</span> <span class="o">=&gt;</span> <span class="nv">$logger</span>
<span class="p">]);</span>

<span class="c1">// Inside the class</span>
<span class="kd">class</span> <span class="nc">UserAggregator</span> <span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$options</span> <span class="o">=</span> <span class="p">[])</span> <span class="p">{</span>
        <span class="nv">$resolver</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">OptionsResolver</span><span class="p">();</span>
        <span class="nv">$resolver</span><span class="o">-&gt;</span><span class="nf">setDefaults</span><span class="p">([</span><span class="s1">'timeout'</span> <span class="o">=&gt;</span> <span class="mi">5</span><span class="p">]);</span>
        <span class="nv">$resolver</span><span class="o">-&gt;</span><span class="nf">setRequired</span><span class="p">([</span><span class="s1">'logger'</span><span class="p">]);</span>

        <span class="nv">$config</span> <span class="o">=</span> <span class="nv">$resolver</span><span class="o">-&gt;</span><span class="nf">resolve</span><span class="p">(</span><span class="nv">$options</span><span class="p">);</span>
        <span class="c1">// ... set properties</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><em>Comparison:</em> This is much looser. The Go pattern provides compile-time safety (you can't pass a "<em>timout</em>" typo option), whereas the PHP array approach needs runtime validation.</p>

<h3>
  
  
  Summary
</h3>

<p>The <em>Functional Options pattern</em> is idiomatic in Go because it aligns with the language's philosophy:</p>

<ol>
<li><p>Explicit over implicit, as this pattern often won't rely on magical DI hydration, and you witness the override with nude eyes.</p></li>
<li><p>Stay away from the <a href="https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/" rel="noopener noreferrer">million dollar mistake</a> and avoid <code>nil</code> pointer exceptions that occur when you forget to call a Setter in OOP. The <code>New</code> function guarantees a valid object.</p></li>
<li><p>"Infinite" extensibility: you can add <code>WithDatabase()</code> later without breaking existing code that calls <code>New()</code>. The options logic is delegated and encapsulated within each corresponding function.</p></li>
</ol>

