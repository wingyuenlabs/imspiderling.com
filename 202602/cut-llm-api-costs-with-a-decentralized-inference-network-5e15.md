---
Title: Cut LLM API costs with a decentralized inference network
Description: 
Author: Daniil Koryto
Date: 2026-02-03T21:49:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>LLM bills can climb fast, especially when you're shipping an MVP, an internal tool, or a workflow-heavy app. We hit that wall and started building <strong>GonkaGate</strong> to make open-source inference usable without a full rewrite.</p>

<p>This is a practical walkthrough. We'll show how it works, how to migrate with minimal code changes, and where the trade-offs are real.</p>

<p>Disclosure: We are the team behind GonkaGate.</p>

<h2>
  
  
  What you'll learn
</h2>

<ul>
<li>Why centralized LLM APIs are expensive</li>
<li>How decentralized inference changes the cost structure</li>
<li>How to switch from the OpenAI SDK with a simple base URL change</li>
<li>When this approach fits and when it doesn't</li>
</ul>

<h2>
  
  
  Why LLM APIs are expensive
</h2>

<p>Centralized providers have a cost stack that adds up:</p>

<ul>
<li>Infrastructure (top-tier GPUs cost tens of thousands of dollars per card)</li>
<li>Frontier-model R&amp;D</li>
<li>Enterprise sales and compliance</li>
<li>Margin</li>
</ul>

<p>Pricing changes frequently, so here are the official pages:</p>

<ul>
<li><a href="https://platform.openai.com/docs/pricing" rel="noopener noreferrer">OpenAI pricing</a></li>
<li><a href="https://platform.claude.com/docs/en/about-claude/pricing" rel="noopener noreferrer">Anthropic pricing</a></li>
<li><a href="https://ai.google.dev/gemini-api/docs/pricing" rel="noopener noreferrer">Google Gemini pricing</a></li>
</ul>

<p>The alternative is self-hosting open-source models, and that has its own pain:</p>

<ul>
<li>Renting and maintaining GPU clusters</li>
<li>CUDA, drivers, and dependency setup</li>
<li>Scaling, load balancing, failover</li>
<li>Ongoing infra maintenance</li>
</ul>

<p>For small teams, both options can hurt in different ways.</p>

<h2>
  
  
  A third path: decentralized inference
</h2>

<p>Gonka is a decentralized network for open-source model inference. The <a href="https://gonka.gg/" rel="noopener noreferrer">public network tracker</a> shows about ~5.4k H100-equivalent GPUs as of February 03, 2026. In December 2025, Bitfury <a href="https://www.businesswire.com/news/home/20251201364475/en/Bitfury-Announces-%2450-Million-Investment-in-Gonkaa-Decentralized-Network-for-Highefficiency-AI-Compute" rel="noopener noreferrer">announced a $50M investment</a> in Gonka as part of its $1B program for decentralized AI projects.</p>

<h3>
  
  
  How the price drops
</h3>

<ol>
<li>
<strong>Compute is nearly fully utilized.</strong> Traditional PoW chains burn a lot of compute on consensus. Gonka uses a mechanism called <strong>Sprint</strong> (Transformer-based Proof-of-Work), described in the <a href="https://gonka.ai/whitepaper.pdf" rel="noopener noreferrer">whitepaper</a>. The work is closer to LLM inference than to hashing.</li>
<li>
<strong>Distributed GPU hosts.</strong> Hardware owners (individual and enterprise) contribute idle compute and earn rewards. The network aggregates existing capacity instead of building centralized data centers. A wide range of GPUs is supported (H100/H200, A100) with a <a href="https://gonka.me/host/hardware-specifications/" rel="noopener noreferrer">48 GB VRAM minimum</a>.</li>
<li>
<strong><a href="https://gonka.ai/wallet/pricing/#references" rel="noopener noreferrer">Dynamic on-chain pricing.</a></strong> Pricing follows network load .</li>
<li>
<strong>Open-source model ecosystem.</strong> The model list changes as operators join the network. New models appear as they are added.</li>
</ol>

<h2>
  
  
  What GonkaGate does
</h2>

<p><strong>GonkaGate</strong> is an API gateway to the Gonka network. You pay in USD, and the integration is compatible with the OpenAI SDK. In most cases, you just swap the base URL and API key.</p>

<h2>
  
  
  Available models
</h2>

<p>These are open-source models, not frontier proprietary ones:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Model</th>
<th>Context</th>
<th>Best for</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>qwen/qwen3-235b-a22b-instruct-2507-fp8</code></td>
<td>262K</td>
<td>Complex reasoning, code</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Honest take:</strong> <code>qwen3-235b</code> is strong at code, summarization, and reasoning. It still lags frontier models on nuanced creative writing and the hardest multi-step tasks. Test on your real use cases.</p>
</blockquote>

<p><strong>Why it's interesting:</strong> open-source means transparency and no vendor lock-in, and the list grows as more operators join.</p>

<h2>
  
  
  Migrate from OpenAI SDK (minimal changes)
</h2>

<p>If you're already using the OpenAI SDK, you only change the endpoint and key.</p>

<h3>
  
  
  Python
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">openai</span> <span class="kn">import</span> <span class="n">OpenAI</span>

<span class="c1"># Before: OpenAI
# client = OpenAI(api_key="sk-...")
</span>
<span class="c1"># After: GonkaGate
</span><span class="n">client</span> <span class="o">=</span> <span class="nc">OpenAI</span><span class="p">(</span>
    <span class="n">base_url</span><span class="o">=</span><span class="sh">"</span><span class="s">https://api.gonkagate.com/v1</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">api_key</span><span class="o">=</span><span class="sh">"</span><span class="s">your-gonkagate-key</span><span class="sh">"</span>
<span class="p">)</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">chat</span><span class="p">.</span><span class="n">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">qwen/qwen3-235b-a22b-instruct-2507-fp8</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[</span>
        <span class="p">{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Explain recursion in simple terms</span><span class="sh">"</span><span class="p">}</span>
    <span class="p">]</span>
<span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">message</span><span class="p">.</span><span class="n">content</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  JavaScript / Node.js
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">OpenAI</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">openai</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">OpenAI</span><span class="p">({</span>
  <span class="na">baseURL</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://api.gonkagate.com/v1</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">GONKAGATE_KEY</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nx">chat</span><span class="p">.</span><span class="nx">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">qwen3-235b-a22b-instruct-2507-fp8</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">messages</span><span class="p">:</span> <span class="p">[{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">'</span><span class="s1">user</span><span class="dl">'</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Solve: 2x + 5 = 13</span><span class="dl">'</span> <span class="p">}],</span>
<span class="p">});</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">message</span><span class="p">.</span><span class="nx">content</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  curl
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl https://api.gonkagate.com/v1/chat/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$GONKAGATE_KEY</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "model": "qwen3-235b-a22b-instruct-2507-fp8",
    "messages": [{"role": "user", "content": "Hello, world!"}]
  }'</span>
</code></pre>

</div>



<p><strong>Supported:</strong> streaming responses, chat completions API, and standard OpenAI SDK methods.</p>

<h2>
  
  
  n8n and workflow automation
</h2>

<p>If you use n8n, GonkaGate is a good fit. n8n allows custom base URLs in OpenAI credentials:</p>

<ol>
<li>Set base URL to <code>https://api.gonkagate.com/v1</code>.</li>
<li>Add your GonkaGate API key.</li>
<li>Use any AI nodes (Chat, Agent, etc.) with lower costs.</li>
</ol>

<p>Why this matters: most n8n workflows are simple tasks (summarization, classification, extraction, basic Q&amp;A). You don't need frontier-level reasoning for that.</p>

<p>With hundreds or thousands of daily workflow runs, the price difference adds up quickly.</p>

<h2>
  
  
  The numbers
</h2>

<p>Current pricing is about <code>$0.0021 per 1M tokens</code> (input + output) for all models at the time of writing. See the <a href="https://gonkagate.com/en/pricing" rel="noopener noreferrer">pricing page</a> for details and fees.</p>

<p>For comparison, here are the official pricing pages of centralized providers:</p>

<ul>
<li><a href="https://platform.openai.com/docs/pricing" rel="noopener noreferrer">OpenAI pricing</a></li>
<li><a href="https://platform.claude.com/docs/en/about-claude/pricing" rel="noopener noreferrer">Anthropic pricing</a></li>
<li><a href="https://ai.google.dev/gemini-api/docs/pricing" rel="noopener noreferrer">Google Gemini pricing</a></li>
</ul>

<p>Simple math: if you currently pay $X per 1M tokens, your GonkaGate budget is roughly <code>current_budget * (0.0021 / X)</code>.</p>

<blockquote>
<p><strong>Important:</strong> These savings are illustrative. Real costs depend on model choice, traffic patterns, and current network load (supply/demand).</p>

<p><strong>Tip:</strong> Use the pricing calculator on the pricing page to estimate your monthly spend based on your actual traffic profile.</p>
</blockquote>

<h2>
  
  
  When it fits (and when it doesn't)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Criterion</th>
<th>GonkaGate</th>
<th>Centralized providers</th>
</tr>
</thead>
<tbody>
<tr>
<td>Budget</td>
<td>Tight</td>
<td>Flexible</td>
</tr>
<tr>
<td>Model quality</td>
<td>Good enough for the task</td>
<td>Frontier-level</td>
</tr>
<tr>
<td>Use case</td>
<td>MVPs, internal tools, n8n workflows</td>
<td>Production, enterprise, critical features</td>
</tr>
<tr>
<td>Models</td>
<td>Prefer open-source</td>
<td>Proprietary is OK</td>
</tr>
<tr>
<td>Stability</td>
<td>Occasional hiccups acceptable</td>
<td>High uptime required</td>
</tr>
<tr>
<td>Vendor lock-in</td>
<td>Want to avoid it</td>
<td>Not a priority</td>
</tr>
</tbody>
</table></div>

<p>If you're closer to the left column, try it. If you need strict SLAs and top-tier quality, stick with centralized providers.</p>

<h2>
  
  
  Limitations and risks
</h2>

<ol>
<li>
<strong>Early-stage network.</strong> Gonka is new. Instability is possible. If your app is mission-critical, plan accordingly.</li>
<li>
<strong>Open-source model ceiling.</strong> <code>qwen3-235b</code> is strong, but it's not a frontier proprietary model. Some tasks will show a gap.</li>
</ol>

<h2>
  
  
  Wrap up
</h2>

<p>If you want a quick way to reduce LLM spend without a big rewrite:</p>

<ol>
<li>Sign up at <a href="https://gonkagate.com/" rel="noopener noreferrer">GonkaGate</a>
</li>
<li>Get an API key</li>
<li>Swap the endpoint in your code</li>
<li>Test on your real use cases</li>
</ol>

<p>Questions about GonkaGate or decentralized inference? Drop them in the comments.</p>

