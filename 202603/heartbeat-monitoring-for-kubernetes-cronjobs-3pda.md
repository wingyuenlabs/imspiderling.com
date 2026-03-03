---
Title: Heartbeat monitoring for Kubernetes CronJobs
Description: 
Author: Joao Thomazinho
Date: 2026-03-03T21:37:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>Kubernetes CronJobs are great until they quietly fail. Maybe the pod image can't pull, or the job exceeds backoff limits. CronObserver adds a simple heartbeat so you know when jobs don't run on time.</p>

<h2>
  
  
  Why use heartbeats on CronJobs?
</h2>

<ul>
<li>CronJobs can be suspended/deleted during deploys.</li>
<li>Out-of-resource errors happen silently.</li>
<li>Logs rotate quickly; by the time you investigate, the evidence is gone.</li>
</ul>

<p>A single HTTP check-in after the pod completes is enough to warn you.</p>

<h2>
  
  
  Step 1 – Create an observer
</h2>

<p>Define schedule/grace to match the CronJob.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Schedule: */30 * * * *
Grace: 5 minutes
Alerts: Email + webhook (Slack/Teams)
</code></pre>

</div>



<h2>
  
  
  Step 2 – Store the URL in a secret
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Secret</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">cronobserver-checkin</span>
<span class="na">stringData</span><span class="pi">:</span>
  <span class="na">url</span><span class="pi">:</span> <span class="s">https://cronobserver.com/checkin/&lt;token&gt;</span>
</code></pre>

</div>



<p>Mount the secret as an env var:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">env</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">CRONOBSERVER_CHECKIN_URL</span>
    <span class="na">valueFrom</span><span class="pi">:</span>
      <span class="na">secretKeyRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">cronobserver-checkin</span>
        <span class="na">key</span><span class="pi">:</span> <span class="s">url</span>
</code></pre>

</div>



<h2>
  
  
  Step 3 – Curl after the job completes
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">command</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">sh"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">-c"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">run-task</span><span class="nv"> </span><span class="s">&amp;&amp;</span><span class="nv"> </span><span class="s">curl</span><span class="nv"> </span><span class="s">-fsS</span><span class="nv"> </span><span class="s">-X</span><span class="nv"> </span><span class="s">POST</span><span class="nv"> </span><span class="s">$CRONOBSERVER_CHECKIN_URL"</span><span class="pi">]</span>
</code></pre>

</div>



<p>Alternative: wrap your script to post success/failure status in JSON.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">command</span><span class="pi">:</span> <span class="pi">[</span>
  <span class="s2">"</span><span class="s">sh"</span><span class="pi">,</span>
  <span class="s2">"</span><span class="s">-c"</span><span class="pi">,</span>
  <span class="s2">"</span><span class="s">run-task</span><span class="nv"> </span><span class="s">&amp;&amp;</span><span class="nv"> </span><span class="s">curl</span><span class="nv"> </span><span class="s">-fsS</span><span class="nv"> </span><span class="s">-X</span><span class="nv"> </span><span class="s">POST</span><span class="nv"> </span><span class="s">$CRONOBSERVER_CHECKIN_URL</span><span class="nv"> </span><span class="s">-H</span><span class="nv"> </span><span class="s">'Content-Type:</span><span class="nv"> </span><span class="s">application/json'</span><span class="nv"> </span><span class="s">-d</span><span class="nv"> </span><span class="s">'{</span><span class="se">\"</span><span class="s">status</span><span class="se">\"</span><span class="s">:</span><span class="se">\"</span><span class="s">success</span><span class="se">\"</span><span class="s">}'"</span>
<span class="pi">]</span>
</code></pre>

</div>



<h2>
  
  
  Optional – Synthetic checks
</h2>

<p>Instead of waiting for a job to ping, CronObserver can proactively ping your endpoint.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">synthetic_check</span><span class="pi">:</span>
  <span class="na">type</span><span class="pi">:</span> <span class="s">httpGet</span>
  <span class="na">url</span><span class="pi">:</span> <span class="s">https://api.example.com/cron/health</span>
  <span class="na">expected_status</span><span class="pi">:</span> <span class="m">200</span>
  <span class="na">timeout_seconds</span><span class="pi">:</span> <span class="m">10</span>
</code></pre>

</div>



<p>Great for queue processors or serverless schedulers.</p>

<h2>
  
  
  Alerting
</h2>

<ul>
<li>Email or webhook per observer.
</li>
<li>Webhook works with Slack, Teams, Zapier.
</li>
<li>Grace periods and cooldowns reduce noise.</li>
</ul>

<h2>
  
  
  Links
</h2>

<ul>
<li>
<a href="https://cronobserver.com/use-cases" rel="noopener noreferrer">CronObserver Use Cases</a>
</li>
<li>
<a href="https://cronobserver.com/integrations" rel="noopener noreferrer">CronObserver Integrations</a>
</li>
<li><a href="https://cronobserver.com/docs/quickstart" rel="noopener noreferrer">Docs Quickstart</a></li>
</ul>

<p>👉 Monitor your first CronJob free: <a href="https://cronobserver.com/sign-up" rel="noopener noreferrer">cronobserver.com/sign-up</a></p>

