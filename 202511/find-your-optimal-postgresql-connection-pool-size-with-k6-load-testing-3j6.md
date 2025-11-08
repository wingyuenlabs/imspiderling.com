---
Title: Find Your Optimal PostgreSQL Connection Pool Size with k6 Load Testing
Description: 
Author: Manash Jyoti Baruah
Date: 2025-11-08T21:50:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  A step-by-step guide to identifying and fixing connection pool bottlenecks in Node.js applications
</h2>

<blockquote>
<p>How I reduced API response time from 1,221ms to 280ms using systematic load testing and pool optimization</p>
</blockquote>




<h2>
  
  
  What You'll Learn
</h2>

<p>In this guide, you'll discover how to:</p>

<ul>
<li>Identify connection pool bottlenecks in your Node.js application</li>
<li>Calculate the optimal pool size using load testing with k6</li>
<li>Monitor pool health with simple metrics</li>
<li>Fix performance issues that cause 1000ms+ response times</li>
</ul>

<p>Let's start with a real production issue I encountered and how I solved it systematically.</p>




<h2>
  
  
  The Problem: Slow Response Times with Low CPU Usage
</h2>

<p>During a routine load test of my Node.js API with 150 concurrent users, I observed these concerning metrics:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Node.js CPU: 45%
RAM: 60%
PostgreSQL CPU: 15%
Average Response Time: 1,221ms
P95 Latency: 2,890ms
</code></pre>

</div>



<p>The symptom was clear: <strong>Database CPU at 15% while response times exceeded 1 second</strong> indicated a bottleneck between the application and database layers, not a database performance issue.</p>




<h2>
  
  
  Step 1: Add Connection Pool Monitoring
</h2>

<p>First, I added monitoring to understand what's happening inside the connection pool. This single line of code revealed everything:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Monitor pool health every 5 seconds</span>
<span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
    <span class="s2">`[POOL] Total: </span><span class="p">${</span><span class="nx">pool</span><span class="p">.</span><span class="nx">totalCount</span><span class="p">}</span><span class="s2"> | Idle: </span><span class="p">${</span><span class="nx">pool</span><span class="p">.</span><span class="nx">idleCount</span><span class="p">}</span><span class="s2"> | Waiting: </span><span class="p">${</span><span class="nx">pool</span><span class="p">.</span><span class="nx">waitingCount</span><span class="p">}</span><span class="s2">`</span>
  <span class="p">);</span>
<span class="p">},</span> <span class="mi">5000</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Key Metrics Explained:</strong></p>

<ul>
<li>
<strong>Total</strong>: Number of connections in the pool</li>
<li>
<strong>Idle</strong>: Available connections ready to use</li>
<li>
<strong>Waiting</strong>: Requests queued because all connections are busy</li>
</ul>

<h3>
  
  
  Basic Server Setup with Monitoring
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">dotenv</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Pool</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">pg</span><span class="dl">"</span><span class="p">;</span>

<span class="nx">dotenv</span><span class="p">.</span><span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">poolConfig</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">host</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_HOST</span><span class="p">,</span>
  <span class="na">port</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PORT</span><span class="p">,</span>
  <span class="na">database</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_NAME</span><span class="p">,</span>
  <span class="na">user</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_USER</span><span class="p">,</span>
  <span class="na">password</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PASSWORD</span><span class="p">,</span>

  <span class="c1">// These are what we're testing</span>
  <span class="na">min</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">POOL_MIN</span><span class="p">)</span> <span class="o">||</span> <span class="mi">2</span><span class="p">,</span>
  <span class="na">max</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">POOL_MAX</span><span class="p">)</span> <span class="o">||</span> <span class="mi">10</span><span class="p">,</span>

  <span class="c1">// Important timeouts</span>
  <span class="na">idleTimeoutMillis</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">POOL_IDLE_TIMEOUT</span><span class="p">)</span> <span class="o">||</span> <span class="mi">30000</span><span class="p">,</span>
  <span class="na">connectionTimeoutMillis</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">POOL_CONNECTION_TIMEOUT</span><span class="p">)</span> <span class="o">||</span> <span class="mi">2000</span><span class="p">,</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">pool</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Pool</span><span class="p">(</span><span class="nx">poolConfig</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">WORKER_ID</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">pid</span><span class="p">;</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">=================================</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[Worker-</span><span class="p">${</span><span class="nx">WORKER_ID</span><span class="p">}</span><span class="s2">]`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  Host: </span><span class="p">${</span><span class="nx">poolConfig</span><span class="p">.</span><span class="nx">host</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  Database: </span><span class="p">${</span><span class="nx">poolConfig</span><span class="p">.</span><span class="nx">database</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  Pool Min: </span><span class="p">${</span><span class="nx">poolConfig</span><span class="p">.</span><span class="nx">min</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  Pool Max: </span><span class="p">${</span><span class="nx">poolConfig</span><span class="p">.</span><span class="nx">max</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">=================================</span><span class="se">\n</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Test database connection</span>
<span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT NOW()</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">err</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="s2">`[Worker-</span><span class="p">${</span><span class="nx">WORKER_ID</span><span class="p">}</span><span class="s2">] Database connection failed:`</span><span class="p">,</span> <span class="nx">err</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
    <span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
  <span class="p">}</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[Worker-</span><span class="p">${</span><span class="nx">WORKER_ID</span><span class="p">}</span><span class="s2">] Database connected at:`</span><span class="p">,</span> <span class="nx">res</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">now</span><span class="p">);</span>
<span class="p">});</span>

<span class="c1">// API endpoint for testing</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">/users</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">startTime</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users LIMIT 100</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">startTime</span><span class="p">;</span>

    <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">count</span><span class="p">:</span> <span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
      <span class="na">data</span><span class="p">:</span> <span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">,</span>
      <span class="na">query_time_ms</span><span class="p">:</span> <span class="nx">duration</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">startTime</span><span class="p">;</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="nx">err</span><span class="p">.</span><span class="nx">message</span><span class="p">,</span>
      <span class="na">query_time_ms</span><span class="p">:</span> <span class="nx">duration</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// The critical monitoring - logs every 5 seconds</span>
<span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
    <span class="s2">`[Worker-</span><span class="p">${</span><span class="nx">WORKER_ID</span><span class="p">}</span><span class="s2">] [POOL] Total: </span><span class="p">${</span><span class="nx">pool</span><span class="p">.</span><span class="nx">totalCount</span><span class="p">}</span><span class="s2"> | Idle: </span><span class="p">${</span><span class="nx">pool</span><span class="p">.</span><span class="nx">idleCount</span><span class="p">}</span><span class="s2"> | Waiting: </span><span class="p">${</span><span class="nx">pool</span><span class="p">.</span><span class="nx">waitingCount</span><span class="p">}</span><span class="s2">`</span>
  <span class="p">);</span>
<span class="p">},</span> <span class="mi">5000</span><span class="p">);</span>

<span class="c1">// Graceful shutdown</span>
<span class="nx">process</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">"</span><span class="s2">SIGTERM</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="se">\n</span><span class="s2">SIGTERM received, closing pool...</span><span class="dl">"</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
  <span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="p">});</span>

<span class="nx">process</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">"</span><span class="s2">SIGINT</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="se">\n</span><span class="s2">SIGINT received, closing pool...</span><span class="dl">"</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
  <span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="p">});</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`\n[Worker-</span><span class="p">${</span><span class="nx">WORKER_ID</span><span class="p">}</span><span class="s2">] Server running on http://localhost:</span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[Worker-</span><span class="p">${</span><span class="nx">WORKER_ID</span><span class="p">}</span><span class="s2">] Users endpoint: http://localhost:</span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">/users\n`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Step 2: Create k6 Load Test
</h2>

<p>Create a realistic load test that simulates production traffic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// load-test.js</span>
<span class="k">import</span> <span class="nx">http</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">k6/http</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">check</span><span class="p">,</span> <span class="nx">sleep</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">k6</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">options</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">stages</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">duration</span><span class="p">:</span> <span class="dl">"</span><span class="s2">10s</span><span class="dl">"</span><span class="p">,</span> <span class="na">target</span><span class="p">:</span> <span class="mi">10</span> <span class="p">},</span>   <span class="c1">// Warm up</span>
    <span class="p">{</span> <span class="na">duration</span><span class="p">:</span> <span class="dl">"</span><span class="s2">20s</span><span class="dl">"</span><span class="p">,</span> <span class="na">target</span><span class="p">:</span> <span class="mi">30</span> <span class="p">},</span>   <span class="c1">// Ramp up</span>
    <span class="p">{</span> <span class="na">duration</span><span class="p">:</span> <span class="dl">"</span><span class="s2">30s</span><span class="dl">"</span><span class="p">,</span> <span class="na">target</span><span class="p">:</span> <span class="mi">60</span> <span class="p">},</span>   <span class="c1">// Increase load</span>
    <span class="p">{</span> <span class="na">duration</span><span class="p">:</span> <span class="dl">"</span><span class="s2">10s</span><span class="dl">"</span><span class="p">,</span> <span class="na">target</span><span class="p">:</span> <span class="mi">100</span> <span class="p">},</span>  <span class="c1">// Push harder</span>
    <span class="p">{</span> <span class="na">duration</span><span class="p">:</span> <span class="dl">"</span><span class="s2">15s</span><span class="dl">"</span><span class="p">,</span> <span class="na">target</span><span class="p">:</span> <span class="mi">150</span> <span class="p">},</span>  <span class="c1">// Peak traffic</span>
    <span class="p">{</span> <span class="na">duration</span><span class="p">:</span> <span class="dl">"</span><span class="s2">10s</span><span class="dl">"</span><span class="p">,</span> <span class="na">target</span><span class="p">:</span> <span class="mi">0</span> <span class="p">},</span>    <span class="c1">// Cool down</span>
  <span class="p">],</span>
  <span class="na">thresholds</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">http_req_duration</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">p(95)&lt;2000</span><span class="dl">"</span><span class="p">],</span> <span class="c1">// 95% of requests under 2s</span>
    <span class="na">http_req_failed</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">rate&lt;0.1</span><span class="dl">"</span><span class="p">],</span>     <span class="c1">// Less than 10% errors</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">function </span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="nx">http</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">http://localhost:3000/users</span><span class="dl">"</span><span class="p">);</span>

  <span class="nf">check</span><span class="p">(</span><span class="nx">res</span><span class="p">,</span> <span class="p">{</span>
    <span class="dl">"</span><span class="s2">status is 200</span><span class="dl">"</span><span class="p">:</span> <span class="p">(</span><span class="nx">r</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="mi">200</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">response time &lt; 500ms</span><span class="dl">"</span><span class="p">:</span> <span class="p">(</span><span class="nx">r</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">timings</span><span class="p">.</span><span class="nx">duration</span> <span class="o">&lt;</span> <span class="mi">500</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="nf">sleep</span><span class="p">(</span><span class="mf">0.1</span><span class="p">);</span> <span class="c1">// Each user waits 100ms between requests</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">handleSummary</span><span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">metrics</span> <span class="o">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">metrics</span><span class="p">;</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="se">\n</span><span class="s2">========================================</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">LOAD TEST RESULTS</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">========================================</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Total Requests: </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">http_reqs</span><span class="p">.</span><span class="nx">values</span><span class="p">.</span><span class="nx">count</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Average Response: </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">http_req_duration</span><span class="p">.</span><span class="nx">values</span><span class="p">.</span><span class="nx">avg</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">ms`</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`P95 Latency: </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">http_req_duration</span><span class="p">.</span><span class="nx">values</span><span class="p">[</span><span class="dl">"</span><span class="s2">p(95)</span><span class="dl">"</span><span class="p">].</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">ms`</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Failed Requests: </span><span class="p">${(</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">http_req_failed</span><span class="p">.</span><span class="nx">values</span><span class="p">.</span><span class="nx">rate</span> <span class="o">*</span> <span class="mi">100</span><span class="p">).</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">%`</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">========================================</span><span class="se">\n</span><span class="dl">"</span><span class="p">);</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">stdout</span><span class="p">:</span> <span class="dl">""</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Step 3: Test Different Pool Sizes Systematically
</h2>

<p>I tested each pool size by editing the <code>.env</code> file and restarting the server:</p>

<h3>
  
  
  <code>.env</code> Configuration
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="c"># Database configuration
</span><span class="py">DB_HOST</span><span class="p">=</span><span class="s">localhost</span>
<span class="py">DB_PORT</span><span class="p">=</span><span class="s">5432</span>
<span class="py">DB_NAME</span><span class="p">=</span><span class="s">testdb</span>
<span class="py">DB_USER</span><span class="p">=</span><span class="s">postgres</span>
<span class="py">DB_PASSWORD</span><span class="p">=</span><span class="s">password</span>

<span class="c"># Pool configuration - Edit POOL_MAX for each test
</span><span class="py">POOL_MIN</span><span class="p">=</span><span class="s">1</span>
<span class="py">POOL_MAX</span><span class="p">=</span><span class="s">10  # Change to: 10, 15, 20, 40</span>
<span class="py">POOL_IDLE_TIMEOUT</span><span class="p">=</span><span class="s">30000</span>
<span class="py">POOL_CONNECTION_TIMEOUT</span><span class="p">=</span><span class="s">10000</span>
</code></pre>

</div>



<h3>
  
  
  Testing Workflow
</h3>

<p>For each pool size test:</p>

<ol>
<li>
<strong>Edit <code>.env</code> file</strong> - Change <code>POOL_MAX</code> value</li>
<li>
<strong>Start the server</strong> - <code>npm run dev</code>
</li>
<li>
<strong>Run load test</strong> - <code>k6 run load-tests/test.js</code> (in another terminal)</li>
<li>
<strong>Collect results</strong> - Watch the console logs and k6 output</li>
<li>
<strong>Stop server</strong> - Ctrl+C and repeat with next pool size
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 1: Set POOL_MAX=10 in .env file</span>
<span class="c"># Step 2: Start server</span>
npm run dev

<span class="c"># Step 3: In another terminal, run k6</span>
k6 run load-tests/test.js

<span class="c"># Step 4: Note the results</span>
<span class="c"># Step 5: Stop server (Ctrl+C)</span>

<span class="c"># Repeat: Edit .env to POOL_MAX=15</span>
npm run dev
k6 run load-tests/test.js

<span class="c"># Repeat: Edit .env to POOL_MAX=20</span>
npm run dev
k6 run load-tests/test.js

<span class="c"># Repeat: Edit .env to POOL_MAX=40</span>
npm run dev
k6 run load-tests/test.js
</code></pre>

</div>






<h2>
  
  
  Step 4: Analyse Test Results
</h2>

<h3>
  
  
  Test #1: Pool Size 10 (Baseline)
</h3>

<p><strong>Server Logs During Load:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Worker-24064] [POOL] Total: 1 | Idle: 0 | Waiting: 0
[Worker-24064] [POOL] Total: 6 | Idle: 1 | Waiting: 0
[Worker-24064] [POOL] Total: 10 | Idle: 3 | Waiting: 0
[Worker-24064] [POOL] Total: 10 | Idle: 1 | Waiting: 0
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 11
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 14
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 22
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 24
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 36
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 37
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 58
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 53
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 69
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 72
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 105
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 123
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 130   PEAK QUEUE
[Worker-24064] [POOL] Total: 10 | Idle: 0 | Waiting: 54
[Worker-24064] [POOL] Total: 10 | Idle: 6 | Waiting: 0
[Worker-24064] [POOL] Total: 10 | Idle: 10 | Waiting: 0
[Worker-24064] [POOL] Total: 0 | Idle: 0 | Waiting: 0
</code></pre>

</div>



<p><strong>k6 Results:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2szbz57sqzf486dvnmle.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2szbz57sqzf486dvnmle.png" alt="Grafana k6  load testing results with pg connection pool limit 10" width="800" height="425"></a></p>

<h3>
  
  
  Test #2: Pool Size 15
</h3>

<p><strong>Server Logs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Worker-29532] [POOL] Total: 1 | Idle: 1 | Waiting: 0
[Worker-29532] [POOL] Total: 5 | Idle: 1 | Waiting: 0
[Worker-29532] [POOL] Total: 8 | Idle: 3 | Waiting: 0
[Worker-29532] [POOL] Total: 11 | Idle: 1 | Waiting: 0
[Worker-29532] [POOL] Total: 15 | Idle: 4 | Waiting: 0
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 0
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 3
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 11
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 16
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 20
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 24
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 30
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 34
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 55
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 72
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 89
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 104
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 120   PEAK QUEUE
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 81
[Worker-29532] [POOL] Total: 15 | Idle: 0 | Waiting: 44
[Worker-29532] [POOL] Total: 15 | Idle: 15 | Waiting: 0
[Worker-29532] [POOL] Total: 14 | Idle: 14 | Waiting: 0
[Worker-29532] [POOL] Total: 0 | Idle: 0 | Waiting: 0
</code></pre>

</div>



<p><strong>k6 Results:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxtfinv4gvvij032yxkjr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxtfinv4gvvij032yxkjr.png" alt="Grafana k6  load testing results with pg connection pool limit 15" width="800" height="406"></a></p>

<h3>
  
  
  Test #3: Pool Size 20
</h3>

<p><strong>Server Logs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Worker-31608] [POOL] Total: 1 | Idle: 1 | Waiting: 0
[Worker-31608] [POOL] Total: 4 | Idle: 2 | Waiting: 0
[Worker-31608] [POOL] Total: 8 | Idle: 4 | Waiting: 0
[Worker-31608] [POOL] Total: 14 | Idle: 2 | Waiting: 0
[Worker-31608] [POOL] Total: 18 | Idle: 1 | Waiting: 0
[Worker-31608] [POOL] Total: 20 | Idle: 8 | Waiting: 0
[Worker-31608] [POOL] Total: 20 | Idle: 3 | Waiting: 0
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 2
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 10
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 18
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 24
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 36
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 48
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 66
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 86
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 99
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 114   PEAK
[Worker-31608] [POOL] Total: 20 | Idle: 0 | Waiting: 60
[Worker-31608] [POOL] Total: 20 | Idle: 8 | Waiting: 0
[Worker-31608] [POOL] Total: 20 | Idle: 20 | Waiting: 0    RECOVERED
[Worker-31608] [POOL] Total: 8 | Idle: 8 | Waiting: 0
[Worker-31608] [POOL] Total: 0 | Idle: 0 | Waiting: 0
</code></pre>

</div>



<p><strong>k6 Results:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqpdcum0ulqrjlxmijwjo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqpdcum0ulqrjlxmijwjo.png" alt="Grafana k6  load testing results with pg connection pool limit 20" width="800" height="396"></a></p>

<h3>
  
  
  Test #4: Pool Size 40
</h3>

<p><strong>Server Logs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Worker-31644] [POOL] Total: 1 | Idle: 1 | Waiting: 0
[Worker-31644] [POOL] Total: 4 | Idle: 0 | Waiting: 0
[Worker-31644] [POOL] Total: 9 | Idle: 0 | Waiting: 0
[Worker-31644] [POOL] Total: 13 | Idle: 2 | Waiting: 0
[Worker-31644] [POOL] Total: 21 | Idle: 4 | Waiting: 0
[Worker-31644] [POOL] Total: 27 | Idle: 8 | Waiting: 0
[Worker-31644] [POOL] Total: 32 | Idle: 12 | Waiting: 0
[Worker-31644] [POOL] Total: 38 | Idle: 1 | Waiting: 0
[Worker-31644] [POOL] Total: 40 | Idle: 4 | Waiting: 0
[Worker-31644] [POOL] Total: 40 | Idle: 6 | Waiting: 0
[Worker-31644] [POOL] Total: 40 | Idle: 13 | Waiting: 0
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 0
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 12
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 36
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 51
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 65
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 88   PEAK (still queuing!)
[Worker-31644] [POOL] Total: 40 | Idle: 0 | Waiting: 19
[Worker-31644] [POOL] Total: 40 | Idle: 36 | Waiting: 0
[Worker-31644] [POOL] Total: 40 | Idle: 40 | Waiting: 0   RECOVERED
[Worker-31644] [POOL] Total: 7 | Idle: 7 | Waiting: 0
[Worker-31644] [POOL] Total: 0 | Idle: 0 | Waiting: 0
</code></pre>

</div>



<p><strong>k6 Results:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbpg335cqaadezbwzt0op.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbpg335cqaadezbwzt0op.png" alt="Grafana k6  load testing results with pg connection pool limit 40" width="800" height="414"></a></p>

<p><strong>Analysis:</strong></p>

<ul>
<li> Best response times: <strong>280ms average</strong> (77% improvement from baseline)</li>
<li> Highest throughput: <strong>13,680 requests</strong> (239% increase)</li>
<li> Still experienced queuing: Peak of 88 waiting despite 40 connections</li>
<li> Diminishing returns: Only 59% improvement over Pool 20</li>
</ul>




<h2>
  
  
  Step 5: Performance Analysis
</h2>

<h3>
  
  
  Results Summary
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Pool Size</th>
<th>Total Requests</th>
<th>Avg Response</th>
<th>P95 Latency</th>
<th>Max Queue</th>
<th>Improvement</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>10</strong></td>
<td>4,040</td>
<td>1,221ms</td>
<td>2,890ms</td>
<td>130</td>
<td>Baseline</td>
</tr>
<tr>
<td><strong>15</strong></td>
<td>6,969</td>
<td>654ms</td>
<td>1,545ms</td>
<td>120</td>
<td>+72.5%</td>
</tr>
<tr>
<td><strong>20</strong></td>
<td>8,605</td>
<td>508ms</td>
<td>1,171ms</td>
<td>114</td>
<td>
<strong>+113%</strong> (Optimal)</td>
</tr>
<tr>
<td><strong>40</strong></td>
<td>13,680</td>
<td>280ms</td>
<td>529ms</td>
<td>88</td>
<td>+239%</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Key Insights
</h3>

<ol>
<li>
<strong>10 → 15</strong>: Significant improvement (72.5% throughput increase, 46% latency reduction)</li>
<li>
<strong>15 → 20</strong>: Strong continued gains (23.5% more throughput, 22% latency reduction)</li>
<li>
<strong>20 → 40</strong>: Diminishing returns (59% more throughput but 2x the connections)</li>
</ol>

<p><strong>VERDICT: Pool Size 20 is Optimal</strong> </p>

<p>While Pool Size 40 delivers the best raw performance, <strong>Pool Size 20 provides the best balance</strong>:</p>

<ul>
<li>
<p><strong>Pool 20 advantages:</strong></p>

<ul>
<li>113% throughput improvement with just 20 connections</li>
<li>Clears queue completely (recovers to 0)</li>
<li>Uses 50% fewer database resources than Pool 40</li>
<li>Already achieves sub-600ms response times</li>
</ul>


</li>

<li>

<p><strong>Pool 40 considerations:</strong></p>

<ul>
<li>Only 59% better than Pool 20 despite 100% more connections</li>
<li>Still experiences queuing (88 peak) during high load</li>
<li>Higher database resource consumption</li>
<li>Better suited if you expect &gt;200 concurrent users</li>
</ul>


</li>

</ul>




<h2>
  
  
  Step 6: Calculate Your Optimal Pool Size
</h2>

<p>Based on the testing, here's a formula to calculate your starting point:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">calculateOptimalPoolSize</span><span class="p">(</span><span class="nx">concurrentUsers</span><span class="p">,</span> <span class="nx">avgQueryTimeMs</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Base calculation</span>
  <span class="kd">const</span> <span class="nx">baseSize</span> <span class="o">=</span> <span class="p">(</span><span class="nx">concurrentUsers</span> <span class="o">*</span> <span class="nx">avgQueryTimeMs</span><span class="p">)</span> <span class="o">/</span> <span class="mi">1000</span><span class="p">;</span>

  <span class="c1">// Add 20% safety buffer</span>
  <span class="kd">const</span> <span class="nx">withBuffer</span> <span class="o">=</span> <span class="nx">baseSize</span> <span class="o">*</span> <span class="mf">1.2</span><span class="p">;</span>

  <span class="k">return</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">ceil</span><span class="p">(</span><span class="nx">withBuffer</span><span class="p">);</span>
<span class="p">}</span>

<span class="c1">// Example for our case</span>
<span class="kd">const</span> <span class="nx">poolSize</span> <span class="o">=</span> <span class="nf">calculateOptimalPoolSize</span><span class="p">(</span><span class="mi">150</span><span class="p">,</span> <span class="mi">150</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">poolSize</span><span class="p">);</span> <span class="c1">// Returns 27</span>

<span class="c1">// Testing showed 20 was sufficient because actual query time was faster</span>
</code></pre>

</div>



<h3>
  
  
  Important Constraints
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Check database limits</span>
<span class="kd">const</span> <span class="nx">dbMaxConnections</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>  <span class="c1">// from SHOW max_connections;</span>
<span class="kd">const</span> <span class="nx">reserveForAdmin</span> <span class="o">=</span> <span class="mi">10</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">availableForApp</span> <span class="o">=</span> <span class="nx">dbMaxConnections</span> <span class="o">-</span> <span class="nx">reserveForAdmin</span><span class="p">;</span>

<span class="c1">// If using multiple workers</span>
<span class="kd">const</span> <span class="nx">workers</span> <span class="o">=</span> <span class="mi">4</span><span class="p">;</span>  <span class="c1">// PM2 cluster mode</span>
<span class="kd">const</span> <span class="nx">poolPerWorker</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">floor</span><span class="p">(</span><span class="nx">availableForApp</span> <span class="o">/</span> <span class="nx">workers</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Each worker gets </span><span class="p">${</span><span class="nx">poolPerWorker</span><span class="p">}</span><span class="s2"> connections`</span><span class="p">);</span>
</code></pre>

</div>






<h2>
  
  
  Step 7: Production Implementation
</h2>

<h3>
  
  
  Enhanced Monitoring Setup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">ALERT_THRESHOLDS</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">waitingQueue</span><span class="p">:</span> <span class="mi">20</span><span class="p">,</span>
  <span class="na">idlePercentage</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
<span class="p">};</span>

<span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">total</span> <span class="o">=</span> <span class="nx">pool</span><span class="p">.</span><span class="nx">totalCount</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">idle</span> <span class="o">=</span> <span class="nx">pool</span><span class="p">.</span><span class="nx">idleCount</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">waiting</span> <span class="o">=</span> <span class="nx">pool</span><span class="p">.</span><span class="nx">waitingCount</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">idlePercentage</span> <span class="o">=</span> <span class="nx">total</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="p">(</span><span class="nx">idle</span> <span class="o">/</span> <span class="nx">total</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span> <span class="p">:</span> <span class="mi">0</span><span class="p">;</span>

  <span class="c1">// Log metrics</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[POOL] Total: </span><span class="p">${</span><span class="nx">total</span><span class="p">}</span><span class="s2"> | Idle: </span><span class="p">${</span><span class="nx">idle</span><span class="p">}</span><span class="s2"> | Waiting: </span><span class="p">${</span><span class="nx">waiting</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

  <span class="c1">// Alert conditions</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">waiting</span> <span class="o">&gt;</span> <span class="nx">ALERT_THRESHOLDS</span><span class="p">.</span><span class="nx">waitingQueue</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="s2">` [ALERT] High queue: </span><span class="p">${</span><span class="nx">waiting</span><span class="p">}</span><span class="s2"> requests waiting`</span><span class="p">);</span>
    <span class="c1">// Send to monitoring service</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">idlePercentage</span> <span class="o">&lt;</span> <span class="nx">ALERT_THRESHOLDS</span><span class="p">.</span><span class="nx">idlePercentage</span> <span class="o">&amp;&amp;</span> <span class="nx">total</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="s2">` [ALERT] Low idle connections: </span><span class="p">${</span><span class="nx">idlePercentage</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">1</span><span class="p">)}</span><span class="s2">%`</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">},</span> <span class="mi">5000</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Database Monitoring Queries
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Check current connections</span>
<span class="k">SELECT</span> <span class="k">count</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">as</span> <span class="n">total</span><span class="p">,</span>
       <span class="k">count</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="n">FILTER</span> <span class="p">(</span><span class="k">WHERE</span> <span class="k">state</span> <span class="o">=</span> <span class="s1">'active'</span><span class="p">)</span> <span class="k">as</span> <span class="n">active</span><span class="p">,</span>
       <span class="k">count</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="n">FILTER</span> <span class="p">(</span><span class="k">WHERE</span> <span class="k">state</span> <span class="o">=</span> <span class="s1">'idle'</span><span class="p">)</span> <span class="k">as</span> <span class="n">idle</span>
<span class="k">FROM</span> <span class="n">pg_stat_activity</span>
<span class="k">WHERE</span> <span class="n">datname</span> <span class="o">=</span> <span class="n">current_database</span><span class="p">();</span>

<span class="c1">-- Find connection hogs</span>
<span class="k">SELECT</span> <span class="n">pid</span><span class="p">,</span> <span class="n">usename</span><span class="p">,</span> <span class="n">application_name</span><span class="p">,</span> <span class="k">state</span><span class="p">,</span>
       <span class="n">NOW</span><span class="p">()</span> <span class="o">-</span> <span class="n">query_start</span> <span class="k">as</span> <span class="n">duration</span><span class="p">,</span>
       <span class="k">substring</span><span class="p">(</span><span class="n">query</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">50</span><span class="p">)</span> <span class="k">as</span> <span class="n">query_preview</span>
<span class="k">FROM</span> <span class="n">pg_stat_activity</span>
<span class="k">WHERE</span> <span class="k">state</span> <span class="o">!=</span> <span class="s1">'idle'</span>
  <span class="k">AND</span> <span class="n">query_start</span> <span class="o">&lt;</span> <span class="n">NOW</span><span class="p">()</span> <span class="o">-</span> <span class="n">INTERVAL</span> <span class="s1">'1 minute'</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">query_start</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  Common Pitfalls to Avoid
</h2>

<h3>
  
  
  1. Not Releasing Connections
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">//  BAD: Connection leak</span>
<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">);</span>
<span class="c1">// Forgot to release!</span>

<span class="c1">//  GOOD: Always release</span>
<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
<span class="k">try</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">);</span>
<span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
  <span class="nx">client</span><span class="p">.</span><span class="nf">release</span><span class="p">();</span>
<span class="p">}</span>

<span class="c1">//  BETTER: Use pool.query() for simple queries</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  2. Holding Connections During External Calls
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">//  BAD: Blocks connection for 2+ seconds</span>
<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users WHERE id = $1</span><span class="dl">"</span><span class="p">,</span> <span class="p">[</span><span class="nx">id</span><span class="p">]);</span>
<span class="kd">const</span> <span class="nx">apiData</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">"</span><span class="s2">https://slow-api.com/data</span><span class="dl">"</span><span class="p">);</span> <span class="c1">// 2 seconds!</span>
<span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">UPDATE users SET data = $1</span><span class="dl">"</span><span class="p">,</span> <span class="p">[</span><span class="nx">apiData</span><span class="p">]);</span>
<span class="nx">client</span><span class="p">.</span><span class="nf">release</span><span class="p">();</span>

<span class="c1">//  GOOD: Release before external call</span>
<span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users WHERE id = $1</span><span class="dl">"</span><span class="p">,</span> <span class="p">[</span><span class="nx">id</span><span class="p">]);</span>
<span class="kd">const</span> <span class="nx">apiData</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">"</span><span class="s2">https://slow-api.com/data</span><span class="dl">"</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">"</span><span class="s2">UPDATE users SET data = $1</span><span class="dl">"</span><span class="p">,</span> <span class="p">[</span><span class="nx">apiData</span><span class="p">]);</span>
</code></pre>

</div>






<h2>
  
  
  Quick Reference
</h2>

<h3>
  
  
  Healthy Pool Indicators
</h3>

<ul>
<li> Waiting &lt; 10 consistently</li>
<li> Idle &gt; 0 (at least 10% of total)</li>
<li> Response time P95 &lt; 1000ms</li>
<li> No connection timeouts</li>
</ul>

<h3>
  
  
  When to Increase Pool Size
</h3>

<ul>
<li> Waiting &gt; 30 consistently</li>
<li> Idle always 0</li>
<li> Response times &gt; 1000ms</li>
<li> Database CPU &lt; 50% under load</li>
</ul>

<h3>
  
  
  Useful Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start server (uses .env file)</span>
npm run dev

<span class="c"># Run load test (in another terminal)</span>
k6 run load-tests/test.js

<span class="c"># PostgreSQL connections check</span>
psql <span class="nt">-c</span> <span class="s2">"SELECT count(*) FROM pg_stat_activity"</span>
</code></pre>

</div>






<h2>
  
  
  Conclusion
</h2>

<p>Connection pool optimization reduced our response time from <strong>1,221ms to 280ms</strong> - a <strong>77% improvement</strong> - through systematic testing with k6 load tests.</p>

<h3>
  
  
  The Complete Performance Picture:
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Pool 10</th>
<th>Pool 20 (Recommended)</th>
<th>Pool 40</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Avg Response</strong></td>
<td>1,221ms</td>
<td>508ms (-58%)</td>
<td>280ms (-77%)</td>
</tr>
<tr>
<td><strong>P95 Latency</strong></td>
<td>2,890ms</td>
<td>1,171ms (-59%)</td>
<td>529ms (-82%)</td>
</tr>
<tr>
<td><strong>Throughput</strong></td>
<td>4,040 req</td>
<td>8,605 req (+113%)</td>
<td>13,680 req (+239%)</td>
</tr>
<tr>
<td><strong>Max Queue</strong></td>
<td>130</td>
<td>114</td>
<td>88</td>
</tr>
<tr>
<td><strong>Efficiency</strong></td>
<td>Poor</td>
<td><strong>Excellent</strong></td>
<td>Good</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Final Verdict: Choose Pool Size 20
</h3>

<p><strong>Why Pool 20 over Pool 40?</strong></p>

<ol>
<li>
<strong>Resource efficiency</strong>: 113% performance gain with just 20 connections</li>
<li>
<strong>Diminishing returns</strong>: Pool 40 only adds 59% more throughput for 100% more connections</li>
<li>
<strong>Cost-benefit</strong>: Pool 20 achieves the "good enough" threshold (&lt;600ms response)</li>
<li>
<strong>Database overhead</strong>: Fewer connections = less memory and CPU on PostgreSQL</li>
</ol>

<h3>
  
  
  Key Takeaways:
</h3>

<ol>
<li>
<strong>Default pool (10) is severely undersized</strong> - caused 1.2s response times</li>
<li>
<strong>Pool 20 hits the sweet spot</strong> - doubles throughput, halves latency</li>
<li>
<strong>Pool 40 shows diminishing returns</strong> - consider only for &gt;200 concurrent users</li>
<li>
<strong>Queue monitoring is critical</strong> - even Pool 40 had 88 requests waiting at peak</li>
</ol>

<p>Remember: The default pool size of 10 resulted in 130 requests queuing. Simply changing to Pool 20 delivered a <strong>2x throughput improvement</strong> with sub-second response times.</p>




<p><strong>Resources:</strong></p>

<ul>
<li>📊 <a href="https://k6.io/docs/" rel="noopener noreferrer">k6 documentation</a>
</li>
</ul>

<p><strong>Questions?</strong> Drop them in the comments. What pool size works for your application?</p>




<h3>
  
  
  💡 Found this helpful?
</h3>

<p><em>Drop a ❤️ and 🦄 if this helped you. Follow for more practical guides and real-world dev experiences.</em></p>

<p>🤝 <strong>Let's connect:</strong></p>

<p><strong>💼 LinkedIn</strong> → <a href="https://www.linkedin.com/in/manashjb/" rel="noopener noreferrer">linkedin.com/in/manashjb</a><br><br>
<strong>📺 YouTube</strong> → <a href="https://www.youtube.com/@CoderInSpace" rel="noopener noreferrer">youtube.com/@CoderInSpace</a><br><br>
<strong>🚀 Portfolio</strong> → <a href="https://manash.dev" rel="noopener noreferrer">manash.dev</a></p>




