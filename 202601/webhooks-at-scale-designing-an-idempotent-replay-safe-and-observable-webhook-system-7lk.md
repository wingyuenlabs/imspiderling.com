---
Title: Webhooks at Scale: Designing an Idempotent, Replay-Safe, and Observable Webhook System
Description: 
Author: Art light
Date: 2026-01-19T21:19:59.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Webhooks look easy until your system processes the same payment 3 times, drops one critical event, and you can’t prove what actually happened.</p>
</blockquote>

<p>This article is a production-grade deep dive into building a webhook ingestion system that survives retries, replays, out-of-order delivery, provider bugs, and your own future self.</p>

<p>If you’ve ever thought “we’ll just verify the signature and store the payload” — this post is for you.</p>

<h2>
  
  
  Why webhooks are deceptively hard
</h2>

<p>Most webhook providers promise:</p>

<ul>
<li>at-least-once delivery</li>
<li>retries on failure</li>
<li>signed payloads</li>
</ul>

<p>What they don’t promise:</p>

<ul>
<li>ordering</li>
<li>uniqueness</li>
<li>consistency</li>
</ul>

<p>sane retry behavior</p>

<p><strong>Reality</strong>: webhooks are an unreliable distributed queue that you do not control.</p>

<p>Treat them as such.</p>

<h2>
  
  
  Failure modes most teams discover too late
</h2>

<ul>
<li>Duplicate events processed twice</li>
<li>Provider retries for hours after success</li>
<li>Events arriving out of order</li>
<li>Partial failures mid-processing</li>
<li>Clock skew breaking signatures</li>
<li>Silent drops with no audit trail</li>
</ul>

<p>A correct design assumes all of these happen daily.</p>

<h2>
  
  
  Architecture overview
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Webhook Provider
   │
   │  POST /webhook
   ▼
Ingress Layer (Fast, Stateless)
   │
   │  enqueue
   ▼
Persistent Event Store
   │
   │  dedupe + order
   ▼
Event Processor
   │
   │  side effects
   ▼
Domain Services
</code></pre>

</div>



<p>Key principle:</p>

<blockquote>
<p>Never do business logic in the webhook handler.</p>
</blockquote>

<h2>
  
  
  Step 1: Fast acknowledgment (or you will get retries)
</h2>

<p>Webhook endpoints must:</p>

<ul>
<li>verify signature</li>
<li>persist raw payload</li>
<li>return 2xx</li>
</ul>

<p>Nothing else.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>app.post('/webhook', async (req, res) =&gt; {
  verifySignature(req)
  await storeRawEvent(req)
  res.status(200).end()
})
</code></pre>

</div>



<p>If your endpoint takes &gt;1–2 seconds, retries are guaranteed.</p>

<h2>
  
  
  Step 2: Raw event persistence (non-negotiable)
</h2>

<p>Store <strong>exactly</strong> what you received:</p>

<ul>
<li>headers</li>
<li>body</li>
<li>timestamp</li>
<li><p>provider event ID (if any)<br>
Why?</p></li>
<li><p>replay</p></li>
<li><p>audits</p></li>
<li><p>debugging provider disputes</p></li>
</ul>

<p>Never transform at this stage.</p>

<h2>
  
  
  Step 3: Idempotency is not optional
</h2>

<blockquote>
<p>If your system is not idempotent, retries are data corruption.</p>
</blockquote>

<p><strong>The wrong approach</strong></p>

<ul>
<li>“We’ll check if status already changed” ❌</li>
<li>“We’ll trust provider event IDs” ❌
<strong>The correct approach</strong>
</li>
</ul>

<p>Create your own idempotency key:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const key = hash(provider + eventType + externalObjectId)
</code></pre>

</div>



<p>Persist it with a unique constraint.</p>

<p>If insert fails → duplicate → skip safely.</p>

<h2>
  
  
  Step 4: Ordering without pretending you control time
</h2>

<p>Providers do not guarantee ordering.</p>

<p>Never assume:</p>

<ul>
<li>event A arrives before event B</li>
<li>timestamps are monotonic</li>
</ul>

<p><strong>Strategy</strong></p>

<ul>
<li>Model events as state transitions</li>
<li>Reject invalid transitions
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if (!isValidTransition(currentState, nextEvent)) {
  logAndIgnore()
}
</code></pre>

</div>



<p>This makes ordering irrelevant.</p>

<h2>
  
  
  Step 5: Exactly-once side effects (the hard part)
</h2>

<p>Databases are transactional. External APIs are not.</p>

<p><strong>Pattern: transactional outbox</strong></p>

<ol>
<li>Write domain change + outbox record in same transaction</li>
<li>Commit</li>
<li>Async worker executes side effects</li>
<li>Mark outbox as done</li>
</ol>

<p>This prevents:</p>

<ul>
<li>double emails</li>
<li>double charges</li>
<li>partial failures</li>
</ul>

<h2>
  
  
  Step 6: Signature verification pitfalls
</h2>

<p>Common mistakes:</p>

<ul>
<li>parsing JSON before verification</li>
<li>ignoring header casing</li>
<li>using system clock blindly</li>
</ul>

<p>Always:</p>

<ul>
<li>verify against raw body</li>
<li>allow small clock skew</li>
<li>fail closed</li>
</ul>

<p>If verification fails → <strong>do not retry internally.</strong></p>

<h2>
  
  
  Step 7: Observability or it didn’t happen
</h2>

<p>You need to answer:</p>

<ul>
<li>did we receive it?</li>
<li>did we process it?</li>
<li>what did it change?</li>
</ul>

<p>Minimum requirements:</p>

<ul>
<li>event ID traceable across logs</li>
<li>processing status persisted</li>
<li>dead-letter queue for failures</li>
</ul>

<p>If you can’t answer these in &lt;5 minutes, your system is blind.</p>

<h2>
  
  
  Production checklist
</h2>

<p>Miss one — and you’ll eventually ship a bug you can’t undo.</p>

<h2>
  
  
  Final thoughts
</h2>

<p>Webhooks are not callbacks. They are untrusted, replayable messages.</p>

<p>Once you treat them that way, they become boring.</p>

<p>And boring infrastructure is the goal.</p>

