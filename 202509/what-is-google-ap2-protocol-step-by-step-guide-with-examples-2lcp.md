---
Title: What is Google AP2 Protocol : Step by Step Guide with Examples
Description: 
Author: vishalmysore
Date: 2025-09-18T21:52:20.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What is the Google AP2 Protocol?
</h2>

<p>The Agent Payments Protocol (AP2) is an open, non-proprietary protocol developed by Google in collaboration with several other organizations in financial services and tech industries. Its purpose is to establish a secure and auditable framework for payments initiated by AI agents. AP2 extends existing protocols like Agent2Agent (A2A) and Model Context Protocol (MCP) to specifically handle payment flows, ensuring transactions are authorized, authentic, and accountable</p>

<p>The rise of AI agents capable of transacting on behalf of users creates new challenges for secure, authenticated, and accountable payments. Traditional payment systems assume a human is directly clicking “buy” on a trusted surface, but autonomous agents break this assumption. The Agentic Payment Protocol (AP2) is an open, shared protocol that provides a common language for secure, compliant transactions between agents and merchants, preventing ecosystem fragmentation.</p>

<p>Code for this article is <a href="https://github.com/vishalmysore/ap2java" rel="noopener noreferrer">here</a></p>

<p><strong>AP2 addresses three critical questions:</strong></p>

<ul>
<li>
<strong>Authorization:</strong> Proving that a user gave an agent specific authority to make a particular purchase.</li>
<li>
<strong>Authenticity:</strong> Enabling merchants to verify that an agent’s request accurately reflects the user’s true intent.</li>
<li>
<strong>Accountability:</strong> Determining responsibility if a fraudulent or incorrect transaction occurs.</li>
</ul>

<p>AP2 supports a wide range of payment types—from cards to stablecoins and real-time bank transfers—ensuring a consistent, secure, and scalable experience for users, merchants, and financial institutions.</p>

<h2>
  
  
  How AP2 Works: Mandates and Verifiable Credentials
</h2>

<p>AP2 builds trust using <strong>Mandates</strong>—tamper-proof, cryptographically-signed digital contracts that serve as verifiable proof of a user’s instructions. Mandates are signed by verifiable credentials (VCs) and act as foundational evidence for every transaction.</p>

<p><strong>Mandates support two primary shopping scenarios:</strong></p>

<ul>
<li><p><strong>Real-time purchases (human present):</strong><br>
The user asks an agent to find a product (e.g., “Find me new white running shoes”). The request is captured in an initial <strong>Intent Mandate</strong>, providing auditable context. After the agent presents a cart, the user’s approval signs a <strong>Cart Mandate</strong>, creating a secure, unchangeable record of the items and price.</p></li>
<li><p><strong>Delegated tasks (human not present):</strong><br>
The user delegates a task (e.g., “Buy concert tickets the moment they go on sale”) by signing a detailed Intent Mandate upfront, specifying rules like price limits and timing. The agent can automatically generate a Cart Mandate once conditions are met.</p></li>
</ul>

<p>In both cases, the chain of evidence links the payment method to the verified contents of the Cart Mandate, creating a non-repudiable audit trail for authorization, authenticity, and accountability.</p>

<h2>
  
  
  Java Implementation: Alignment with AP2 Protocol and Python Samples
</h2>

<p>My Java implementation closely follows the AP2 protocol and the reference Python samples:</p>

<ul>
<li>
<strong>Mandate System:</strong> Implements IntentMandate, CartMandate, and PaymentMandate as verifiable credentials, each with constraints, references, and cryptographic proofs.</li>
<li>
<strong>Agent Roles:</strong> Defines agent classes for Shopping Agent, Merchant Agent, and Payment Processor, mirroring the modular structure of the Python samples.</li>
<li>
<strong>Payment Lifecycle:</strong> Supports creation, authorization, capture, refund, and status queries using interfaces and domain classes similar to those in Python.</li>
<li>
<strong>Security and Audit:</strong> Enforces signature verification, mandate chaining, and audit logging for accountability and security.</li>
<li>
<strong>Agent Communication:</strong> Uses JSON-RPC and a2ajava for agent-to-agent messaging, just as the Python samples use HTTP and A2A message formats.</li>
</ul>

<p>This alignment ensures that implementation is interoperable, secure, and true to the AP2 protocol’s design.</p>

<h2>
  
  
  Typical Use Case: Agentic Commerce Flow
</h2>

<ol>
<li>
<p><strong>User Authorization (Intent Mandate):</strong></p>

<ul>
<li>The user instructs an agent (e.g., “Buy running shoes under $100”).</li>
<li>The agent creates an Intent Mandate capturing the user’s intent and constraints.</li>
</ul>
</li>
<li>
<p><strong>Cart Building &amp; Approval (Cart Mandate):</strong></p>

<ul>
<li>The agent finds products and builds a cart.</li>
<li>The user reviews and approves the cart, resulting in a Cart Mandate.</li>
</ul>
</li>
<li>
<p><strong>Payment Execution (Payment Mandate):</strong></p>

<ul>
<li>The agent initiates payment using the approved cart.</li>
<li>A Payment Mandate is created and sent to the payment processor.</li>
</ul>
</li>
<li>
<p><strong>Payment Lifecycle:</strong></p>

<ul>
<li>The payment processor handles creation, authorization, capture, refund, and status queries.</li>
</ul>
</li>
<li>
<p><strong>Mandate Verification &amp; Audit:</strong></p>

<ul>
<li>All mandates are verified for signature and references, with audit logs for traceability.</li>
</ul>
</li>
<li>
<p><strong>Agent-to-Agent Communication:</strong></p>

<ul>
<li>Mandates and messages are exchanged between agents, merchants, and processors using secure protocols.</li>
</ul>
</li>
</ol>




<p>This flow demonstrates how AP2 enables secure, user-controlled, and auditable agentic commerce, with your Java implementation faithfully reflecting the protocol’s architecture and intent.</p>

<h2>
  
  
  Java Code Samples: Mandate Flow and Payment Lifecycle
</h2>

<p>Below are sample code snippets from the Java implementation, demonstrating how AP2 mandates and payment flows are handled. These examples are directly inspired by the official AP2 sample implementations and closely follow their structure and logic.</p>

<h3>
  
  
  1. Intent Mandate
</h3>

<p>When a user instructs an agent (e.g., "Buy running shoes under $100"), the agent creates an Intent Mandate:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">IntentMandate</span> <span class="n">mandate</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">IntentMandate</span><span class="o">(</span>
   <span class="s">"intent-123"</span><span class="o">,</span>                 <span class="c1">// Unique ID</span>
   <span class="s">"user-agent-456"</span><span class="o">,</span>             <span class="c1">// Requesting agent</span>
   <span class="s">"merchant-789"</span><span class="o">,</span>               <span class="c1">// Receiving agent/merchant</span>
   <span class="k">new</span> <span class="nf">BigDecimal</span><span class="o">(</span><span class="s">"100.00"</span><span class="o">),</span>     <span class="c1">// Maximum allowed amount</span>
   <span class="kc">true</span>                          <span class="c1">// Requires human approval</span>
<span class="o">);</span>
<span class="n">mandate</span><span class="o">.</span><span class="na">setAllowedCategories</span><span class="o">(</span><span class="nc">Collections</span><span class="o">.</span><span class="na">singletonList</span><span class="o">(</span><span class="s">"shoes"</span><span class="o">));</span>
</code></pre>

</div>



<h3>
  
  
  2. Cart Mandate
</h3>

<p>After finding options, the agent builds a cart and seeks user approval. The Cart Mandate references the original Intent Mandate and contains specific items and prices:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">List</span><span class="o">&lt;</span><span class="nc">CartMandate</span><span class="o">.</span><span class="na">CartItem</span><span class="o">&gt;</span> <span class="n">items</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayList</span><span class="o">&lt;&gt;();</span>
<span class="n">items</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="k">new</span> <span class="nc">CartMandate</span><span class="o">.</span><span class="na">CartItem</span><span class="o">(</span>
   <span class="s">"item-123"</span><span class="o">,</span>                   <span class="c1">// Item ID</span>
   <span class="s">"Nike Air Zoom Pegasus"</span><span class="o">,</span>      <span class="c1">// Description</span>
   <span class="k">new</span> <span class="nf">BigDecimal</span><span class="o">(</span><span class="s">"95.00"</span><span class="o">),</span>      <span class="c1">// Price</span>
   <span class="mi">1</span>                             <span class="c1">// Quantity</span>
<span class="o">));</span>

<span class="nc">CartMandate</span> <span class="n">cartMandate</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CartMandate</span><span class="o">(</span>
   <span class="s">"cart-456"</span><span class="o">,</span>                   <span class="c1">// Unique ID</span>
   <span class="s">"user-agent-456"</span><span class="o">,</span>             <span class="c1">// Requesting agent</span>
   <span class="s">"merchant-789"</span><span class="o">,</span>               <span class="c1">// Receiving agent/merchant</span>
   <span class="s">"intent-123"</span><span class="o">,</span>                 <span class="c1">// Parent Intent Mandate ID</span>
   <span class="n">items</span><span class="o">,</span>                        <span class="c1">// List of items</span>
   <span class="s">"USD"</span><span class="o">,</span>                        <span class="c1">// Currency</span>
   <span class="nc">Instant</span><span class="o">.</span><span class="na">now</span><span class="o">().</span><span class="na">plusMinutes</span><span class="o">(</span><span class="mi">30</span><span class="o">)</span> <span class="c1">// Expiration</span>
<span class="o">);</span>
</code></pre>

</div>



<h3>
  
  
  3. Payment Mandate
</h3>

<p>With the cart approved, the agent executes payment. The Payment Mandate references the Cart Mandate and includes payment details:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">Map</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">Object</span><span class="o">&gt;</span> <span class="n">paymentDetails</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">HashMap</span><span class="o">&lt;&gt;();</span>
<span class="n">paymentDetails</span><span class="o">.</span><span class="na">put</span><span class="o">(</span><span class="s">"paymentMethodId"</span><span class="o">,</span> <span class="s">"card-123"</span><span class="o">);</span>

<span class="nc">PaymentMandate</span> <span class="n">paymentMandate</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">PaymentMandate</span><span class="o">(</span>
   <span class="s">"payment-789"</span><span class="o">,</span>                <span class="c1">// Unique ID</span>
   <span class="s">"user-agent-456"</span><span class="o">,</span>             <span class="c1">// Requesting agent</span>
   <span class="s">"merchant-789"</span><span class="o">,</span>               <span class="c1">// Receiving agent/merchant</span>
   <span class="s">"order-321"</span><span class="o">,</span>                  <span class="c1">// Payment reference</span>
   <span class="k">new</span> <span class="nf">BigDecimal</span><span class="o">(</span><span class="s">"95.00"</span><span class="o">),</span>      <span class="c1">// Amount</span>
   <span class="s">"USD"</span><span class="o">,</span>                        <span class="c1">// Currency</span>
   <span class="s">"cart-456"</span><span class="o">,</span>                   <span class="c1">// Parent Cart Mandate ID</span>
   <span class="n">paymentDetails</span><span class="o">,</span>                <span class="c1">// Payment details</span>
   <span class="nc">Instant</span><span class="o">.</span><span class="na">now</span><span class="o">().</span><span class="na">plusMinutes</span><span class="o">(</span><span class="mi">15</span><span class="o">)</span>  <span class="c1">// Expiration</span>
<span class="o">);</span>
</code></pre>

</div>



<h3>
  
  
  4. Mandate Verification
</h3>

<p>Mandates are verified for signature, expiration, and correct references:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kt">boolean</span> <span class="n">isValid</span> <span class="o">=</span> <span class="n">mandateVerifier</span><span class="o">.</span><span class="na">verifyIntentMandate</span><span class="o">(</span><span class="n">intentMandate</span><span class="o">,</span> <span class="n">paymentRequest</span><span class="o">);</span>
<span class="k">if</span> <span class="o">(</span><span class="n">isValid</span><span class="o">)</span> <span class="o">{</span>
   <span class="n">isValid</span> <span class="o">=</span> <span class="n">mandateVerifier</span><span class="o">.</span><span class="na">verifyCartMandate</span><span class="o">(</span><span class="n">cartMandate</span><span class="o">,</span> <span class="n">intentMandate</span><span class="o">,</span> <span class="n">paymentRequest</span><span class="o">);</span>
   <span class="c1">// ...</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  5. Payment Processing Flow
</h3>

<p>The payment processor handles creation, authorization, capture, refund, and status queries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">PaymentProcessor</span> <span class="n">paymentProcessor</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SamplePaymentProcessor</span><span class="o">();</span>
<span class="nc">AP2Client</span> <span class="n">ap2Client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AP2Client</span><span class="o">(</span><span class="n">paymentProcessor</span><span class="o">,</span> <span class="n">agentCard</span><span class="o">);</span>

<span class="nc">PaymentRequest</span> <span class="n">paymentRequest</span> <span class="o">=</span> <span class="nc">PaymentRequest</span><span class="o">.</span><span class="na">builder</span><span class="o">()</span>
      <span class="o">.</span><span class="na">amount</span><span class="o">(</span><span class="k">new</span> <span class="nc">BigDecimal</span><span class="o">(</span><span class="s">"50.00"</span><span class="o">))</span>
      <span class="o">.</span><span class="na">currencyCode</span><span class="o">(</span><span class="s">"USD"</span><span class="o">)</span>
      <span class="o">.</span><span class="na">requestingAgentId</span><span class="o">(</span><span class="s">"agent-123"</span><span class="o">)</span>
      <span class="o">.</span><span class="na">receivingAgentId</span><span class="o">(</span><span class="s">"merchant-456"</span><span class="o">)</span>
      <span class="o">.</span><span class="na">description</span><span class="o">(</span><span class="s">"Service payment"</span><span class="o">)</span>
      <span class="o">.</span><span class="na">build</span><span class="o">();</span>

<span class="nc">PaymentResponse</span> <span class="n">response</span> <span class="o">=</span> <span class="n">ap2Client</span><span class="o">.</span><span class="na">createPayment</span><span class="o">(</span><span class="n">paymentRequest</span><span class="o">).</span><span class="na">get</span><span class="o">();</span>
</code></pre>

</div>



<h2>
  
  
  Alignment with Official AP2 Sample Implementations
</h2>

<p>The above Java samples directly mirror the official AP2 sample implementations (Python and Android):</p>

<ul>
<li>
<strong>Mandate Chain:</strong> The use of Intent, Cart, and Payment Mandates as verifiable credentials is identical to the reference samples.</li>
<li>
<strong>Agent Roles:</strong> The modular agent structure (Shopping Agent, Merchant Agent, Payment Processor) matches the official design.</li>
<li>
<strong>Payment Flow:</strong> The lifecycle (create, authorize, capture, refund, status) and mandate verification logic are implemented as described in the AP2 specification and samples.</li>
<li>
<strong>Security:</strong> Signature verification, expiration, and audit logging are enforced, just as in the official samples.</li>
<li>
<strong>Communication:</strong> The use of JSON-RPC and agent-to-agent messaging aligns with the protocol’s recommended transport and message formats.</li>
</ul>

