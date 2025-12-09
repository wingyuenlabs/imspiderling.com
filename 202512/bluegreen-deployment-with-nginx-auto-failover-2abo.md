---
Title: Blue/Green Deployment with Nginx Auto-Failover
Description: 
Author: Sherifdeen Adebayo
Date: 2025-12-09T21:16:01.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  What is Blue/Green Deployment?
</h3>

<p>Imagine you're running a restaurant. You have two identical kitchens: Kitchen Blue (currently serving customers) and Kitchen Green (on standby). When you want to update the menu or equipment, you:</p>

<ol>
<li>Update Kitchen Green while Kitchen Blue serves customers</li>
<li>Test Kitchen Green thoroughly</li>
<li>Switch all orders to Kitchen Green</li>
<li>Now Kitchen Blue is on standby for the next update</li>
</ol>

<p>That's exactly how blue/green deployment works in software! You maintain two identical production environments and can switch between them instantly.</p>

<h3>
  
  
  Why Use This Approach?
</h3>

<p><strong>Traditional deployment problems:</strong></p>

<ul>
<li>Downtime during updates (users see "Site under maintenance")</li>
<li>No easy rollback if something breaks</li>
<li>Risk of breaking production with untested changes</li>
</ul>

<p><strong>Blue/Green deployment benefits:</strong></p>

<ul>
<li>Zero downtime - users never notice the switch</li>
<li>Instant rollback - just switch back if issues arise</li>
<li>Safe testing - new version runs in production environment before switch</li>
</ul>

<h3>
  
  
  Project Architecture
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                          ┌─────────────┐
                          │   Nginx     │
                          │   :8080     │
                          └──────┬──────┘
                                 │
          ┌──────────────────────┴──────────────────────┐
          │                                              │
┌─────────▼──────────┐                        ┌─────────▼──────────┐
│   Blue (Primary)   │                        │  Green (Backup)    │
│      :8081         │                        │      :8082         │
└────────────────────┘                        └────────────────────┘
</code></pre>

</div>



<p><strong>How it works:</strong></p>

<ol>
<li>
<strong>Nginx</strong> sits at port 8080 (your main entry point)</li>
<li>
<strong>Blue service</strong> at port 8081 (currently handling all traffic)</li>
<li>
<strong>Green service</strong> at port 8082 (backup, ready to take over)</li>
<li>When Blue fails, Nginx automatically routes traffic to Green</li>
<li>No requests are dropped during the transition</li>
</ol>

<h3>
  
  
  Setting Up the Project
</h3>

<h4>
  
  
  Step 1: Project Structure
</h4>

<p>Create your project directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>blue-green-deployment
<span class="nb">cd </span>blue-green-deployment
</code></pre>

</div>



<p>Your final structure will look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>blue-green-deployment/
├── docker-compose.yml      # Orchestrates all services
├── nginx.conf.template     # Nginx configuration
├── entrypoint.sh          # Nginx startup script
├── env.example            # Environment variables template
├── .env                   # Your actual environment variables
├── Makefile               # Convenient commands
├── test-failover.sh       # Automated testing script
├── README.md              # Documentation
├── DECISION.md            # Technical decisions
└── PART_B_RESEARCH.md     # Infrastructure research
</code></pre>

</div>



<h4>
  
  
  Step 2: Docker Compose Configuration
</h4>

<p>The <code>docker-compose.yml</code> file is the heart of this setup. Let's break it down:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="c1"># Nginx acts as the load balancer with automatic failover</span>
  <span class="na">nginx</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">nginx:alpine</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">nginx-lb</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8080:80"</span>           <span class="c1"># Main entry point for users</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">./nginx.conf.template:/etc/nginx/nginx.conf:ro</span>
      <span class="pi">-</span> <span class="s">./entrypoint.sh:/entrypoint.sh:ro</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">ACTIVE_POOL=${ACTIVE_POOL:-blue}</span>    <span class="c1"># Which pool is primary?</span>
      <span class="pi">-</span> <span class="s">BLUE_UPSTREAM=app_blue:${PORT:-3000}</span>
      <span class="pi">-</span> <span class="s">GREEN_UPSTREAM=app_green:${PORT:-3000}</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app_blue</span>
      <span class="pi">-</span> <span class="s">app_green</span>
    <span class="na">entrypoint</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">/bin/sh"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">/entrypoint.sh"</span><span class="pi">]</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Blue pool - primary by default</span>
  <span class="na">app_blue</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">${BLUE_IMAGE}</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">app_blue</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8081:${PORT:-3000}"</span>    <span class="c1"># Direct access for testing</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">APP_POOL=blue</span>
      <span class="pi">-</span> <span class="s">RELEASE_ID=${RELEASE_ID_BLUE:-v1.0.0-blue}</span>
      <span class="pi">-</span> <span class="s">PORT=${PORT:-3000}</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">wget"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--quiet"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--tries=1"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--spider"</span><span class="pi">,</span>
             <span class="s2">"</span><span class="s">http://localhost:${PORT:-3000}/healthz"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">5s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">3s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">3</span>

  <span class="c1"># Green pool - backup by default</span>
  <span class="na">app_green</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">${GREEN_IMAGE}</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">app_green</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8082:${PORT:-3000}"</span>    <span class="c1"># Direct access for testing</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">APP_POOL=green</span>
      <span class="pi">-</span> <span class="s">RELEASE_ID=${RELEASE_ID_GREEN:-v1.0.0-green}</span>
      <span class="pi">-</span> <span class="s">PORT=${PORT:-3000}</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">wget"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--quiet"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--tries=1"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--spider"</span><span class="pi">,</span>
             <span class="s2">"</span><span class="s">http://localhost:${PORT:-3000}/healthz"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">5s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">3s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">3</span>

<span class="na">networks</span><span class="pi">:</span>
  <span class="na">app-network</span><span class="pi">:</span>
    <span class="na">driver</span><span class="pi">:</span> <span class="s">bridge</span>
</code></pre>

</div>



<p><strong>Key points to understand:</strong></p>

<ol>
<li>
<p><strong>Port Strategy:</strong></p>

<ul>
<li>Port 8080: Public-facing Nginx (users hit this)</li>
<li>Port 8081: Direct access to Blue (for chaos testing)</li>
<li>Port 8082: Direct access to Green (for chaos testing)</li>
</ul>
</li>
<li>
<p><strong>Health Checks:</strong></p>

<ul>
<li>Every 5 seconds, Docker checks if the service is healthy</li>
<li>Uses the <code>/healthz</code> endpoint</li>
<li>After 3 failed attempts (3s timeout each), marks as unhealthy</li>
</ul>
</li>
<li>
<p><strong>Environment Variables:</strong></p>

<ul>
<li>
<code>ACTIVE_POOL</code>: Which service gets traffic first (blue/green)</li>
<li>
<code>RELEASE_ID</code>: Tracks which version is running</li>
<li>
<code>PORT</code>: Application port (default: 3000)</li>
</ul>
</li>
</ol>

<h4>
  
  
  Step 3: Nginx Configuration Magic
</h4>

<p>The <code>nginx.conf.template</code> is where the failover magic happens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">events</span> <span class="p">{</span>
    <span class="kn">worker_connections</span> <span class="mi">1024</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">http</span> <span class="p">{</span>
    <span class="c1"># Logging to help debug issues</span>
    <span class="kn">access_log</span> <span class="n">/var/log/nginx/access.log</span><span class="p">;</span>
    <span class="kn">error_log</span> <span class="n">/var/log/nginx/error.log</span> <span class="s">warn</span><span class="p">;</span>

    <span class="c1"># Combined upstream with failover logic</span>
    <span class="kn">upstream</span> <span class="s">app_backend</span> <span class="p">{</span>
        <span class="c1"># Active pool (primary)</span>
        <span class="kn">server</span> $<span class="p">{</span><span class="kn">ACTIVE_UPSTREAM</span><span class="err">}</span> <span class="s">max_fails=2</span> <span class="s">fail_timeout=5s</span><span class="p">;</span>

        <span class="c1"># Backup pool - only used if primary fails</span>
        <span class="kn">server</span> $<span class="p">{</span><span class="kn">BACKUP_UPSTREAM</span><span class="err">}</span> <span class="s">max_fails=2</span> <span class="s">fail_timeout=5s</span> <span class="s">backup</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="kn">server</span> <span class="p">{</span>
        <span class="kn">listen</span> <span class="mi">80</span><span class="p">;</span>
        <span class="kn">server_name</span> <span class="s">localhost</span><span class="p">;</span>

        <span class="c1"># Aggressive timeouts for quick failover detection</span>
        <span class="kn">proxy_connect_timeout</span> <span class="s">2s</span><span class="p">;</span>
        <span class="kn">proxy_send_timeout</span> <span class="s">3s</span><span class="p">;</span>
        <span class="kn">proxy_read_timeout</span> <span class="s">3s</span><span class="p">;</span>

        <span class="c1"># Retry logic - crucial for zero-downtime failover</span>
        <span class="kn">proxy_next_upstream</span> <span class="s">error</span> <span class="s">timeout</span> <span class="s">http_500</span> <span class="s">http_502</span> <span class="s">http_503</span> <span class="s">http_504</span><span class="p">;</span>
        <span class="kn">proxy_next_upstream_tries</span> <span class="mi">2</span><span class="p">;</span>
        <span class="kn">proxy_next_upstream_timeout</span> <span class="s">10s</span><span class="p">;</span>

        <span class="c1"># Don't buffer - we want real-time responses</span>
        <span class="kn">proxy_buffering</span> <span class="no">off</span><span class="p">;</span>

        <span class="c1"># Forward original client info</span>
        <span class="kn">proxy_set_header</span> <span class="s">Host</span> <span class="nv">$host</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">X-Real-IP</span> <span class="nv">$remote_addr</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">X-Forwarded-For</span> <span class="nv">$proxy_add_x_forwarded_for</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">X-Forwarded-Proto</span> <span class="nv">$scheme</span><span class="p">;</span>

        <span class="kn">location</span> <span class="n">/</span> <span class="p">{</span>
            <span class="kn">proxy_pass</span> <span class="s">http://app_backend</span><span class="p">;</span>
            <span class="kn">proxy_pass_request_headers</span> <span class="no">on</span><span class="p">;</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Let's decode this configuration:</strong></p>

<p><strong>1. Upstream Block:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">upstream</span> <span class="s">app_backend</span> <span class="p">{</span>
    <span class="kn">server</span> $<span class="p">{</span><span class="kn">ACTIVE_UPSTREAM</span><span class="err">}</span> <span class="s">max_fails=2</span> <span class="s">fail_timeout=5s</span><span class="p">;</span>
    <span class="kn">server</span> $<span class="p">{</span><span class="kn">BACKUP_UPSTREAM</span><span class="err">}</span> <span class="s">max_fails=2</span> <span class="s">fail_timeout=5s</span> <span class="s">backup</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<ul>
<li>
<code>max_fails=2</code>: After 2 failed requests, mark server as down</li>
<li>
<code>fail_timeout=5s</code>: Server is down for 5 seconds before retry</li>
<li>
<code>backup</code>: This server only receives traffic when primary fails</li>
</ul>

<p><strong>2. Timeout Settings:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">proxy_connect_timeout</span> <span class="s">2s</span><span class="p">;</span>    <span class="c1"># Can't connect? Fail fast</span>
<span class="k">proxy_read_timeout</span> <span class="s">3s</span><span class="p">;</span>       <span class="c1"># No response? Move to backup</span>
</code></pre>

</div>



<p>These are aggressive timeouts for quick failure detection. In production, you might increase these based on your app's response times.</p>

<p><strong>3. Retry Logic:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">proxy_next_upstream</span> <span class="s">error</span> <span class="s">timeout</span> <span class="s">http_500</span> <span class="s">http_502</span> <span class="s">http_503</span> <span class="s">http_504</span><span class="p">;</span>
<span class="k">proxy_next_upstream_tries</span> <span class="mi">2</span><span class="p">;</span>
</code></pre>

</div>



<p>This tells Nginx: "If you get an error, timeout, or 5xx status code, try the next server (backup) automatically."</p>

<h4>
  
  
  Step 4: The Entrypoint Script
</h4>

<p>Nginx doesn't support environment variables natively in its config. We use <code>envsubst</code> to template it at runtime:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/sh</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="c"># Figure out which upstream is active and which is backup</span>
<span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$ACTIVE_POOL</span><span class="s2">"</span> <span class="o">=</span> <span class="s2">"blue"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nv">ACTIVE_UPSTREAM</span><span class="o">=</span><span class="s2">"</span><span class="nv">$BLUE_UPSTREAM</span><span class="s2">"</span>
    <span class="nv">BACKUP_UPSTREAM</span><span class="o">=</span><span class="s2">"</span><span class="nv">$GREEN_UPSTREAM</span><span class="s2">"</span>
<span class="k">else
    </span><span class="nv">ACTIVE_UPSTREAM</span><span class="o">=</span><span class="s2">"</span><span class="nv">$GREEN_UPSTREAM</span><span class="s2">"</span>
    <span class="nv">BACKUP_UPSTREAM</span><span class="o">=</span><span class="s2">"</span><span class="nv">$BLUE_UPSTREAM</span><span class="s2">"</span>
<span class="k">fi

</span><span class="nb">echo</span> <span class="s2">"==&gt; Setting up Nginx with active pool: </span><span class="nv">$ACTIVE_POOL</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"    Active upstream: </span><span class="nv">$ACTIVE_UPSTREAM</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"    Backup upstream: </span><span class="nv">$BACKUP_UPSTREAM</span><span class="s2">"</span>

<span class="c"># Use envsubst to replace variables in the template</span>
<span class="nb">export </span>ACTIVE_UPSTREAM
<span class="nb">export </span>BACKUP_UPSTREAM

envsubst <span class="s1">'${ACTIVE_UPSTREAM} ${BACKUP_UPSTREAM}'</span> <span class="se">\</span>
    &lt; /etc/nginx/nginx.conf <span class="o">&gt;</span> /tmp/nginx.conf

<span class="c"># Test the config before starting (ALWAYS do this!)</span>
nginx <span class="nt">-t</span> <span class="nt">-c</span> /tmp/nginx.conf

<span class="c"># Start nginx in foreground</span>
<span class="nb">echo</span> <span class="s2">"==&gt; Starting Nginx..."</span>
<span class="nb">exec </span>nginx <span class="nt">-g</span> <span class="s1">'daemon off;'</span> <span class="nt">-c</span> /tmp/nginx.conf
</code></pre>

</div>



<p><strong>What this script does:</strong></p>

<ol>
<li>Reads the <code>ACTIVE_POOL</code> environment variable</li>
<li>Sets up which upstream is active vs backup</li>
<li>Replaces placeholders in nginx.conf</li>
<li>Tests the configuration (prevents broken configs from starting)</li>
<li>Starts Nginx with the processed config</li>
</ol>

<h4>
  
  
  Step 5: Environment Configuration
</h4>

<p>Create your <code>.env</code> file from the example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cp </span>env.example .env
</code></pre>

</div>



<p>The <code>.env</code> file contents:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Docker images for blue and green pools</span>
<span class="nv">BLUE_IMAGE</span><span class="o">=</span>your-registry/your-app:blue
<span class="nv">GREEN_IMAGE</span><span class="o">=</span>your-registry/your-app:green

<span class="c"># Which pool should be active? (blue or green)</span>
<span class="nv">ACTIVE_POOL</span><span class="o">=</span>blue

<span class="c"># Release identifiers - show up in X-Release-Id header</span>
<span class="nv">RELEASE_ID_BLUE</span><span class="o">=</span>v1.0.0-blue-20250129
<span class="nv">RELEASE_ID_GREEN</span><span class="o">=</span>v1.0.0-green-20250129

<span class="c"># Application port</span>
<span class="nv">PORT</span><span class="o">=</span>3000
</code></pre>

</div>



<p><strong>Pro tip:</strong> In a real deployment, <code>BLUE_IMAGE</code> and <code>GREEN_IMAGE</code> might point to different versions of your app:</p>

<ul>
<li>Blue: <code>myapp:v1.2.3</code>
</li>
<li>Green: <code>myapp:v1.2.4</code> (new version being tested)</li>
</ul>

<h4>
  
  
  Step 6: Makefile for Convenience
</h4>

<p>The <code>Makefile</code> provides friendly commands:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight make"><code><span class="nl">.PHONY</span><span class="o">:</span> <span class="nf">help up down restart logs test clean</span>

<span class="nl">help</span><span class="o">:</span>  <span class="c">##</span><span class="nf"> Show available commands</span>
    <span class="p">@</span><span class="nb">grep</span> <span class="nt">-E</span> <span class="s1">'^[a-zA-Z_-]+:.*?## .*$$'</span> <span class="p">$(</span>MAKEFILE_LIST<span class="p">)</span> | <span class="se">\</span>
    <span class="nb">awk</span> <span class="s1">'BEGIN {FS = ":.*?## "}; {printf "  %-15s %s\n", $$1, $$2}'</span>

<span class="nl">up</span><span class="o">:</span>  <span class="c">##</span><span class="nf"> Start all services</span>
    <span class="p">@</span><span class="k">if</span> <span class="o">[</span> <span class="o">!</span> <span class="nt">-f</span> .env <span class="o">]</span><span class="p">;</span> <span class="k">then</span> <span class="se">\</span>
        <span class="nb">echo</span> <span class="s2">"Creating .env from env.example..."</span><span class="p">;</span> <span class="se">\</span>
        <span class="nb">cp </span>env.example .env<span class="p">;</span> <span class="se">\</span>
    <span class="k">fi</span>
    docker-compose up <span class="nt">-d</span>
    <span class="p">@</span><span class="nb">echo</span> <span class="s2">"Services started!"</span>
    <span class="p">@</span><span class="nb">echo</span> <span class="s2">"Nginx:  http://localhost:8080"</span>
    <span class="p">@</span><span class="nb">echo</span> <span class="s2">"Blue:   http://localhost:8081"</span>
    <span class="p">@</span><span class="nb">echo</span> <span class="s2">"Green:  http://localhost:8082"</span>

<span class="nl">down</span><span class="o">:</span>  <span class="c">##</span><span class="nf"> Stop all services</span>
    docker-compose down

<span class="nl">test</span><span class="o">:</span>  <span class="c">##</span><span class="nf"> Run failover test</span>
    ./test-failover.sh

<span class="nl">blue</span><span class="o">:</span>  <span class="c">##</span><span class="nf"> Switch to blue as active</span>
    <span class="p">@</span><span class="nb">sed</span> <span class="nt">-i</span>.bak <span class="s1">'s/ACTIVE_POOL=.*/ACTIVE_POOL=blue/'</span> .env
    <span class="p">@$(</span>MAKE<span class="p">)</span> restart

<span class="nl">green</span><span class="o">:</span>  <span class="c">##</span><span class="nf"> Switch to green as active</span>
    <span class="p">@</span><span class="nb">sed</span> <span class="nt">-i</span>.bak <span class="s1">'s/ACTIVE_POOL=.*/ACTIVE_POOL=green/'</span> .env
    <span class="p">@$(</span>MAKE<span class="p">)</span> restart
</code></pre>

</div>



<p><strong>Usage:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>make <span class="nb">help</span>      <span class="c"># See all commands</span>
make up        <span class="c"># Start everything</span>
make <span class="nb">test</span>      <span class="c"># Run failover tests</span>
make green     <span class="c"># Switch to green pool</span>
</code></pre>

</div>



<h3>
  
  
  Running the Project
</h3>

<h4>
  
  
  Start the Stack
</h4>

<p><strong>Option 1: Using Make (recommended)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>make up
</code></pre>

</div>



<p><strong>Option 2: Manual</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cp </span>env.example .env
docker-compose up <span class="nt">-d</span>
</code></pre>

</div>



<h4>
  
  
  Verify It's Working
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check service status</span>
docker-compose ps

<span class="c"># Hit the main endpoint</span>
curl <span class="nt">-i</span> http://localhost:8080/version
</code></pre>

</div>



<p>You should see headers like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>HTTP/1.1 200 OK
X-App-Pool: blue
X-Release-Id: v1.0.0-blue-20250129
</code></pre>

</div>



<h3>
  
  
  Testing the Failover
</h3>

<p>This is where it gets exciting! Let's break the blue service and watch Nginx automatically switch to green.</p>

<h4>
  
  
  Automated Test Script
</h4>

<p>The <code>test-failover.sh</code> script automates the entire test:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="nb">echo</span> <span class="s2">"🔵 Blue/Green Failover Test"</span>
<span class="nb">echo</span> <span class="s2">"============================"</span>
<span class="nb">echo</span> <span class="s2">""</span>

<span class="c"># Step 1: Baseline check</span>
<span class="nb">echo</span> <span class="s2">"📊 Step 1: Checking baseline (should be blue)..."</span>
<span class="k">for </span>i <span class="k">in</span> <span class="o">{</span>1..3<span class="o">}</span><span class="p">;</span> <span class="k">do
    </span><span class="nv">response</span><span class="o">=</span><span class="si">$(</span>curl <span class="nt">-s</span> <span class="nt">-i</span> http://localhost:8080/version<span class="si">)</span>
    <span class="nv">pool</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$response</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-i</span> <span class="s2">"X-App-Pool:"</span> | <span class="nb">cut</span> <span class="nt">-d</span>: <span class="nt">-f2</span> | <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">' \r'</span><span class="si">)</span>
    <span class="nb">echo</span> <span class="s2">"  Request </span><span class="nv">$i</span><span class="s2">: Pool=</span><span class="nv">$pool</span><span class="s2">"</span>
<span class="k">done</span>

<span class="c"># Step 2: Trigger chaos on blue</span>
<span class="nb">echo</span> <span class="s2">"💥 Step 2: Triggering chaos on blue..."</span>
curl <span class="nt">-s</span> <span class="nt">-X</span> POST <span class="s2">"http://localhost:8081/chaos/start?mode=error"</span>
<span class="nb">sleep </span>1

<span class="c"># Step 3: Test failover</span>
<span class="nb">echo</span> <span class="s2">"🔄 Step 3: Testing failover (should switch to green)..."</span>
<span class="nv">success_count</span><span class="o">=</span>0
<span class="nv">green_count</span><span class="o">=</span>0
<span class="k">for </span>i <span class="k">in</span> <span class="o">{</span>1..10<span class="o">}</span><span class="p">;</span> <span class="k">do
    </span><span class="nv">http_code</span><span class="o">=</span><span class="si">$(</span>curl <span class="nt">-s</span> <span class="nt">-o</span> /dev/null <span class="nt">-w</span> <span class="s2">"%{http_code}"</span> http://localhost:8080/version<span class="si">)</span>
    <span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$http_code</span><span class="s2">"</span> <span class="o">=</span> <span class="s2">"200"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
        </span><span class="nv">success_count</span><span class="o">=</span><span class="k">$((</span>success_count <span class="o">+</span> <span class="m">1</span><span class="k">))</span>
        <span class="nv">response</span><span class="o">=</span><span class="si">$(</span>curl <span class="nt">-s</span> <span class="nt">-i</span> http://localhost:8080/version<span class="si">)</span>
        <span class="nv">pool</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$response</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-i</span> <span class="s2">"X-App-Pool:"</span> | <span class="nb">cut</span> <span class="nt">-d</span>: <span class="nt">-f2</span> | <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">' \r'</span><span class="si">)</span>
        <span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$pool</span><span class="s2">"</span> <span class="o">=</span> <span class="s2">"green"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
            </span><span class="nv">green_count</span><span class="o">=</span><span class="k">$((</span>green_count <span class="o">+</span> <span class="m">1</span><span class="k">))</span>
        <span class="k">fi
        </span><span class="nb">echo</span> <span class="s2">"  Request </span><span class="nv">$i</span><span class="s2">: HTTP </span><span class="nv">$http_code</span><span class="s2">, Pool=</span><span class="nv">$pool</span><span class="s2"> ✓"</span>
    <span class="k">else
        </span><span class="nb">echo</span> <span class="s2">"  Request </span><span class="nv">$i</span><span class="s2">: HTTP </span><span class="nv">$http_code</span><span class="s2"> ✗ FAILED"</span>
    <span class="k">fi
    </span><span class="nb">sleep </span>0.5
<span class="k">done</span>

<span class="c"># Step 4: Stop chaos</span>
<span class="nb">echo</span> <span class="s2">"🛑 Step 4: Stopping chaos..."</span>
curl <span class="nt">-s</span> <span class="nt">-X</span> POST <span class="s2">"http://localhost:8081/chaos/stop"</span>

<span class="c"># Results</span>
<span class="nb">echo</span> <span class="s2">"📈 Results:"</span>
<span class="nb">echo</span> <span class="s2">"  ├─ Total requests: 10"</span>
<span class="nb">echo</span> <span class="s2">"  ├─ Successful (200): </span><span class="nv">$success_count</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"  └─ Routed to green: </span><span class="nv">$green_count</span><span class="s2">"</span>

<span class="k">if</span> <span class="o">[</span> <span class="nv">$success_count</span> <span class="nt">-eq</span> 10 <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="o">[</span> <span class="nv">$green_count</span> <span class="nt">-ge</span> 9 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"✅ Test PASSED - Failover working correctly!"</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"❌ Test FAILED - Check the logs"</span>
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>Run the test:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">chmod</span> +x test-failover.sh
./test-failover.sh
</code></pre>

</div>



<p><strong>Expected output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🔵 Blue/Green Failover Test
============================

📊 Step 1: Checking baseline (should be blue)...
  Request 1: Pool=blue
  Request 2: Pool=blue
  Request 3: Pool=blue

💥 Step 2: Triggering chaos on blue...
  Chaos initiated: {"status":"chaos_started","mode":"error"}

🔄 Step 3: Testing failover (should switch to green)...
  Request 1: HTTP 200, Pool=green ✓
  Request 2: HTTP 200, Pool=green ✓
  Request 3: HTTP 200, Pool=green ✓
  ...
  Request 10: HTTP 200, Pool=green ✓

📈 Results:
  ├─ Total requests: 10
  ├─ Successful (200): 10
  └─ Routed to green: 10

✅ Test PASSED - Failover working correctly!
</code></pre>

</div>



<p><strong>What just happened?</strong></p>

<ol>
<li>All requests initially went to blue</li>
<li>We triggered chaos mode (blue starts returning 500 errors)</li>
<li>Nginx detected blue was failing</li>
<li>
<strong>Zero requests failed</strong> - Nginx automatically retried on green</li>
<li>All subsequent requests went to green</li>
</ol>

<h3>
  
  
  Manual Testing (Understanding Each Step)
</h3>

<p>Let's do it manually to understand what's happening:</p>

<p><strong>1. Check baseline:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">for </span>i <span class="k">in</span> <span class="o">{</span>1..5<span class="o">}</span><span class="p">;</span> <span class="k">do
  </span>curl <span class="nt">-s</span> http://localhost:8080/version | <span class="nb">grep</span> <span class="nt">-E</span> <span class="s2">"pool|release"</span>
<span class="k">done</span>
</code></pre>

</div>



<p>All responses show <code>"pool": "blue"</code>.</p>

<p><strong>2. Break blue service:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Trigger chaos mode - makes blue return 500 errors</span>
curl <span class="nt">-X</span> POST <span class="s2">"http://localhost:8081/chaos/start?mode=error"</span>
</code></pre>

</div>



<p><strong>3. Watch the magic:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Keep hitting the endpoint</span>
<span class="k">for </span>i <span class="k">in</span> <span class="o">{</span>1..10<span class="o">}</span><span class="p">;</span> <span class="k">do
  </span>curl <span class="nt">-s</span> <span class="nt">-w</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">Status: %{http_code}</span><span class="se">\n</span><span class="s2">"</span> http://localhost:8080/version | <span class="se">\</span>
    <span class="nb">grep</span> <span class="nt">-E</span> <span class="s2">"pool|release|Status"</span>
  <span class="nb">sleep </span>1
<span class="k">done</span>
</code></pre>

</div>



<p><strong>You'll see:</strong></p>

<ul>
<li>All requests still return 200 (no failures!)</li>
<li>Pool changes from "blue" to "green"</li>
<li>Headers now show <code>"pool": "green"</code>
</li>
</ul>

<p><strong>4. Fix blue:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"http://localhost:8081/chaos/stop"</span>
</code></pre>

</div>



<p>After a few seconds, traffic goes back to blue (it's the primary).</p>

<h3>
  
  
  Key Concepts Explained
</h3>

<h4>
  
  
  Why Aggressive Timeouts?
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">proxy_connect_timeout</span> <span class="s">2s</span><span class="p">;</span>
<span class="k">proxy_read_timeout</span> <span class="s">3s</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Scenario:</strong> Your blue service starts hanging (taking 10+ seconds to respond).</p>

<p><strong>With loose timeouts (10s):</strong></p>

<ul>
<li>User makes request → Nginx waits 10s on blue → Fails → Retries on green</li>
<li>User waited 10+ seconds (bad experience)</li>
</ul>

<p><strong>With tight timeouts (2-3s):</strong></p>

<ul>
<li>User makes request → Nginx waits 2s on blue → Fails fast → Retries on green</li>
<li>User gets response in ~2-3 seconds (much better!)</li>
</ul>

<p><strong>Trade-off:</strong> If your app legitimately takes 5 seconds to respond, 3s timeouts will cause false failures. Tune based on your app's performance.</p>

<h4>
  
  
  The Backup Directive
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">server</span> $<span class="p">{</span><span class="kn">ACTIVE_UPSTREAM</span><span class="err">}</span> <span class="s">max_fails=2</span> <span class="s">fail_timeout=5s</span><span class="p">;</span>
<span class="kn">server</span> $<span class="p">{</span><span class="kn">BACKUP_UPSTREAM</span><span class="err">}</span> <span class="s">max_fails=2</span> <span class="s">fail_timeout=5s</span> <span class="s">backup</span><span class="p">;</span>
</code></pre>

</div>



<p>The <code>backup</code> keyword is crucial:</p>

<ul>
<li>
<strong>Without it:</strong> Nginx load-balances 50/50 between blue and green</li>
<li>
<strong>With it:</strong> Nginx sends all traffic to blue, green only gets traffic when blue is down</li>
</ul>

<p>This is what makes it true blue/green deployment (not load balancing).</p>

<h4>
  
  
  Health Checks vs Failover
</h4>

<p><strong>Docker health checks:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">healthcheck</span><span class="pi">:</span>
  <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">wget"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--quiet"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--tries=1"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">--spider"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">http://localhost:3000/healthz"</span><span class="pi">]</span>
  <span class="na">interval</span><span class="pi">:</span> <span class="s">5s</span>
</code></pre>

</div>



<p><strong>Nginx failover:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">max_fails=2</span> <span class="s">fail_timeout=5s</span>
</code></pre>

</div>



<p>These serve different purposes:</p>

<ul>
<li>
<strong>Docker health checks:</strong> Tell Docker orchestration layer if container is healthy</li>
<li>
<strong>Nginx failover:</strong> Actual traffic routing decisions</li>
</ul>

<p>Both are important, but Nginx failover is what keeps users happy during failures.</p>

<h3>
  
  
  Production Considerations
</h3>

<h4>
  
  
  What Would Change in Production?
</h4>

<p><strong>1. Timeouts would be tuned:</strong></p>

<ul>
<li>Measure your app's real response times</li>
<li>Set timeouts slightly above p95 latency</li>
<li>Balance fast failover vs false positives</li>
</ul>

<p><strong>2. Monitoring &amp; Alerting:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="c1"># Add Prometheus metrics</span>
<span class="k">location</span> <span class="n">/metrics</span> <span class="p">{</span>
    <span class="kn">stub_status</span> <span class="no">on</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>3. Structured Logging:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">log_format</span> <span class="s">json</span> <span class="s">escape=json</span> <span class="s">'</span><span class="p">{</span><span class="kn">'</span>
  <span class="s">'"time":"</span><span class="nv">$time_iso8601</span><span class="s">",'</span>
  <span class="s">'"remote_addr":"</span><span class="nv">$remote_addr</span><span class="s">",'</span>
  <span class="s">'"request":"</span><span class="nv">$request</span><span class="s">",'</span>
  <span class="s">'"status":</span><span class="nv">$status</span><span class="s">,'</span>
  <span class="s">'"upstream":"</span><span class="nv">$upstream_addr</span><span class="s">"'</span>
<span class="s">'</span><span class="err">}</span><span class="s">'</span><span class="p">;</span>

<span class="kn">access_log</span> <span class="n">/var/log/nginx/access.log</span> <span class="s">json</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>4. SSL/TLS:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">server</span> <span class="p">{</span>
    <span class="kn">listen</span> <span class="mi">443</span> <span class="s">ssl</span> <span class="s">http2</span><span class="p">;</span>
    <span class="kn">ssl_certificate</span> <span class="n">/etc/ssl/certs/cert.pem</span><span class="p">;</span>
    <span class="kn">ssl_certificate_key</span> <span class="n">/etc/ssl/private/key.pem</span><span class="p">;</span>
    <span class="c1"># ... rest of config</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>5. Rate Limiting:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">limit_req_zone</span> <span class="nv">$binary_remote_addr</span> <span class="s">zone=api:10m</span> <span class="s">rate=10r/s</span><span class="p">;</span>

<span class="k">location</span> <span class="n">/</span> <span class="p">{</span>
    <span class="kn">limit_req</span> <span class="s">zone=api</span> <span class="s">burst=20</span> <span class="s">nodelay</span><span class="p">;</span>
    <span class="kn">proxy_pass</span> <span class="s">http://app_backend</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Troubleshooting Guide
</h3>

<h4>
  
  
  Problem: All requests failing
</h4>

<p><strong>Check if containers are running:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker-compose ps
</code></pre>

</div>



<p><strong>Expected output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>NAME        STATE    PORTS
nginx-lb    Up       0.0.0.0:8080-&gt;80/tcp
app_blue    Up       0.0.0.0:8081-&gt;3000/tcp
app_green   Up       0.0.0.0:8082-&gt;3000/tcp
</code></pre>

</div>



<p><strong>If containers are down:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker-compose logs
</code></pre>

</div>



<h4>
  
  
  Problem: Not seeing failover
</h4>

<p><strong>Check Nginx logs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker logs nginx-lb

<span class="c"># Or live tail:</span>
docker logs <span class="nt">-f</span> nginx-lb
</code></pre>

</div>



<p><strong>Look for:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[error] ... upstream timed out ... while connecting to upstream
[warn] ... marking server app_blue:3000 as down
</code></pre>

</div>



<h4>
  
  
  Problem: Services won't start
</h4>

<p><strong>Check if ports are already in use:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lsof <span class="nt">-i</span> :8080
lsof <span class="nt">-i</span> :8081
lsof <span class="nt">-i</span> :8082
</code></pre>

</div>



<p><strong>If something is using these ports:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Kill the process</span>
<span class="nb">kill</span> <span class="nt">-9</span> &lt;PID&gt;

<span class="c"># Or change ports in docker-compose.yml</span>
ports:
  - <span class="s2">"9080:80"</span>  <span class="c"># Use 9080 instead of 8080</span>
</code></pre>

</div>



<h4>
  
  
  Problem: Changes to .env not taking effect
</h4>

<p><strong>You need to recreate containers:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker-compose down
docker-compose up <span class="nt">-d</span>
</code></pre>

</div>



<p>Just restarting isn't enough - environment variables are set at container creation time.</p>

<h3>
  
  
  Real-World Use Cases
</h3>

<h4>
  
  
  Scenario 1: Deploying a New Version
</h4>

<p><strong>Without blue/green:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Take site down</span>
docker-compose down

<span class="c"># Update to new version</span>
docker-compose up <span class="nt">-d</span>

<span class="c"># Hope nothing broke (users see downtime)</span>
</code></pre>

</div>



<p><strong>With blue/green:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Update green to new version while blue serves traffic</span>
<span class="nb">sed</span> <span class="nt">-i</span> <span class="s1">'s/GREEN_IMAGE=.*/GREEN_IMAGE=myapp:v2.0.0/'</span> .env

<span class="c"># Start green with new version</span>
docker-compose up <span class="nt">-d</span> app_green

<span class="c"># Test green directly</span>
curl http://localhost:8082/version

<span class="c"># Switch traffic to green (instant, zero downtime)</span>
make green

<span class="c"># If something's wrong, instant rollback</span>
make blue
</code></pre>

</div>



<h4>
  
  
  Scenario 2: Database Migration
</h4>

<p><strong>Challenge:</strong> New version needs schema changes.</p>

<p><strong>Approach:</strong></p>

<ol>
<li>Make schema changes backward-compatible</li>
<li>Deploy new version to green (with migration)</li>
<li>Test thoroughly</li>
<li>Switch traffic</li>
<li>Keep blue running for quick rollback</li>
<li>After validation, update blue too</li>
</ol>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Update green with new code + migrations</span>
<span class="nv">GREEN_IMAGE</span><span class="o">=</span>myapp:v2.0.0 docker-compose up <span class="nt">-d</span> app_green

<span class="c"># Run migrations on green</span>
docker <span class="nb">exec </span>app_green npm run migrate

<span class="c"># Test green</span>
curl http://localhost:8082/test-endpoint

<span class="c"># Switch traffic</span>
make green

<span class="c"># Monitor for issues</span>
docker logs <span class="nt">-f</span> app_green

<span class="c"># If problems occur, instant rollback</span>
make blue
</code></pre>

</div>



