---
Title: 🚀 Quo.js v0.5.0 🚀 - Event-Driven Architecture and MIT License
Description: 
Author: quojs
Date: 2026-01-08T21:55:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>After months of community feedback and real-world usage, we're excited to announce <strong>Quo.js v0.5.0</strong>, a major release that clarifies our architecture and removes adoption barriers.</p>

<p>Two key changes:</p>

<ol>
<li>
<strong>Event-bus terminology</strong> — API now reflects our event-driven architecture</li>
<li>
<strong>MIT License</strong> — Removed enterprise adoption barriers</li>
</ol>

<p>Let's dive into the technical details and rationale behind these decisions.</p>




<h2>
  
  
  What is Quo.js?
</h2>

<p>Before we discuss what changed, let's clarify what Quo.js <em>is</em>:</p>

<p><strong>Quo.js is an event-driven, async-first state container with atomic subscriptions.</strong></p>

<p>It combines:</p>

<ul>
<li>
<strong>Event-driven architecture</strong> with channel-based routing</li>
<li>
<strong>FIFO queue</strong> for predictable, serialized event processing</li>
<li>
<strong>Atomic path subscriptions</strong> for zero unnecessary re-renders</li>
<li>
<strong>Native async support</strong> via built-in middleware and effects</li>
</ul>

<p>If Redux and Zustand had a baby that grew up studying event sourcing and CQRS, it would be Quo.js.</p>




<h2>
  
  
  Part 1: From Dispatch to Emit — Why Terminology Matters
</h2>

<h3>
  
  
  The Problem with "Dispatch"
</h3>

<p>When we launched Quo.js, we borrowed Redux's terminology: <code>dispatch()</code>, <code>Action</code>, <code>ActionMap</code>. This made sense—Redux is familiar, and the migration path seemed clear.</p>

<p>But as users adopted Quo.js, confusion emerged:</p>

<blockquote>
<p>"Why do I await dispatch? Redux dispatch is synchronous."<br><br>
"What's the difference between dispatch and emit?"<br><br>
"Is this just Redux with extra steps?"</p>
</blockquote>

<p><strong>The truth:</strong> Quo.js isn't a flux dispatcher. It's an <strong>event-driven state container</strong> with a fundamentally different execution model.</p>

<h3>
  
  
  Quo.js's Architecture: Event Bus + FIFO Queue
</h3>

<p>Let's look at what actually happens when you call <code>store.emit()</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// 1. Event is created with unique ID (deduplication)</span>
<span class="kd">const</span> <span class="nx">event</span> <span class="o">=</span> <span class="p">{</span> 
  <span class="na">channel</span><span class="p">:</span> <span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span> 
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">add</span><span class="dl">'</span><span class="p">,</span> 
  <span class="na">payload</span><span class="p">:</span> <span class="nx">todo</span><span class="p">,</span>
  <span class="na">id</span><span class="p">:</span> <span class="nc">Symbol</span><span class="p">()</span> 
<span class="p">};</span>

<span class="c1">// 2. Event is enqueued in FIFO queue</span>
<span class="k">this</span><span class="p">.</span><span class="nx">eventQueue</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">event</span><span class="p">);</span>

<span class="c1">// 3. If queue is processing, return (backpressure)</span>
<span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">isProcessingQueue</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

<span class="c1">// 4. Drain queue sequentially</span>
<span class="k">while </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">eventQueue</span><span class="p">.</span><span class="nx">length</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">event</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">eventQueue</span><span class="p">.</span><span class="nf">shift</span><span class="p">();</span>

  <span class="c1">// 5. Run async middleware pipeline</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">mw</span> <span class="k">of</span> <span class="k">this</span><span class="p">.</span><span class="nx">middleware</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">ok</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">mw</span><span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">event</span><span class="p">,</span> <span class="nx">emit</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">ok</span><span class="p">)</span> <span class="k">break</span><span class="p">;</span> <span class="c1">// Middleware can cancel</span>
  <span class="p">}</span>

  <span class="c1">// 6. Apply reducers (synchronous)</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">reducerBus</span><span class="p">.</span><span class="nf">emit</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">channel</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="kd">type</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">payload</span><span class="p">);</span>

  <span class="c1">// 7. Run async effects</span>
  <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">notifyEffects</span><span class="p">(</span><span class="nx">event</span><span class="p">);</span>

  <span class="c1">// 8. Notify subscribers (if state changed)</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">stateChanged</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">listeners</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">l</span> <span class="o">=&gt;</span> <span class="nf">l</span><span class="p">());</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is <strong>not dispatch</strong>. This is <strong>event emission</strong> through a queue with:</p>

<ul>
<li>Async pipeline</li>
<li>Event deduplication</li>
<li>Serialized processing</li>
<li>Built-in backpressure</li>
</ul>

<p>The word "dispatch" implies synchronous, immediate execution. Quo.js does neither.</p>

<h3>
  
  
  Enter: Event-Bus Terminology
</h3>

<p>In v0.5.0, we've adopted terminology that reflects reality:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Old (v0.4.x)</th>
<th>New (v0.5.0)</th>
<th>Reason</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>dispatch()</code></td>
<td><code>emit()</code></td>
<td>Events are emitted, not dispatched</td>
</tr>
<tr>
<td><code>Action</code></td>
<td><code>Event</code></td>
<td>We emit events, not actions</td>
</tr>
<tr>
<td><code>ActionMap</code></td>
<td><code>EventMap</code></td>
<td>Maps channels to event types</td>
</tr>
<tr>
<td><code>action.event</code></td>
<td><code>event.type</code></td>
<td>Clearer: "the event's type"</td>
</tr>
</tbody>
</table></div>

<p><strong>Code example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// BEFORE (v0.4.x) ❌</span>
<span class="nx">store</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="dl">'</span><span class="s1">analytics</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">track</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">page</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/home</span><span class="dl">'</span> <span class="p">});</span>

<span class="kd">const</span> <span class="nx">reducer</span> <span class="o">=</span> <span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">action</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">action</span><span class="p">.</span><span class="nx">event</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">track</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// ...</span>
  <span class="p">}</span>
<span class="p">};</span>

<span class="c1">// AFTER (v0.5.0) ✅</span>
<span class="nx">store</span><span class="p">.</span><span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">analytics</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">track</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">page</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/home</span><span class="dl">'</span> <span class="p">});</span>

<span class="kd">const</span> <span class="nx">reducer</span> <span class="o">=</span> <span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">track</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// ...</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>The new terminology instantly communicates:</p>

<ul>
<li>
<strong>Events</strong> flow through the system (event sourcing)</li>
<li>
<strong>Channels</strong> organize events (pub/sub model)</li>
<li>
<strong>Emit</strong> suggests async, queued processing</li>
</ul>

<h3>
  
  
  Migration Path: Deprecation, Not Deletion
</h3>

<p>We understand changing terminology is disruptive. That's why we've taken a <strong>gradual deprecation approach</strong>:</p>

<p>✅ <strong>Old APIs still work</strong> (with dev warnings)<br><br>
✅ <strong>Type aliases provided</strong> (<code>Action</code> = <code>Event</code>)<br><br>
✅ <strong>Clear timeline</strong> (removal in v1.0.0)<br><br>
✅ <strong>Comprehensive migration guide</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// This still works in v0.5.0 (with warning):</span>
<span class="nx">store</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">add</span><span class="dl">'</span><span class="p">,</span> <span class="nx">todo</span><span class="p">);</span>

<span class="c1">// Console (development only):</span>
<span class="c1">// [@quojs/core] `store.dispatch()` is deprecated. </span>
<span class="c1">// Use `store.emit()` instead. Will be removed in v1.0.0.</span>
</code></pre>

</div>



<p><strong>Only one true breaking change:</strong> <code>action.event</code> → <code>event.type</code></p>

<p>We couldn't alias this one because it would break TypeScript inference. But the fix is straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Find and replace in your reducers:</span>
<span class="c1">// action.event → event.type</span>
</code></pre>

</div>



<h3>
  
  
  Why This Matters
</h3>

<p>Terminology isn't just semantics—it shapes mental models.</p>

<p>When you see <code>store.emit('analytics', 'track', data)</code>, you immediately understand:</p>

<ul>
<li>This is <strong>asynchronous</strong> (like emitting an event)</li>
<li>It's <strong>fire-and-forget</strong> (queue handles it)</li>
<li>It's <strong>event-driven</strong> (not command-driven like Redux)</li>
</ul>

<p>This alignment between naming and behavior reduces cognitive load and makes Quo.js easier to understand, especially for developers coming from event-driven systems (Kafka, RabbitMQ, EventEmitter).</p>




<h2>
  
  
  Part 2: From MPL-2.0 to MIT — Removing Adoption Barriers
</h2>

<h3>
  
  
  The License Problem We Didn't See Coming
</h3>

<p>When we launched Quo.js, we chose <strong>Mozilla Public License 2.0 (MPL-2.0)</strong> for good reasons:</p>

<ul>
<li>File-level copyleft (modifications stay open)</li>
<li>Patent protection clauses</li>
<li>Permissive for most use cases</li>
</ul>

<p>We thought: "MPL-2.0 is permissive! Libraries like Firefox use it!"</p>

<p><strong>Reality check:</strong> Enterprise legal departments don't care about nuance.</p>

<p>Over the past months, we heard from multiple teams:</p>

<blockquote>
<p>"Our legal team won't approve MPL-2.0. Can you switch to MIT?"<br><br>
"We can't use Quo.js until the license changes."<br><br>
"Adding to our project requires a legal review—3-6 weeks minimum."</p>
</blockquote>

<p>The pattern was clear: <strong>MPL-2.0 was silently killing adoption</strong>.</p>

<h3>
  
  
  Why Enterprises Fear MPL-2.0
</h3>

<p>It's not that MPL-2.0 is restrictive—it's <strong>unfamiliar</strong>:</p>

<ol>
<li>
<strong>Whitelist policies</strong>: Most companies have "MIT/Apache/BSD-only" policies</li>
<li>
<strong>Legal bandwidth</strong>: Non-standard licenses require review (slow, expensive)</li>
<li>
<strong>Risk aversion</strong>: Lawyers say "no" to unknowns by default</li>
<li>
<strong>Contributor concerns</strong>: Open-source contributors prefer MIT for portfolios</li>
</ol>

<p>Meanwhile, every major JavaScript library uses MIT:</p>

<ul>
<li>React, Vue, Angular → MIT</li>
<li>Redux, Zustand, Jotai, MobX → MIT</li>
<li>Express, Fastify, Next.js → MIT</li>
</ul>

<p>We were the outlier.</p>

<h3>
  
  
  The Decision: MIT
</h3>

<p>After consulting with the community, contributors, and users, we made the switch:</p>

<p><strong>Quo.js v0.5.0 is MIT licensed.</strong></p>

<h4>
  
  
  What We Gain
</h4>

<p>✅ <strong>Instant enterprise approval</strong> — No legal review needed<br><br>
✅ <strong>Community trust</strong> — Industry-standard terms everyone knows<br><br>
✅ <strong>Contributor friendliness</strong> — MIT is the default for OSS portfolios<br><br>
✅ <strong>Ecosystem alignment</strong> — Matches React, Redux, and every major library  </p>
<h4>
  
  
  What We Don't Lose
</h4>

<p>The license is <strong>more</strong> permissive, not less:</p>

<p>✅ <strong>Open-source commitment</strong> — Still 100% open<br><br>
✅ <strong>Copyright ownership</strong> — Contributors still own their work<br><br>
✅ <strong>Community governance</strong> — Unchanged<br><br>
✅ <strong>Commercial use</strong> — Fully allowed (as before)  </p>

<p>The only difference: Derivatives don't <em>have</em> to be open-source. But in practice, most Quo.js usage is integration (not forking), so this rarely matters.</p>
<h3>
  
  
  A Note on Pragmatism
</h3>

<p>Some might see this as "selling out" to enterprise demands. We see it differently:</p>

<p><strong>Open-source impact = adoption × value</strong></p>

<p>If MPL-2.0 blocks 50% of potential users, we're cutting our impact in half. MIT maximizes reach without compromising values.</p>

<p>We're not a protocol (where copyleft matters). We're a <strong>library</strong>. Our success comes from <em>widespread use</em>, not <em>forced contributions</em>.</p>

<p>MIT is the pragmatic choice for maximum impact.</p>


<h2>
  
  
  Part 3: What Didn't Change (And Why That's Good)
</h2>

<p>Amidst these changes, Quo.js's core value proposition remains <strong>exactly the same</strong>:</p>
<h3>
  
  
  1. Atomic Subscriptions Still Eliminate Re-renders
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// Only re-renders when THIS specific todo's title changes</span>
<span class="kd">const</span> <span class="nx">title</span> <span class="o">=</span> <span class="nf">useAtomicProp</span><span class="p">({</span> 
  <span class="na">reducer</span><span class="p">:</span> <span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span> 
  <span class="na">property</span><span class="p">:</span> <span class="dl">'</span><span class="s1">items.0.title</span><span class="dl">'</span> 
<span class="p">});</span>

<span class="c1">// Wildcard patterns for collections</span>
<span class="kd">const</span> <span class="nx">allTitles</span> <span class="o">=</span> <span class="nf">useAtomicProp</span><span class="p">({</span>
  <span class="na">reducer</span><span class="p">:</span> <span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">property</span><span class="p">:</span> <span class="dl">'</span><span class="s1">items.*.title</span><span class="dl">'</span>
<span class="p">});</span>
</code></pre>

</div>


<p>This is <strong>still</strong> the killer feature. No other library offers this by default.</p>
<h3>
  
  
  2. Async Pipeline Still Built-in
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Async middleware (no thunks needed)</span>
<span class="kd">const</span> <span class="nx">authMiddleware</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">event</span><span class="p">,</span> <span class="nx">emit</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">login</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">authenticateUser</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">payload</span><span class="p">);</span>
    <span class="k">await</span> <span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">auth</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">loginSuccess</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="nx">token</span> <span class="p">});</span>
    <span class="k">return</span> <span class="kc">false</span><span class="p">;</span> <span class="c1">// Cancel original event</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
<span class="p">};</span>

<span class="c1">// Async effects (no sagas needed)</span>
<span class="kd">const</span> <span class="nx">analyticsEffect</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">,</span> <span class="nx">getState</span><span class="p">,</span> <span class="nx">emit</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nf">trackEvent</span><span class="p">(</span><span class="nx">event</span><span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>


<p>Async is <strong>first-class</strong>, not an afterthought.</p>
<h3>
  
  
  3. TypeScript Inference Still Excellent
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">AppEM</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">todos</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">add</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">title</span><span class="p">:</span> <span class="kr">string</span> <span class="p">};</span>
    <span class="nl">toggle</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span> <span class="p">};</span>
  <span class="p">};</span>
<span class="p">};</span>

<span class="c1">// Full autocomplete:</span>
<span class="k">await</span> <span class="nx">store</span><span class="p">.</span><span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">add</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">title</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Buy milk</span><span class="dl">'</span>
<span class="p">});</span>

<span class="c1">// TypeScript catches errors:</span>
<span class="k">await</span> <span class="nx">store</span><span class="p">.</span><span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">add</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span> <span class="p">});</span> 
<span class="c1">// ❌ Error: id must be string</span>
</code></pre>

</div>


<p>Types are <strong>inferred</strong>, not manually specified.</p>
<h3>
  
  
  4. Universal Runtime Still Supported
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Works everywhere:</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createStore</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@quojs/core</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Browser, Node.js, Deno, Bun</span>
<span class="kd">const</span> <span class="nx">store</span> <span class="o">=</span> <span class="nf">createStore</span><span class="p">({</span> <span class="cm">/* ... */</span> <span class="p">});</span>
</code></pre>

</div>


<p>Zero DOM dependencies. Use it in servers, CLI tools, microservices.</p>


<h2>
  
  
  Performance: The Numbers Don't Lie
</h2>

<p>We keep mentioning "zero unnecessary re-renders." Let's prove it.</p>
<h3>
  
  
  Benchmark: Todo App (50 items)
</h3>

<p>Same app. Same interactions. Different libraries.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Library</th>
<th>Re-renders (Toggling a TODO item)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Redux Toolkit</strong></td>
<td>142</td>
</tr>
<tr>
<td><strong>Quo.js</strong></td>
<td><strong>1</strong></td>
</tr>
</tbody>
</table></div>

<p><a href="https://github.com/quojs/quojs/blob/main/examples/v0/quojs-in-react/redux-quojs-profiler.md#quojs-vs-redux-toolkit-profiler-comparison" rel="noopener noreferrer">See it for yourself!</a></p>
<h3>
  
  
  Why?
</h3>

<p>Redux/Zustand re-render every component subscribed to <code>state.todos</code>. Even if the change was to <code>todos[49]</code>, components displaying <code>todos[0]</code> re-render.</p>

<p>Quo.js <strong>only</strong> re-renders components subscribed to the changed path:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// This component ONLY re-renders when items[0].title changes</span>
<span class="kd">function</span> <span class="nf">TodoItem</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">title</span> <span class="o">=</span> <span class="nf">useAtomicProp</span><span class="p">({</span>
    <span class="na">reducer</span><span class="p">:</span> <span class="dl">'</span><span class="s1">todos</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">property</span><span class="p">:</span> <span class="dl">'</span><span class="s1">items.0.title</span><span class="dl">'</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">title</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;;</span>
<span class="p">}</span>

<span class="c1">// Adding todos[50]? This component doesn't care.</span>
</code></pre>

</div>



<p>Atomic subscriptions = free performance.</p>




<h2>
  
  
  Migration Guide: Step-by-Step
</h2>

<p>Migrating from v0.4.x to v0.5.0 takes <strong>~15 minutes</strong> for most codebases.</p>

<h3>
  
  
  Step 1: Update Packages
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @quojs/core@0.5.0 @quojs/react@0.5.0
</code></pre>

</div>



<h3>
  
  
  Step 2: Search and Replace
</h3>

<p>Use your IDE's find-and-replace:</p>

<ol>
<li>
<p><strong>dispatch → emit</strong></p>

<ul>
<li>
<code>store.dispatch(</code> → <code>store.emit(</code>
</li>
<li>
<code>useDispatch()</code> → <code>useEmit()</code>
</li>
</ul>
</li>
<li>
<p><strong>action → event</strong></p>

<ul>
<li>
<code>action.event</code> → <code>event.type</code>
</li>
<li>
<code>: Action</code> → <code>: Event</code> (in type annotations)</li>
</ul>
</li>
<li>
<p><strong>ActionMap → EventMap</strong></p>

<ul>
<li>
<code>type AppAM</code> → <code>type AppEM</code>
</li>
<li>
<code>: ActionMap</code> → <code>: EventMap</code>
</li>
</ul>
</li>
</ol>

<h3>
  
  
  Step 3: Test
</h3>

<p>Run your tests. The only breaking change is <code>action.event</code> → <code>event.type</code>, which TypeScript will catch.</p>

<h3>
  
  
  Step 4: Remove Warnings
</h3>

<p>Check your dev console. You'll see deprecation warnings for any missed renames.</p>

<p>Fix them now—they'll be errors in v1.0.0.</p>

<h2>
  
  
  Try It Today
</h2>

<p><a href="https://github.com/quojs/quojs/blob/main/docs/en/QUICK_START_GUIDE.md" rel="noopener noreferrer">Quick Start Guide</a>.</p>

<p><strong>Resources:</strong></p>

<ul>
<li>🌐 <a href="https://quojs.dev" rel="noopener noreferrer">Quo.js Website</a>
</li>
<li>📚 <a href="https://github.com/quojs/quojs/blob/main/packages/core/docs/README.md#quojscore" rel="noopener noreferrer">Core Docs</a>
</li>
<li>📚 <a href="https://github.com/quojs/quojs/blob/main/packages/react/docs/README.md#quojsreact" rel="noopener noreferrer">React Docs</a>
</li>
<li>🔧 <a href="https://github.com/quojs/quojs/blob/main/README.md#live-examples" rel="noopener noreferrer">Examples</a>
</li>
<li>📊 <a href="https://github.com/quojs/quojs/blob/main/docs/en/design/state-management-library-comparison.md" rel="noopener noreferrer">Library Comparison</a>
</li>
<li>🏗️ <a href="https://github.com/quojs/quojs/blob/main/docs/en/design/event-queue-architecture.md" rel="noopener noreferrer">Architecture Deep-Dive</a>
</li>
<li>💬 <a href="https://github.com/quojs/quojs/discussions" rel="noopener noreferrer">GitHub Discussions</a>
</li>
</ul>




<h2>
  
  
  Community Feedback Welcome
</h2>

<p>We built Quo.js because we needed it—but we're shaping it based on <em>your</em> feedback.</p>

<p>What do you think of v0.5.0?</p>

<ul>
<li>Does the event-bus terminology make sense?</li>
<li>Relieved about MIT?</li>
<li>What features do you need next?</li>
</ul>

<p>Drop a comment below or join the discussion on <a href="https://github.com/quojs/quojs/discussions" rel="noopener noreferrer">GitHub</a>.</p>




<h2>
  
  
  Thank You
</h2>

<p>To everyone who:</p>

<ul>
<li>Provided feedback on terminology</li>
<li>Shared concerns about MPL-2.0</li>
<li>Tested RC builds</li>
<li>Filed issues and PRs</li>
<li>Used Quo.js in production</li>
</ul>

<p>Made in 🇲🇽, for the world.</p>

<p>— The Quo.js Team</p>




<p><em>P.S. If Quo.js helps you, give us a star on <a href="https://github.com/quojs/quojs" rel="noopener noreferrer">GitHub</a>! ⭐</em></p>

