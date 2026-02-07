---
Title: Why Interface + Factory? The Java Pattern That Makes Everything Replaceable
Description: 
Author: Sergiy Yevtushenko
Date: 2026-02-07T21:39:17.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Why Interface + Factory? The Java Pattern That Makes Everything Replaceable
</h1>

<h2>
  
  
  The Pattern
</h2>

<p>Every component — use case, processing step, adapter — is defined as an interface with a static factory method:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">interface</span> <span class="nc">ProcessOrder</span> <span class="o">{</span>
    <span class="n">record</span> <span class="nf">Request</span><span class="o">(</span><span class="nc">String</span> <span class="n">orderId</span><span class="o">,</span> <span class="nc">String</span> <span class="n">paymentToken</span><span class="o">)</span> <span class="o">{}</span>
    <span class="n">record</span> <span class="nf">Response</span><span class="o">(</span><span class="nc">OrderConfirmation</span> <span class="n">confirmation</span><span class="o">)</span> <span class="o">{}</span>

    <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Response</span><span class="o">&gt;</span> <span class="nf">execute</span><span class="o">(</span><span class="nc">Request</span> <span class="n">request</span><span class="o">);</span>

    <span class="kd">interface</span> <span class="nc">ValidateInput</span> <span class="o">{</span>
        <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">ValidRequest</span><span class="o">&gt;</span> <span class="nf">apply</span><span class="o">(</span><span class="nc">Request</span> <span class="n">raw</span><span class="o">);</span>
    <span class="o">}</span>
    <span class="kd">interface</span> <span class="nc">ReserveInventory</span> <span class="o">{</span>
        <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Reservation</span><span class="o">&gt;</span> <span class="nf">apply</span><span class="o">(</span><span class="nc">ValidRequest</span> <span class="n">req</span><span class="o">);</span>
    <span class="o">}</span>
    <span class="kd">interface</span> <span class="nc">ProcessPayment</span> <span class="o">{</span>
        <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Payment</span><span class="o">&gt;</span> <span class="nf">apply</span><span class="o">(</span><span class="nc">Reservation</span> <span class="n">reservation</span><span class="o">);</span>
    <span class="o">}</span>
    <span class="kd">interface</span> <span class="nc">ConfirmOrder</span> <span class="o">{</span>
        <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Response</span><span class="o">&gt;</span> <span class="nf">apply</span><span class="o">(</span><span class="nc">Payment</span> <span class="n">payment</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="kd">static</span> <span class="nc">ProcessOrder</span> <span class="nf">processOrder</span><span class="o">(</span><span class="nc">ValidateInput</span> <span class="n">validate</span><span class="o">,</span>
                                     <span class="nc">ReserveInventory</span> <span class="n">reserve</span><span class="o">,</span>
                                     <span class="nc">ProcessPayment</span> <span class="n">processPayment</span><span class="o">,</span>
                                     <span class="nc">ConfirmOrder</span> <span class="n">confirm</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="n">request</span> <span class="o">-&gt;</span> <span class="n">validate</span><span class="o">.</span><span class="na">apply</span><span class="o">(</span><span class="n">request</span><span class="o">)</span>
                                  <span class="o">.</span><span class="na">flatMap</span><span class="o">(</span><span class="nl">reserve:</span><span class="o">:</span><span class="n">apply</span><span class="o">)</span>
                                  <span class="o">.</span><span class="na">flatMap</span><span class="o">(</span><span class="nl">processPayment:</span><span class="o">:</span><span class="n">apply</span><span class="o">)</span>
                                  <span class="o">.</span><span class="na">flatMap</span><span class="o">(</span><span class="nl">confirm:</span><span class="o">:</span><span class="n">apply</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Four steps. Each is a single-method interface. The factory method accepts all dependencies as parameters and returns a lambda implementing the use case. The body reads exactly like the business process: validate, reserve, process payment, confirm.</p>

<p>This isn't arbitrary convention. There are three specific reasons this structure exists.</p>

<h2>
  
  
  Reason 1: Substitutability Without Magic
</h2>

<p>Anyone can implement the interface. No framework. No inheritance hierarchy. No annotations.</p>

<p>Testing becomes trivial:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Test</span>
<span class="kt">void</span> <span class="nf">order_fails_when_inventory_insufficient</span><span class="o">()</span> <span class="o">{</span>
    <span class="kt">var</span> <span class="n">useCase</span> <span class="o">=</span> <span class="nc">ProcessOrder</span><span class="o">.</span><span class="na">processOrder</span><span class="o">(</span>
        <span class="n">request</span> <span class="o">-&gt;</span> <span class="nc">Result</span><span class="o">.</span><span class="na">success</span><span class="o">(</span><span class="k">new</span> <span class="nc">ValidRequest</span><span class="o">(</span><span class="n">request</span><span class="o">)),</span>  <span class="c1">// always valid</span>
        <span class="n">req</span> <span class="o">-&gt;</span> <span class="no">INSUFFICIENT_INVENTORY</span><span class="o">.</span><span class="na">result</span><span class="o">(),</span>                <span class="c1">// always fails</span>
        <span class="n">reservation</span> <span class="o">-&gt;</span> <span class="o">{</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">AssertionError</span><span class="o">(</span><span class="s">"unreachable"</span><span class="o">);</span> <span class="o">},</span>
        <span class="n">payment</span> <span class="o">-&gt;</span> <span class="o">{</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">AssertionError</span><span class="o">(</span><span class="s">"unreachable"</span><span class="o">);</span> <span class="o">}</span>
    <span class="o">);</span>

    <span class="n">useCase</span><span class="o">.</span><span class="na">execute</span><span class="o">(</span><span class="k">new</span> <span class="nc">Request</span><span class="o">(</span><span class="s">"order-1"</span><span class="o">,</span> <span class="s">"tok_123"</span><span class="o">))</span>
           <span class="o">.</span><span class="na">onSuccess</span><span class="o">(</span><span class="nl">Assertions:</span><span class="o">:</span><span class="n">fail</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>No mocking framework. No <code>@Mock</code> annotations. No <code>when().thenReturn()</code> chains. The test constructs the exact scenario it needs with plain lambdas.</p>

<p>Stubbing incomplete implementations during development is equally straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// Payment gateway isn't ready yet? Stub it.</span>
<span class="kt">var</span> <span class="n">useCase</span> <span class="o">=</span> <span class="nc">ProcessOrder</span><span class="o">.</span><span class="na">processOrder</span><span class="o">(</span>
    <span class="n">realValidator</span><span class="o">,</span>
    <span class="n">realInventoryService</span><span class="o">,</span>
    <span class="n">reservation</span> <span class="o">-&gt;</span> <span class="nc">Result</span><span class="o">.</span><span class="na">success</span><span class="o">(</span><span class="k">new</span> <span class="nc">Payment</span><span class="o">(</span><span class="s">"stub-"</span> <span class="o">+</span> <span class="n">reservation</span><span class="o">.</span><span class="na">id</span><span class="o">(),</span> <span class="nc">Money</span><span class="o">.</span><span class="na">ZERO</span><span class="o">)),</span>
    <span class="n">realConfirmation</span>
<span class="o">);</span>
</code></pre>

</div>



<p>The team working on inventory doesn't need to wait for the payment team. Each step is independently implementable.</p>

<h2>
  
  
  Reason 2: Implementation Isolation
</h2>

<p>Each implementation is self-contained. No shared base classes. No abstract methods to override. No coupling between implementations whatsoever.</p>

<p>Compare with the typical abstract class approach:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// The abstract class trap</span>
<span class="kd">public</span> <span class="kd">abstract</span> <span class="kd">class</span> <span class="nc">AbstractOrderProcessor</span> <span class="o">{</span>
    <span class="kd">protected</span> <span class="kd">final</span> <span class="nc">Logger</span> <span class="n">log</span> <span class="o">=</span> <span class="nc">LoggerFactory</span><span class="o">.</span><span class="na">getLogger</span><span class="o">(</span><span class="n">getClass</span><span class="o">());</span>

    <span class="kd">public</span> <span class="kd">final</span> <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Response</span><span class="o">&gt;</span> <span class="nf">execute</span><span class="o">(</span><span class="nc">Request</span> <span class="n">request</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"Processing order: {}"</span><span class="o">,</span> <span class="n">request</span><span class="o">.</span><span class="na">orderId</span><span class="o">());</span>
        <span class="kt">var</span> <span class="n">result</span> <span class="o">=</span> <span class="n">doExecute</span><span class="o">(</span><span class="n">request</span><span class="o">);</span>
        <span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"Order result: {}"</span><span class="o">,</span> <span class="n">result</span><span class="o">);</span>
        <span class="k">return</span> <span class="n">result</span><span class="o">;</span>
    <span class="o">}</span>

    <span class="kd">protected</span> <span class="kd">abstract</span> <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Response</span><span class="o">&gt;</span> <span class="nf">doExecute</span><span class="o">(</span><span class="nc">Request</span> <span class="n">request</span><span class="o">);</span>
    <span class="kd">protected</span> <span class="kd">abstract</span> <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">ValidRequest</span><span class="o">&gt;</span> <span class="nf">validate</span><span class="o">(</span><span class="nc">Request</span> <span class="n">request</span><span class="o">);</span>

    <span class="c1">// "Shared utility" that every subclass now depends on</span>
    <span class="kd">protected</span> <span class="nc">Result</span><span class="o">&lt;</span><span class="nc">Money</span><span class="o">&gt;</span> <span class="nf">calculateTotal</span><span class="o">(</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">LineItem</span><span class="o">&gt;</span> <span class="n">items</span><span class="o">)</span> <span class="o">{</span>
        <span class="c1">// 47 lines of logic that one subclass needed once</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Now every implementation is coupled to the base class. Change <code>calculateTotal</code> and you need to understand every subclass. Add logging to <code>execute</code> and every implementation gets it whether appropriate or not. The base class becomes a gravity well — accumulating shared code that creates invisible dependencies between implementations that should have nothing in common.</p>

<p>With interface + factory, there is no shared implementation code. Period. Each intersection between implementations is unnecessary coupling with corresponding maintenance overhead — up to needing deep understanding of two projects instead of one, with zero benefit.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// Implementation A: uses database</span>
<span class="kd">static</span> <span class="nc">ProcessPayment</span> <span class="nf">databasePayment</span><span class="o">(</span><span class="nc">PaymentRepository</span> <span class="n">repo</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">return</span> <span class="n">reservation</span> <span class="o">-&gt;</span> <span class="n">repo</span><span class="o">.</span><span class="na">charge</span><span class="o">(</span><span class="n">reservation</span><span class="o">.</span><span class="na">paymentToken</span><span class="o">(),</span> <span class="n">reservation</span><span class="o">.</span><span class="na">total</span><span class="o">())</span>
                              <span class="o">.</span><span class="na">map</span><span class="o">(</span><span class="nl">Payment:</span><span class="o">:</span><span class="n">fromRecord</span><span class="o">);</span>
<span class="o">}</span>

<span class="c1">// Implementation B: uses external API</span>
<span class="kd">static</span> <span class="nc">ProcessPayment</span> <span class="nf">stripePayment</span><span class="o">(</span><span class="nc">StripeClient</span> <span class="n">client</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">return</span> <span class="n">reservation</span> <span class="o">-&gt;</span> <span class="n">client</span><span class="o">.</span><span class="na">createCharge</span><span class="o">(</span><span class="n">reservation</span><span class="o">.</span><span class="na">total</span><span class="o">(),</span> <span class="n">reservation</span><span class="o">.</span><span class="na">paymentToken</span><span class="o">())</span>
                                <span class="o">.</span><span class="na">map</span><span class="o">(</span><span class="nl">Payment:</span><span class="o">:</span><span class="n">fromStripe</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>These implementations don't know about each other. They don't share code. They don't share a base class. They share a contract — the interface — and nothing else.</p>

<h2>
  
  
  Reason 3: Disposable Implementation
</h2>

<p>Here's the subtle one. The factory method returns a lambda or local record. It can't be referenced externally by class name.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">static</span> <span class="nc">ProcessOrder</span> <span class="nf">processOrder</span><span class="o">(</span><span class="nc">ValidateInput</span> <span class="n">validate</span><span class="o">,</span>
                                 <span class="nc">ReserveInventory</span> <span class="n">reserve</span><span class="o">,</span>
                                 <span class="nc">ProcessPayment</span> <span class="n">processPayment</span><span class="o">,</span>
                                 <span class="nc">ConfirmOrder</span> <span class="n">confirm</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">return</span> <span class="n">request</span> <span class="o">-&gt;</span> <span class="n">validate</span><span class="o">.</span><span class="na">apply</span><span class="o">(</span><span class="n">request</span><span class="o">)</span>          <span class="c1">// this lambda IS the implementation</span>
                              <span class="o">.</span><span class="na">flatMap</span><span class="o">(</span><span class="nl">reserve:</span><span class="o">:</span><span class="n">apply</span><span class="o">)</span>
                              <span class="o">.</span><span class="na">flatMap</span><span class="o">(</span><span class="nl">processPayment:</span><span class="o">:</span><span class="n">apply</span><span class="o">)</span>
                              <span class="o">.</span><span class="na">flatMap</span><span class="o">(</span><span class="nl">confirm:</span><span class="o">:</span><span class="n">apply</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>No code anywhere says <code>new ProcessOrderImpl()</code>. No code depends on the implementation class. The implementation is replaceable by definition — because nothing can reference it.</p>

<p>The interface is the design artifact. The implementation is incidental.</p>

<p>This might sound academic until you need to:</p>

<ul>
<li>Replace a synchronous implementation with an async one</li>
<li>Swap a database adapter for an API adapter</li>
<li>Add a caching layer around an existing step</li>
<li>Completely rewrite a step's internals</li>
</ul>

<p>In each case, the interface stays. The factory method signature stays. The implementation — which nothing references — gets replaced. No downstream changes. No adapter layers. No "backwards compatibility."</p>

<h2>
  
  
  The Compound Effect
</h2>

<p>Each reason is valuable on its own. Together, they create a system where:</p>

<p><strong>Testing is configuration.</strong> You assemble the exact combination of real and stubbed components you need. No mocking framework overhead. No "mock everything" test fragility.</p>

<p><strong>Refactoring is safe.</strong> Replacing an implementation can't break other implementations because they don't share code. The compiler enforces the contract through the interface.</p>

<p><strong>Complexity is bounded.</strong> Understanding one implementation requires understanding only that implementation and the interfaces it consumes. Not a base class hierarchy. Not shared utilities. Not other implementations.</p>

<p><strong>Incremental development is natural.</strong> Stub what's not ready. Replace stubs with real implementations one at a time. Each step can be developed, tested, and deployed independently.</p>

<h2>
  
  
  When Does This Not Apply?
</h2>

<p>When there genuinely is one implementation and always will be. Pure utility functions, mathematical computations, simple data transformations — these don't need the interface + factory treatment. A static method is fine.</p>

<p>The pattern pays for itself when there's any possibility of multiple implementations — including the test implementation, which almost always exists.</p>

<h2>
  
  
  The Shift
</h2>

<p>Most Java codebases default to classes. Interface extraction happens later, reluctantly, when testing forces it or when the second implementation appears.</p>

<p>Flip this. Start with the interface. Define the contract. The implementation follows naturally — and when it needs to change, nothing else does.</p>

<p>The interface is what you design. The implementation is what you happen to write today.</p>

