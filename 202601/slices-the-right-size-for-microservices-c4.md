---
Title: Slices: The Right Size for Microservices
Description: 
Author: Sergiy Yevtushenko
Date: 2026-01-18T21:49:19.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Slices: The Right Size for Microservices
</h1>

<h2>
  
  
  The Granularity Trap
</h2>

<p>Every team that adopts microservices eventually hits the same wall: how big should a service be?</p>

<p>Go too small and you drown in network calls, distributed transactions, and deployment complexity. Your simple "get user profile" operation now involves five services, three of which are just proxies for database tables. Latency compounds. Debugging becomes archaeology.</p>

<p>Go too large and you're back to the monolith. Different teams step on each other. Deployments require coordination. The "micro" in microservices becomes ironic.</p>

<p>The standard advice—"one service per bounded context" or "services should be independently deployable"—sounds reasonable but provides no actionable guidance. Where does one context end and another begin? What exactly makes something "independently deployable"?</p>

<p>Teams oscillate between extremes, refactoring services that are "too small" into larger ones, then splitting services that grew "too large." The cycle repeats because the fundamental question remains unanswered: what determines the right boundary?</p>

<h2>
  
  
  Boundaries Before Size
</h2>

<p>The problem isn't size. It's boundaries.</p>

<p>A well-defined boundary has specific properties:</p>

<ul>
<li>
<strong>Clear contract</strong>: callers know exactly what they can request and what they'll receive</li>
<li>
<strong>Explicit dependencies</strong>: the component declares what it needs from outside</li>
<li>
<strong>Internal freedom</strong>: implementation details can change without affecting callers</li>
</ul>

<p>Size follows from boundaries, not the other way around. A component is the right size when it fully owns its boundaries—when everything needed to fulfill its contract lives inside, and everything outside is accessed through explicit dependencies.</p>

<p>This is where most microservice designs fail. They draw boundaries based on technical layers (API gateway, business logic, database access) or organizational structure (team ownership). Neither approach produces stable boundaries because neither focuses on the actual contracts between components.</p>

<h2>
  
  
  What Is a Slice?
</h2>

<p>A slice is a deployable unit defined by its contract. You write an interface with a single annotation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Slice</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">OrderService</span> <span class="o">{</span>
    <span class="nc">Promise</span><span class="o">&lt;</span><span class="nc">OrderResponse</span><span class="o">&gt;</span> <span class="nf">createOrder</span><span class="o">(</span><span class="nc">CreateOrderRequest</span> <span class="n">request</span><span class="o">);</span>
    <span class="nc">Promise</span><span class="o">&lt;</span><span class="nc">OrderResponse</span><span class="o">&gt;</span> <span class="nf">getOrder</span><span class="o">(</span><span class="nc">GetOrderRequest</span> <span class="n">request</span><span class="o">);</span>
    <span class="nc">Promise</span><span class="o">&lt;</span><span class="nc">Unit</span><span class="o">&gt;</span> <span class="nf">cancelOrder</span><span class="o">(</span><span class="nc">CancelOrderRequest</span> <span class="n">request</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>That's it. The annotation processor generates everything else—factory methods, dependency wiring, deployment metadata. You define the contract; the tooling handles the infrastructure.</p>

<p>This interface is the boundary:</p>

<ul>
<li>
<strong>Methods define the contract</strong>: each takes a request, returns a promise of response</li>
<li>
<strong>Request/response types are explicit</strong>: no hidden parameters, no ambient context</li>
<li>
<strong>Async by default</strong>: <code>Promise&lt;T&gt;</code> handles both success and failure paths</li>
</ul>

<p>The implementation lives behind this interface. It might be simple or complex. It might call other slices or be completely self-contained. The boundary doesn't care.</p>

<h2>
  
  
  Aether and Forge: Development Made Simple
</h2>

<p>Slices run on Aether, a distributed runtime designed around slice contracts. You don't configure service discovery, serialization, or inter-slice communication—Aether handles it based on what the slice interfaces declare. Every inter-slice call eventually succeeds if the cluster is alive; the runtime manages retries, failover, and recovery transparently.</p>

<p>Forge provides a development environment for testing slices under realistic conditions—load generation, chaos injection, backend simulation. Instead of deploying to staging to see how your slices behave under pressure, you run Forge locally and observe.</p>

<p>The development experience stays simple: write <code>@Slice</code> interfaces, implement them, test with Forge, deploy to Aether. The annotation processor generates all the boilerplate—factories, dependency wiring, routing metadata.</p>

<h2>
  
  
  Dependencies That Don't Lie
</h2>

<p>Traditional service architectures bury dependencies in configuration files, environment variables, or runtime discovery. You find out what a service needs by reading its code, tracing its network calls, or waiting for it to fail in production.</p>

<p>Slices declare dependencies in the interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Slice</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">OrderService</span> <span class="o">{</span>
    <span class="nc">Promise</span><span class="o">&lt;</span><span class="nc">OrderResponse</span><span class="o">&gt;</span> <span class="nf">createOrder</span><span class="o">(</span><span class="nc">CreateOrderRequest</span> <span class="n">request</span><span class="o">);</span>
    <span class="c1">// Other methods...</span>

    <span class="c1">// Factory method declares dependencies explicitly</span>
    <span class="kd">static</span> <span class="nc">OrderService</span> <span class="nf">orderService</span><span class="o">(</span><span class="nc">InventoryService</span> <span class="n">inventory</span><span class="o">,</span> <span class="nc">PaymentService</span> <span class="n">payment</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">OrderServiceFactory</span><span class="o">.</span><span class="na">orderService</span><span class="o">(</span><span class="nc">Aspect</span><span class="o">.</span><span class="na">identity</span><span class="o">(),</span> <span class="n">inventory</span><span class="o">,</span> <span class="n">payment</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>The annotation processor generates the factory that wires everything:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">final</span> <span class="kd">class</span> <span class="nc">OrderServiceFactory</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="nf">OrderServiceFactory</span><span class="o">()</span> <span class="o">{}</span>

    <span class="kd">public</span> <span class="kd">static</span> <span class="nc">OrderService</span> <span class="nf">orderService</span><span class="o">(</span>
            <span class="nc">Aspect</span><span class="o">&lt;</span><span class="nc">OrderService</span><span class="o">&gt;</span> <span class="n">aspect</span><span class="o">,</span>
            <span class="nc">InventoryService</span> <span class="n">inventory</span><span class="o">,</span>
            <span class="nc">PaymentService</span> <span class="n">payment</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="n">aspect</span><span class="o">.</span><span class="na">apply</span><span class="o">(</span><span class="k">new</span> <span class="nc">OrderServiceImpl</span><span class="o">(</span><span class="n">inventory</span><span class="o">,</span> <span class="n">payment</span><span class="o">));</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>The factory method signature declares dependencies. No service locators, no runtime discovery, no configuration files that might or might not match reality. Dependencies are visible at compile time and verified before deployment.</p>

<p>This explicitness matters. You can trace the dependency graph by reading code. You can test with substitutes by passing different implementations. Forge validates the entire graph before starting anything.</p>

<h2>
  
  
  Same Code, Different Environments
</h2>

<p>The same slices run unchanged across three runtime modes:</p>

<ul>
<li>
<strong>Ember</strong>: Single-process runtime with multiple cluster nodes. Fast startup, simple debugging. Perfect for local development.</li>
<li>
<strong>Forge</strong>: Ember plus load generation and chaos injection. Test how slices behave under pressure without deploying anywhere.</li>
<li>
<strong>Aether</strong>: Full distributed cluster. Production deployment with all the resilience guarantees.</li>
</ul>

<p>Your code doesn't know which mode it's running in. Slice interfaces, implementations, and dependencies stay identical. The runtime handles the difference—whether inter-slice calls are in-process or cross-network is transparent.</p>

<p>This changes the development workflow. You write and debug in Ember. You stress-test in Forge. You deploy to Aether. At no point do you modify slice code to accommodate the environment.</p>

<h2>
  
  
  Right-Sized by Definition
</h2>

<p>With boundaries explicit and deployment flexible, the "right size" question dissolves.</p>

<p>A slice is the right size when:</p>

<ol>
<li>Its interface captures a coherent set of operations</li>
<li>Its dependencies accurately reflect what it needs</li>
<li>Its implementation can fulfill its contract</li>
</ol>

<p>There's no minimum or maximum. An authentication slice might have two methods. An order processing slice might have twenty. Size follows from the domain, not from arbitrary rules about lines of code or team structure.</p>

<p>More importantly, getting it wrong is recoverable. Split a slice that grew too complex? The boundary changes, but callers just see a new interface. Merge slices that were artificially separated? Same story. Refactoring slices is refactoring code, not rewriting infrastructure.</p>

<h2>
  
  
  The JBCT Connection
</h2>

<p>Slices are where JBCT patterns live.</p>

<p>Each slice method is a data transformation pipeline:</p>

<ul>
<li>Parse input (validated request types)</li>
<li>Gather data (dependencies, other slices)</li>
<li>Process (business logic)</li>
<li>Respond (typed response)</li>
</ul>

<p>The six patterns—Leaf, Sequencer, Fork-Join, Condition, Iteration, Aspects—compose within and across slice methods. A slice is simply the deployment boundary around a set of related transformations.</p>

<p>This is why the combination works. JBCT gives you consistent structure within slices. Slices give you consistent boundaries between them. Together, they eliminate the two sources of architectural entropy: inconsistent implementation patterns and unclear service boundaries.</p>

<h2>
  
  
  Conclusion
</h2>

<p>The microservices granularity problem persists because it asks the wrong question. "How big should a service be?" has no good answer. "What are the contracts between components?" has a precise one.</p>

<p>Slices shift focus from size to boundaries. Define the interface. Declare dependencies explicitly. Let deployment topology adapt to operational needs rather than dictating code structure.</p>

<p>The result: boundaries that are clear by construction, dependencies that are visible by design, and deployment flexibility that doesn't require rewriting code.</p>




<p><em>Part of <a href="//../README.md">Java Backend Coding Technology</a> - a methodology for writing predictable, testable backend code.</em></p>

