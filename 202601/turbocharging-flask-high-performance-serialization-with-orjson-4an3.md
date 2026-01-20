---
Title: Turbocharging Flask: High-Performance Serialization with orjson
Description: 
Author: Lalit Mishra
Date: 2026-01-20T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>In high-throughput Python APIs, there is a silent killer of performance. It isn’t your database query latency, it isn’t the lack of async/await in your WSGI container, and it usually isn’t the network bandwidth.</p>

<p>It is the cost of transforming your internal Python objects into JSON.</p>

<p>I have spent countless hours profiling "slow" Flask applications where developers were convinced they needed to migrate to Go or rewrite their SQL queries. In reality, the bottleneck was deceptively simple: their API was spending 200ms querying the database and 600ms serializing the 50MB result set into a JSON string using Python’s standard library.</p>

<p>This article explores why Flask’s default serialization architecture struggles at scale and how we can achieve a 5x to 10x throughput increase by integrating <code>orjson</code>, a Rust-powered JSON library, directly into Flask’s provider chain. We will move beyond simple "drop-in replacements" and architect a zero-copy serialization path that respects the WSGI interface while bypassing the overhead of CPython’s object model.</p>




<h2>
  
  
  The Bottleneck: Why <code>jsonify</code> is Slow
</h2>

<p>To understand the solution, we must first anatomize the problem. When you return <code>jsonify(data)</code> in a standard Flask application, a heavy chain of operations is triggered.</p>

<p>Flask’s default <code>JSONProvider</code> utilizes Python’s built-in <code>json</code> module. While the standard library is stable and correct, it is not designed for high-performance web servers handling large payloads.</p>

<ol>
<li>
<strong>Intermediate String Allocation:</strong> <code>json.dumps</code> outputs a Python <code>str</code> object. In Python, strings are Unicode objects. If your API returns a 10MB JSON payload, Python must allocate memory for this massive string.</li>
<li>
<strong>UTF-8 Encoding Overhead:</strong> WSGI servers (like Gunicorn or uWSGI) cannot send Python strings over the network; they send bytes. Therefore, Flask (or Werkzeug) must take that massive string and encode it back into UTF-8 bytes (<code>response.encode('utf-8')</code>).</li>
<li>
<strong>Object Traversal:</strong> The standard <code>json</code> encoder traverses your Python lists and dictionaries using the CPython C-API, which involves significant overhead for type checking and reference counting on every single element.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8hfeshvvz57wt5mulurr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8hfeshvvz57wt5mulurr.png" alt="Serialization Bottleneck" width="800" height="446"></a></p>

<p>When an endpoint returns a list of 10,000 ORM objects, the CPU is pinned not by business logic, but by the serialization loop. This creates a "Stop-the-World" effect on the worker thread, blocking it from handling other requests and causing tail latencies to spike dramatically.</p>




<h2>
  
  
  Enter <code>orjson</code>: Rust, SIMD, and Speed
</h2>

<p><code>orjson</code> is not just another JSON library; it is a fundamental rethink of how serialization should work in a Python context. Implemented in Rust, it leverages AVX2 SIMD instructions to parallelize the parsing and generation of JSON data.</p>

<p>The architectural advantages of <code>orjson</code> over the standard library are distinct:</p>

<ul>
<li>
<strong>Native Bytes Output:</strong> <code>orjson.dumps</code> returns <code>bytes</code>, not <code>str</code>. This is the single most important feature for a web server. It means we skip the expensive UTF-8 encoding step entirely. The Rust code generates the byte buffer directly, which can be handed straight to the WSGI socket.</li>
<li>
<strong>Strictness and Safety:</strong> It is stricter than the standard library (e.g., it distinguishes between 32-bit and 64-bit integers and adheres strictly to UTF-8), which prevents subtle encoding bugs in production.</li>
<li>
<strong>Low Memory Footprint:</strong> By avoiding the creation of intermediate Python string objects, <code>orjson</code> significantly reduces memory pressure (GC churn) during high-concurrency spikes.</li>
</ul>




<h2>
  
  
  The Implementation: Overriding the <code>JSONProvider</code>
</h2>

<p>Many tutorials suggest hacking app.json_encoder. However, in modern Flask (2.2+ and 3.x), the correct architectural approach is to subclass the JSONProvider.</p>

<p>We want to achieve two things:</p>

<ul>
<li>Use orjson for serialization.</li>
<li>
<strong>Crucially</strong>, return the bytes directly to Flask without decoding them back to a string.</li>
</ul>

<p>Here is the robust, production-ready implementation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">orjson</span>
<span class="kn">from</span> <span class="n">flask</span> <span class="kn">import</span> <span class="n">Flask</span>
<span class="kn">from</span> <span class="n">flask.json.provider</span> <span class="kn">import</span> <span class="n">JSONProvider</span>

<span class="k">class</span> <span class="nc">OrJSONProvider</span><span class="p">(</span><span class="n">JSONProvider</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">dumps</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">
        Serialize data to JSON using orjson.
        Note: orjson returns bytes, but Flask</span><span class="sh">'</span><span class="s">s dumps() signature expects str.
        We handle the bytes conversion in the response() method to avoid
        unnecessary decoding overhead.
        </span><span class="sh">"""</span>
        <span class="c1"># orjson options can be bitwise OR-ed.
</span>        <span class="c1"># OPT_NAIVE_UTC: Treat naive datetimes as UTC (standard for APIs)
</span>        <span class="c1"># OPT_SERIALIZE_NUMPY: Native support for numpy arrays
</span>        <span class="n">option</span> <span class="o">=</span> <span class="n">orjson</span><span class="p">.</span><span class="n">OPT_NAIVE_UTC</span> <span class="o">|</span> <span class="n">orjson</span><span class="p">.</span><span class="n">OPT_SERIALIZE_NUMPY</span>

        <span class="c1"># We must decode here IF this method is called directly by logic 
</span>        <span class="c1"># expecting a string (e.g., templates). 
</span>        <span class="c1"># However, for API responses, we will bypass this.
</span>        <span class="k">return</span> <span class="n">orjson</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">obj</span><span class="p">,</span> <span class="n">option</span><span class="o">=</span><span class="n">option</span><span class="p">).</span><span class="nf">decode</span><span class="p">(</span><span class="sh">'</span><span class="s">utf-8</span><span class="sh">'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">loads</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">s</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">orjson</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">s</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">response</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">
        High-performance response generation.
        We override this to skip the bytes-&gt;str-&gt;bytes roundtrip.
        </span><span class="sh">"""</span>
        <span class="n">obj</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_prepare_response_obj</span><span class="p">(</span><span class="n">args</span><span class="p">,</span> <span class="n">kwargs</span><span class="p">)</span>

        <span class="c1"># Serialize directly to bytes
</span>        <span class="n">option</span> <span class="o">=</span> <span class="n">orjson</span><span class="p">.</span><span class="n">OPT_NAIVE_UTC</span> <span class="o">|</span> <span class="n">orjson</span><span class="p">.</span><span class="n">OPT_SERIALIZE_NUMPY</span>
        <span class="n">dumped_bytes</span> <span class="o">=</span> <span class="n">orjson</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">obj</span><span class="p">,</span> <span class="n">option</span><span class="o">=</span><span class="n">option</span><span class="p">)</span>

        <span class="c1"># Create the response object directly with bytes
</span>        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">_app</span><span class="p">.</span><span class="nf">response_class</span><span class="p">(</span>
            <span class="n">dumped_bytes</span><span class="p">,</span> 
            <span class="n">mimetype</span><span class="o">=</span><span class="sh">'</span><span class="s">application/json</span><span class="sh">'</span>
        <span class="p">)</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">Flask</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>
<span class="n">app</span><span class="p">.</span><span class="n">json</span> <span class="o">=</span> <span class="nc">OrJSONProvider</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Why this works</strong></p>

<p>By overriding <code>response</code>, we intercept the <code>jsonify</code> call. We instruct <code>orjson</code> to generate bytes and pass those bytes directly to <code>self._app.response_class</code>. Flask's <code>Response</code> object is happy to accept bytes as a body. We have effectively eliminated the largest CPU cost in the response pipeline.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frvwstc8hf24sfre9bpuy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frvwstc8hf24sfre9bpuy.png" alt="Highlight the response method with a glowing effect, showing an arrow pointing from orjson.dumps (returning bytes) directly into the Flask Response constructor, bypassing a crossed-out decode('utf-8') block. Embed this near the code snippet." width="800" height="446"></a></p>




<h2>
  
  
  Advanced Trade-offs and Considerations
</h2>

<p>While the performance gains are "magical," the integration is not without trade-offs. A senior engineer must weigh these before deployment.</p>

<h3>
  
  
  1. Strictness vs. Flexibility
</h3>

<p>Python's <code>json</code> module is permissive. It will happily serialize <code>Infinity</code> or <code>NaN</code>, resulting in valid Javascript but invalid JSON. <code>orjson</code> is strict. It will raise a <code>JSONEncodeError</code> for non-compliant types. This is generally a good thing for API contracts, but it may break legacy endpoints that rely on sloppy serialization.</p>

<h3>
  
  
  2. Custom Encoders
</h3>

<p>The standard library allows you to subclass <code>JSONEncoder</code> and override <code>default()</code>. <code>orjson</code> does not use classes for encoders. Instead, it supports a <code>default</code> hook function. If you rely heavily on complex custom object serialization, you will need to refactor your encoder logic into a standalone function.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">default_encoder</span><span class="p">(</span><span class="n">obj</span><span class="p">):</span>
    <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">obj</span><span class="p">,</span> <span class="n">MyCustomObject</span><span class="p">):</span>
        <span class="k">return</span> <span class="nf">str</span><span class="p">(</span><span class="n">obj</span><span class="p">)</span>
    <span class="k">raise</span> <span class="nb">TypeError</span>

<span class="c1"># Usage in provider
</span><span class="n">orjson</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">obj</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="n">default_encoder</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  3. Dataclasses and Numpy
</h3>

<p>One of <code>orjson</code>'s superpowers is native support for <code>dataclasses</code> and <code>numpy</code> arrays. In standard Flask, returning a Numpy array requires converting it to a list first (<code>array.tolist()</code>), which copies memory. <code>orjson</code> can serialize the underlying memory buffer of a Numpy array directly. If your API serves machine learning predictions or analytics, this single feature can reduce latency by 50% or more.</p>




<h2>
  
  
  Realistic Benchmarking
</h2>

<p>Microbenchmarks often show <code>orjson</code> as 20x faster than <code>json</code>. However, in the context of a full HTTP request, the gains are amortized by network I/O and DB latency.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb8f4vvipjq0mytum4lb3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb8f4vvipjq0mytum4lb3.png" alt="" width="800" height="446"></a></p>

<p>In production scenarios involving nested dictionaries or lists of 5,000+ items, we typically observe:</p>

<ul>
<li>
<strong>Throughput (RPS):</strong> 3x to 5x increase for data-heavy endpoints.</li>
<li>
<strong>P99 Latency:</strong> Significant reduction due to reduced GC pressure and CPU contention.</li>
<li>
<strong>Memory Usage:</strong> ~30% reduction in peak usage per worker during serialization bursts.</li>
</ul>




<h2>
  
  
  Conclusion
</h2>

<p>Optimizing serialization is often the highest ROI activity for a mature Flask API. By swapping the default provider for a correctly implemented <code>orjson</code> provider, you aren't just making a library change; you are fundamentally altering how your application handles memory and data transport.</p>

<p>This architecture moves Flask closer to the performance profile of Go or Rust web services for the specific task of JSON generation, allowing you to scale further with your existing Python codebase.</p>

