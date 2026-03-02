---
Title: Day 17 – Building SaaS-Grade Analytics for My AI Phishing Defense Platform
Description: 
Author: Arslon Erkinov
Date: 2026-03-02T21:40:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>Today I focused entirely on backend robustness.</p>

<p>Instead of jumping to UI, I strengthened the analytics layer.</p>

<p>What I Added<br>
1️⃣ API Usage Monitoring<br>
Status codes<br>
Request latency (ms)<br>
Per-API-key tracking<br>
Error rate calculation</p>

<p>2️⃣ Risk Intelligence Metrics<br>
High / Medium / Low distribution<br>
Risk percentage breakdown<br>
7-day usage trends<br>
Average risk score over time</p>

<p>3️⃣ Performance Metrics<br>
Average latency<br>
Error rate (%)<br>
Per-plan breakdown<br>
Real Bug I Faced<br>
I hit this error:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sqlite3.OperationalError: no such column: analyzer_apiusage.latency_ms
</code></pre>

</div>



<p>The model was updated, but the database schema wasn’t.<br>
Classic migration issue.</p>

<p>The fix:<br>
Run makemigrations<br>
Run migrate<br>
Ensure correct module imports in models/init.py</p>

<p>Lesson:<br>
Your ORM and your DB schema must always stay aligned.<br>
Why This Matters<br>
Before building UI, backend must be stable.</p>

<p>Now the system:<br>
Tracks performance<br>
Measures abuse<br>
Prepares for production<br>
Supports SaaS analytics dashboards</p>

<p>Next step: Backend hardening before frontend.<br>
Security is not just detection — it’s architecture.</p>

<h1>
  
  
  django #ai #cybersecurity #backend #saas
</h1>

