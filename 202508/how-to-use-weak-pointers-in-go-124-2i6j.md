---
Title: How to Use Weak Pointers in Go 1.24
Description: 
Author: Leapcell
Date: 2025-08-25T21:58:52.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://leapcell.io/?lc_t=d_goweakp" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq4ns62a2a3yxa07gg538.png" alt="Cover" width="800" height="336"></a></p>

<p>In Go, a <strong>weak pointer</strong> refers to a reference that does not prevent the garbage collector (GC) from reclaiming the target object.</p>

<p>When an object is only referenced by weak pointers and has no strong references, the GC will still treat it as unreachable and reclaim it; afterwards, all weak pointers to it will automatically become <code>nil</code>.</p>

<p>In short, a weak pointer does not increase an object’s reference count. When an object is referenced only by weak pointers, the garbage collector can free it. Therefore, before attempting to use the value of a weak pointer, you should check whether it is <code>nil</code>.</p>

<h2>
  
  
  The <code>weak</code> Package in Go 1.24
</h2>

<p>Go 1.24 introduces the <code>weak</code> package, which provides a concise API for creating and using weak pointers.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">import</span> <span class="s">"weak"</span>

<span class="k">type</span> <span class="n">MyStruct</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Data</span> <span class="kt">string</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">obj</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">MyStruct</span><span class="p">{</span><span class="n">Data</span><span class="o">:</span> <span class="s">"example"</span><span class="p">}</span>
    <span class="n">wp</span> <span class="o">:=</span> <span class="n">weak</span><span class="o">.</span><span class="n">Make</span><span class="p">(</span><span class="n">obj</span><span class="p">)</span> <span class="c">// create weak pointer</span>
    <span class="n">val</span> <span class="o">:=</span> <span class="n">wp</span><span class="o">.</span><span class="n">Value</span><span class="p">()</span>    <span class="c">// get strong reference or nil</span>
    <span class="k">if</span> <span class="n">val</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">val</span><span class="o">.</span><span class="n">Data</span><span class="p">)</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"object has been garbage collected"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In the example above, <code>weak.Make(obj)</code> creates a weak pointer to <code>obj</code>.<br>
When calling <code>wp.Value()</code>, it returns a strong reference if the object is still alive; otherwise, it returns <code>nil</code>.</p>
<h3>
  
  
  Testing Weak Pointers
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">import</span> <span class="p">(</span>
    <span class="s">"fmt"</span>
    <span class="s">"runtime"</span>
    <span class="s">"weak"</span>
<span class="p">)</span>

<span class="k">type</span> <span class="n">MyStruct</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Data</span> <span class="kt">string</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">obj</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">MyStruct</span><span class="p">{</span><span class="n">Data</span><span class="o">:</span> <span class="s">"test"</span><span class="p">}</span>
    <span class="n">wp</span> <span class="o">:=</span> <span class="n">weak</span><span class="o">.</span><span class="n">Make</span><span class="p">(</span><span class="n">obj</span><span class="p">)</span>
    <span class="n">obj</span> <span class="o">=</span> <span class="no">nil</span> <span class="c">// remove strong reference</span>
    <span class="n">runtime</span><span class="o">.</span><span class="n">GC</span><span class="p">()</span>
    <span class="k">if</span> <span class="n">wp</span><span class="o">.</span><span class="n">Value</span><span class="p">()</span> <span class="o">==</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"object has been garbage collected"</span><span class="p">)</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"object is still alive"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>


<p>By setting the strong reference <code>obj</code> to <code>nil</code> and explicitly triggering GC,<br>
you can observe that the weak pointer returns <code>nil</code> once the object has been collected.</p>
<h2>
  
  
  Differences Between Weak Pointers and Strong References
</h2>
<h3>
  
  
  Impact on Garbage Collection (GC):
</h3>

<ul>
<li>Strong references keep objects alive.</li>
<li>Weak references do not keep objects alive.</li>
</ul>
<h3>
  
  
  Nil Values:
</h3>

<ul>
<li>A strong reference is <code>nil</code> when it explicitly points to nothing.</li>
<li>A weak reference is <code>nil</code> usually because the target object has already been collected, or it has not yet been assigned.</li>
</ul>
<h3>
  
  
  Access Method:
</h3>

<ul>
<li>A strong reference can directly dereference the object.</li>
<li>A weak reference requires calling the <code>Value()</code> method before accessing the object.</li>
</ul>
<h2>
  
  
  Example 1: Using Weak Pointers for Temporary Caching
</h2>

<p>A typical scenario for weak pointers is storing entries in a cache without preventing them from being collected by the GC.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"fmt"</span>
    <span class="s">"runtime"</span>
    <span class="s">"sync"</span>
    <span class="s">"weak"</span>
<span class="p">)</span>

<span class="k">type</span> <span class="n">User</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Name</span> <span class="kt">string</span>
<span class="p">}</span>

<span class="k">var</span> <span class="n">cache</span> <span class="n">sync</span><span class="o">.</span><span class="n">Map</span> <span class="c">// map[int]weak.Pointer[*User]</span>

<span class="k">func</span> <span class="n">GetUser</span><span class="p">(</span><span class="n">id</span> <span class="kt">int</span><span class="p">)</span> <span class="o">*</span><span class="n">User</span> <span class="p">{</span>
    <span class="c">// ① Try to fetch from cache first</span>
    <span class="k">if</span> <span class="n">wp</span><span class="p">,</span> <span class="n">ok</span> <span class="o">:=</span> <span class="n">cache</span><span class="o">.</span><span class="n">Load</span><span class="p">(</span><span class="n">id</span><span class="p">);</span> <span class="n">ok</span> <span class="p">{</span>
        <span class="k">if</span> <span class="n">u</span> <span class="o">:=</span> <span class="n">wp</span><span class="o">.</span><span class="p">(</span><span class="n">weak</span><span class="o">.</span><span class="n">Pointer</span><span class="p">[</span><span class="n">User</span><span class="p">])</span><span class="o">.</span><span class="n">Value</span><span class="p">();</span> <span class="n">u</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"cache hit"</span><span class="p">)</span>
            <span class="k">return</span> <span class="n">u</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="c">// ② Actually load (here, just construct)</span>
    <span class="n">u</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">User</span><span class="p">{</span><span class="n">Name</span><span class="o">:</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"user-%d"</span><span class="p">,</span> <span class="n">id</span><span class="p">)}</span>
    <span class="n">cache</span><span class="o">.</span><span class="n">Store</span><span class="p">(</span><span class="n">id</span><span class="p">,</span> <span class="n">weak</span><span class="o">.</span><span class="n">Make</span><span class="p">(</span><span class="n">u</span><span class="p">))</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"load from DB"</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">u</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">u</span> <span class="o">:=</span> <span class="n">GetUser</span><span class="p">(</span><span class="m">1</span><span class="p">)</span> <span class="c">// load from DB</span>
    <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">u</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

    <span class="n">runtime</span><span class="o">.</span><span class="n">GC</span><span class="p">()</span> <span class="c">// even if GC runs immediately, `main` holds a strong ref, so User survives</span>

    <span class="n">u</span> <span class="o">=</span> <span class="no">nil</span>      <span class="c">// release the last strong reference</span>
    <span class="n">runtime</span><span class="o">.</span><span class="n">GC</span><span class="p">()</span> <span class="c">// trigger GC, User may be collected</span>

    <span class="n">_</span> <span class="o">=</span> <span class="n">GetUser</span><span class="p">(</span><span class="m">1</span><span class="p">)</span> <span class="c">// if collected, it will load from DB again</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In this cache implementation, entries are stored as weak pointers. If the object has no other strong references, the GC can reclaim it; on the next call to <code>GetUser</code>, the data will be reloaded.</p>

<p>Running the code above produces the following output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>go run cache.go
load from DB
user-1
load from DB
</code></pre>

</div>



<h3>
  
  
  Why Use Weak Pointers?
</h3>

<p>Common scenarios include:</p>

<ul>
<li>
<strong>Caching</strong>: Store objects without forcing them to remain in memory; if not used elsewhere, they can be collected.</li>
<li>
<strong>Observer pattern</strong>: Keep references to observers without preventing them from being collected.</li>
<li>
<strong>Canonicalization</strong>: Ensure only one instance of the same object exists, while allowing unused ones to be collected.</li>
<li>
<strong>Dependency graphs</strong>: Prevent reference cycles in structures like trees or graphs.</li>
</ul>

<h3>
  
  
  Notes on Using Weak Pointers
</h3>

<ul>
<li>
<strong>Always check for nil</strong>: Objects may be collected at any GC cycle; the result of <code>Value()</code> must not be cached.</li>
<li>
<strong>Avoid circular dependencies</strong>: Don’t let objects referenced by weak pointers hold strong references back to the container, or cycles may still form.</li>
<li>
<strong>Performance trade-offs</strong>: Accessing a weak pointer requires an extra call, and frequently reloading objects that fall back to <code>nil</code> may cause churn.</li>
</ul>

<h2>
  
  
  Example 2: Normal Use of Strong Pointers
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
   <span class="s">"fmt"</span>
   <span class="s">"runtime"</span>
<span class="p">)</span>

<span class="k">type</span> <span class="n">Session</span> <span class="k">struct</span> <span class="p">{</span>
   <span class="n">ID</span> <span class="kt">string</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
   <span class="n">s</span> <span class="o">:=</span> <span class="nb">new</span><span class="p">(</span><span class="n">Session</span><span class="p">)</span> <span class="c">// equivalent to &amp;Session{}</span>
   <span class="n">s</span><span class="o">.</span><span class="n">ID</span> <span class="o">=</span> <span class="s">"abc123"</span>

   <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"strong ref alive:"</span><span class="p">,</span> <span class="n">s</span><span class="o">.</span><span class="n">ID</span><span class="p">)</span>

   <span class="n">s</span> <span class="o">=</span> <span class="no">nil</span>           <span class="c">// remove the last strong reference</span>
   <span class="n">runtime</span><span class="o">.</span><span class="n">GC</span><span class="p">()</span>      <span class="c">// attempt to trigger GC (for demo only, actual timing depends on runtime)</span>

   <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"done"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Here, <code>s</code> is a strong pointer. As long as it remains reachable, the <code>Session</code> object will never be collected by the GC.</p>

<h3>
  
  
  When Is an Object Pointed to by a Strong Pointer Collected?
</h3>

<ul>
<li>
<p><strong>Reachability analysis</strong>: Go uses a mark-and-sweep GC. At the start of a GC cycle, the runtime traverses all strong references from root objects (stack, global variables, current registers, etc.).</p>

<ul>
<li>Objects reachable through strong references are considered <em>reachable</em> and will stay alive.</li>
<li>All others are marked <em>unreachable</em> and freed during the sweep phase.</li>
</ul>


</li>

<li><p><strong>No reference counting</strong>: Go only checks whether an object is reachable from roots. The number of references or whether values are equal does not affect collection.</p></li>

<li><p><strong>Uncertain timing</strong>: GC cycles are triggered automatically by the scheduler. Developers can call <code>runtime.GC()</code> as a <em>hint</em>, but this does not guarantee immediate collection.</p></li>

<li><p><strong>Variables themselves may be collected</strong>: If a strong pointer variable <code>s</code> resides on the heap and the structure holding it is no longer reachable, then <code>s</code> itself may be collected. Stack variables are freed when the function returns.</p></li>

</ul>

<h2>
  
  
  Summary
</h2>

<p><strong>A strong pointer guarantees that the object it points to remains in the “live set” across all GC cycles. Once the last strong reference is broken, the object will be automatically released during the next GC cycle.</strong></p>




<h3>
  
  
  <a href="https://leapcell.io/?lc_t=d_goweakp" rel="noopener noreferrer">We are Leapcell, your top choice for hosting Go projects.</a>
</h3>

<p><a href="https://leapcell.io/?lc_t=d_goweakp" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1spys5a7iav85ostyvur.png" alt="Leapcell" width="800" height="180"></a></p>

<p><a href="https://leapcell.io/?lc_t=d_goweakp" rel="noopener noreferrer">Leapcell</a> is the Next-Gen Serverless Platform for Web Hosting, Async Tasks, and Redis:</p>

<p><strong>Multi-Language Support</strong></p>

<ul>
<li>Develop with Node.js, Python, Go, or Rust.</li>
</ul>

<p><strong>Deploy unlimited projects for free</strong></p>

<ul>
<li>pay only for usage — no requests, no charges.</li>
</ul>

<p><strong>Unbeatable Cost Efficiency</strong></p>

<ul>
<li>Pay-as-you-go with no idle charges.</li>
<li>Example: $25 supports 6.94M requests at a 60ms average response time.</li>
</ul>

<p><strong>Streamlined Developer Experience</strong></p>

<ul>
<li>Intuitive UI for effortless setup.</li>
<li>Fully automated CI/CD pipelines and GitOps integration.</li>
<li>Real-time metrics and logging for actionable insights.</li>
</ul>

<p><strong>Effortless Scalability and High Performance</strong></p>

<ul>
<li>Auto-scaling to handle high concurrency with ease.</li>
<li>Zero operational overhead — just focus on building.</li>
</ul>

<p>Explore more in the <a href="https://docs.leapcell.io/" rel="noopener noreferrer">Documentation</a>!</p>

<p><a href="https://leapcell.io/?lc_t=d_goweakp" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7jysgde178rto5utpb08.png" alt="Try Leapcell" width="513" height="120"></a></p>

<p>Follow us on X: <a href="https://x.com/LeapcellHQ" rel="noopener noreferrer">@LeapcellHQ</a></p>




<p><a href="https://leapcell.io/blog/understanding-weak-pointers-in-go-1-24" rel="noopener noreferrer">Read on our blog</a></p>

