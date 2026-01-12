---
Title: C#.NET - day 07
Description: 
Author: Sabin Sim
Date: 2026-01-12T21:20:53.000Z
Robots: noindex,nofollow
Template: index
---
<h1>day 07: Interface &amp; Dependency Inversion </h1>

<p><em>Decoupling services from concrete implementations using contracts</em></p>

<h2>Introduction</h2>

<p>
Up to this point, the system worked well.
However, the Service layer was still tightly coupled to a concrete Repository implementation.
</p>

<p>
This step introduces <strong>interfaces</strong> and <strong>dependency inversion</strong>
to ensure that the Service does not depend on a specific storage mechanism.
</p>




<h2>🌊 System Flow</h2>

<ol>
  <li>
<strong>Request:</strong> The Controller receives a request.</li>
  <li>
<strong>Delegation:</strong> The Controller forwards the request to the Service.</li>
  <li>
    <strong>Dependency Inversion:</strong>
    The Service does not know the concrete repository.
    It only calls methods defined in <code>IHelloRepository</code>.
  </li>
  <li>
    <strong>Execution:</strong>
    <code>Program.cs</code> injects the actual implementation (<code>HelloRepository</code>)
    that fulfills the contract.
  </li>
</ol>




<h2>1️⃣ Model: Data Contract</h2>

<p>
The model defines the shape of the data exchanged across layers.
</p>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">namespace</span> <span class="nn">HelloFlow.Models</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">HelloResponse</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Message</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>
    <span class="k">public</span> <span class="n">DateTime</span> <span class="n">CreatedAt</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Location</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>2️⃣ Repository Contract: IHelloRepository</h2>

<p>
The interface acts as a contract.
It defines <em>what must be done</em>, not <em>how it is done</em>.
</p>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">HelloFlow.Models</span><span class="p">;</span>

<span class="k">namespace</span> <span class="nn">HelloFlow.Repositories</span><span class="p">;</span>

<span class="k">public</span> <span class="k">interface</span> <span class="nc">IHelloRepository</span>
<span class="p">{</span>
    <span class="k">void</span> <span class="nf">Save</span><span class="p">(</span><span class="n">HelloResponse</span> <span class="n">data</span><span class="p">);</span>
    <span class="n">List</span><span class="p">&lt;</span><span class="n">HelloResponse</span><span class="p">&gt;</span> <span class="nf">GetAll</span><span class="p">();</span>
    <span class="n">List</span><span class="p">&lt;</span><span class="n">HelloResponse</span><span class="p">&gt;</span> <span class="nf">SearchAdvanced</span><span class="p">(</span><span class="kt">string</span> <span class="n">keyword</span><span class="p">,</span> <span class="kt">int</span> <span class="n">pageNumber</span><span class="p">,</span> <span class="kt">int</span> <span class="n">pageSize</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>
Any repository implementation that follows this contract can be used interchangeably.
</p>




<h2>3️⃣ Repository Implementation</h2>

<p>
This class fulfills the contract using an in-memory, thread-safe data structure.
</p>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System.Collections.Concurrent</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">HelloFlow.Models</span><span class="p">;</span>

<span class="k">namespace</span> <span class="nn">HelloFlow.Repositories</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">HelloRepository</span> <span class="p">:</span> <span class="n">IHelloRepository</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">ConcurrentDictionary</span><span class="p">&lt;</span><span class="kt">string</span><span class="p">,</span> <span class="n">HelloResponse</span><span class="p">&gt;</span> <span class="n">_storage</span> <span class="p">=</span> <span class="k">new</span><span class="p">();</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">Save</span><span class="p">(</span><span class="n">HelloResponse</span> <span class="n">data</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">key</span> <span class="p">=</span> <span class="n">Guid</span><span class="p">.</span><span class="nf">NewGuid</span><span class="p">().</span><span class="nf">ToString</span><span class="p">();</span>
        <span class="n">_storage</span><span class="p">.</span><span class="nf">TryAdd</span><span class="p">(</span><span class="n">key</span><span class="p">,</span> <span class="n">data</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="n">List</span><span class="p">&lt;</span><span class="n">HelloResponse</span><span class="p">&gt;</span> <span class="nf">GetAll</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="n">_storage</span><span class="p">.</span><span class="n">Values</span><span class="p">.</span><span class="nf">ToList</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="n">List</span><span class="p">&lt;</span><span class="n">HelloResponse</span><span class="p">&gt;</span> <span class="nf">SearchAdvanced</span><span class="p">(</span><span class="kt">string</span> <span class="n">keyword</span><span class="p">,</span> <span class="kt">int</span> <span class="n">pageNumber</span><span class="p">,</span> <span class="kt">int</span> <span class="n">pageSize</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">query</span> <span class="p">=</span> <span class="n">_storage</span><span class="p">.</span><span class="n">Values</span><span class="p">.</span><span class="nf">AsEnumerable</span><span class="p">();</span>

        <span class="k">if</span> <span class="p">(!</span><span class="kt">string</span><span class="p">.</span><span class="nf">IsNullOrWhiteSpace</span><span class="p">(</span><span class="n">keyword</span><span class="p">))</span>
        <span class="p">{</span>
            <span class="n">query</span> <span class="p">=</span> <span class="n">query</span><span class="p">.</span><span class="nf">Where</span><span class="p">(</span><span class="n">x</span> <span class="p">=&gt;</span>
                <span class="n">x</span><span class="p">.</span><span class="n">Message</span><span class="p">.</span><span class="nf">Contains</span><span class="p">(</span><span class="n">keyword</span><span class="p">,</span> <span class="n">StringComparison</span><span class="p">.</span><span class="n">OrdinalIgnoreCase</span><span class="p">));</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="n">query</span>
            <span class="p">.</span><span class="nf">OrderByDescending</span><span class="p">(</span><span class="n">x</span> <span class="p">=&gt;</span> <span class="n">x</span><span class="p">.</span><span class="n">CreatedAt</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">Skip</span><span class="p">((</span><span class="n">pageNumber</span> <span class="p">-</span> <span class="m">1</span><span class="p">)</span> <span class="p">*</span> <span class="n">pageSize</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">Take</span><span class="p">(</span><span class="n">pageSize</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">ToList</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>4️⃣ Service Layer: Depending on the Contract</h2>

<p>
The Service no longer depends on a concrete repository.
It works with <code>IHelloRepository</code>, not <code>HelloRepository</code>.
</p>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">HelloFlow.Models</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">HelloFlow.Repositories</span><span class="p">;</span>

<span class="k">namespace</span> <span class="nn">HelloFlow.Services</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">HelloService</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">IHelloRepository</span> <span class="n">_repository</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">HelloService</span><span class="p">(</span><span class="n">IHelloRepository</span> <span class="n">repository</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_repository</span> <span class="p">=</span> <span class="n">repository</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="n">HelloResponse</span> <span class="nf">GetHello</span><span class="p">(</span><span class="kt">string</span> <span class="n">name</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">response</span> <span class="p">=</span> <span class="k">new</span> <span class="n">HelloResponse</span>
        <span class="p">{</span>
            <span class="n">Message</span> <span class="p">=</span> <span class="s">$"Hello, </span><span class="p">{</span><span class="n">name</span><span class="p">}</span><span class="s">!"</span><span class="p">,</span>
            <span class="n">CreatedAt</span> <span class="p">=</span> <span class="n">DateTime</span><span class="p">.</span><span class="n">Now</span><span class="p">,</span>
            <span class="n">Location</span> <span class="p">=</span> <span class="s">"Cazis, Switzerland"</span>
        <span class="p">};</span>

        <span class="n">_repository</span><span class="p">.</span><span class="nf">Save</span><span class="p">(</span><span class="n">response</span><span class="p">);</span>
        <span class="k">return</span> <span class="n">response</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="n">List</span><span class="p">&lt;</span><span class="n">HelloResponse</span><span class="p">&gt;</span> <span class="nf">FindHelloAdvanced</span><span class="p">(</span><span class="kt">string</span> <span class="n">keyword</span><span class="p">,</span> <span class="kt">int</span> <span class="n">page</span><span class="p">,</span> <span class="kt">int</span> <span class="n">size</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">int</span> <span class="n">safePage</span> <span class="p">=</span> <span class="n">page</span> <span class="p">&lt;</span> <span class="m">1</span> <span class="p">?</span> <span class="m">1</span> <span class="p">:</span> <span class="n">page</span><span class="p">;</span>
        <span class="kt">int</span> <span class="n">safeSize</span> <span class="p">=</span> <span class="n">size</span> <span class="p">&gt;</span> <span class="m">50</span> <span class="p">?</span> <span class="m">50</span> <span class="p">:</span> <span class="n">size</span><span class="p">;</span>

        <span class="k">return</span> <span class="n">_repository</span><span class="p">.</span><span class="nf">SearchAdvanced</span><span class="p">(</span><span class="n">keyword</span><span class="p">,</span> <span class="n">safePage</span><span class="p">,</span> <span class="n">safeSize</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>
This change represents the core of dependency inversion:
the Service depends on an abstraction, not a concrete implementation.
</p>




<h2>5️⃣ Controller: Request Handling Only</h2>


<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">Microsoft.AspNetCore.Mvc</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">HelloFlow.Services</span><span class="p">;</span>

<span class="k">namespace</span> <span class="nn">HelloFlow.Controllers</span><span class="p">;</span>

<span class="p">[</span><span class="n">ApiController</span><span class="p">]</span>
<span class="p">[</span><span class="nf">Route</span><span class="p">(</span><span class="s">"api/[controller]"</span><span class="p">)]</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">HelloController</span> <span class="p">:</span> <span class="n">ControllerBase</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">HelloService</span> <span class="n">_service</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">HelloController</span><span class="p">(</span><span class="n">HelloService</span> <span class="n">service</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_service</span> <span class="p">=</span> <span class="n">service</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="n">HttpGet</span><span class="p">]</span>
    <span class="k">public</span> <span class="n">IActionResult</span> <span class="nf">SayHello</span><span class="p">(</span><span class="kt">string</span> <span class="n">name</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">result</span> <span class="p">=</span> <span class="n">_service</span><span class="p">.</span><span class="nf">GetHello</span><span class="p">(</span><span class="n">name</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">Ok</span><span class="p">(</span><span class="n">result</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="nf">HttpGet</span><span class="p">(</span><span class="s">"search"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="n">IActionResult</span> <span class="nf">SearchHello</span><span class="p">(</span>
        <span class="p">[</span><span class="n">FromQuery</span><span class="p">]</span> <span class="kt">string</span><span class="p">?</span> <span class="n">name</span><span class="p">,</span>
        <span class="p">[</span><span class="n">FromQuery</span><span class="p">]</span> <span class="kt">int</span> <span class="n">page</span> <span class="p">=</span> <span class="m">1</span><span class="p">,</span>
        <span class="p">[</span><span class="n">FromQuery</span><span class="p">]</span> <span class="kt">int</span> <span class="n">size</span> <span class="p">=</span> <span class="m">10</span>
    <span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">results</span> <span class="p">=</span> <span class="n">_service</span><span class="p">.</span><span class="nf">FindHelloAdvanced</span><span class="p">(</span><span class="n">name</span> <span class="p">??</span> <span class="s">""</span><span class="p">,</span> <span class="n">page</span><span class="p">,</span> <span class="n">size</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">Ok</span><span class="p">(</span><span class="n">results</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<h2>6️⃣ Program.cs: Dependency Wiring</h2>

<p>
This is where the abstraction is connected to its concrete implementation.
</p>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">HelloFlow.Services</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">HelloFlow.Repositories</span><span class="p">;</span>

<span class="kt">var</span> <span class="n">builder</span> <span class="p">=</span> <span class="n">WebApplication</span><span class="p">.</span><span class="nf">CreateBuilder</span><span class="p">(</span><span class="n">args</span><span class="p">);</span>

<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="nf">AddControllers</span><span class="p">();</span>
<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="nf">AddEndpointsApiExplorer</span><span class="p">();</span>
<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="nf">AddSwaggerGen</span><span class="p">();</span>

<span class="c1">// Interface → Implementation mapping</span>
<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="n">AddSingleton</span><span class="p">&lt;</span><span class="n">IHelloRepository</span><span class="p">,</span> <span class="n">HelloRepository</span><span class="p">&gt;();</span>
<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="n">AddScoped</span><span class="p">&lt;</span><span class="n">HelloService</span><span class="p">&gt;();</span>

<span class="kt">var</span> <span class="n">app</span> <span class="p">=</span> <span class="n">builder</span><span class="p">.</span><span class="nf">Build</span><span class="p">();</span>

<span class="k">if</span> <span class="p">(</span><span class="n">app</span><span class="p">.</span><span class="n">Environment</span><span class="p">.</span><span class="nf">IsDevelopment</span><span class="p">())</span>
<span class="p">{</span>
    <span class="n">app</span><span class="p">.</span><span class="nf">UseSwagger</span><span class="p">();</span>
    <span class="n">app</span><span class="p">.</span><span class="nf">UseSwaggerUI</span><span class="p">();</span>
<span class="p">}</span>

<span class="n">app</span><span class="p">.</span><span class="nf">UseHttpsRedirection</span><span class="p">();</span>
<span class="n">app</span><span class="p">.</span><span class="nf">MapControllers</span><span class="p">();</span>
<span class="n">app</span><span class="p">.</span><span class="nf">Run</span><span class="p">();</span>
</code></pre>

</div>






<h2>🧠 Why This Matters</h2>

<p>
By introducing an interface:
</p>

<ul>
  <li>The Service does not care how data is stored</li>
  <li>The Repository can be replaced without changing Service logic</li>
  <li>Future database integration becomes trivial</li>
</ul>

<p>
Only the DI configuration changes when swapping implementations.
</p>




<h2>🧠 One-Sentence Summary</h2>

<p>
<strong>
This step is not about writing more code,
but about designing a system that can change without breaking.
</strong>
</p>





<h2>✍️ My Notes &amp; Reflections</h2>

<ul>
  <li>
    Each step adds more complexity, but it also gives me a stronger sense of structure.
  </li>
  <li>
    It feels as if I am slowly watching a factory come to life,
    understanding how its parts begin to move and connect with each other.
  </li>
</ul>

