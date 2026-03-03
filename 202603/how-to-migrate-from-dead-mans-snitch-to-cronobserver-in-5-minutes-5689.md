---
Title: How to migrate from Dead Man's Snitch to CronObserver in 5 minutes
Description: 
Author: Joao Thomazinho
Date: 2026-03-03T21:38:20.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Silent job failures usually hurt the most. This guide shows how to move from Dead Man's Snitch (DMS) to CronObserver without rewriting every scheduled job.</p>
</blockquote>

<h2>
  
  
  Why teams migrate
</h2>

<p>Dead Man's Snitch works well for basic heartbeat monitoring, but teams often hit limitations:</p>

<ul>
<li>Limited payload visibility (no context on why the job failed)</li>
<li>Hard to keep history beyond simple tables</li>
<li>Alerting restricted to email or basic webhook</li>
<li>No synthetic checks or observability integrations</li>
</ul>

<p>CronObserver follows the same "check-in" model but adds payload logging, synthetic checks, per-observer grace periods, and integrations (Datadog, Prometheus, etc.).</p>

<h2>
  
  
  Migration checklist
</h2>

<ol>
<li>[x] Create a CronObserver account (free tier covers 1 job)
</li>
<li>[x] Create an observer for each DMS snitch (copy schedule + grace period)
</li>
<li>[x] Replace the DMS POST call with CronObserver token URL
</li>
<li>[x] Test and archive the old snitch</li>
</ol>

<h2>
  
  
  Step 1 – Create observers
</h2>

<p>Use the CronObserver dashboard to create observers with the same frequency you used in DMS. Example: an hourly job with a 5-minute grace period.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Schedule: Every hour on the hour
Grace: 5 minutes
Alert channels: Email + webhook (Slack/Teams)
</code></pre>

</div>



<h2>
  
  
  Step 2 – Update the check-in call
</h2>

<p>Replace your DMS URL with the CronObserver token. Example for Linux cron:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># old</span>
curl <span class="nt">-fsS</span> <span class="nt">-X</span> POST https://deadmanssnitch.com/your-snitch

<span class="c"># new</span>
curl <span class="nt">-fsS</span> <span class="nt">-X</span> POST https://cronobserver.com/checkin/&lt;token&gt;
</code></pre>

</div>



<h3>
  
  
  Node.js / TypeScript
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">await</span> <span class="nf">runJob</span><span class="p">();</span>
<span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">"</span><span class="s2">https://cronobserver.com/checkin/&lt;token&gt;</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span> <span class="p">},</span>
  <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">status</span><span class="p">:</span> <span class="dl">"</span><span class="s2">success</span><span class="dl">"</span><span class="p">,</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Job finished</span><span class="dl">"</span> <span class="p">}),</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Kubernetes CronJob
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">containers</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">worker</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">alpine:3</span>
    <span class="na">env</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">CHECKIN_URL</span>
        <span class="na">value</span><span class="pi">:</span> <span class="s2">"</span><span class="s">https://cronobserver.com/checkin/&lt;token&gt;"</span>
    <span class="na">command</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">sh</span>
      <span class="pi">-</span> <span class="s">-c</span>
      <span class="pi">-</span> <span class="pi">|</span>
        <span class="s">run-job &amp;&amp; curl -fsS -X POST "$CHECKIN_URL"</span>
</code></pre>

</div>



<h2>
  
  
  Step 3 – Use payloads and thresholds
</h2>

<p>CronObserver lets you send structured JSON metadata.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST https://cronobserver.com/checkin/&lt;token&gt; <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "status": "success",
    "message": "Reports generated",
    "metadata": { "reportId": 42, "duration_ms": 2300 }
  }'</span>
</code></pre>

</div>



<ul>
<li>Payloads show up in the dashboard for debugging.
</li>
<li>You can set payload thresholds (e.g., warn if duration_ms exceeds 10,000).</li>
</ul>

<h2>
  
  
  Step 4 – Add synthetic checks (optional)
</h2>

<p>Instead of waiting for a job to ping you, CronObserver can actively probe your endpoint.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">synthetic_check</span><span class="pi">:</span>
  <span class="na">type</span><span class="pi">:</span> <span class="s">httpGet</span>
  <span class="na">url</span><span class="pi">:</span> <span class="s">https://api.example.com/cron/health</span>
  <span class="na">expected_status</span><span class="pi">:</span> <span class="m">200</span>
  <span class="na">timeout_seconds</span><span class="pi">:</span> <span class="m">10</span>
</code></pre>

</div>



<p>Great for verifying queue workers or lambda triggers.</p>

<h2>
  
  
  Bonus – Quickstart video
</h2>

<p>Embed a quickstart snippet directly in your docs or ask CronObserver support for the import template.</p>

<h2>
  
  
  Wrap-up
</h2>

<ul>
<li>Free plan lets you monitor one critical job before committing.
</li>
<li>Grace periods and cooldowns reduce alert fatigue.
</li>
<li>Built-in observability integrations (Datadog, New Relic, Prometheus, Sentry, Dynatrace).</li>
</ul>

<p>👉 Try it now: <a href="https://cronobserver.com/dead-mans-snitch-alternative" rel="noopener noreferrer">cronobserver.com/dead-mans-snitch-alternative</a></p>

<p>Let me know if you need the import CSV – happy to share the template.</p>

