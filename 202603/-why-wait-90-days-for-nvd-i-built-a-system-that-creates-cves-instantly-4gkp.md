---
Title: # Why Wait 90 Days for NVD? I Built a System That Creates CVEs Instantly.
Description: 
Author: Malika
Date: 2026-03-09T22:10:23.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <em>tags: security, ai, cybersecurity, n8n</em>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx5pbsuzw4uuynxm7bo9t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx5pbsuzw4uuynxm7bo9t.png" alt="SOC-CERT: AI-Powered Threat Intelligence — showing data sources (NIST, CERT-FR, BleepingComputer, CISA), CVE enrichment pipeline, Cohere AI agent, and multi-channel output (Gmail, Slack, Sheets) with 99.8% uptime and 0 false positives" width="800" height="533"></a></p>

<p>I won a dev.to challenge last August by building something I desperately needed: an automated threat intelligence system that doesn't drown your team in noise.</p>

<p>But this article isn't about winning. It's about what I learned building <strong>SOC-CERT</strong> — and why every developer should think differently about security automation in 2025.</p>




<h2>
  
  
  The Problem Nobody Talks About
</h2>

<p>Most security advice stops at "keep your dependencies updated, run a static scanner." That's necessary but nowhere near enough.</p>

<p>Here's the real problem: <strong>NVD publishes CVEs 30 to 90 days after a vulnerability is discovered.</strong> That gap is your exposure window. You're running dependencies with known exploits, and the official database hasn't catalogued them yet.</p>

<p>Meanwhile, your security team — if you even have one — is drowning in alerts from 5 different feeds, all reporting the same CVE with slightly different formatting. Alert fatigue sets in. Things get missed.</p>

<p>I built SOC-CERT to fix exactly this.</p>




<h2>
  
  
  What SOC-CERT Actually Does
</h2>

<p>SOC-CERT is an automated threat intelligence pipeline built with <strong>n8n</strong> and <strong>Bright Data</strong> that:</p>

<ul>
<li>Monitors <strong>CISA, NIST NVD, CERT-FR, and BleepingComputer</strong> simultaneously</li>
<li>Runs every CVE through <strong>Cohere Command-R</strong> for AI-powered severity scoring</li>
<li>Uses <strong>hash-based deduplication</strong> so the same CVE from 5 different sources generates exactly one alert</li>
<li>Delivers structured notifications to Slack, Gmail, and Google Sheets in under 5 minutes from detection</li>
</ul>

<p>It processes 100+ CVEs daily with 99.8% uptime, completely free using tier services.</p>

<p>The architecture looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Data Sources (CISA, NIST, CERT-FR, BleepingComputer)
        ↓
Bright Data (proxy rotation, anti-bot bypass)
        ↓
n8n Orchestration Layer
        ↓
Cohere AI Agent (severity scoring + structured output)
        ↓
Hash-based dedup + change detection
        ↓
Multi-channel notifications (Slack / Gmail / Sheets)
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fimgbr8j3jt0a9tynee3r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fimgbr8j3jt0a9tynee3r.png" alt="Full SOC-CERT pipeline diagram — from Cron trigger and Rate Limiter through NIST, CERT-FR, BleepingComputer and CISA data collection, enrichment via CISA KEV, CIRCL and AlienVault OTX, AI agent with Cohere + memory, through to Slack, Gmail and Google Sheets output" width="794" height="2665"></a></p>




<h2>
  
  
  The Technical Decisions That Actually Mattered
</h2>

<h3>
  
  
  1. Hash-based deduplication — the most underrated feature
</h3>

<p>This was the insight that changed everything. Instead of trying to parse and compare CVE IDs across sources (which fail constantly due to formatting differences), I hash the normalized CVE content and track changes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Simplified dedup logic in n8n</span>
<span class="kd">const</span> <span class="nx">contentHash</span> <span class="o">=</span> <span class="nx">crypto</span>
  <span class="p">.</span><span class="nf">createHash</span><span class="p">(</span><span class="dl">'</span><span class="s1">sha256</span><span class="dl">'</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">cve</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="na">description</span><span class="p">:</span> <span class="nx">cve</span><span class="p">.</span><span class="nx">description</span> <span class="p">}))</span>
  <span class="p">.</span><span class="nf">digest</span><span class="p">(</span><span class="dl">'</span><span class="s1">hex</span><span class="dl">'</span><span class="p">);</span>

<span class="k">if </span><span class="p">(</span><span class="nx">seenHashes</span><span class="p">.</span><span class="nf">has</span><span class="p">(</span><span class="nx">contentHash</span><span class="p">))</span> <span class="p">{</span>
  <span class="k">return</span><span class="p">;</span> <span class="c1">// Already processed, skip</span>
<span class="p">}</span>

<span class="nx">seenHashes</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="nx">contentHash</span><span class="p">);</span>
<span class="c1">// Process and alert...</span>
</code></pre>

</div>



<p>Result: <strong>zero duplicate alerts</strong> across 4 sources running 24/7.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0jgklctz2u9ihikea93v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0jgklctz2u9ihikea93v.png" alt="n8n workflow in 3 sections — top: data collection from NIST, CERT-FR, BleepingComputer, CISA with Bright Data and CISA KEV enrichment; middle: Normalize → Diff/Hash Check → AI Agent (Cohere + Simple Memory) → AI Output Parser → Structure Alerts; bottom: Structure Alerts branching to Health Check, Critical Alerts, Append to Sheet, Slack Notify and Gmail Alert Message" width="800" height="916"></a></p>

<h3>
  
  
  2. Constraining the AI output — not trusting it blindly
</h3>

<p>The biggest mistake in AI-powered automation is treating the model as a decision-maker. Cohere Command-R is excellent at structured extraction — but only if you constrain it hard.</p>

<p>My system prompt wasn't "analyze this CVE." It was a strict schema definition:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Analyze the following CVE data and return ONLY valid JSON with these exact fields:
- severity: CRITICAL | HIGH | MEDIUM | LOW
- cvss_estimated: float between 0-10
- affected_components: array of strings
- exploitation_likelihood: integer 1-5
- summary: max 100 words

Do not include any text outside the JSON object.
</code></pre>

</div>



<p>This + an output parser node in n8n = consistent, machine-readable data every time.</p>

<h3>
  
  
  3. Retry logic and graceful degradation
</h3>

<p>NIST NVD goes down. CERT-FR rate-limits aggressively. BleepingComputer blocks scrapers regularly.</p>

<p>The pipeline uses a <strong>Continue on Error</strong> pattern with 3 retry attempts and exponential backoff per source. If a source fails, the pipeline logs the failure, alerts the admin, and continues processing the other sources. <strong>Partial data is infinitely better than a crashed pipeline.</strong></p>




<h2>
  
  
  The 90-Day Gap Problem — and the Virtual CVE Solution
</h2>

<p>After winning the n8n challenge, I took this further. Building the Chrome extension version of SOC-CERT revealed something deeper:</p>

<p>Even with a perfect pipeline, you're still reactive. You're tracking vulnerabilities after they've been named.</p>

<p>The solution I built: <strong>Virtual CVE Intelligence.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzsjlc22aucbcfpttw3ai.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzsjlc22aucbcfpttw3ai.png" alt="Side-by-side comparison: NVD process (90-day documentation delay, emerging threats invisible, enterprises vulnerable) vs SOC-CERT Guardian protection (2.3-second detection, Virtual CVE intelligence, immediate protection) — showing CVE-2026-148724 detected as Emerging Threat with score 75" width="800" height="436"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Normal CVE lifecycle:
Day 0:  Vulnerability discovered
Day 30: Research completed
Day 60: CVE submitted to MITRE
Day 90: Published in NVD
→ 90-day window with zero tracking

Virtual CVE approach:
Second 0:  User visits suspicious URL
Second 2:  Gemini Nano detects threat pattern
Second 5:  Virtual CVE generated (CVE-2026-XXXXX)
Second 10: Alert with remediation steps
→ Immediate tracking from detection moment
</code></pre>

</div>



<p>The Chrome extension combines local Gemini Nano analysis (for speed and privacy) with server-side n8n enrichment against the CISA KEV catalog. Two-stage analysis: fast local result in 2 seconds, enriched result with real CVE correlation shortly after.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd9g38ku0jip1nqjq9n24.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd9g38ku0jip1nqjq9n24.png" alt="SOC-CERT Guardian Chrome extension panel showing: analyzed URL testphp.vulnweb.com rated Malicious at 90% risk score with 95% confidence, SQL Injection and URL Encoding threat indicators, Gemini Nano AI analysis results, and deep CVE correlation identifying CVE-2020-0618 Critical with remediation recommendations" width="610" height="2518"></a></p>




<h2>
  
  
  What This Means for Any Web Stack
</h2>

<p>Here's where I want to talk to you directly as a developer.</p>

<p>Your app is probably pulling in dozens of third-party dependencies. Each one is a potential vulnerability surface. Running a dependency scanner in CI is good practice — but it only catches CVEs that are already in the advisory database.</p>

<p>A few patterns from SOC-CERT that translate directly to any stack:</p>

<p><strong>Pattern 1: Monitor, don't just audit</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8pgvcm4lwi3bbi5wao1s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8pgvcm4lwi3bbi5wao1s.png" alt="Virtual CVE Intelligence dashboard showing real-time stats: 224 Virtual CVEs Created, 36 Threats Detected in 24h, 2.3s average detection time, 87% AI confidence — SOC-CERT Guardian v1.0" width="660" height="682"></a></p>

<p>Set up a lightweight n8n workflow (or even a cron job) that queries the CISA KEV API daily for any CVE tagged with your key dependencies.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Example for Ruby/Rails — adapt the keywords to your own stack</span>
<span class="c1"># (Node.js → express, lodash / Python → django, flask / PHP → wordpress, laravel)</span>
<span class="n">task</span> <span class="ss">:check_kev</span> <span class="o">=&gt;</span> <span class="ss">:environment</span> <span class="k">do</span>
  <span class="n">response</span> <span class="o">=</span> <span class="no">HTTParty</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="s2">"https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json"</span><span class="p">)</span>
  <span class="n">kev</span> <span class="o">=</span> <span class="no">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="nf">body</span><span class="p">)[</span><span class="s2">"vulnerabilities"</span><span class="p">]</span>

  <span class="n">your_stack</span> <span class="o">=</span> <span class="p">[</span><span class="s2">"rails"</span><span class="p">,</span> <span class="s2">"rack"</span><span class="p">,</span> <span class="s2">"devise"</span><span class="p">,</span> <span class="s2">"activerecord"</span><span class="p">]</span> <span class="c1"># ← change this</span>

  <span class="n">relevant</span> <span class="o">=</span> <span class="n">kev</span><span class="p">.</span><span class="nf">select</span> <span class="k">do</span> <span class="o">|</span><span class="n">cve</span><span class="o">|</span>
    <span class="n">your_stack</span><span class="p">.</span><span class="nf">any?</span> <span class="p">{</span> <span class="o">|</span><span class="n">dep</span><span class="o">|</span> <span class="n">cve</span><span class="p">[</span><span class="s2">"product"</span><span class="p">].</span><span class="nf">downcase</span><span class="p">.</span><span class="nf">include?</span><span class="p">(</span><span class="n">dep</span><span class="p">)</span> <span class="p">}</span>
  <span class="k">end</span>

  <span class="n">relevant</span><span class="p">.</span><span class="nf">each</span> <span class="k">do</span> <span class="o">|</span><span class="n">cve</span><span class="o">|</span>
    <span class="no">SlackNotifier</span><span class="p">.</span><span class="nf">alert</span><span class="p">(</span><span class="s2">"🚨 KEV match: </span><span class="si">#{</span><span class="n">cve</span><span class="p">[</span><span class="s1">'cveID'</span><span class="p">]</span><span class="si">}</span><span class="s2"> affects </span><span class="si">#{</span><span class="n">cve</span><span class="p">[</span><span class="s1">'product'</span><span class="p">]</span><span class="si">}</span><span class="s2">"</span><span class="p">)</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p><strong>Pattern 2: Deduplicate your alerts</strong></p>

<p>If you're piping security alerts to Slack from multiple sources, implement the same hash-based dedup approach. Your team will thank you.</p>

<p><strong>Pattern 3: Treat AI as a structured data extractor, not an oracle</strong></p>

<p>When using AI to triage security alerts, give it a strict output schema. Vague prompts produce vague outputs. In security, vagueness costs you.</p>




<h2>
  
  
  The Stack — Full Transparency
</h2>

<p><strong>SOC-CERT uses entirely free tiers:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Tool</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Orchestration</td>
<td>n8n (self-hosted)</td>
<td>Free</td>
</tr>
<tr>
<td>Scraping</td>
<td>Bright Data</td>
<td>Free tier</td>
</tr>
<tr>
<td>AI scoring</td>
<td>Cohere Command-R</td>
<td>Free tier</td>
</tr>
<tr>
<td>Notifications</td>
<td>Slack + Gmail</td>
<td>Free</td>
</tr>
<tr>
<td>Logging</td>
<td>Google Sheets</td>
<td>Free</td>
</tr>
<tr>
<td>CVE Extension AI</td>
<td>Gemini Nano (Chrome)</td>
<td>Free</td>
</tr>
</tbody>
</table></div>

<p>Total infrastructure cost: <strong>$0/month.</strong></p>

<p>This matters because one of the biggest barriers to security tooling for small teams is cost. Enterprise threat intelligence platforms cost $50K+/year. SOC-CERT proves you don't need that.</p>




<h2>
  
  
  What I'd Do Differently
</h2>

<p>Honest retrospective:</p>

<p><strong>I'd implement MITRE ATT&amp;CK mapping from day one.</strong> Right now, SOC-CERT tells you a CVE exists and its severity. What it doesn't do well yet is map that to actual attacker TTPs — "here's how they'd exploit this in practice." That's the next level.</p>

<p><strong>I'd add time-window deduplication earlier.</strong> Hash-based dedup prevents the same CVE from being alerted twice in the same run. But if a CVE resurfaces 3 weeks later in a new source, it alerts again. Redis-based state tracking with TTL would fix this cleanly.</p>

<p><strong>I'd separate the enrichment pipeline from the alerting pipeline.</strong> Currently they're coupled in a single n8n workflow. Decoupling them would allow async enrichment without blocking alerts.</p>




<h2>
  
  
  The Bigger Picture
</h2>

<p>Three months, three challenges, one ecosystem:</p>

<ul>
<li>
<strong>August 2025:</strong> n8n pipeline — won the AI Agents Challenge</li>
<li>
<strong>September 2025:</strong> KendoReact dashboard — enterprise visualization layer</li>
<li>
<strong>October 2025:</strong> Chrome AI extension — proactive browser-level detection</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc60p9heqbfx3or035h6w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc60p9heqbfx3or035h6w.png" alt="SOC-CERT Dashboard built with KendoReact — showing real-time alerts table (Failed login attempt HIGH, Malware activity MEDIUM, Failed login attempt LOW), severity distribution donut chart (45% HIGH, 20% MEDIUM, 35% LOW), and AI Analyst panel" width="800" height="350"></a></p>

<p>Each product reused the same core intelligence layer. That's the real lesson: <strong>build systems, not features.</strong> A well-designed automation pipeline compounds in value every time you extend it.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li>🔗 SOC-CERT original article: <a href="https://dev.to/joupify/soc-cert-automated-threat-intelligence-system-with-n8n-ai-5722">dev.to/joupify/soc-cert-automated-threat-intelligence-system-with-n8n-ai-5722</a>
</li>
<li>🔗 Chrome Extension: <a href="https://github.com/joupify/soc-cert-guardian-extension" rel="noopener noreferrer">github.com/joupify/soc-cert-guardian-extension</a>
</li>
<li>🔗 CISA KEV Catalog API: <a href="https://www.cisa.gov/known-exploited-vulnerabilities-catalog" rel="noopener noreferrer">cisa.gov/known-exploited-vulnerabilities-catalog</a>
</li>
<li>🔗 n8n docs: <a href="https://docs.n8n.io" rel="noopener noreferrer">docs.n8n.io</a>
</li>
</ul>




<p>If you're a developer or small team thinking about security automation — and you can't afford enterprise tooling — I hope this gives you a concrete starting point.</p>

<p>Drop your questions in the comments. Always happy to dig into implementation details. 🛡️</p>

<p><em>— Malika | Full-stack Rails dev, Paris | <a href="https://github.com/joupify" rel="noopener noreferrer">github.com/joupify</a></em></p>

