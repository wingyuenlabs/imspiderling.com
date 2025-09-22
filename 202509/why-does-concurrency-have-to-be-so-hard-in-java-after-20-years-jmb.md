---
Title: Why Does Concurrency Have to Be So Hard in Java After 20 Years?
Description: 
Author: Rob D
Date: 2025-09-22T21:12:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>Java has been around for nearly three decades, and we’ve had threads since day one. We got <code>java.util.concurrent</code> in 2004, lambdas in 2014, CompletableFuture improvements, reactive streams, and virtual threads in 2023…</p>

<p>And yet, writing <em>correct concurrent code</em> in Java still feels like navigating a minefield.</p>

<h1>
  
  
  Why is this still so hard?
</h1>




<h2>
  
  
  The Problem: Too Many Half-Solutions
</h2>

<p>Let’s take a simple case:<br><br>
Fetch data from three APIs concurrently, process results, handle errors, and respect a timeout.</p>

<h3>
  
  
  1. Threads (1995)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="nf">fetchData</span><span class="o">()</span> <span class="o">{</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">results</span> <span class="o">=</span> <span class="nc">Collections</span><span class="o">.</span><span class="na">synchronizedList</span><span class="o">(</span><span class="k">new</span> <span class="nc">ArrayList</span><span class="o">&lt;&gt;());</span>
    <span class="nc">CountDownLatch</span> <span class="n">latch</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CountDownLatch</span><span class="o">(</span><span class="mi">3</span><span class="o">);</span>

    <span class="nc">Thread</span> <span class="n">t1</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Thread</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="o">{</span>
        <span class="k">try</span> <span class="o">{</span>
            <span class="n">results</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="n">callApi</span><span class="o">(</span><span class="s">"api1"</span><span class="o">));</span>
        <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">Exception</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
            <span class="c1">// What do we do here?</span>
        <span class="o">}</span> <span class="k">finally</span> <span class="o">{</span>
            <span class="n">latch</span><span class="o">.</span><span class="na">countDown</span><span class="o">();</span>
        <span class="o">}</span>
    <span class="o">});</span>

    <span class="c1">// … repeat for t2, t3 …</span>

    <span class="n">t1</span><span class="o">.</span><span class="na">start</span><span class="o">();</span> <span class="n">t2</span><span class="o">.</span><span class="na">start</span><span class="o">();</span> <span class="n">t3</span><span class="o">.</span><span class="na">start</span><span class="o">();</span>

    <span class="k">try</span> <span class="o">{</span>
        <span class="n">latch</span><span class="o">.</span><span class="na">await</span><span class="o">(</span><span class="mi">10</span><span class="o">,</span> <span class="nc">TimeUnit</span><span class="o">.</span><span class="na">SECONDS</span><span class="o">);</span> <span class="c1">// What if it times out?</span>
    <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">InterruptedException</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
        <span class="c1">// Now what? Cancel the threads?</span>
    <span class="o">}</span>

    <span class="k">return</span> <span class="n">results</span><span class="o">;</span> <span class="c1">// Hope for the best</span>
<span class="o">}</span>
<span class="nl">Problems:</span>

<span class="nc">Manual</span> <span class="n">lifecycle</span> <span class="n">management</span>

<span class="nc">No</span> <span class="n">consistent</span> <span class="n">error</span> <span class="n">propagation</span>

<span class="nc">Cancellation</span> <span class="n">is</span> <span class="n">basically</span> <span class="n">impossible</span>

<span class="mi">2</span><span class="o">.</span> <span class="nc">ExecutorService</span> <span class="o">(</span><span class="mi">2004</span><span class="o">)</span>
<span class="n">java</span>
<span class="nc">Copy</span> <span class="n">code</span>
<span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="nf">fetchData</span><span class="o">()</span> <span class="kd">throws</span> <span class="nc">Exception</span> <span class="o">{</span>
    <span class="nc">ExecutorService</span> <span class="n">executor</span> <span class="o">=</span> <span class="nc">Executors</span><span class="o">.</span><span class="na">newFixedThreadPool</span><span class="o">(</span><span class="mi">3</span><span class="o">);</span>
    <span class="k">try</span> <span class="o">{</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Future</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;&gt;</span> <span class="n">futures</span> <span class="o">=</span> <span class="nc">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span>
            <span class="n">executor</span><span class="o">.</span><span class="na">submit</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api1"</span><span class="o">)),</span>
            <span class="n">executor</span><span class="o">.</span><span class="na">submit</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api2"</span><span class="o">)),</span>
            <span class="n">executor</span><span class="o">.</span><span class="na">submit</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api3"</span><span class="o">))</span>
        <span class="o">);</span>

        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">results</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayList</span><span class="o">&lt;&gt;();</span>
        <span class="k">for</span> <span class="o">(</span><span class="nc">Future</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">f</span> <span class="o">:</span> <span class="n">futures</span><span class="o">)</span> <span class="o">{</span>
            <span class="k">try</span> <span class="o">{</span>
                <span class="n">results</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="n">f</span><span class="o">.</span><span class="na">get</span><span class="o">(</span><span class="mi">10</span><span class="o">,</span> <span class="nc">TimeUnit</span><span class="o">.</span><span class="na">SECONDS</span><span class="o">));</span>
            <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">TimeoutException</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
                <span class="n">f</span><span class="o">.</span><span class="na">cancel</span><span class="o">(</span><span class="kc">true</span><span class="o">);</span>
            <span class="o">}</span>
        <span class="o">}</span>
        <span class="k">return</span> <span class="n">results</span><span class="o">;</span>
    <span class="o">}</span> <span class="k">finally</span> <span class="o">{</span>
        <span class="n">executor</span><span class="o">.</span><span class="na">shutdown</span><span class="o">();</span>
        <span class="n">executor</span><span class="o">.</span><span class="na">awaitTermination</span><span class="o">(</span><span class="mi">5</span><span class="o">,</span> <span class="nc">TimeUnit</span><span class="o">.</span><span class="na">SECONDS</span><span class="o">);</span>
        <span class="n">executor</span><span class="o">.</span><span class="na">shutdownNow</span><span class="o">();</span> <span class="c1">// Fingers crossed</span>
    <span class="o">}</span>
<span class="o">}</span>
<span class="nc">Better</span><span class="o">,</span> <span class="n">but</span> <span class="n">still</span> <span class="n">verbose</span> <span class="n">and</span> <span class="n">error</span><span class="o">-</span><span class="n">prone</span><span class="o">.</span>

<span class="mi">3</span><span class="o">.</span> <span class="nc">CompletableFuture</span> <span class="o">(</span><span class="mi">2014</span><span class="o">)</span>
<span class="n">java</span>
<span class="nc">Copy</span> <span class="n">code</span>
<span class="kd">public</span> <span class="nc">CompletableFuture</span><span class="o">&lt;</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;&gt;</span> <span class="nf">fetchData</span><span class="o">()</span> <span class="o">{</span>
    <span class="kt">var</span> <span class="n">f1</span> <span class="o">=</span> <span class="nc">CompletableFuture</span><span class="o">.</span><span class="na">supplyAsync</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api1"</span><span class="o">));</span>
    <span class="kt">var</span> <span class="n">f2</span> <span class="o">=</span> <span class="nc">CompletableFuture</span><span class="o">.</span><span class="na">supplyAsync</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api2"</span><span class="o">));</span>
    <span class="kt">var</span> <span class="n">f3</span> <span class="o">=</span> <span class="nc">CompletableFuture</span><span class="o">.</span><span class="na">supplyAsync</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api3"</span><span class="o">));</span>

    <span class="k">return</span> <span class="nc">CompletableFuture</span><span class="o">.</span><span class="na">allOf</span><span class="o">(</span><span class="n">f1</span><span class="o">,</span> <span class="n">f2</span><span class="o">,</span> <span class="n">f3</span><span class="o">)</span>
        <span class="o">.</span><span class="na">thenApply</span><span class="o">(</span><span class="n">v</span> <span class="o">-&gt;</span> <span class="nc">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="n">f1</span><span class="o">.</span><span class="na">join</span><span class="o">(),</span> <span class="n">f2</span><span class="o">.</span><span class="na">join</span><span class="o">(),</span> <span class="n">f3</span><span class="o">.</span><span class="na">join</span><span class="o">()))</span>
        <span class="o">.</span><span class="na">orTimeout</span><span class="o">(</span><span class="mi">10</span><span class="o">,</span> <span class="nc">TimeUnit</span><span class="o">.</span><span class="na">SECONDS</span><span class="o">)</span>
        <span class="o">.</span><span class="na">exceptionally</span><span class="o">(</span><span class="n">ex</span> <span class="o">-&gt;</span> <span class="nc">List</span><span class="o">.</span><span class="na">of</span><span class="o">());</span>
<span class="o">}</span>
<span class="nc">Cleaner</span><span class="o">,</span> <span class="nl">but:</span>

<span class="nc">No</span> <span class="n">structured</span> <span class="n">concurrency</span>

<span class="nc">Cancellation</span> <span class="n">is</span> <span class="n">awkward</span>

<span class="nc">Error</span> <span class="n">handling</span> <span class="n">is</span> <span class="n">ad</span><span class="o">-</span><span class="n">hoc</span>

<span class="mi">4</span><span class="o">.</span> <span class="nc">Virtual</span> <span class="nf">Threads</span> <span class="o">(</span><span class="mi">2023</span><span class="o">)</span>
<span class="n">java</span>
<span class="nc">Copy</span> <span class="n">code</span>
<span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="nf">fetchData</span><span class="o">()</span> <span class="kd">throws</span> <span class="nc">Exception</span> <span class="o">{</span>
    <span class="k">try</span> <span class="o">(</span><span class="kt">var</span> <span class="n">executor</span> <span class="o">=</span> <span class="nc">Executors</span><span class="o">.</span><span class="na">newVirtualThreadPerTaskExecutor</span><span class="o">())</span> <span class="o">{</span>
        <span class="kt">var</span> <span class="n">futures</span> <span class="o">=</span> <span class="nc">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span>
            <span class="n">executor</span><span class="o">.</span><span class="na">submit</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api1"</span><span class="o">)),</span>
            <span class="n">executor</span><span class="o">.</span><span class="na">submit</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api2"</span><span class="o">)),</span>
            <span class="n">executor</span><span class="o">.</span><span class="na">submit</span><span class="o">(()</span> <span class="o">-&gt;</span> <span class="n">callApi</span><span class="o">(</span><span class="s">"api3"</span><span class="o">))</span>
        <span class="o">);</span>

        <span class="k">return</span> <span class="n">futures</span><span class="o">.</span><span class="na">stream</span><span class="o">()</span>
            <span class="o">.</span><span class="na">map</span><span class="o">(</span><span class="n">f</span> <span class="o">-&gt;</span> <span class="o">{</span>
                <span class="k">try</span> <span class="o">{</span>
                    <span class="k">return</span> <span class="n">f</span><span class="o">.</span><span class="na">get</span><span class="o">(</span><span class="mi">10</span><span class="o">,</span> <span class="nc">TimeUnit</span><span class="o">.</span><span class="na">SECONDS</span><span class="o">);</span>
                <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">Exception</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
                    <span class="k">throw</span> <span class="k">new</span> <span class="nf">RuntimeException</span><span class="o">(</span><span class="n">e</span><span class="o">);</span>
                <span class="o">}</span>
            <span class="o">})</span>
            <span class="o">.</span><span class="na">toList</span><span class="o">();</span>
    <span class="o">}</span>
<span class="o">}</span>
<span class="nc">Virtual</span> <span class="n">threads</span> <span class="n">help</span> <span class="n">performance</span><span class="o">,</span> <span class="n">but</span> <span class="n">the</span> <span class="n">core</span> <span class="n">problems</span> <span class="nl">remain:</span>

<span class="nc">No</span> <span class="n">automatic</span> <span class="n">cancellation</span>

<span class="nc">No</span> <span class="n">clear</span> <span class="n">error</span> <span class="n">boundaries</span>

<span class="nc">Timeouts</span> <span class="n">are</span> <span class="n">still</span> <span class="n">manual</span>

<span class="err">#</span> <span class="nc">What</span><span class="err">’</span><span class="n">s</span> <span class="nc">Wrong</span> <span class="nc">Here</span><span class="o">?</span>
<span class="nc">After</span> <span class="mi">20</span><span class="o">+</span> <span class="n">years</span><span class="o">,</span> <span class="nc">Java</span><span class="err">’</span><span class="n">s</span> <span class="n">concurrency</span> <span class="n">is</span> <span class="n">still</span> <span class="nl">missing:</span>

<span class="nc">Structured</span> <span class="nc">Concurrency</span> <span class="err">—</span> <span class="n">no</span> <span class="n">parent</span><span class="o">/</span><span class="n">child</span> <span class="n">lifetimes</span><span class="o">,</span> <span class="n">leading</span> <span class="n">to</span> <span class="n">leaks</span><span class="o">.</span>

<span class="nc">Reliable</span> <span class="nc">Cancellation</span> <span class="err">—</span> <span class="n">interruption</span> <span class="n">is</span> <span class="n">unreliable</span> <span class="n">and</span> <span class="n">inconsistent</span><span class="o">.</span>

<span class="nc">Consistent</span> <span class="nc">Error</span> <span class="nc">Handling</span> <span class="err">—</span> <span class="n">failures</span> <span class="n">don</span><span class="err">’</span><span class="n">t</span> <span class="n">cleanly</span> <span class="n">propagate</span><span class="o">.</span>

<span class="nc">Resource</span> <span class="nc">Safety</span> <span class="err">—</span> <span class="nc">Executors</span> <span class="n">and</span> <span class="n">threads</span> <span class="n">must</span> <span class="n">be</span> <span class="n">closed</span> <span class="n">manually</span><span class="o">.</span>

<span class="nc">Context</span> <span class="err">—</span> <span class="n">no</span> <span class="n">standard</span> <span class="n">way</span> <span class="n">to</span> <span class="n">pass</span> <span class="n">cancellation</span> <span class="n">tokens</span><span class="o">,</span> <span class="n">timeouts</span><span class="o">,</span> <span class="n">or</span> <span class="n">tracing</span><span class="o">.</span>

<span class="nc">Other</span> <span class="n">languages</span> <span class="n">got</span> <span class="k">this</span> <span class="nl">right:</span>

<span class="nc">Go</span> <span class="n">has</span> <span class="n">context</span><span class="o">.</span><span class="na">WithTimeout</span> <span class="k">for</span> <span class="n">group</span> <span class="n">cancellation</span><span class="o">.</span>

<span class="nc">Kotlin</span> <span class="n">has</span> <span class="n">coroutines</span> <span class="n">with</span> <span class="n">structured</span> <span class="n">scopes</span><span class="o">.</span>

<span class="no">C</span><span class="err">#</span> <span class="n">has</span> <span class="nc">Tasks</span> <span class="n">with</span> <span class="nc">CancellationToken</span><span class="o">.</span>

<span class="err">#</span> <span class="nc">What</span> <span class="nc">We</span> <span class="nc">Actually</span> <span class="nc">Want</span>

</code></pre>

</div>



<p>try (var scope = new CoroutineScope()) {<br>
    var results = List.of("api1", "api2", "api3").stream()<br>
        .map(api -&gt; scope.async(suspend -&gt; callApi(suspend, api)))<br>
        .map(handle -&gt; handle.join())<br>
        .toList();<br>
    return results;<br>
} // Automatic cleanup, cancellation, and error propagation</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>

This is:

Structured — parent/child relationships are explicit

Cancellable — cooperative and consistent

Safe — resources cleaned up automatically

Transparent — errors bubble naturally

# Enter JCoroutines 🚀
Concurrency should feel like an elevator: press a few clear buttons and trust the machinery.

That’s what I set out to build with JCoroutines:

Structured concurrency by default

Explicit context passing (cancellation, timeouts, schedulers)

No compiler magic — just clean Java APIs on top of virtual threads

It’s small, explicit, and available today.

Try It Out
On Maven Central:

maven xml
&lt;dependency&gt;
  &lt;groupId&gt;tech.robd&lt;/groupId&gt;
  &lt;artifactId&gt;jcoroutines&lt;/artifactId&gt;
  &lt;version&gt;0.1.0&lt;/version&gt;
&lt;/dependency&gt;
Or Gradle:

kotlin gradle.build.kts

implementation("tech.robd:jcoroutines:0.1.0")

The Path Forward
Java itself is heading this way (see JEP 428 on structured concurrency), but it will take years before that’s fully stable.

Meanwhile, JCoroutines gives you these patterns now — using just Java 21+ and virtual threads.

[📦 Maven Central](https://central.sonatype.com/artifact/tech.robd/jcoroutines)

[💻 GitHub repo](https://github.com/robdeas/jcoroutines)


</code></pre>

</div>

