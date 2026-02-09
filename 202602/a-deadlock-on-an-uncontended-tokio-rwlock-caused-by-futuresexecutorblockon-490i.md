---
Title: A deadlock on an uncontended Tokio RwLock (caused by futures::executor::block_on)
Description: 
Author: Cristian Spinetta
Date: 2026-02-09T21:52:52.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  A weird test hang
</h2>

<p>I recently hit a Rust test that hung forever:</p>

<ul>
<li>no panic</li>
<li>no error</li>
<li>no timeout</li>
<li>just… silence</li>
</ul>

<p>The confusing part: the code path only used <code>tokio::sync::RwLock::read()</code>. No writers, no contention. Two reads succeeded. The third <code>read().await</code> never returned.</p>

<p>That sounds impossible if you're thinking in "classic lock" terms.</p>

<p>But the root cause wasn't contention. It was <strong>executor semantics</strong>.</p>

<h2>
  
  
  The minimal reproducer
</h2>

<p>I published a tiny repro repo that demonstrates the behavior:</p>

<ul>
<li>
<code>cargo run</code> → hangs (expected)
</li>
<li>
<code>cargo run -- --fixed</code> → completes</li>
</ul>

<p><a href="https://github.com/cspinetta/tokio-block-on-deadlock" rel="noopener noreferrer">tokio-block-on-deadlock</a></p>

<p>The repro intentionally forces Tokio's cooperative scheduler to yield, then attempts another uncontended read lock.</p>

<h2>
  
  
  The mental model mismatch
</h2>

<p>A common assumption is:</p>

<blockquote>
<p>If a lock is uncontended, acquiring it can't block.</p>
</blockquote>

<p>That's true for synchronous locks, but async is different. In async Rust, <code>.await</code> means:</p>

<ul>
<li>Poll the future</li>
<li>If it can't make progress <em>right now</em>, return <code>Poll::Pending</code>
</li>
<li>The executor will poll it again later</li>
</ul>

<p>Crucially:</p>

<blockquote>
<p><code>Pending</code> does not always mean "the resource isn't ready".</p>
</blockquote>

<p>Sometimes it means "yield for fairness".</p>

<h2>
  
  
  Tokio's cooperative scheduling (the key)
</h2>

<p>Tokio uses <strong>cooperative scheduling</strong>. Each task has a limited budget to prevent a single task from starving others.</p>

<p>When that budget is exhausted, Tokio can intentionally return <code>Poll::Pending</code> to force a yield — even if the underlying resource is available.</p>

<p>Under a normal Tokio runtime, that's fine:</p>

<ul>
<li>the task gets re-polled</li>
<li>the budget is reset</li>
<li>the operation completes on the next poll</li>
</ul>

<p>This is usually invisible.</p>

<h2>
  
  
  Where it goes wrong: <code>futures::executor::block_on</code>
</h2>

<p>In my case, I discovered that part of the test setup was driving Tokio async code using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="nn">futures</span><span class="p">::</span><span class="nn">executor</span><span class="p">::</span><span class="nf">block_on</span><span class="p">(</span><span class="k">async</span> <span class="p">{</span> <span class="o">...</span> <span class="p">})</span>
</code></pre>

</div>



<p>This is a common pattern in tests/mocks when you have a synchronous callback but need to call async code.</p>

<p>The problem is: <strong><code>futures::executor::block_on</code> is not the Tokio runtime.</strong></p>

<p>What it does is roughly:</p>

<ul>
<li>Poll the future on the current OS thread</li>
<li>If it returns <code>Pending</code>, park the thread and wait for a wake-up</li>
</ul>

<p>So when Tokio returns <code>Pending</code> for cooperative scheduling reasons, <code>block_on</code> treats it as "I should sleep until someone wakes me".</p>

<p>But in this scenario, there isn't a normal "external wake-up" to wait for. Tokio expected the runtime to re-poll the task.</p>

<p>Result:</p>

<ul>
<li>the OS thread parks</li>
<li>the future never gets re-polled</li>
<li>the test hangs forever</li>
</ul>

<p>And the lock looks "deadlocked" even though it's uncontended.</p>

<h2>
  
  
  Why it happened on the <em>third</em> read
</h2>

<p>Because the cooperative budget is cumulative.</p>

<p>The first awaits didn't exhaust the budget. The third happened to be the first place where Tokio decided "you must yield now", returning <code>Pending</code>.</p>

<p>With small code changes, the hang could move earlier/later or disappear entirely — which is part of what makes this class of bug hard to debug.</p>

<h2>
  
  
  A practical fix
</h2>

<p>The simplest rule:</p>

<blockquote>
<p>If async code uses Tokio primitives, it must be driven by Tokio.</p>
</blockquote>

<p>If you need to run async code from a synchronous context (tests/mocks/callbacks), one safe pattern is to drive it with Tokio's runtime handle on a separate OS thread:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">fn</span> <span class="n">tokio_block_on</span><span class="o">&lt;</span><span class="n">F</span><span class="p">:</span> <span class="nn">std</span><span class="p">::</span><span class="nn">future</span><span class="p">::</span><span class="n">Future</span> <span class="o">+</span> <span class="nb">Send</span><span class="o">&gt;</span><span class="p">(</span><span class="n">fut</span><span class="p">:</span> <span class="n">F</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="nn">F</span><span class="p">::</span><span class="n">Output</span>
<span class="k">where</span>
    <span class="nn">F</span><span class="p">::</span><span class="n">Output</span><span class="p">:</span> <span class="nb">Send</span><span class="p">,</span>
<span class="p">{</span>
    <span class="k">let</span> <span class="n">handle</span> <span class="o">=</span> <span class="nn">tokio</span><span class="p">::</span><span class="nn">runtime</span><span class="p">::</span><span class="nn">Handle</span><span class="p">::</span><span class="nf">current</span><span class="p">();</span>
    <span class="nn">std</span><span class="p">::</span><span class="nn">thread</span><span class="p">::</span><span class="nf">scope</span><span class="p">(|</span><span class="n">s</span><span class="p">|</span> <span class="n">s</span><span class="nf">.spawn</span><span class="p">(||</span> <span class="n">handle</span><span class="nf">.block_on</span><span class="p">(</span><span class="n">fut</span><span class="p">))</span><span class="nf">.join</span><span class="p">()</span><span class="nf">.unwrap</span><span class="p">())</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In the repro repo, <code>cargo run -- --fixed</code> uses this approach and completes successfully.</p>

<h2>
  
  
  Takeaways
</h2>

<ul>
<li>
<code>Poll::Pending</code> doesn't always mean "waiting for a resource"</li>
<li>Tokio relies on its runtime to re-poll tasks; mixing executors can break that assumption</li>
<li>Avoid <code>futures::executor::block_on</code> for Tokio tasks</li>
<li>If you must cross sync/async boundaries, drive futures using Tokio itself (or redesign the boundary)</li>
</ul>

<p>This bug is rare, but it sits in a very common pattern: sync test/mocking glue calling async code. When it happens, it's deeply confusing — and completely silent.</p>

