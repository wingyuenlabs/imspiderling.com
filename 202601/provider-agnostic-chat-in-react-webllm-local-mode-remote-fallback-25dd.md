---
Title: Provider-Agnostic Chat in React: WebLLM Local Mode + Remote Fallback
Description: 
Author: Kaemon Lovendahl
Date: 2026-01-07T20:51:34.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Intro
</h2>

<p>Most LLM apps have the same shape. Ship text to a server, pay per token, and pray the Wi-Fi stays up.</p>

<p>WebLLM is the fun twist. It runs an LLM <strong>inside the browser</strong> using WebGPU. That unlocks privacy-friendly demos, offline-ish behavior, and a new kind of “deployment” where your biggest backend cost is your user’s laptop fan spinning up like it just saw a Dark Souls boss.</p>

<p>The goal here is simple: one chat UI, one message format, two interchangeable “brains”:</p>

<ul>
<li>
<strong>Local provider:</strong> WebLLM in the browser (WebGPU)</li>
<li>
<strong>Remote provider:</strong> a server endpoint with an OpenAI-compatible shape (Next.js route handler)</li>
</ul>

<blockquote>
<p>tldr; Build a tiny chat app where switching between local WebLLM and a remote model is just a dropdown.</p>
</blockquote>

<h2>
  
  
  Setting Up / Prerequisites
</h2>

<ul>
<li>Node 18+ (20+ preferred)</li>
<li>A modern Chromium browser with <strong>WebGPU enabled</strong> (Chrome or Edge is easiest)</li>
<li>Basic React + TypeScript comfort</li>
</ul>

<p>Optional but recommended:</p>

<ul>
<li>A machine with decent RAM. Smaller laptops can run it, but you will feel the pain sooner.</li>
<li>Patience for the first model download.</li>
</ul>

<h2>
  
  
  Implementation Steps
</h2>

<h3>
  
  
  Step 1: Create the app (Vite + React)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm create vite@latest webllm-dual-provider-chat <span class="nt">--</span> <span class="nt">--template</span> react-ts
<span class="nb">cd </span>webllm-dual-provider-chat
npm i
npm i @mlc-ai/web-llm
npm run dev
</code></pre>

</div>



<p>You now have a normal React app that will become a “two-brain” chat UI.</p>

<h3>
  
  
  Step 2: Define a provider interface
</h3>

<p>This interface is the entire trick. The UI does not care <em>how</em> tokens appear, only that they stream in.</p>

<p>In src/ai/types.ts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">Role</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">assistant</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">ChatMessage</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">role</span><span class="p">:</span> <span class="nx">Role</span><span class="p">;</span>
  <span class="nl">content</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">StreamChunk</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">delta</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">done</span><span class="p">?:</span> <span class="nx">boolean</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">ChatProvider</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">label</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>

  <span class="c1">// Called once when selecting this provider (load model, warmup, etc.)</span>
  <span class="nl">init</span><span class="p">?:</span> <span class="p">(</span><span class="nx">opts</span><span class="p">?:</span> <span class="p">{</span>
    <span class="nx">signal</span><span class="p">?:</span> <span class="nx">AbortSignal</span><span class="p">;</span>
    <span class="nl">onStatus</span><span class="p">?:</span> <span class="p">(</span><span class="na">s</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="p">})</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="c1">// Stream response tokens/chunks</span>
  <span class="nl">streamChat</span><span class="p">:</span> <span class="p">(</span><span class="na">args</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">messages</span><span class="p">:</span> <span class="nx">ChatMessage</span><span class="p">[];</span>
    <span class="nl">signal</span><span class="p">?:</span> <span class="nx">AbortSignal</span><span class="p">;</span>
    <span class="nl">onChunk</span><span class="p">:</span> <span class="p">(</span><span class="na">chunk</span><span class="p">:</span> <span class="nx">StreamChunk</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
    <span class="nl">onStatus</span><span class="p">?:</span> <span class="p">(</span><span class="na">s</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
  <span class="p">})</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="c1">// Optional cleanup</span>
  <span class="nl">dispose</span><span class="p">?:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<p>From this point forward, everything is just “implement the interface.”</p>

<h3>
  
  
  Step 3: Implement the WebLLM local provider
</h3>

<p>Two important realities:</p>

<ul>
<li>
<strong>First run downloads a model.</strong> This can be big. Show status text so it does not look frozen.</li>
<li>
<strong>WebGPU is not universal.</strong> Feature detect and fall back.</li>
</ul>

<p>Also, use a model ID that actually works, this one worked at time of writing:</p>

<ul>
<li><code>Llama-3.1-8B-Instruct-q4f32_1-MLC</code></li>
</ul>

<p>In src/ai/webllmProvider.ts<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">ChatMessage</span><span class="p">,</span> <span class="nx">ChatProvider</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./types</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">webllm</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@mlc-ai/web-llm</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">toWebLLMMessages</span><span class="p">(</span>
  <span class="nx">messages</span><span class="p">:</span> <span class="nx">ChatMessage</span><span class="p">[]</span>
<span class="p">):</span> <span class="nx">webllm</span><span class="p">.</span><span class="nx">ChatCompletionMessageParam</span><span class="p">[]</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">messages</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">m</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">role</span><span class="p">:</span> <span class="nx">m</span><span class="p">.</span><span class="nx">role</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">m</span><span class="p">.</span><span class="nx">content</span> <span class="p">}));</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">createWebLLMProvider</span><span class="p">(</span>
  <span class="nx">modelId</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">Llama-3.1-8B-Instruct-q4f32_1-MLC</span><span class="dl">"</span>
<span class="p">):</span> <span class="nx">ChatProvider</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="nx">engine</span><span class="p">:</span> <span class="nx">webllm</span><span class="p">.</span><span class="nx">MLCEngineInterface</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">init</span><span class="p">:</span> <span class="nx">ChatProvider</span><span class="p">[</span><span class="dl">"</span><span class="s2">init</span><span class="dl">"</span><span class="p">]</span> <span class="o">=</span> <span class="k">async </span><span class="p">({</span> <span class="nx">signal</span><span class="p">,</span> <span class="nx">onStatus</span> <span class="p">}</span> <span class="o">=</span> <span class="p">{})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="p">(</span><span class="dl">"</span><span class="s2">gpu</span><span class="dl">"</span> <span class="k">in</span> <span class="nb">navigator</span><span class="p">))</span> <span class="p">{</span>
      <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">WebGPU not available in this browser.</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">engine</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

    <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Initializing WebLLM engine...</span><span class="dl">"</span><span class="p">);</span>
    <span class="nx">engine</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">webllm</span><span class="p">.</span><span class="nc">CreateMLCEngine</span><span class="p">(</span><span class="nx">modelId</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">initProgressCallback</span><span class="p">:</span> <span class="p">(</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">msg</span> <span class="o">=</span>
          <span class="k">typeof</span> <span class="nx">p</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span> <span class="p">?</span> <span class="nx">p</span> <span class="p">:</span> <span class="p">(</span><span class="nx">p</span> <span class="k">as</span> <span class="kr">any</span><span class="p">)?.</span><span class="nx">text</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">Loading model...</span><span class="dl">"</span><span class="p">;</span>
        <span class="nx">onStatus</span><span class="p">?.(</span><span class="nx">msg</span><span class="p">);</span>
      <span class="p">},</span>
    <span class="p">});</span>

    <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Warming up...</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">engine</span><span class="p">.</span><span class="nx">chat</span><span class="p">.</span><span class="nx">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
      <span class="na">messages</span><span class="p">:</span> <span class="p">[{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Say 'ready'.</span><span class="dl">"</span> <span class="p">}],</span>
      <span class="na">temperature</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Ready.</span><span class="dl">"</span><span class="p">);</span>
    <span class="nx">signal</span><span class="p">?.</span><span class="nx">throwIfAborted</span><span class="p">?.();</span>
  <span class="p">};</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">local-webllm</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Local (WebLLM)</span><span class="dl">"</span><span class="p">,</span>
    <span class="nx">init</span><span class="p">,</span>

    <span class="na">streamChat</span><span class="p">:</span> <span class="k">async </span><span class="p">({</span> <span class="nx">messages</span><span class="p">,</span> <span class="nx">signal</span><span class="p">,</span> <span class="nx">onChunk</span><span class="p">,</span> <span class="nx">onStatus</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">engine</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Engine not initialized. Initializing now...</span><span class="dl">"</span><span class="p">);</span>
        <span class="k">await</span> <span class="nf">init</span><span class="p">({</span> <span class="nx">signal</span><span class="p">,</span> <span class="nx">onStatus</span> <span class="p">});</span>
      <span class="p">}</span>
      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">engine</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">WebLLM engine failed to initialize.</span><span class="dl">"</span><span class="p">);</span>

      <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Generating...</span><span class="dl">"</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">resp</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">engine</span><span class="p">.</span><span class="nx">chat</span><span class="p">.</span><span class="nx">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
        <span class="na">messages</span><span class="p">:</span> <span class="nf">toWebLLMMessages</span><span class="p">(</span><span class="nx">messages</span><span class="p">),</span>
        <span class="na">stream</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
        <span class="na">temperature</span><span class="p">:</span> <span class="mf">0.7</span><span class="p">,</span>
      <span class="p">});</span>

      <span class="k">for</span> <span class="k">await </span><span class="p">(</span><span class="kd">const</span> <span class="nx">event</span> <span class="k">of</span> <span class="nx">resp</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">signal</span><span class="p">?.</span><span class="nx">throwIfAborted</span><span class="p">?.();</span>
        <span class="kd">const</span> <span class="nx">delta</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">choices</span><span class="p">?.[</span><span class="mi">0</span><span class="p">]?.</span><span class="nx">delta</span><span class="p">?.</span><span class="nx">content</span> <span class="o">??</span> <span class="dl">""</span><span class="p">;</span>

        <span class="c1">// Optional cleanup if your model spits template markers</span>
        <span class="kd">const</span> <span class="nx">cleaned</span> <span class="o">=</span> <span class="nx">delta</span>
          <span class="p">.</span><span class="nf">replaceAll</span><span class="p">(</span><span class="dl">"</span><span class="s2">&lt;|start_header_id|&gt;</span><span class="dl">"</span><span class="p">,</span> <span class="dl">""</span><span class="p">)</span>
          <span class="p">.</span><span class="nf">replaceAll</span><span class="p">(</span><span class="dl">"</span><span class="s2">&lt;|end_header_id|&gt;</span><span class="dl">"</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">cleaned</span><span class="p">)</span> <span class="nf">onChunk</span><span class="p">({</span> <span class="na">delta</span><span class="p">:</span> <span class="nx">cleaned</span> <span class="p">});</span>
      <span class="p">}</span>

      <span class="nf">onChunk</span><span class="p">({</span> <span class="na">delta</span><span class="p">:</span> <span class="dl">""</span><span class="p">,</span> <span class="na">done</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
      <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Done.</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">},</span>

    <span class="na">dispose</span><span class="p">:</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// Some builds expose engine.dispose(). If not, dropping the reference is fine.</span>
      <span class="c1">// @ts-expect-error optional</span>
      <span class="k">await</span> <span class="nx">engine</span><span class="p">?.</span><span class="nx">dispose</span><span class="p">?.();</span>
      <span class="nx">engine</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="p">},</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>Model IDs can change across releases and builds. If a model ID fails to load you just need to find the updated one.</p>
</blockquote>

<h3>
  
  
  Step 4: Implement the remote provider client
</h3>

<p>Same contract, same streaming shape. The UI should not have to care if the text came from WebGPU wizardry or a server in a trench coat.<br>
You can skip this step if you prefer to only have a local provider.</p>

<p>In src/ai/remoteProvider.ts<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">ChatProvider</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./types</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">createRemoteProvider</span><span class="p">(</span><span class="nx">endpoint</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">/api/chat</span><span class="dl">"</span><span class="p">):</span> <span class="nx">ChatProvider</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">remote</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Remote (Server)</span><span class="dl">"</span><span class="p">,</span>

    <span class="na">streamChat</span><span class="p">:</span> <span class="k">async </span><span class="p">({</span> <span class="nx">messages</span><span class="p">,</span> <span class="nx">signal</span><span class="p">,</span> <span class="nx">onChunk</span><span class="p">,</span> <span class="nx">onStatus</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Contacting server...</span><span class="dl">"</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">endpoint</span><span class="p">,</span> <span class="p">{</span>
        <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span> <span class="p">},</span>
        <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="nx">messages</span> <span class="p">}),</span>
        <span class="nx">signal</span><span class="p">,</span>
      <span class="p">});</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">res</span><span class="p">.</span><span class="nx">ok</span> <span class="o">||</span> <span class="o">!</span><span class="nx">res</span><span class="p">.</span><span class="nx">body</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Remote provider error: </span><span class="p">${</span><span class="nx">res</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Streaming...</span><span class="dl">"</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">reader</span> <span class="o">=</span> <span class="nx">res</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nf">getReader</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">decoder</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TextDecoder</span><span class="p">();</span>

      <span class="k">while </span><span class="p">(</span><span class="kc">true</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">value</span><span class="p">,</span> <span class="nx">done</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">reader</span><span class="p">.</span><span class="nf">read</span><span class="p">();</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">done</span><span class="p">)</span> <span class="k">break</span><span class="p">;</span>

        <span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="nx">decoder</span><span class="p">.</span><span class="nf">decode</span><span class="p">(</span><span class="nx">value</span><span class="p">,</span> <span class="p">{</span> <span class="na">stream</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">text</span><span class="p">)</span> <span class="nf">onChunk</span><span class="p">({</span> <span class="na">delta</span><span class="p">:</span> <span class="nx">text</span> <span class="p">});</span>
      <span class="p">}</span>

      <span class="nf">onChunk</span><span class="p">({</span> <span class="na">delta</span><span class="p">:</span> <span class="dl">""</span><span class="p">,</span> <span class="na">done</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
      <span class="nx">onStatus</span><span class="p">?.(</span><span class="dl">"</span><span class="s2">Done.</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">},</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Step 5: Add a Next.js route handler for <code>/api/chat</code>
</h3>

<p>You can skip this step if you prefer to only have a local provider.</p>

<p>This route handler:</p>

<ul>
<li>receives <code>{ messages }</code> from the client</li>
<li>calls OpenAI’s Responses API with <code>stream: true</code>
</li>
<li>converts the SSE stream into a plain text stream your Vite client already understands</li>
</ul>

<p>In app/api/chat/route.ts<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">runtime</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">edge</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">Role</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">assistant</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">ChatMessage</span> <span class="o">=</span> <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="nx">Role</span><span class="p">;</span> <span class="nl">content</span><span class="p">:</span> <span class="kr">string</span> <span class="p">};</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">POST</span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">messages</span> <span class="p">}</span> <span class="o">=</span> <span class="p">(</span><span class="k">await</span> <span class="nx">req</span><span class="p">.</span><span class="nf">json</span><span class="p">())</span> <span class="k">as</span> <span class="p">{</span> <span class="na">messages</span><span class="p">:</span> <span class="nx">ChatMessage</span><span class="p">[]</span> <span class="p">};</span>

  <span class="kd">const</span> <span class="nx">apiKey</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">OPENAI_API_KEY</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">apiKey</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nc">Response</span><span class="p">(</span><span class="dl">"</span><span class="s2">Missing OPENAI_API_KEY</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">500</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">model</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">OPENAI_MODEL</span> <span class="o">||</span> <span class="dl">"</span><span class="s2">gpt-4o-mini</span><span class="dl">"</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">upstream</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">"</span><span class="s2">https://api.openai.com/v1/responses</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">apiKey</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
      <span class="nx">model</span><span class="p">,</span>
      <span class="na">stream</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">input</span><span class="p">:</span> <span class="nx">messages</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">m</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
        <span class="na">role</span><span class="p">:</span> <span class="nx">m</span><span class="p">.</span><span class="nx">role</span><span class="p">,</span>
        <span class="na">content</span><span class="p">:</span> <span class="p">[{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">input_text</span><span class="dl">"</span><span class="p">,</span> <span class="na">text</span><span class="p">:</span> <span class="nx">m</span><span class="p">.</span><span class="nx">content</span> <span class="p">}],</span>
      <span class="p">})),</span>
      <span class="na">text</span><span class="p">:</span> <span class="p">{</span> <span class="na">format</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span> <span class="p">}</span> <span class="p">},</span>
    <span class="p">}),</span>
  <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">upstream</span><span class="p">.</span><span class="nx">ok</span> <span class="o">||</span> <span class="o">!</span><span class="nx">upstream</span><span class="p">.</span><span class="nx">body</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">errText</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">upstream</span><span class="p">.</span><span class="nf">text</span><span class="p">().</span><span class="k">catch</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="dl">""</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nc">Response</span><span class="p">(</span><span class="s2">`Upstream error (</span><span class="p">${</span><span class="nx">upstream</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">): </span><span class="p">${</span><span class="nx">errText</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">status</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">encoder</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TextEncoder</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">decoder</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TextDecoder</span><span class="p">();</span>

  <span class="kd">let</span> <span class="nx">buffer</span> <span class="o">=</span> <span class="dl">""</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">stream</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">ReadableStream</span><span class="o">&lt;</span><span class="nb">Uint8Array</span><span class="o">&gt;</span><span class="p">({</span>
    <span class="k">async</span> <span class="nf">start</span><span class="p">(</span><span class="nx">controller</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">reader</span> <span class="o">=</span> <span class="nx">upstream</span><span class="p">.</span><span class="nx">body</span><span class="o">!</span><span class="p">.</span><span class="nf">getReader</span><span class="p">();</span>

      <span class="k">try</span> <span class="p">{</span>
        <span class="k">while </span><span class="p">(</span><span class="kc">true</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="p">{</span> <span class="nx">value</span><span class="p">,</span> <span class="nx">done</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">reader</span><span class="p">.</span><span class="nf">read</span><span class="p">();</span>
          <span class="k">if </span><span class="p">(</span><span class="nx">done</span><span class="p">)</span> <span class="k">break</span><span class="p">;</span>

          <span class="nx">buffer</span> <span class="o">+=</span> <span class="nx">decoder</span><span class="p">.</span><span class="nf">decode</span><span class="p">(</span><span class="nx">value</span><span class="p">,</span> <span class="p">{</span> <span class="na">stream</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>

          <span class="c1">// SSE events are separated by a blank line</span>
          <span class="kd">let</span> <span class="nx">idx</span><span class="p">;</span>
          <span class="k">while </span><span class="p">((</span><span class="nx">idx</span> <span class="o">=</span> <span class="nx">buffer</span><span class="p">.</span><span class="nf">indexOf</span><span class="p">(</span><span class="dl">"</span><span class="se">\n\n</span><span class="dl">"</span><span class="p">))</span> <span class="o">!==</span> <span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">const</span> <span class="nx">rawEvent</span> <span class="o">=</span> <span class="nx">buffer</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">idx</span><span class="p">);</span>
            <span class="nx">buffer</span> <span class="o">=</span> <span class="nx">buffer</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="nx">idx</span> <span class="o">+</span> <span class="mi">2</span><span class="p">);</span>

            <span class="kd">const</span> <span class="nx">dataLines</span> <span class="o">=</span> <span class="nx">rawEvent</span>
              <span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="dl">"</span><span class="se">\n</span><span class="dl">"</span><span class="p">)</span>
              <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">line</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">line</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">"</span><span class="s2">data:</span><span class="dl">"</span><span class="p">))</span>
              <span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">line</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">line</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/^data:</span><span class="se">\s?</span><span class="sr">/</span><span class="p">,</span> <span class="dl">""</span><span class="p">).</span><span class="nf">trim</span><span class="p">());</span>

            <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">data</span> <span class="k">of</span> <span class="nx">dataLines</span><span class="p">)</span> <span class="p">{</span>
              <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">data</span><span class="p">)</span> <span class="k">continue</span><span class="p">;</span>

              <span class="k">if </span><span class="p">(</span><span class="nx">data</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">[DONE]</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
                <span class="nx">controller</span><span class="p">.</span><span class="nf">close</span><span class="p">();</span>
                <span class="k">return</span><span class="p">;</span>
              <span class="p">}</span>

              <span class="kd">let</span> <span class="na">evt</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span>
              <span class="k">try</span> <span class="p">{</span>
                <span class="nx">evt</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">data</span><span class="p">);</span>
              <span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
                <span class="k">continue</span><span class="p">;</span>
              <span class="p">}</span>

              <span class="k">if </span><span class="p">(</span>
                <span class="nx">evt</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">response.output_text.delta</span><span class="dl">"</span> <span class="o">&amp;&amp;</span>
                <span class="k">typeof</span> <span class="nx">evt</span><span class="p">.</span><span class="nx">delta</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span>
              <span class="p">)</span> <span class="p">{</span>
                <span class="nx">controller</span><span class="p">.</span><span class="nf">enqueue</span><span class="p">(</span><span class="nx">encoder</span><span class="p">.</span><span class="nf">encode</span><span class="p">(</span><span class="nx">evt</span><span class="p">.</span><span class="nx">delta</span><span class="p">));</span>
              <span class="p">}</span>
            <span class="p">}</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">controller</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">e</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
        <span class="nx">controller</span><span class="p">.</span><span class="nf">close</span><span class="p">();</span>
      <span class="p">}</span>
    <span class="p">},</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="k">new</span> <span class="nc">Response</span><span class="p">(</span><span class="nx">stream</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
      <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text/plain; charset=utf-8</span><span class="dl">"</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">Cache-Control</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">no-cache, no-transform</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Running Next.js alongside Vite without CORS pain
</h4>

<p>If the chat UI is running on Vite (<code>localhost:5173</code>) and Next.js is running on <code>localhost:3000</code>, calling <code>/api/chat</code> from Vite will hit Vite’s server, not Next. The easy fix is a dev proxy.</p>

<p>Update <code>vite.config.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">vite</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">react</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@vitejs/plugin-react</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">plugins</span><span class="p">:</span> <span class="p">[</span><span class="nf">react</span><span class="p">()],</span>
  <span class="na">server</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">proxy</span><span class="p">:</span> <span class="p">{</span>
      <span class="dl">"</span><span class="s2">/api</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">http://localhost:3000</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Now the client can keep using <code>createRemoteProvider("/api/chat")</code> and Vite will forward it to Next.</p>

<h4>
  
  
  Environment variables for Next.js
</h4>

<p>Create <code>.env.local</code> in the Next.js project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">OPENAI_API_KEY</span><span class="o">=</span>your_key_here
<span class="nv">OPENAI_MODEL</span><span class="o">=</span>gpt-4o-mini
</code></pre>

</div>



<h3>
  
  
  Step 6: Build the chat hook (provider-agnostic brain socket)
</h3>

<p>The whole job of this hook is to:</p>

<ul>
<li>manage messages</li>
<li>manage streaming state</li>
<li>route the “append these tokens” events into the last assistant message</li>
</ul>

<p>The sharp edge: streaming makes state bugs very obvious. If you mutate the last message in place, React will punish you with duplication weirdness, especially in dev.</p>

<p>So we update the last message <strong>immutably</strong>.</p>

<p>In src/hooks/useChat.ts<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useMemo</span><span class="p">,</span> <span class="nx">useRef</span><span class="p">,</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">react</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">ChatMessage</span><span class="p">,</span> <span class="nx">ChatProvider</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../ai/types</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">useChat</span><span class="p">(</span><span class="nx">providers</span><span class="p">:</span> <span class="nx">ChatProvider</span><span class="p">[])</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">providerId</span><span class="p">,</span> <span class="nx">setProviderId</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="nx">providers</span><span class="p">[</span><span class="mi">0</span><span class="p">]?.</span><span class="nx">id</span> <span class="o">??</span> <span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">provider</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(</span>
    <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">providers</span><span class="p">.</span><span class="nf">find</span><span class="p">((</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">p</span><span class="p">.</span><span class="nx">id</span> <span class="o">===</span> <span class="nx">providerId</span><span class="p">)</span><span class="o">!</span><span class="p">,</span>
    <span class="p">[</span><span class="nx">providers</span><span class="p">,</span> <span class="nx">providerId</span><span class="p">]</span>
  <span class="p">);</span>

  <span class="kd">const</span> <span class="p">[</span><span class="nx">messages</span><span class="p">,</span> <span class="nx">setMessages</span><span class="p">]</span> <span class="o">=</span> <span class="nx">useState</span><span class="o">&lt;</span><span class="nx">ChatMessage</span><span class="p">[]</span><span class="o">&gt;</span><span class="p">([</span>
    <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="dl">"</span><span class="s2">You are a helpful assistant.</span><span class="dl">"</span> <span class="p">},</span>
  <span class="p">]);</span>

  <span class="kd">const</span> <span class="p">[</span><span class="nx">status</span><span class="p">,</span> <span class="nx">setStatus</span><span class="p">]</span> <span class="o">=</span> <span class="nx">useState</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">isStreaming</span><span class="p">,</span> <span class="nx">setIsStreaming</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>

  <span class="kd">const</span> <span class="nx">abortRef</span> <span class="o">=</span> <span class="nx">useRef</span><span class="o">&lt;</span><span class="nx">AbortController</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>

  <span class="k">async</span> <span class="kd">function</span> <span class="nf">selectProvider</span><span class="p">(</span><span class="nx">nextId</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">abortRef</span><span class="p">.</span><span class="nx">current</span><span class="p">?.</span><span class="nf">abort</span><span class="p">();</span>
    <span class="nf">setProviderId</span><span class="p">(</span><span class="nx">nextId</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">next</span> <span class="o">=</span> <span class="nx">providers</span><span class="p">.</span><span class="nf">find</span><span class="p">((</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">p</span><span class="p">.</span><span class="nx">id</span> <span class="o">===</span> <span class="nx">nextId</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">next</span><span class="p">?.</span><span class="nx">init</span><span class="p">)</span> <span class="p">{</span>
      <span class="nf">setStatus</span><span class="p">(</span><span class="dl">"</span><span class="s2">Preparing provider...</span><span class="dl">"</span><span class="p">);</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">next</span><span class="p">.</span><span class="nf">init</span><span class="p">({</span> <span class="na">onStatus</span><span class="p">:</span> <span class="nx">setStatus</span> <span class="p">});</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
        <span class="nf">setStatus</span><span class="p">(</span><span class="nx">e</span><span class="p">?.</span><span class="nx">message</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">Failed to initialize provider.</span><span class="dl">"</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="kd">function</span> <span class="nf">send</span><span class="p">(</span><span class="nx">userText</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">userText</span><span class="p">.</span><span class="nf">trim</span><span class="p">())</span> <span class="k">return</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">isStreaming</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

    <span class="nx">abortRef</span><span class="p">.</span><span class="nx">current</span><span class="p">?.</span><span class="nf">abort</span><span class="p">();</span>
    <span class="nx">abortRef</span><span class="p">.</span><span class="nx">current</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AbortController</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">userMsg</span><span class="p">:</span> <span class="nx">ChatMessage</span> <span class="o">=</span> <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">userText</span> <span class="p">};</span>

    <span class="c1">// Add user + placeholder assistant</span>
    <span class="nf">setMessages</span><span class="p">((</span><span class="nx">prev</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">[...</span><span class="nx">prev</span><span class="p">,</span> <span class="nx">userMsg</span><span class="p">,</span> <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">assistant</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="dl">""</span> <span class="p">}]);</span>
    <span class="nf">setIsStreaming</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="nf">setStatus</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">provider</span><span class="p">.</span><span class="nf">streamChat</span><span class="p">({</span>
        <span class="na">messages</span><span class="p">:</span> <span class="p">[...</span><span class="nx">messages</span><span class="p">,</span> <span class="nx">userMsg</span><span class="p">],</span> <span class="c1">// good enough for a demo</span>
        <span class="na">signal</span><span class="p">:</span> <span class="nx">abortRef</span><span class="p">.</span><span class="nx">current</span><span class="p">.</span><span class="nx">signal</span><span class="p">,</span>
        <span class="na">onStatus</span><span class="p">:</span> <span class="nx">setStatus</span><span class="p">,</span>
        <span class="na">onChunk</span><span class="p">:</span> <span class="p">({</span> <span class="nx">delta</span><span class="p">,</span> <span class="nx">done</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="k">if </span><span class="p">(</span><span class="nx">delta</span><span class="p">)</span> <span class="p">{</span>
            <span class="nf">setMessages</span><span class="p">((</span><span class="nx">prev</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
              <span class="kd">const</span> <span class="nx">last</span> <span class="o">=</span> <span class="nx">prev</span><span class="p">[</span><span class="nx">prev</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">1</span><span class="p">];</span>
              <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">last</span> <span class="o">||</span> <span class="nx">last</span><span class="p">.</span><span class="nx">role</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">assistant</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="nx">prev</span><span class="p">;</span>

              <span class="c1">// Immutable update</span>
              <span class="kd">const</span> <span class="nx">updatedLast</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">last</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">last</span><span class="p">.</span><span class="nx">content</span> <span class="o">+</span> <span class="nx">delta</span> <span class="p">};</span>
              <span class="k">return</span> <span class="p">[...</span><span class="nx">prev</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">),</span> <span class="nx">updatedLast</span><span class="p">];</span>
            <span class="p">});</span>
          <span class="p">}</span>

          <span class="k">if </span><span class="p">(</span><span class="nx">done</span><span class="p">)</span> <span class="nf">setIsStreaming</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
        <span class="p">},</span>
      <span class="p">});</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
      <span class="nf">setIsStreaming</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
      <span class="nf">setStatus</span><span class="p">(</span><span class="nx">e</span><span class="p">?.</span><span class="nx">message</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">Error while streaming.</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="kd">function</span> <span class="nf">stop</span><span class="p">()</span> <span class="p">{</span>
    <span class="nx">abortRef</span><span class="p">.</span><span class="nx">current</span><span class="p">?.</span><span class="nf">abort</span><span class="p">();</span>
    <span class="nf">setIsStreaming</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="nf">setStatus</span><span class="p">(</span><span class="dl">"</span><span class="s2">Stopped.</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="nx">providers</span><span class="p">,</span>
    <span class="nx">providerId</span><span class="p">,</span>
    <span class="nx">provider</span><span class="p">,</span>
    <span class="nx">messages</span><span class="p">,</span>
    <span class="nx">status</span><span class="p">,</span>
    <span class="nx">isStreaming</span><span class="p">,</span>
    <span class="nx">selectProvider</span><span class="p">,</span>
    <span class="nx">send</span><span class="p">,</span>
    <span class="nx">stop</span><span class="p">,</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>React state closure note: <code>messages: [...messages, userMsg]</code> uses the current render’s <code>messages</code>. For normal chat usage, that is fine. If you want to harden it, store <code>messages</code> in a ref and read from that when starting the stream.</p>
</blockquote>

<h3>
  
  
  Step 7: UI component
</h3>

<p>Keep it simple. Treat the provider dropdown as a “brain toggle” and let the rest of the UI stay boring on purpose.</p>

<p>In src/App.tsx<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useEffect</span><span class="p">,</span> <span class="nx">useMemo</span><span class="p">,</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">react</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createWebLLMProvider</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./ai/webllmProvider</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createRemoteProvider</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./ai/remoteProvider</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useChat</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./hooks/useChat</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">App</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">providers</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(</span>
    <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="nf">createWebLLMProvider</span><span class="p">(),</span> <span class="nf">createRemoteProvider</span><span class="p">(</span><span class="dl">"</span><span class="s2">/api/chat</span><span class="dl">"</span><span class="p">)],</span>
    <span class="p">[]</span>
  <span class="p">);</span>

  <span class="kd">const</span> <span class="nx">chat</span> <span class="o">=</span> <span class="nf">useChat</span><span class="p">(</span><span class="nx">providers</span><span class="p">);</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">input</span><span class="p">,</span> <span class="nx">setInput</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">chat</span><span class="p">.</span><span class="nf">selectProvider</span><span class="p">(</span><span class="nx">chat</span><span class="p">.</span><span class="nx">providerId</span><span class="p">);</span>
    <span class="c1">// eslint-disable-next-line react-hooks/exhaustive-deps</span>
  <span class="p">},</span> <span class="p">[]);</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">maxWidth</span><span class="p">:</span> <span class="mi">900</span><span class="p">,</span> <span class="na">margin</span><span class="p">:</span> <span class="dl">"</span><span class="s2">0 auto</span><span class="dl">"</span><span class="p">,</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">16</span><span class="p">,</span> <span class="na">fontFamily</span><span class="p">:</span> <span class="dl">"</span><span class="s2">system-ui</span><span class="dl">"</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">h1</span><span class="p">&gt;</span>Dual Provider Chat<span class="p">&lt;/</span><span class="nt">h1</span><span class="p">&gt;</span>

      <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">display</span><span class="p">:</span> <span class="dl">"</span><span class="s2">flex</span><span class="dl">"</span><span class="p">,</span> <span class="na">gap</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">alignItems</span><span class="p">:</span> <span class="dl">"</span><span class="s2">center</span><span class="dl">"</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">label</span><span class="p">&gt;</span>
          Provider<span class="si">{</span><span class="dl">"</span><span class="s2"> </span><span class="dl">"</span><span class="si">}</span>
          <span class="p">&lt;</span><span class="nt">select</span>
            <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">providerId</span><span class="si">}</span>
            <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">chat</span><span class="p">.</span><span class="nf">selectProvider</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)</span><span class="si">}</span>
            <span class="na">disabled</span><span class="p">=</span><span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">isStreaming</span><span class="si">}</span>
          <span class="p">&gt;</span>
            <span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">providers</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
              <span class="p">&lt;</span><span class="nt">option</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">p</span><span class="p">.</span><span class="nx">id</span><span class="si">}</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">p</span><span class="p">.</span><span class="nx">id</span><span class="si">}</span><span class="p">&gt;</span>
                <span class="si">{</span><span class="nx">p</span><span class="p">.</span><span class="nx">label</span><span class="si">}</span>
              <span class="p">&lt;/</span><span class="nt">option</span><span class="p">&gt;</span>
            <span class="p">))</span><span class="si">}</span>
          <span class="p">&lt;/</span><span class="nt">select</span><span class="p">&gt;</span>
        <span class="p">&lt;/</span><span class="nt">label</span><span class="p">&gt;</span>

        <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">opacity</span><span class="p">:</span> <span class="mf">0.8</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">status</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">isStreaming</span> <span class="o">&amp;&amp;</span> <span class="p">&lt;</span><span class="nt">button</span> <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">stop</span><span class="si">}</span><span class="p">&gt;</span>Stop<span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span><span class="si">}</span>
      <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>

      <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">marginTop</span><span class="p">:</span> <span class="mi">16</span><span class="p">,</span> <span class="na">border</span><span class="p">:</span> <span class="dl">"</span><span class="s2">1px solid #ddd</span><span class="dl">"</span><span class="p">,</span> <span class="na">borderRadius</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">minHeight</span><span class="p">:</span> <span class="mi">300</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">messages</span>
          <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">m</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">m</span><span class="p">.</span><span class="nx">role</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span><span class="p">)</span>
          <span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">m</span><span class="p">,</span> <span class="nx">idx</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
            <span class="p">&lt;</span><span class="nt">div</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">idx</span><span class="si">}</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">marginBottom</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
              <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">fontWeight</span><span class="p">:</span> <span class="mi">700</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">m</span><span class="p">.</span><span class="nx">role</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
              <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">whiteSpace</span><span class="p">:</span> <span class="dl">"</span><span class="s2">pre-wrap</span><span class="dl">"</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">m</span><span class="p">.</span><span class="nx">content</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
            <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
          <span class="p">))</span><span class="si">}</span>
      <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>

      <span class="p">&lt;</span><span class="nt">form</span>
        <span class="na">onSubmit</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
          <span class="nx">chat</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="nx">input</span><span class="p">);</span>
          <span class="nf">setInput</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>
        <span class="p">}</span><span class="si">}</span>
        <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">display</span><span class="p">:</span> <span class="dl">"</span><span class="s2">flex</span><span class="dl">"</span><span class="p">,</span> <span class="na">gap</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span> <span class="na">marginTop</span><span class="p">:</span> <span class="mi">12</span> <span class="p">}</span><span class="si">}</span>
      <span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">input</span>
          <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">input</span><span class="si">}</span>
          <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">setInput</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)</span><span class="si">}</span>
          <span class="na">placeholder</span><span class="p">=</span><span class="s">"Say something..."</span>
          <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">flex</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">10</span> <span class="p">}</span><span class="si">}</span>
          <span class="na">disabled</span><span class="p">=</span><span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">isStreaming</span><span class="si">}</span>
        <span class="p">/&gt;</span>
        <span class="p">&lt;</span><span class="nt">button</span> <span class="na">type</span><span class="p">=</span><span class="s">"submit"</span> <span class="na">disabled</span><span class="p">=</span><span class="si">{</span><span class="nx">chat</span><span class="p">.</span><span class="nx">isStreaming</span><span class="si">}</span><span class="p">&gt;</span>
          Send
        <span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span>
      <span class="p">&lt;/</span><span class="nt">form</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>At this point you have:</p>

<ul>
<li>a local WebGPU chat provider</li>
<li>a remote API chat provider</li>
<li>a UI that can swap between them without rewriting anything</li>
</ul>

<h2>
  
  
  Why this setup is worth having
</h2>

<h3>
  
  
  1) Privacy-first features without a backend
</h3>

<p>If the user’s text is sensitive (journaling, medical notes, internal docs), local mode keeps content on-device by default.</p>

<h3>
  
  
  2) Cost control and “free” demos
</h3>

<p>Local mode is effectively “free per token” after the download. It is great for:</p>

<ul>
<li>prototypes</li>
<li>workshops</li>
<li>dev tooling</li>
<li>weekend projects that should not come with a monthly bill</li>
</ul>

<h3>
  
  
  3) Graceful degradation
</h3>

<p>Local mode can be an upgrade path instead of a requirement:</p>

<ul>
<li>WebGPU available: local</li>
<li>WebGPU missing: remote fallback</li>
</ul>

<h3>
  
  
  4) Offline-ish UX for specific workflows
</h3>

<p>Full offline is tricky, but “no server call needed for this” is still a huge win for:</p>

<ul>
<li>rewriting text</li>
<li>summarizing</li>
<li>quick Q&amp;A over content already in the browser</li>
</ul>

<h2>
  
  
  Real Talk
</h2>

<ul>
<li>
<strong>Download time:</strong> first load can be chunky. People will assume it is broken unless you show progress.</li>
<li>
<strong>Device limits:</strong> mobile can struggle. Low-RAM machines can crash tabs or throttle hard.</li>
<li>
<strong>WebGPU support:</strong> treat local mode as progressive enhancement, not a hard dependency.</li>
<li>
<strong>Privacy win:</strong> local mode avoids shipping user text to your server by default.</li>
<li>
<strong>Cost win:</strong> local mode shifts the cost to user compute, which is nice until it is not.</li>
</ul>

<h2>
  
  
  Watch outs and gotchas
</h2>

<h3>
  
  
  Token junk like <code>&lt;|start_header_id|&gt;</code>
</h3>

<p>Some model builds emit template markers. Filtering them out is fine for demos. For cleaner output long-term, experiment with model choices and chat templates.</p>

<h3>
  
  
  Local models are not remote models
</h3>

<p>Expect differences:</p>

<ul>
<li>weaker instruction following</li>
<li>more formatting quirks</li>
<li>occasional “why are you like this” moments</li>
</ul>

<h2>
  
  
  Possible Improvements
</h2>

<ol>
<li><strong>Model picker UI</strong></li>
</ol>

<ul>
<li>dropdown of model IDs</li>
<li>persist selection in <code>localStorage</code>
</li>
<li>show estimated download size if available</li>
</ul>

<ol>
<li><strong>Provider router</strong></li>
</ol>

<ul>
<li>auto-pick local if WebGPU exists</li>
<li>auto-fallback to remote if init fails</li>
<li>show a small badge: “Local” or “Remote”</li>
</ul>

<ol>
<li><strong>Conversation memory controls</strong></li>
</ol>

<ul>
<li>send last N messages only</li>
<li>auto-summarize older messages (local if possible)</li>
</ul>

<ol>
<li><strong>Structured output mode</strong></li>
</ol>

<ul>
<li>have the assistant return JSON “actions”</li>
<li>validate with zod before rendering anything</li>
</ul>

<h2>
  
  
  Outro
</h2>

<p>A provider boundary is one of those small architectural choices that pays rent forever. Models change, vendors change, pricing changes, browser capabilities evolve. A chat UI that can swap brains is a lot harder to paint into a corner.</p>

<p>Also, it is extremely satisfying to flip a dropdown and make your browser turn into a tiny AI workstation.</p>

