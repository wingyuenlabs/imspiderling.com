---
Title: Parallel Programming in Zig: Threads, Shared Memory, and Synchronization
Description: 
Author: Seven
Date: 2026-02-26T21:36:38.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>This post continues my exploration of low-level programming and the Zig language. Today, we will explore the fundamental concepts of parallel programming. We'll start by defining what threads are, then move on to spawning them in Zig, and finally, we'll see how to handle shared memory safely using synchronization primitives.</p>




<h2>
  
  
  Step 1: The Basic Process
</h2>

<p>Every program runs as a process with at least one thread. Let's start by creating a simple task and running it in our <code>main</code> function. At this stage, everything is sequential.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight zig"><code><span class="k">const</span> <span class="n">std</span> <span class="o">=</span> <span class="nb">@import</span><span class="p">(</span><span class="s">"std"</span><span class="p">);</span>

<span class="k">pub</span> <span class="k">fn</span> <span class="n">main</span><span class="p">()</span> <span class="o">!</span><span class="k">void</span> <span class="p">{</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Starting main thread...</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{});</span>
    <span class="n">task</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Finished main thread.</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{});</span>
<span class="p">}</span>

<span class="k">fn</span> <span class="n">task</span><span class="p">(</span><span class="n">id</span><span class="p">:</span> <span class="kt">usize</span><span class="p">)</span> <span class="k">void</span> <span class="p">{</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"task {} is running</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="n">id</span><span class="p">});</span>
    <span class="k">var</span> <span class="n">ts</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">posix</span><span class="p">.</span><span class="py">timespec</span><span class="p">{</span> <span class="p">.</span><span class="py">sec</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="p">.</span><span class="py">nsec</span> <span class="o">=</span> <span class="mi">0</span> <span class="p">};</span>
    <span class="mi">_</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">posix</span><span class="p">.</span><span class="py">system</span><span class="p">.</span><span class="nf">nanosleep</span><span class="p">(</span><span class="o">&amp;</span><span class="n">ts</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">ts</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Output:
Starting main thread...
task 1 is running
Finished main thread.
</code></pre>

</div>






<h2>
  
  
  Step 2: Spawning Your First Thread
</h2>

<p>Now, let's use <code>std.Thread.spawn</code> to run the task on a separate path of execution. We use <code>thread.join()</code> to tell the main thread to wait for the worker to finish.</p>

<p>Note that in Zig 0.16, <code>main</code> can receive a <code>std.process.Init</code> argument, which gives us access to <code>std.Io</code> — used here to measure elapsed time with <code>std.Io.Clock</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight zig"><code><span class="k">const</span> <span class="n">std</span> <span class="o">=</span> <span class="nb">@import</span><span class="p">(</span><span class="s">"std"</span><span class="p">);</span>

<span class="k">pub</span> <span class="k">fn</span> <span class="n">main</span><span class="p">(</span><span class="n">init</span><span class="p">:</span> <span class="n">std</span><span class="p">.</span><span class="py">process</span><span class="p">.</span><span class="py">Init</span><span class="p">)</span> <span class="o">!</span><span class="k">void</span> <span class="p">{</span>
    <span class="k">const</span> <span class="n">start</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">.</span><span class="py">Clock</span><span class="p">.</span><span class="nf">now</span><span class="p">(.</span><span class="py">real</span><span class="p">,</span> <span class="n">init</span><span class="p">.</span><span class="py">io</span><span class="p">);</span>
    <span class="k">const</span> <span class="n">thread</span> <span class="o">=</span> <span class="k">try</span> <span class="n">std</span><span class="p">.</span><span class="py">Thread</span><span class="p">.</span><span class="nf">spawn</span><span class="p">(</span><span class="o">.</span><span class="p">{},</span> <span class="n">task</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="mi">1</span><span class="p">});</span>
    <span class="n">thread</span><span class="p">.</span><span class="nf">join</span><span class="p">();</span> <span class="c">// This blocks the main thread until the task is done</span>
    <span class="k">const</span> <span class="n">end</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">.</span><span class="py">Clock</span><span class="p">.</span><span class="nf">now</span><span class="p">(.</span><span class="py">real</span><span class="p">,</span> <span class="n">init</span><span class="p">.</span><span class="py">io</span><span class="p">);</span>
    <span class="k">const</span> <span class="n">duration</span> <span class="o">=</span> <span class="n">start</span><span class="p">.</span><span class="nf">durationTo</span><span class="p">(</span><span class="n">end</span><span class="p">);</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Time: {}ms</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="n">duration</span><span class="p">.</span><span class="nf">toMilliseconds</span><span class="p">()});</span>
<span class="p">}</span>

<span class="k">fn</span> <span class="n">task</span><span class="p">(</span><span class="n">id</span><span class="p">:</span> <span class="kt">usize</span><span class="p">)</span> <span class="k">void</span> <span class="p">{</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Task {} is running thread: {} </span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span> <span class="n">id</span><span class="p">,</span> <span class="n">std</span><span class="p">.</span><span class="py">Thread</span><span class="p">.</span><span class="nf">getCurrentId</span><span class="p">()</span> <span class="p">});</span>
    <span class="k">var</span> <span class="n">ts</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">posix</span><span class="p">.</span><span class="py">timespec</span><span class="p">{</span> <span class="p">.</span><span class="py">sec</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="p">.</span><span class="py">nsec</span> <span class="o">=</span> <span class="mi">0</span> <span class="p">};</span>
    <span class="mi">_</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">posix</span><span class="p">.</span><span class="py">system</span><span class="p">.</span><span class="nf">nanosleep</span><span class="p">(</span><span class="o">&amp;</span><span class="n">ts</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">ts</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Output:
Task 1 is running thread: 1134137
Time: 1000ms
</code></pre>

</div>






<h2>
  
  
  Step 3: Running in Parallel
</h2>

<p>To use your CPU cores effectively, we can spawn multiple threads. By storing them in an array and joining them after spawning all of them, they all work at the same time.</p>

<p>Notice that 4 threads each sleeping for 1 second still complete in ~1000ms total — they truly run in parallel.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight zig"><code><span class="k">const</span> <span class="n">std</span> <span class="o">=</span> <span class="nb">@import</span><span class="p">(</span><span class="s">"std"</span><span class="p">);</span>

<span class="k">pub</span> <span class="k">fn</span> <span class="n">main</span><span class="p">(</span><span class="n">init</span><span class="p">:</span> <span class="n">std</span><span class="p">.</span><span class="py">process</span><span class="p">.</span><span class="py">Init</span><span class="p">)</span> <span class="o">!</span><span class="k">void</span> <span class="p">{</span>
    <span class="k">const</span> <span class="n">start</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">.</span><span class="py">Clock</span><span class="p">.</span><span class="nf">now</span><span class="p">(.</span><span class="py">real</span><span class="p">,</span> <span class="n">init</span><span class="p">.</span><span class="py">io</span><span class="p">);</span>
    <span class="k">var</span> <span class="n">threads</span><span class="p">:</span> <span class="p">[</span><span class="mi">4</span><span class="p">]</span><span class="n">std</span><span class="p">.</span><span class="py">Thread</span> <span class="o">=</span> <span class="k">undefined</span><span class="p">;</span>
    <span class="k">for</span> <span class="p">(</span><span class="o">&amp;</span><span class="n">threads</span><span class="p">,</span> <span class="mi">0</span><span class="o">..</span><span class="p">)</span> <span class="p">|</span><span class="o">*</span><span class="n">t</span><span class="p">,</span> <span class="n">i</span><span class="p">|</span> <span class="p">{</span>
        <span class="n">t</span><span class="o">.*</span> <span class="o">=</span> <span class="k">try</span> <span class="n">std</span><span class="p">.</span><span class="py">Thread</span><span class="p">.</span><span class="nf">spawn</span><span class="p">(</span><span class="o">.</span><span class="p">{},</span> <span class="n">task</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="n">i</span><span class="p">});</span>
    <span class="p">}</span>
    <span class="k">for</span> <span class="p">(</span><span class="n">threads</span><span class="p">)</span> <span class="p">|</span><span class="n">t</span><span class="p">|</span> <span class="n">t</span><span class="p">.</span><span class="nf">join</span><span class="p">();</span>
    <span class="k">const</span> <span class="n">end</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">.</span><span class="py">Clock</span><span class="p">.</span><span class="nf">now</span><span class="p">(.</span><span class="py">real</span><span class="p">,</span> <span class="n">init</span><span class="p">.</span><span class="py">io</span><span class="p">);</span>
    <span class="k">const</span> <span class="n">duration</span> <span class="o">=</span> <span class="n">start</span><span class="p">.</span><span class="nf">durationTo</span><span class="p">(</span><span class="n">end</span><span class="p">);</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Time: {}ms</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="n">duration</span><span class="p">.</span><span class="nf">toMilliseconds</span><span class="p">()});</span>
<span class="p">}</span>

<span class="k">fn</span> <span class="n">task</span><span class="p">(</span><span class="n">id</span><span class="p">:</span> <span class="kt">usize</span><span class="p">)</span> <span class="k">void</span> <span class="p">{</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Task {} is running thread: {} </span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span> <span class="n">id</span><span class="p">,</span> <span class="n">std</span><span class="p">.</span><span class="py">Thread</span><span class="p">.</span><span class="nf">getCurrentId</span><span class="p">()</span> <span class="p">});</span>
    <span class="k">var</span> <span class="n">ts</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">posix</span><span class="p">.</span><span class="py">timespec</span><span class="p">{</span> <span class="p">.</span><span class="py">sec</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="p">.</span><span class="py">nsec</span> <span class="o">=</span> <span class="mi">0</span> <span class="p">};</span>
    <span class="mi">_</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">posix</span><span class="p">.</span><span class="py">system</span><span class="p">.</span><span class="nf">nanosleep</span><span class="p">(</span><span class="o">&amp;</span><span class="n">ts</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">ts</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Output:
Task 0 is running thread: 1134350
Task 1 is running thread: 1134351
Task 2 is running thread: 1134352
Task 3 is running thread: 1134353
Time: 1000ms
</code></pre>

</div>






<h2>
  
  
  Step 4: The Shared Memory Problem (Race Condition)
</h2>

<p>Threads share the same memory space. If multiple threads try to update the same variable at once, they will overwrite each other's changes, causing a <strong>race condition</strong> — the final result will be inconsistent and unpredictable.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight zig"><code><span class="k">const</span> <span class="n">std</span> <span class="o">=</span> <span class="nb">@import</span><span class="p">(</span><span class="s">"std"</span><span class="p">);</span>

<span class="c">// This will produce inconsistent results!</span>
<span class="k">pub</span> <span class="k">fn</span> <span class="n">main</span><span class="p">()</span> <span class="o">!</span><span class="k">void</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">arr</span> <span class="o">=</span> <span class="p">[</span><span class="mi">_</span><span class="p">]</span><span class="kt">i32</span><span class="p">{</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span> <span class="p">};</span>
    <span class="k">var</span> <span class="n">threads</span><span class="p">:</span> <span class="p">[</span><span class="mi">5</span><span class="p">]</span><span class="n">std</span><span class="p">.</span><span class="py">Thread</span> <span class="o">=</span> <span class="k">undefined</span><span class="p">;</span>
    <span class="k">for</span> <span class="p">(</span><span class="o">&amp;</span><span class="n">threads</span><span class="p">)</span> <span class="p">|</span><span class="o">*</span><span class="n">t</span><span class="p">|</span> <span class="p">{</span>
        <span class="n">t</span><span class="o">.*</span> <span class="o">=</span> <span class="k">try</span> <span class="n">std</span><span class="p">.</span><span class="py">Thread</span><span class="p">.</span><span class="nf">spawn</span><span class="p">(</span><span class="o">.</span><span class="p">{},</span> <span class="n">task</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="o">&amp;</span><span class="n">arr</span><span class="p">});</span>
    <span class="p">}</span>
    <span class="k">for</span> <span class="p">(</span><span class="n">threads</span><span class="p">)</span> <span class="p">|</span><span class="n">t</span><span class="p">|</span> <span class="n">t</span><span class="p">.</span><span class="nf">join</span><span class="p">();</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Result: {any}</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="n">arr</span><span class="p">});</span>
<span class="p">}</span>

<span class="k">fn</span> <span class="n">task</span><span class="p">(</span><span class="n">arr</span><span class="p">:</span> <span class="o">*</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span><span class="kt">i32</span><span class="p">)</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">for</span> <span class="p">(</span><span class="mi">0</span><span class="o">..</span><span class="mi">100000</span><span class="p">)</span> <span class="p">|</span><span class="mi">_</span><span class="p">|</span> <span class="p">{</span>
        <span class="k">for</span> <span class="p">(</span><span class="mi">0</span><span class="o">..</span><span class="mi">3</span><span class="p">)</span> <span class="p">|</span><span class="n">j</span><span class="p">|</span> <span class="n">arr</span><span class="p">[</span><span class="n">j</span><span class="p">]</span> <span class="o">+=</span> <span class="mi">1</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Output:
Result: { 311264, 289236, 273695 }
</code></pre>

</div>



<p>The expected result would be <code>{ 500000, 500000, 500000 }</code> (5 threads × 100000 iterations), but we get something different — and it changes on every run. That is the race condition in action.</p>




<h2>
  
  
  Step 5: Fixing it with Mutex and Defer
</h2>

<p>To fix the bug, we use a <code>Mutex</code> to lock the critical section — the block of code that accesses shared data. Only one thread can hold the lock at a time, so the others wait their turn.</p>

<p>In Zig 0.16, the mutex is <code>std.Io.Mutex</code> and its <code>lock</code>/<code>unlock</code> methods require passing the <code>std.Io</code> handle. We also pass <code>io</code> down to the task function for this reason.</p>

<p>Notice that we lock and unlock <strong>inside</strong> the loop, wrapping only the minimal critical section. Locking outside the loop would force threads to run one at a time for their entire duration, eliminating any parallelism benefit.</p>

<p>We also use <code>defer mutex.unlock(io)</code> immediately after the lock. This ensures the lock is always released when the block exits, even if an error occurs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight zig"><code><span class="k">const</span> <span class="n">std</span> <span class="o">=</span> <span class="nb">@import</span><span class="p">(</span><span class="s">"std"</span><span class="p">);</span>

<span class="k">pub</span> <span class="k">fn</span> <span class="n">main</span><span class="p">(</span><span class="n">init</span><span class="p">:</span> <span class="n">std</span><span class="p">.</span><span class="py">process</span><span class="p">.</span><span class="py">Init</span><span class="p">)</span> <span class="o">!</span><span class="k">void</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">arr</span> <span class="o">=</span> <span class="p">[</span><span class="mi">_</span><span class="p">]</span><span class="kt">i32</span><span class="p">{</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span> <span class="p">};</span>
    <span class="k">var</span> <span class="n">mutex</span> <span class="o">=</span> <span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">.</span><span class="py">Mutex</span><span class="p">.</span><span class="py">init</span><span class="p">;</span>
    <span class="k">var</span> <span class="n">threads</span><span class="p">:</span> <span class="p">[</span><span class="mi">5</span><span class="p">]</span><span class="n">std</span><span class="p">.</span><span class="py">Thread</span> <span class="o">=</span> <span class="k">undefined</span><span class="p">;</span>
    <span class="k">for</span> <span class="p">(</span><span class="o">&amp;</span><span class="n">threads</span><span class="p">)</span> <span class="p">|</span><span class="o">*</span><span class="n">t</span><span class="p">|</span> <span class="p">{</span>
        <span class="n">t</span><span class="o">.*</span> <span class="o">=</span> <span class="k">try</span> <span class="n">std</span><span class="p">.</span><span class="py">Thread</span><span class="p">.</span><span class="nf">spawn</span><span class="p">(</span><span class="o">.</span><span class="p">{},</span> <span class="n">task</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span> <span class="o">&amp;</span><span class="n">arr</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">mutex</span><span class="p">,</span> <span class="n">init</span><span class="p">.</span><span class="py">io</span> <span class="p">});</span>
    <span class="p">}</span>
    <span class="k">for</span> <span class="p">(</span><span class="n">threads</span><span class="p">)</span> <span class="p">|</span><span class="n">t</span><span class="p">|</span> <span class="n">t</span><span class="p">.</span><span class="nf">join</span><span class="p">();</span>
    <span class="n">std</span><span class="p">.</span><span class="py">debug</span><span class="p">.</span><span class="nf">print</span><span class="p">(</span><span class="s">"Result: {any}</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="o">.</span><span class="p">{</span><span class="n">arr</span><span class="p">});</span>
<span class="p">}</span>

<span class="k">fn</span> <span class="n">task</span><span class="p">(</span><span class="n">arr</span><span class="p">:</span> <span class="o">*</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span><span class="kt">i32</span><span class="p">,</span> <span class="n">mutex</span><span class="p">:</span> <span class="o">*</span><span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">.</span><span class="py">Mutex</span><span class="p">,</span> <span class="n">io</span><span class="p">:</span> <span class="n">std</span><span class="p">.</span><span class="py">Io</span><span class="p">)</span> <span class="o">!</span><span class="k">void</span> <span class="p">{</span>
    <span class="k">for</span> <span class="p">(</span><span class="mi">0</span><span class="o">..</span><span class="mi">100000</span><span class="p">)</span> <span class="p">|</span><span class="mi">_</span><span class="p">|</span> <span class="p">{</span>
        <span class="p">{</span>
            <span class="k">try</span> <span class="n">mutex</span><span class="p">.</span><span class="nf">lock</span><span class="p">(</span><span class="n">io</span><span class="p">);</span>
            <span class="k">defer</span> <span class="n">mutex</span><span class="p">.</span><span class="nf">unlock</span><span class="p">(</span><span class="n">io</span><span class="p">);</span> <span class="c">// released at the end of this block</span>
            <span class="k">for</span> <span class="p">(</span><span class="mi">0</span><span class="o">..</span><span class="mi">3</span><span class="p">)</span> <span class="p">|</span><span class="n">j</span><span class="p">|</span> <span class="n">arr</span><span class="p">[</span><span class="n">j</span><span class="p">]</span> <span class="o">+=</span> <span class="mi">1</span><span class="p">;</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Output:
Result: { 500000, 500000, 500000 }
</code></pre>

</div>



<p>Now the result is consistent and correct on every run.</p>

<blockquote>
<p><strong>Tip:</strong> For simple numeric operations on a single variable, Zig also provides <code>std.atomic.Value</code>, which can be more efficient than a Mutex since it avoids the overhead of locking entirely.</p>
</blockquote>




<h2>
  
  
  Conclusion
</h2>

<p>Parallel programming is a powerful tool for building high-performance software, but it requires a solid understanding of how threads interact. We've seen how easy it is to spawn threads in Zig, but also how quickly shared memory can lead to subtle bugs. By using tools like <code>std.Io.Mutex</code> — and understanding <em>where</em> to apply them — we can protect our data and ensure our programs remain correct and reliable as they scale across multiple CPU cores.</p>




<h2>
  
  
  References
</h2>

<h3>
  
  
  Sources &amp; Further Reading
</h3>

<ul>
<li><a href="https://ziglang.org/" rel="noopener noreferrer">Zig Language Official Site</a></li>
<li><a href="https://pedropark99.github.io/zig-book/Chapters/14-threads.html" rel="noopener noreferrer">Zig-Book: Threads Chapter</a></li>
<li><a href="https://andrewkelley.me/post/zig-new-async-io-text-version.html" rel="noopener noreferrer">Andrew Kelley: Zig's New Async/IO</a></li>
<li><a href="https://www.reddit.com/r/Zig/comments/mytkpv/multithreading_in_zig/" rel="noopener noreferrer">Reddit: Multithreading in Zig Discussion</a></li>
<li><a href="https://www.youtube.com/watch?v=axphlFa3xB4" rel="noopener noreferrer">Visualizing Threading Concepts (Video)</a></li>
</ul>




<h2>
  
  
  Contact
</h2>

<p>Feel free to reach out or follow my work:</p>

<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/llllOllOOll/zig_thread_tutorial" rel="noopener noreferrer">source</a>
</li>
<li>
<strong>x:</strong> <a href="https://x.com/1111O11OO11" rel="noopener noreferrer">https://x.com/1111O11OO11</a>
</li>
<li>
<strong>Email:</strong> <a href="mailto:7b37b3@gmail.com">7b37b3@gmail.com</a>
</li>
</ul>




<p><em>Written with 0.16.0-dev.2565+684032671 — All code tested and verified.</em></p>

