---
Title: Stop Getting Blocked: Professional API Rate Limiting Strategies
Description: 
Author: Olamide Olaniyan
Date: 2025-11-20T21:54:58.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Originally published on <a href="https://sociavault.com/blog/api-rate-limiting-strategies-scale" rel="noopener noreferrer">SociaVault Blog</a></em></p>

<p>You've built a scraper. It works perfectly for 10 minutes. Then crashes with a 429 error.</p>

<p>Rate limit exceeded. Blocked for 15 minutes.</p>

<p>You restart. Same thing. Your data pipeline is broken. Users are waiting.</p>

<p>I've hit every rate limit imaginable. Instagram blocked me for a day. Twitter cut me off mid-scrape. TikTok throttled me to nothing.</p>

<p>Now I handle millions of API requests daily without issues. Let me show you how.</p>

<h2>
  
  
  The Problem: Why APIs Have Limits
</h2>

<p>APIs protect their infrastructure with limits:</p>

<ul>
<li>
<strong>Per second</strong>: 10 requests/sec</li>
<li>
<strong>Per minute</strong>: 100 requests/min
</li>
<li>
<strong>Per hour</strong>: 5,000 requests/hr</li>
<li>
<strong>Per day</strong>: 50,000 requests/day</li>
</ul>

<p>Hit any limit → blocked temporarily.</p>

<h2>
  
  
  Solution 1: Exponential Backoff
</h2>

<p>When rate limited, wait before retrying. But wait <strong>longer each time</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">async</span> <span class="kd">function</span> <span class="nf">requestWithBackoff</span><span class="p">(</span><span class="nx">url</span><span class="p">,</span> <span class="nx">headers</span><span class="p">,</span> <span class="nx">maxRetries</span> <span class="o">=</span> <span class="mi">5</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="nx">retries</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
  <span class="kd">let</span> <span class="nx">delay</span> <span class="o">=</span> <span class="mi">1000</span><span class="p">;</span> <span class="c1">// Start with 1 second</span>

  <span class="k">while </span><span class="p">(</span><span class="nx">retries</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">try</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">axios</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">url</span><span class="p">,</span> <span class="p">{</span> <span class="nx">headers</span> <span class="p">});</span>
      <span class="k">return</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">;</span>

    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">response</span><span class="p">?.</span><span class="nx">status</span> <span class="o">===</span> <span class="mi">429</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">retries</span><span class="o">++</span><span class="p">;</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">retries</span> <span class="o">&gt;=</span> <span class="nx">maxRetries</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Max retries exceeded</span><span class="dl">'</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Rate limited. Waiting </span><span class="p">${</span><span class="nx">delay</span><span class="p">}</span><span class="s2">ms (retry </span><span class="p">${</span><span class="nx">retries</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">maxRetries</span><span class="p">}</span><span class="s2">)`</span><span class="p">);</span>
        <span class="k">await</span> <span class="k">new</span> <span class="nc">Promise</span><span class="p">(</span><span class="nx">resolve</span> <span class="o">=&gt;</span> <span class="nf">setTimeout</span><span class="p">(</span><span class="nx">resolve</span><span class="p">,</span> <span class="nx">delay</span><span class="p">));</span>

        <span class="nx">delay</span> <span class="o">*=</span> <span class="mi">2</span><span class="p">;</span> <span class="c1">// Exponential: 1s, 2s, 4s, 8s, 16s</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Why this works</strong>: First retry after 1s. Second after 2s. Third after 4s. Gives the API time to recover.</p>

<h2>
  
  
  Solution 2: Request Queue with Rate Limiting
</h2>

<p>Instead of bursts, queue requests and send at controlled rate:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">class</span> <span class="nc">RateLimitedQueue</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">requestsPerSecond</span> <span class="o">=</span> <span class="mi">5</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">queue</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">requestsPerSecond</span> <span class="o">=</span> <span class="nx">requestsPerSecond</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">interval</span> <span class="o">=</span> <span class="mi">1000</span> <span class="o">/</span> <span class="nx">requestsPerSecond</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">processing</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nf">add</span><span class="p">(</span><span class="nx">requestFunction</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nc">Promise</span><span class="p">((</span><span class="nx">resolve</span><span class="p">,</span> <span class="nx">reject</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">queue</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span> <span class="nx">requestFunction</span><span class="p">,</span> <span class="nx">resolve</span><span class="p">,</span> <span class="nx">reject</span> <span class="p">});</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="k">this</span><span class="p">.</span><span class="nx">processing</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">process</span><span class="p">();</span>
      <span class="p">}</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">process</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">processing</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>

    <span class="k">while </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">queue</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">item</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">queue</span><span class="p">.</span><span class="nf">shift</span><span class="p">();</span>

      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">item</span><span class="p">.</span><span class="nf">requestFunction</span><span class="p">();</span>
        <span class="nx">item</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="nx">result</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">item</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="nx">error</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="c1">// Wait before next request</span>
      <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">queue</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">await</span> <span class="k">new</span> <span class="nc">Promise</span><span class="p">(</span><span class="nx">resolve</span> <span class="o">=&gt;</span> <span class="nf">setTimeout</span><span class="p">(</span><span class="nx">resolve</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">interval</span><span class="p">));</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">this</span><span class="p">.</span><span class="nx">processing</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="kd">const</span> <span class="nx">queue</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RateLimitedQueue</span><span class="p">(</span><span class="mi">5</span><span class="p">);</span> <span class="c1">// 5 requests per second</span>

<span class="k">await</span> <span class="nx">queue</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">return</span> <span class="k">await</span> <span class="nf">fetchData</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://api.example.com/data</span><span class="dl">'</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Why this works</strong>: Guarantees you never exceed rate limit. Requests spaced evenly. No bursts.</p>

<h2>
  
  
  Solution 3: Token Bucket Algorithm
</h2>

<p>More sophisticated rate limiting for variable loads:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">class</span> <span class="nc">TokenBucket</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">capacity</span><span class="p">,</span> <span class="nx">refillRate</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">capacity</span> <span class="o">=</span> <span class="nx">capacity</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">tokens</span> <span class="o">=</span> <span class="nx">capacity</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">refillRate</span> <span class="o">=</span> <span class="nx">refillRate</span><span class="p">;</span> <span class="c1">// tokens per second</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">lastRefill</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="nf">refill</span><span class="p">()</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">now</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">timePassed</span> <span class="o">=</span> <span class="p">(</span><span class="nx">now</span> <span class="o">-</span> <span class="k">this</span><span class="p">.</span><span class="nx">lastRefill</span><span class="p">)</span> <span class="o">/</span> <span class="mi">1000</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">tokensToAdd</span> <span class="o">=</span> <span class="nx">timePassed</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">refillRate</span><span class="p">;</span>

    <span class="k">this</span><span class="p">.</span><span class="nx">tokens</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">capacity</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">tokens</span> <span class="o">+</span> <span class="nx">tokensToAdd</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">lastRefill</span> <span class="o">=</span> <span class="nx">now</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">consume</span><span class="p">(</span><span class="nx">tokens</span> <span class="o">=</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">refill</span><span class="p">();</span>

    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">tokens</span> <span class="o">&gt;=</span> <span class="nx">tokens</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">tokens</span> <span class="o">-=</span> <span class="nx">tokens</span><span class="p">;</span>
      <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// Wait for tokens to refill</span>
    <span class="kd">const</span> <span class="nx">tokensNeeded</span> <span class="o">=</span> <span class="nx">tokens</span> <span class="o">-</span> <span class="k">this</span><span class="p">.</span><span class="nx">tokens</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">waitTime</span> <span class="o">=</span> <span class="p">(</span><span class="nx">tokensNeeded</span> <span class="o">/</span> <span class="k">this</span><span class="p">.</span><span class="nx">refillRate</span><span class="p">)</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">;</span>

    <span class="k">await</span> <span class="k">new</span> <span class="nc">Promise</span><span class="p">(</span><span class="nx">resolve</span> <span class="o">=&gt;</span> <span class="nf">setTimeout</span><span class="p">(</span><span class="nx">resolve</span><span class="p">,</span> <span class="nx">waitTime</span><span class="p">));</span>

    <span class="k">this</span><span class="p">.</span><span class="nf">refill</span><span class="p">();</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">tokens</span> <span class="o">-=</span> <span class="nx">tokens</span><span class="p">;</span>
    <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="kd">const</span> <span class="nx">bucket</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TokenBucket</span><span class="p">(</span><span class="mi">100</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span> <span class="c1">// 100 max, refill 10/sec</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">makeRequest</span><span class="p">(</span><span class="nx">url</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">bucket</span><span class="p">.</span><span class="nf">consume</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
  <span class="k">return</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">url</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Why this works</strong>: Allows bursts when tokens available, smooths over time. More flexible than fixed rate.</p>

<h2>
  
  
  Solution 4: Distributed Rate Limiting with Redis
</h2>

<p>When multiple servers make requests, coordinate with Redis:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">Redis</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">ioredis</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">redis</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Redis</span><span class="p">();</span>

<span class="kd">class</span> <span class="nc">DistributedRateLimiter</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="nx">limit</span><span class="p">,</span> <span class="nx">windowSeconds</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">key</span> <span class="o">=</span> <span class="nx">key</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">limit</span> <span class="o">=</span> <span class="nx">limit</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">windowSeconds</span> <span class="o">=</span> <span class="nx">windowSeconds</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">checkLimit</span><span class="p">()</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">now</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">windowStart</span> <span class="o">=</span> <span class="nx">now</span> <span class="o">-</span> <span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">windowSeconds</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">);</span>

    <span class="c1">// Remove old requests</span>
    <span class="k">await</span> <span class="nx">redis</span><span class="p">.</span><span class="nf">zremrangebyscore</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">key</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nx">windowStart</span><span class="p">);</span>

    <span class="c1">// Count requests in window</span>
    <span class="kd">const</span> <span class="nx">requestCount</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">redis</span><span class="p">.</span><span class="nf">zcard</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">key</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">requestCount</span> <span class="o">&lt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">limit</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">redis</span><span class="p">.</span><span class="nf">zadd</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">key</span><span class="p">,</span> <span class="nx">now</span><span class="p">,</span> <span class="s2">`</span><span class="p">${</span><span class="nx">now</span><span class="p">}</span><span class="s2">-</span><span class="p">${</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">()}</span><span class="s2">`</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">redis</span><span class="p">.</span><span class="nf">expire</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">key</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">windowSeconds</span> <span class="o">*</span> <span class="mi">2</span><span class="p">);</span>
      <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="kc">false</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">waitForSlot</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">while </span><span class="p">(</span><span class="kc">true</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">checkLimit</span><span class="p">())</span> <span class="p">{</span>
        <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
      <span class="p">}</span>
      <span class="k">await</span> <span class="k">new</span> <span class="nc">Promise</span><span class="p">(</span><span class="nx">resolve</span> <span class="o">=&gt;</span> <span class="nf">setTimeout</span><span class="p">(</span><span class="nx">resolve</span><span class="p">,</span> <span class="mi">100</span><span class="p">));</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage across multiple servers</span>
<span class="kd">const</span> <span class="nx">limiter</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DistributedRateLimiter</span><span class="p">(</span><span class="dl">'</span><span class="s1">api:limit</span><span class="dl">'</span><span class="p">,</span> <span class="mi">100</span><span class="p">,</span> <span class="mi">60</span><span class="p">);</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">makeDistributedRequest</span><span class="p">(</span><span class="nx">url</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">limiter</span><span class="p">.</span><span class="nf">waitForSlot</span><span class="p">();</span>
  <span class="k">return</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">url</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Why this works</strong>: All servers share the same Redis counter. Total requests never exceed limit. Critical for scaling.</p>

<h2>
  
  
  Real Results
</h2>

<p><strong>Before rate limiting strategies</strong>:</p>

<ul>
<li>Hit limit every 2 minutes</li>
<li>Blocked 15 minutes each time</li>
<li>Could scrape 100 profiles/hour</li>
<li>Constant errors</li>
</ul>

<p><strong>After implementing these</strong>:</p>

<ul>
<li>Never hit limits</li>
<li>5,000 profiles/hour</li>
<li>Zero errors</li>
<li>Smooth operation</li>
</ul>

<h2>
  
  
  Read the Full Guide
</h2>

<p>This is a condensed version. The full guide includes:</p>

<ul>
<li>Smart caching strategies</li>
<li>Multiple API key rotation</li>
<li>Complete production-ready code</li>
<li>Python implementations</li>
<li>Monitoring and alerting</li>
</ul>

<p><strong><a href="https://sociavault.com/blog/api-rate-limiting-strategies-scale-2025" rel="noopener noreferrer">Read the complete guide on SociaVault →</a></strong></p>




<p><em>Building social media data tools? <a href="https://sociavault.com" rel="noopener noreferrer">SociaVault</a> handles rate limiting infrastructure for you. Focus on your app, not API limits.</em></p>

<h2>
  
  
  Discussion
</h2>

<p>What rate limiting strategies have worked for you? Share in the comments! 👇</p>

<h1>
  
  
  webdev #api #ratelimiting #nodejs #javascript
</h1>

