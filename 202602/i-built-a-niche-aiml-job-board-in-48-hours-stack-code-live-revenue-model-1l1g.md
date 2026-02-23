---
Title: I Built a Niche AI/ML Job Board in 48 Hours — Stack, Code & Live Revenue Model
Description: 
Author: Daniel Vermillion
Date: 2026-02-23T21:37:07.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  I Built a Niche AI/ML Job Board in 48 Hours — Here's the Exact Stack
</h1>

<p>I just launched <a href="https://aijobsboard.oblivionlabz.net" rel="noopener noreferrer">aijobsboard.oblivionlabz.net</a> — a fully autonomous AI/ML remote jobs board that auto-aggregates listings, serves job seekers for free, and charges employers to feature listings.</p>

<h2>
  
  
  The Business Model
</h2>

<ul>
<li>
<strong>Job seekers</strong>: Free. Always.</li>
<li>
<strong>Employers</strong>: $99-$299 to feature/pin a listing (via Stripe Checkout)</li>
<li>
<strong>Zero manual work</strong>: Jobs auto-populate from APIs every 30 minutes</li>
</ul>

<p>Proven model: a similar niche board made $21k in its first 4.5 months of 2025.</p>

<h2>
  
  
  The Tech Stack
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Next.js 14 (App Router)
TypeScript
Tailwind CSS
Stripe (Checkout + Webhooks)
Vercel (hosting)
Cloudflare DNS
</code></pre>

</div>



<h2>
  
  
  How Jobs Auto-Populate
</h2>

<p>I pull from 3 free sources with zero scraping — all RSS/JSON feeds:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">SOURCES</span> <span class="o">=</span> <span class="p">[</span>
  <span class="dl">'</span><span class="s1">https://remoteok.com/api?tag=ai</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">https://remotive.com/api/remote-jobs?category=software-dev&amp;limit=50</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">https://jobicy.com/?feed=job_feed&amp;job_category=dev&amp;job_types=full-time</span><span class="dl">'</span>
<span class="p">]</span>
</code></pre>

</div>



<p>No scraping. No paid APIs. Free to consume.</p>

<h2>
  
  
  The Stripe Flow
</h2>

<ol>
<li>Employer hits <code>/post-job</code>
</li>
<li>Fills out company, title, URL, listing type</li>
<li>Redirects to Stripe Checkout ($99 or $299)</li>
<li>On success, webhook fires and job gets featured
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">POST</span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">event</span> <span class="o">=</span> <span class="nx">stripe</span><span class="p">.</span><span class="nx">webhooks</span><span class="p">.</span><span class="nf">constructEvent</span><span class="p">(</span>
    <span class="nx">body</span><span class="p">,</span> <span class="nx">sig</span><span class="p">,</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">STRIPE_WEBHOOK_SECRET</span><span class="o">!</span>
  <span class="p">)</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">checkout.session.completed</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Store featured job, send confirmation</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  What Makes It Autonomous
</h2>

<ul>
<li>Jobs refresh via Vercel ISR (revalidate every 30 min)</li>
<li>Payments process automatically via Stripe</li>
<li>No database needed</li>
<li>Zero moderation for organic listings</li>
</ul>

<h2>
  
  
  The SEO Play
</h2>

<p>Niche job boards rank fast for long-tail terms:</p>

<ul>
<li>"remote machine learning jobs"</li>
<li>"AI engineer remote jobs 2026"</li>
<li>"LLM engineer remote work"</li>
</ul>

<p>Each job listing is a static page with proper meta tags and schema markup.</p>

<h2>
  
  
  The Numbers
</h2>

<ul>
<li>
<strong>Build time</strong>: 48 hours</li>
<li>
<strong>Monthly cost</strong>: $0 (Vercel free tier + free APIs)</li>
<li>
<strong>Revenue model</strong>: $99-$299 per featured listing</li>
<li>
<strong>Autonomy level</strong>: 97%</li>
</ul>

<p>The board is live: <a href="https://aijobsboard.oblivionlabz.net" rel="noopener noreferrer">aijobsboard.oblivionlabz.net</a></p>




<p><em>Built by <a href="https://oblivionlabz.net" rel="noopener noreferrer">Oblivion Labz</a></em></p>

