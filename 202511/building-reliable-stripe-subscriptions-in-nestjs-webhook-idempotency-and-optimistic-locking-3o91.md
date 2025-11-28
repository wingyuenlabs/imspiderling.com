---
Title: Building Reliable Stripe Subscriptions in NestJS: Webhook Idempotency and Optimistic Locking
Description: 
Author: Aniefon Umanah
Date: 2025-11-28T21:35:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>Subscriptions power modern SaaS apps, but integrating payment providers like Stripe requires handling webhooks reliably. In a recent commit to the CommitLore backend, I implemented a full subscription system using Stripe. The standout technical choice was ensuring idempotent webhook processing to avoid duplicate updates during retries or failures.</p>

<p>This approach uses a dedicated <code>webhook_events</code> table to track Stripe events and optimistic locking on the <code>subscriptions</code> table to prevent race conditions. It's a practical pattern for any NestJS app dealing with external async events. Let's break it down.</p>

<h2>
  
  
  Why Idempotency Matters for Stripe Webhooks
</h2>

<p>Stripe webhooks notify your server of events like subscription upgrades, cancellations, or payment failures. But networks are unreliable—events can arrive multiple times, out of order, or during downtime. Without safeguards, this leads to duplicate charges, inconsistent subscription states, or infinite loops.</p>

<p>The solution: Store each webhook in a database first, process it only once, and update the subscription atomically. This commit adds:</p>

<ul>
<li>A new <code>webhook_events</code> table to log incoming events with status tracking.</li>
<li>Optimistic locking via a <code>version</code> column on <code>subscriptions</code> to detect concurrent modifications.</li>
</ul>

<p>This prevents issues like double-processing a <code>customer.subscription.updated</code> event, which could incorrectly downgrade a plan.</p>

<h2>
  
  
  The Database Schema: Tracking Webhook Events
</h2>

<p>The migration creates a simple but robust table. Here's the key SQL from the TypeORM migration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/database/migrations/1764287633270-AddWebhookEventsAndOptimisticLocking.ts</span>
<span class="k">await</span> <span class="nx">queryRunner</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
  CREATE TABLE "webhook_events" (
    "id" uuid NOT NULL DEFAULT uuid_generate_v4(),
    "createdAt" TIMESTAMP NOT NULL DEFAULT now(),
    "updatedAt" TIMESTAMP NOT NULL DEFAULT now(),
    "stripeEventId" character varying(255) NOT NULL,
    "eventType" character varying(100) NOT NULL,
    "status" character varying(20) NOT NULL DEFAULT 'processing',
    "errorMessage" text,
    "processedAt" TIMESTAMP,
    CONSTRAINT "UQ_2321876280160661b78693399a5" UNIQUE ("stripeEventId"),
    CONSTRAINT "PK_4cba37e6a0acb5e1fc49c34ebfd" PRIMARY KEY ("id")
  )
`</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">queryRunner</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`CREATE INDEX "IDX_2321876280160661b78693399a" ON "webhook_events" ("stripeEventId")`</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">queryRunner</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`ALTER TABLE "subscriptions" ADD "version" integer NOT NULL DEFAULT 1`</span><span class="p">);</span>
</code></pre>

</div>



<ul>
<li>
<strong>Unique <code>stripeEventId</code></strong>: Ensures no duplicates—Stripe guarantees this ID is unique per event.</li>
<li>
<strong>Status enum</strong> (<code>processing</code>, <code>processed</code>, <code>failed</code>): Tracks lifecycle for retries or debugging.</li>
<li>
<strong>Index on <code>stripeEventId</code></strong>: Fast lookups during webhook receipt.</li>
</ul>

<p>The entity looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/modules/subscriptions/domain/entities/webhook-event.entity.ts</span>
<span class="p">@</span><span class="nd">Entity</span><span class="p">(</span><span class="dl">'</span><span class="s1">webhook_events</span><span class="dl">'</span><span class="p">)</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">WebhookEvent</span> <span class="kd">extends</span> <span class="nc">BaseEntity</span> <span class="p">{</span>
  <span class="p">@</span><span class="nd">Column</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">varchar</span><span class="dl">'</span><span class="p">,</span> <span class="na">length</span><span class="p">:</span> <span class="mi">255</span><span class="p">,</span> <span class="na">unique</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
  <span class="p">@</span><span class="nd">Index</span><span class="p">()</span>
  <span class="nx">stripeEventId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>

  <span class="p">@</span><span class="nd">Column</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">varchar</span><span class="dl">'</span><span class="p">,</span> <span class="na">length</span><span class="p">:</span> <span class="mi">100</span> <span class="p">})</span>
  <span class="nx">eventType</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>

  <span class="p">@</span><span class="nd">Column</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">varchar</span><span class="dl">'</span><span class="p">,</span> <span class="na">length</span><span class="p">:</span> <span class="mi">20</span><span class="p">,</span> <span class="na">default</span><span class="p">:</span> <span class="nx">WebhookEventStatus</span><span class="p">.</span><span class="nx">PROCESSING</span> <span class="p">})</span>
  <span class="nx">status</span><span class="p">:</span> <span class="nx">WebhookEventStatus</span><span class="p">;</span>

  <span class="p">@</span><span class="nd">Column</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span> <span class="na">nullable</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
  <span class="nx">errorMessage</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>

  <span class="p">@</span><span class="nd">Column</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">timestamp</span><span class="dl">'</span><span class="p">,</span> <span class="na">nullable</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
  <span class="nx">processedAt</span><span class="p">?:</span> <span class="nb">Date</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Handling Webhooks in the Use Case
</h2>

<p>The <code>handle-stripe-webhook.use-case.ts</code> (expanded in this commit) follows a clear flow:</p>

<ol>
<li>
<strong>Verify and Parse</strong>: Use Stripe's SDK to construct the event from payload and signature.</li>
<li>
<strong>Idempotency Check</strong>: Query <code>webhook_events</code> by <code>stripeEventId</code>. If already processed, return 200 early.</li>
<li>
<strong>Store Event</strong>: Insert as "processing".</li>
<li>
<strong>Process Event</strong>: Switch on <code>eventType</code> to update subscriptions (e.g., upgrade on <code>customer.subscription.updated</code>).</li>
<li>
<strong>Atomic Update</strong>: Use optimistic locking—TypeORM's <code>@VersionColumn()</code> throws if the version mismatches.</li>
<li>
<strong>Mark Complete</strong>: Update to "processed" or "failed" with error details.</li>
</ol>

<p>Here's a simplified excerpt of the processing logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Pseudo-code from handle-stripe-webhook.use-case.ts</span>
<span class="k">async</span> <span class="nf">execute</span><span class="p">(</span><span class="nx">payload</span><span class="p">:</span> <span class="nx">Buffer</span><span class="p">,</span> <span class="nx">signature</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">event</span> <span class="o">=</span> <span class="nx">stripe</span><span class="p">.</span><span class="nf">constructWebhookEvent</span><span class="p">(</span><span class="nx">payload</span><span class="p">,</span> <span class="nx">signature</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">existing</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">webhookEventRepository</span><span class="p">.</span><span class="nf">findByStripeEventId</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">existing</span><span class="p">?.</span><span class="nx">status</span> <span class="o">===</span> <span class="nx">WebhookEventStatus</span><span class="p">.</span><span class="nx">PROCESSED</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span> <span class="p">};</span> <span class="c1">// Idempotent skip</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">webhookEvent</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">webhookEventRepository</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
    <span class="na">stripeEventId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
    <span class="na">eventType</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="kd">type</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="k">switch </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="kd">type</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">case</span> <span class="dl">'</span><span class="s1">customer.subscription.updated</span><span class="dl">'</span><span class="p">:</span>
        <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">updateSubscription</span><span class="p">(</span><span class="nx">subscriptionId</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">object</span><span class="p">);</span>
        <span class="k">break</span><span class="p">;</span>
      <span class="c1">// ... other cases like 'invoice.payment_failed'</span>
    <span class="p">}</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">webhookEventRepository</span><span class="p">.</span><span class="nf">updateStatus</span><span class="p">(</span><span class="nx">webhookEvent</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">WebhookEventStatus</span><span class="p">.</span><span class="nx">PROCESSED</span><span class="p">,</span> <span class="kc">undefined</span><span class="p">,</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">());</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">webhookEventRepository</span><span class="p">.</span><span class="nf">updateStatus</span><span class="p">(</span><span class="nx">webhookEvent</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">WebhookEventStatus</span><span class="p">.</span><span class="nx">FAILED</span><span class="p">,</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
    <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The optimistic lock shines in <code>updateSubscription</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// In subscription entity</span>
<span class="p">@</span><span class="nd">VersionColumn</span><span class="p">()</span>
<span class="nx">version</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="c1">// Increments on each save, fails save if concurrent change detected</span>
</code></pre>

</div>



<p>If two webhooks hit simultaneously (rare but possible during retries), TypeORM rolls back the conflicting update, letting the other succeed.</p>

<h2>
  
  
  Cleanup and Monitoring
</h2>

<p>To avoid table bloat, a <code>CleanupWebhookEventsUseCase</code> deletes processed events older than 30 days:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/modules/subscriptions/application/use-cases/cleanup-webhook-events.use-case.ts</span>
<span class="k">async</span> <span class="nf">execute</span><span class="p">(</span><span class="nx">retentionDays</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">30</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">number</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">cutoffDate</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">();</span>
  <span class="nx">cutoffDate</span><span class="p">.</span><span class="nf">setDate</span><span class="p">(</span><span class="nx">cutoffDate</span><span class="p">.</span><span class="nf">getDate</span><span class="p">()</span> <span class="o">-</span> <span class="nx">retentionDays</span><span class="p">);</span>
  <span class="k">return</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">webhookEventRepository</span><span class="p">.</span><span class="nf">deleteProcessedOlderThan</span><span class="p">(</span><span class="nx">cutoffDate</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Schedule this via cron in production. For monitoring, log failures and expose metrics on unprocessed events.</p>

<h2>
  
  
  Gotchas and Lessons
</h2>

<ul>
<li>
<strong>Signature Verification</strong>: Always verify webhook signatures first—Stripe's test mode uses different keys.</li>
<li>
<strong>Retry Logic</strong>: Stripe retries failed webhooks, so idempotency is non-negotiable. But don't process "failed" events again unless you implement custom retries.</li>
<li>
<strong>Version Conflicts</strong>: In high-traffic apps, optimistic locking might conflict often; consider pessimistic if needed, but it's overkill here.</li>
<li>
<strong>Testing</strong>: Use Stripe CLI for local webhooks. Mock the repo in unit tests to simulate duplicates.</li>
</ul>

<p>This setup scales well—events are lightweight, and the unique constraint catches issues early. For a NestJS + Stripe combo, it's a solid foundation. If you're building payments, start with this pattern to save debugging headaches later.</p>

<p>(Word count: ~850 | Char count: ~5200)</p>

