---
Title: Built a Self-Hostable Plausible + Sentry Alternative in One Day
Description: 
Author: Om Dongaonkar
Date: 2026-02-27T21:29:46.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  I Built a Self-Hostable Plausible + Sentry Alternative in One Day - That Runs on Shared Hosting
</h1>

<p>I worked on projects that run on shared hosting. PHP + MySQL, no root access. It's a real environment that a lot of developers actually ship to - and almost no tooling is built for it.</p>

<p>But every analytics or error tracking tool I looked at assumed you had a VPS at minimum, or were happy paying a SaaS bill every month. Plausible is great - but it's paid per site, and self-hosting it means Docker, which means a VPS. Sentry's free tier is generous until it isn't. And there was a hard requirement: no third-party services touching user data.</p>

<p>So I built one. In a day. It's called <strong>Micrologs</strong>.</p>




<h2>
  
  
  What it does
</h2>

<p>Drop one script tag on your site:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;script
  </span><span class="na">src=</span><span class="s">"https://yourdomain.com/snippet/micrologs.js"</span>
  <span class="na">data-public-key=</span><span class="s">"your_public_key"</span>
  <span class="na">data-environment=</span><span class="s">"production"</span>
  <span class="na">async</span><span class="nt">&gt;</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>



<p>From that point, you get:</p>

<ul>
<li>Pageviews, sessions, unique visitors, bounce rate</li>
<li>Country / region / city breakdown (via local MaxMind GeoLite2 — no runtime API calls)</li>
<li>Device, OS, browser breakdown</li>
<li>Referrer categorization — organic, social, email, referral, direct</li>
<li>UTM campaign tracking</li>
<li>JS errors auto-caught — <code>window.onerror</code> and <code>unhandledrejection</code>, grouped by fingerprint</li>
<li>Manual error tracking from any backend over a single HTTP call</li>
<li>Audit logging</li>
<li>Tracked link shortener with click analytics</li>
</ul>

<p>All of it hits your own database. Nothing leaves your server.</p>




<h2>
  
  
  The constraint that shaped everything
</h2>

<p>Shared hosting means no Redis, no background workers, no daemons, no WebSockets. You get PHP and MySQL and that's it.</p>

<p>This forced some interesting decisions.</p>

<p><strong>Rate limiting without Redis.</strong> Most rate limiters use Redis or a DB table with a timestamp. I went file-based — append-only <code>.req</code> files, one per request, inside a per-IP directory. Counting recent requests = counting files newer than the window. No locking, no DB writes on every request, works on any host.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// Count recent attempts — each file is one request</span>
<span class="k">foreach</span> <span class="p">(</span><span class="nb">glob</span><span class="p">(</span><span class="nv">$userDir</span> <span class="mf">.</span> <span class="s2">"/*.req"</span><span class="p">)</span> <span class="k">as</span> <span class="nv">$file</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if</span> <span class="p">(</span><span class="nv">$now</span> <span class="o">-</span> <span class="nb">filemtime</span><span class="p">(</span><span class="nv">$file</span><span class="p">)</span> <span class="o">&lt;=</span> <span class="nv">$windowSeconds</span><span class="p">)</span> <span class="p">{</span>
        <span class="nv">$attempts</span><span class="o">++</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="o">@</span><span class="nb">unlink</span><span class="p">(</span><span class="nv">$file</span><span class="p">);</span> <span class="c1">// clean up expired ones while we're here</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Cleanup runs probabilistically — 1% chance on each request. No cron job needed.</p>

<p><strong>Geolocation without API calls.</strong> MaxMind's GeoLite2 database ships as a <code>.mmdb</code> file you drop on your server. Every lookup is local. Zero latency overhead, zero external dependency at runtime.</p>

<p><strong>Visitor identification without being creepy.</strong> No raw IPs stored, ever. IPs are SHA-256 hashed with a salt immediately on ingestion. For visitor tracking, a UUID is stored in a cookie for 365 days. If the cookie gets cleared, a canvas fingerprint kicks in as a fallback to re-identify the same visitor — then re-associates the new cookie hash so the next visit is seamless again.</p>




<h2>
  
  
  The architecture is designed in stages
</h2>

<p>v1 is a clean REST API — works on shared hosting, zero extra infrastructure.</p>

<p>v2 will add Redis caching, async queuing, and webhook alerts — opt-in for people who have a VPS.</p>

<p>v3 will add WebSockets and a live dashboard feed — opt-in for people who want realtime.</p>

<p>The key decision: each stage is strictly opt-in. Shared hosting users will never be broken by what VPS users unlock. v1 will always work on v1 infrastructure.</p>




<h2>
  
  
  Some implementation details worth talking about
</h2>

<p><strong>Error grouping by fingerprint.</strong> This is how Sentry works and it's the right approach. When an error comes in, I hash <code>project_id + error_type + message + file + line</code> into a SHA-256 fingerprint. Same error fires 1000 times — 1 group, 1000 occurrences. If you mark an error as resolved and it fires again, it automatically reopens.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$fingerprint</span> <span class="o">=</span> <span class="nb">hash</span><span class="p">(</span>
    <span class="s2">"sha256"</span><span class="p">,</span>
    <span class="nv">$projectId</span> <span class="mf">.</span> <span class="nv">$errorType</span> <span class="mf">.</span> <span class="nv">$message</span> <span class="mf">.</span> <span class="nv">$file</span> <span class="mf">.</span> <span class="p">(</span><span class="nv">$line</span> <span class="o">??</span> <span class="s2">""</span><span class="p">)</span>
<span class="p">);</span>
</code></pre>

</div>



<p><strong>Public key vs secret key separation.</strong> The JS snippet uses a public key — safe to expose in the browser, locked to a whitelist of allowed domains. Analytics queries and link management use a secret key — server-side only, never in frontend code. One install supports multiple projects, each with their own key pair.</p>

<p><strong>SPA support in the snippet.</strong> History-based routing doesn't trigger page loads, so <code>pushState</code> and <code>replaceState</code> get patched to fire pageviews on navigation. <code>popstate</code> and <code>hashchange</code> are also handled.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">history</span><span class="p">.</span><span class="nx">pushState</span> <span class="o">=</span> <span class="nf">function </span><span class="p">()</span> <span class="p">{</span>
    <span class="nx">_push</span><span class="p">.</span><span class="nf">apply</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="nx">arguments</span><span class="p">);</span>
    <span class="nf">onUrlChange</span><span class="p">();</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>Bot filtering.</strong> UA string matching is obvious, but real browsers always send <code>Accept-Language</code> and <code>Accept</code> headers. If those are missing, it's a bot or a script regardless of what the UA says.</p>




<h2>
  
  
  What the API looks like
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl https://yourdomain.com/api/analytics/visitors.php?range<span class="o">=</span>30d <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-API-Key: your_secret_key"</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"success"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"unique_visitors"</span><span class="p">:</span><span class="w"> </span><span class="mi">1842</span><span class="p">,</span><span class="w">
    </span><span class="nl">"total_pageviews"</span><span class="p">:</span><span class="w"> </span><span class="mi">5631</span><span class="p">,</span><span class="w">
    </span><span class="nl">"total_sessions"</span><span class="p">:</span><span class="w"> </span><span class="mi">2109</span><span class="p">,</span><span class="w">
    </span><span class="nl">"bounce_rate"</span><span class="p">:</span><span class="w"> </span><span class="mf">43.2</span><span class="p">,</span><span class="w">
    </span><span class="nl">"over_time"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"date"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-01-28"</span><span class="p">,</span><span class="w"> </span><span class="nl">"pageviews"</span><span class="p">:</span><span class="w"> </span><span class="mi">178</span><span class="p">,</span><span class="w"> </span><span class="nl">"unique_visitors"</span><span class="p">:</span><span class="w"> </span><span class="mi">91</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"date"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-01-29"</span><span class="p">,</span><span class="w"> </span><span class="nl">"pageviews"</span><span class="p">:</span><span class="w"> </span><span class="mi">204</span><span class="p">,</span><span class="w"> </span><span class="nl">"unique_visitors"</span><span class="p">:</span><span class="w"> </span><span class="mi">113</span><span class="w"> </span><span class="p">}</span><span class="w">
    </span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>It's an engine, not a dashboard. The data comes back as JSON — what you do with it is up to you. Build a dashboard, pipe it into Grafana, query it from your admin panel, whatever fits your stack.</p>




<h2>
  
  
  Stack
</h2>

<ul>
<li>PHP 8.1+</li>
<li>MySQL 8.0+ / MariaDB 10.4+</li>
<li>MaxMind GeoLite2 (local <code>.mmdb</code> file)</li>
<li>Vanilla JS snippet, zero dependencies, ~3KB</li>
</ul>

<p>No Node, no Docker, no Redis, no build step. Clone it, import the schema, fill in the env file, drop the snippet. That's the entire setup.</p>




<h2>
  
  
  It's already in production
</h2>

<p>We shipped it internally at the company the same day I built it. It's tracking real traffic right now. That's the other reason for the shared hosting constraint — it wasn't a hypothetical, it was the actual target environment.</p>




<h2>
  
  
  Open source, MIT
</h2>

<p>GitHub: <a href="https://github.com/OmDongaonkar03/Micrologs" rel="noopener noreferrer">github.com/OmDongaonkar03/Micrologs</a></p>

<p>If you're on shared hosting, running a privacy-first product, or just tired of paying for tools you could own — give it a try. Issues and PRs are open.</p>

<p>v2 is next. Webhooks for error alerts, Redis caching as opt-in, async queuing. If you have thoughts on what matters most, open an issue.</p>

