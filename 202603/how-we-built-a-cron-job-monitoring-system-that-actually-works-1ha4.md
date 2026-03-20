---
Title: How We Built a Cron Job Monitoring System That Actually Works
Description: 
Author: quietpulse
Date: 2026-03-20T21:18:26.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  How We Built a Cron Job Monitoring System That Actually Works
</h1>

<h2>
  
  
  The Problem We Couldn't Ignore
</h2>

<p>Cron jobs are the backbone of many SaaS applications. But when they fail silently, you don't find out until it's too late.</p>

<p>We built QuietPulse because we needed a simple, reliable way to know when our background tasks go missing — without breaking the bank.</p>

<h2>
  
  
  What QuietPulse Does
</h2>

<p>QuietPulse is a heartbeat monitoring service for cron jobs. Here's the core idea:</p>

<ol>
<li>
<strong>Create a job</strong> in QuietPulse with a minimum interval (e.g., every 5 minutes)</li>
<li>
<strong>Ping QuietPulse</strong> after your cron job finishes successfully via a simple GET request</li>
<li>
<strong>Get alerted</strong> (via Telegram) if the job doesn't check in on time</li>
</ol>

<p>That's it. No complex UIs, no overwhelming dashboards. Just one question: "Did my job run when it was supposed to?"</p>

<h2>
  
  
  Architecture: Simple Stack, Reliable Delivery
</h2>

<p>We chose technologies we knew would work reliably in production:</p>

<ul>
<li>
<strong>Backend</strong>: NestJS (TypeScript, dependency injection, great for APIs)</li>
<li>
<strong>Frontend</strong>: Angular 21 (standalone components, modern tooling)</li>
<li>
<strong>Database</strong>: SQLite with Prisma ORM (fast, zero-config, easy backups)</li>
<li>
<strong>Hosting</strong>: DigitalOcean VPS ($4/month) in Frankfurt</li>
<li>
<strong>Payments</strong>: NOWPayments (crypto invoices, no Stripe/PayPal needed)</li>
<li>
<strong>Alerts</strong>: Telegram Bot API (instant notifications)</li>
</ul>

<p>The whole thing runs on a single $4 VPS. No Kubernetes, no microservices — just one monolith that does its job well.</p>

<h2>
  
  
  How the Heartbeat Works
</h2>

<p>Each job gets a unique HTTP endpoint:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight http"><code><span class="err">GET https://quietpulse.xyz/ping/{jobToken}
</span></code></pre>

</div>



<p>Your cron job makes a request after completion:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="s2">"https://quietpulse.xyz/ping/AbCdEf123..."</span>
</code></pre>

</div>



<p>QuietPulse records the timestamp. If the expected interval passes without a new ping, it sends a Telegram alert to the user's configured chat.</p>

<p>Simple, stateless, and works from any environment (even without outbound internet, as long as it can reach our endpoint).</p>

<h2>
  
  
  Payment Challenges: Why Crypto?
</h2>

<p>We're based in Belarus. Stripe and PayPal don't work here. So we integrated NOWPayments, which accepts dozens of cryptocurrencies and converts them to stablecoins.</p>

<p>Key features:</p>

<ul>
<li>User picks their preferred crypto at checkout</li>
<li>Webhook notifies us when payment confirms</li>
<li>No monthly subscription headaches</li>
</ul>

<p>It's not perfect, but it works globally without banking corruption.</p>

<h2>
  
  
  Pricing: Freemium That Actually Helps
</h2>

<p>We kept pricing simple:</p>

<ul>
<li>
<strong>Free</strong>: 2 jobs, minimum interval 1 hour</li>
<li>
<strong>Starter</strong>: $7/month (or $20 for 3 months) — 20 jobs, min interval 5 minutes</li>
<li>
<strong>Unlimited</strong>: $25/month — unlimited jobs, min interval 1 minute</li>
</ul>

<p>We want small projects to be able to monitor their critical tasks for free. If you need more, you upgrade. No hidden quotas.</p>

<h2>
  
  
  Lessons Learned
</h2>

<ol>
<li>
<strong>Start with Telegram</strong>. Notifications via Telegram are more reliable than email. Users already have Telegram installed.</li>
<li>
<strong>SQLite scales further than you think</strong>. With proper connection pooling and WAL mode, a single file handles thousands of pings per minute.</li>
<li>
<strong>Don't overcomplicate the dashboard</strong>. Our entire UI is one page with a table and a "create job" modal. Users don't need analytics, they need alerts.</li>
<li>
<strong>Test your own cron jobs</strong>. We run QuietPulse to monitor QuietPulse's own maintenance tasks. If we miss an alert, we know something's broken.</li>
<li>
<strong>Crypto payments are viable</strong>. Yes, there's volatility, but NOWPayments auto-converts to USDT. The friction is mostly on the user side, not ours.</li>
</ol>

<h2>
  
  
  What's Next?
</h2>

<p>We're polishing the first stable release and writing documentation. The first blog post about detecting missed cron jobs is already live at <a href="https://quietpulse.xyz/blog" rel="noopener noreferrer">quietpulse.xyz/blog</a>.</p>

<p>If you have cron jobs that matter, give QuietPulse a try. It's free to start, and setup takes less than 2 minutes.</p>

<p><strong><a href="https://quietpulse.xyz" rel="noopener noreferrer">Create your first job → quietpulse.xyz</a></strong></p>




<p><em>Built with TypeScript, Angular, and a pressing need to sleep better at night.</em></p>

