---
Title: How to Generate Images Using LLM Gateway and the Vercel AI SDK
Description: 
Author: smakosh
Date: 2026-02-28T22:14:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>Image generation has become a core feature of modern AI applications. But integrating with multiple image providers — each with different APIs, pricing, and capabilities — can be a pain.</p>

<p><a href="https://llmgateway.io" rel="noopener noreferrer">LLM Gateway</a> simplifies this by giving you a single, OpenAI-compatible API for image generation across providers like Google Gemini, Alibaba Qwen, ByteDance Seedream, and more. In this guide, we'll walk through generating images using both the <strong>OpenAI SDK</strong> and the <strong>Vercel AI SDK</strong>.</p>

<h2>
  
  
  Prerequisites
</h2>

<ol>
<li>Sign up at <a href="https://llmgateway.io/signup" rel="noopener noreferrer">llmgateway.io</a> and create a project</li>
<li>Copy your API key</li>
<li>Export it in your environment:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">LLM_GATEWAY_API_KEY</span><span class="o">=</span><span class="s2">"llmgtwy_XXXXXXXXXXXXXXXX"</span>
</code></pre>

</div>



<h2>
  
  
  Option 1: The Images API (<code>/v1/images/generations</code>)
</h2>

<p>This is the simplest approach — a drop-in replacement for OpenAI's image generation endpoint.</p>

<h3>
  
  
  Using curl
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"https://api.llmgateway.io/v1/images/generations"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$LLM_GATEWAY_API_KEY</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "model": "gemini-3-pro-image-preview",
    "prompt": "A cute cat wearing a tiny top hat",
    "n": 1,
    "size": "1024x1024"
  }'</span>
</code></pre>

</div>



<h3>
  
  
  Using the OpenAI SDK (Node.js)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">OpenAI</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">openai</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">writeFileSync</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">fs</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">OpenAI</span><span class="p">({</span>
  <span class="na">baseURL</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.llmgateway.io/v1</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LLM_GATEWAY_API_KEY</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nx">images</span><span class="p">.</span><span class="nf">generate</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">gemini-3-pro-image-preview</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A futuristic city skyline at sunset with flying cars</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">n</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
  <span class="na">size</span><span class="p">:</span> <span class="dl">"</span><span class="s2">1024x1024</span><span class="dl">"</span><span class="p">,</span>
<span class="p">});</span>

<span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">image</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">image</span><span class="p">.</span><span class="nx">b64_json</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">buf</span> <span class="o">=</span> <span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">image</span><span class="p">.</span><span class="nx">b64_json</span><span class="p">,</span> <span class="dl">"</span><span class="s2">base64</span><span class="dl">"</span><span class="p">);</span>
    <span class="nf">writeFileSync</span><span class="p">(</span><span class="s2">`image-</span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2">.png`</span><span class="p">,</span> <span class="nx">buf</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>That's it — point <code>baseURL</code> to LLM Gateway and use the standard OpenAI SDK. No new libraries needed.</p>

<h2>
  
  
  Option 2: The Vercel AI SDK (<code>generateImage</code>)
</h2>

<p>If you're already using the <a href="https://ai-sdk.dev" rel="noopener noreferrer">Vercel AI SDK</a>, LLM Gateway has a native provider package: <code>@llmgateway/ai-sdk-provider</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @llmgateway/ai-sdk-provider ai
</code></pre>

</div>



<h3>
  
  
  Simple Image Generation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createLLMGateway</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@llmgateway/ai-sdk-provider</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">generateImage</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">ai</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">writeFileSync</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">fs</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">llmgateway</span> <span class="o">=</span> <span class="nf">createLLMGateway</span><span class="p">({</span>
  <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LLM_GATEWAY_API_KEY</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">generateImage</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="nx">llmgateway</span><span class="p">.</span><span class="nf">image</span><span class="p">(</span><span class="dl">"</span><span class="s2">gemini-3-pro-image-preview</span><span class="dl">"</span><span class="p">),</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="dl">"</span><span class="s2">A cozy cabin in a snowy mountain landscape at night with aurora borealis</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">size</span><span class="p">:</span> <span class="dl">"</span><span class="s2">1024x1024</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">n</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
<span class="p">});</span>

<span class="nx">result</span><span class="p">.</span><span class="nx">images</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">image</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">buf</span> <span class="o">=</span> <span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">image</span><span class="p">.</span><span class="nx">base64</span><span class="p">,</span> <span class="dl">"</span><span class="s2">base64</span><span class="dl">"</span><span class="p">);</span>
  <span class="nf">writeFileSync</span><span class="p">(</span><span class="s2">`image-</span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2">.png`</span><span class="p">,</span> <span class="nx">buf</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Conversational Image Generation with <code>streamText</code>
</h3>

<p>The real power comes when you combine image generation with chat. Using the chat completions approach, you can build conversational image generation — ask the model to create an image, then refine it through follow-up messages.</p>

<p>Here's a Next.js API route:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">streamText</span><span class="p">,</span> <span class="kd">type</span> <span class="nx">UIMessage</span><span class="p">,</span> <span class="nx">convertToModelMessages</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">ai</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createLLMGateway</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@llmgateway/ai-sdk-provider</span><span class="dl">"</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">ChatRequestBody</span> <span class="p">{</span>
  <span class="nl">messages</span><span class="p">:</span> <span class="nx">UIMessage</span><span class="p">[];</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">POST</span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">body</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">req</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">messages</span> <span class="p">}:</span> <span class="nx">ChatRequestBody</span> <span class="o">=</span> <span class="nx">body</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">llmgateway</span> <span class="o">=</span> <span class="nf">createLLMGateway</span><span class="p">({</span>
    <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LLM_GATEWAY_API_KEY</span><span class="p">,</span>
    <span class="na">baseUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.llmgateway.io/v1</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">streamText</span><span class="p">({</span>
      <span class="na">model</span><span class="p">:</span> <span class="nx">llmgateway</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span><span class="dl">"</span><span class="s2">gemini-3-pro-image-preview</span><span class="dl">"</span><span class="p">),</span>
      <span class="na">messages</span><span class="p">:</span> <span class="nf">convertToModelMessages</span><span class="p">(</span><span class="nx">messages</span><span class="p">),</span>
    <span class="p">});</span>

    <span class="k">return</span> <span class="nx">result</span><span class="p">.</span><span class="nf">toUIMessageStreamResponse</span><span class="p">();</span>
  <span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nc">Response</span><span class="p">(</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">LLM Gateway Chat request failed</span><span class="dl">"</span> <span class="p">}),</span>
      <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">500</span> <span class="p">},</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>On the frontend, use the <code>useChat</code> hook from <code>@ai-sdk/react</code> and render images from the assistant's response parts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useChat</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@ai-sdk/react</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">ImageChat</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">messages</span><span class="p">,</span> <span class="nx">status</span><span class="p">,</span> <span class="nx">sendMessage</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useChat</span><span class="p">();</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="si">{</span><span class="nx">messages</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">m</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">m</span><span class="p">.</span><span class="nx">role</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">assistant</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">textContent</span> <span class="o">=</span> <span class="nx">m</span><span class="p">.</span><span class="nx">parts</span>
            <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">p</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">p</span><span class="p">.</span><span class="nx">text</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>

          <span class="kd">const</span> <span class="nx">imageParts</span> <span class="o">=</span> <span class="nx">m</span><span class="p">.</span><span class="nx">parts</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span>
            <span class="p">(</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">p</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">file</span><span class="dl">"</span> <span class="o">&amp;&amp;</span> <span class="nx">p</span><span class="p">.</span><span class="nx">mediaType</span><span class="p">?.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">"</span><span class="s2">image/</span><span class="dl">"</span><span class="p">),</span>
          <span class="p">);</span>

          <span class="k">return </span><span class="p">(</span>
            <span class="p">&lt;</span><span class="nt">div</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">m</span><span class="p">.</span><span class="nx">id</span><span class="si">}</span><span class="p">&gt;</span>
              <span class="si">{</span><span class="nx">textContent</span> <span class="o">&amp;&amp;</span> <span class="p">&lt;</span><span class="nt">p</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">textContent</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">p</span><span class="p">&gt;</span><span class="si">}</span>
              <span class="si">{</span><span class="nx">imageParts</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">part</span><span class="p">,</span> <span class="nx">idx</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
                <span class="p">&lt;</span><span class="nt">img</span>
                  <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">idx</span><span class="si">}</span>
                  <span class="na">src</span><span class="p">=</span><span class="si">{</span><span class="s2">`data:</span><span class="p">${</span><span class="nx">part</span><span class="p">.</span><span class="nx">mediaType</span><span class="p">}</span><span class="s2">;base64,</span><span class="p">${</span><span class="nx">part</span><span class="p">.</span><span class="nx">url</span><span class="p">}</span><span class="s2">`</span><span class="si">}</span>
                  <span class="na">alt</span><span class="p">=</span><span class="s">"Generated image"</span>
                <span class="p">/&gt;</span>
              <span class="p">))</span><span class="si">}</span>
            <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
          <span class="p">);</span>
        <span class="p">}</span>

        <span class="k">return </span><span class="p">(</span>
          <span class="p">&lt;</span><span class="nt">div</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">m</span><span class="p">.</span><span class="nx">id</span><span class="si">}</span><span class="p">&gt;</span>
            <span class="si">{</span><span class="nx">m</span><span class="p">.</span><span class="nx">parts</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">p</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span>
              <span class="nx">p</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span> <span class="p">?</span> <span class="p">&lt;</span><span class="nt">p</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">i</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">p</span><span class="p">.</span><span class="nx">text</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">p</span><span class="p">&gt;</span> <span class="p">:</span> <span class="kc">null</span>
            <span class="p">)</span><span class="si">}</span>
          <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
        <span class="p">);</span>
      <span class="p">})</span><span class="si">}</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<p>You can also use the pre-built <code>&lt;Image /&gt;</code> component from <a href="https://ai-sdk.dev/elements/components/image" rel="noopener noreferrer">ai-elements</a> for a polished rendering experience.</p>

<h2>
  
  
  Option 3: Image Editing
</h2>

<p>LLM Gateway also supports editing existing images via <code>/v1/images/edits</code>. Pass an image URL (HTTPS or base64 data URL) along with your edit prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"https://api.llmgateway.io/v1/images/edits"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$LLM_GATEWAY_API_KEY</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "images": [
      {
        "image_url": "https://example.com/source-image.png"
      }
    ],
    "prompt": "Add a watercolor effect to this image",
    "model": "gemini-3-pro-image-preview",
    "quality": "high",
    "size": "1024x1024"
  }'</span>
</code></pre>

</div>



<h2>
  
  
  Switching Providers
</h2>

<p>The best part: switching image providers is a one-line change. Just swap the model string:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Google Gemini</span>
<span class="nx">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">gemini-3-pro-image-preview</span><span class="dl">"</span>

<span class="c1">// Alibaba Qwen (from $0.03/image)</span>
<span class="nx">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">alibaba/qwen-image-plus</span><span class="dl">"</span>

<span class="c1">// ByteDance Seedream (up to 4K output)</span>
<span class="nx">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bytedance/seedream-4-5</span><span class="dl">"</span>

<span class="c1">// Z.AI CogView (great for text in images)</span>
<span class="nx">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">zai/cogview-4</span><span class="dl">"</span>
</code></pre>

</div>



<p>Each provider has different strengths:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Provider</th>
<th>Model</th>
<th>Price</th>
<th>Best For</th>
</tr>
</thead>
<tbody>
<tr>
<td>Google</td>
<td><code>gemini-3-pro-image-preview</code></td>
<td>Varies</td>
<td>General purpose, high quality</td>
</tr>
<tr>
<td>Alibaba</td>
<td><code>alibaba/qwen-image-max</code></td>
<td>$0.075/image</td>
<td>Highest quality</td>
</tr>
<tr>
<td>Alibaba</td>
<td><code>alibaba/qwen-image-plus</code></td>
<td>$0.03/image</td>
<td>Best value</td>
</tr>
<tr>
<td>ByteDance</td>
<td><code>bytedance/seedream-4-5</code></td>
<td>$0.045/image</td>
<td>Up to 4K, multi-image fusion</td>
</tr>
<tr>
<td>Z.AI</td>
<td><code>zai/cogview-4</code></td>
<td>$0.01/image</td>
<td>Cheapest, bilingual text rendering</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Customizing Output
</h2>

<p>Use the <code>image_config</code> parameter (chat completions) or standard <code>size</code>/<code>quality</code> params (images API) to control output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Google: aspect ratio + resolution</span>
<span class="s2">"image_config"</span>: <span class="o">{</span>
  <span class="s2">"aspect_ratio"</span>: <span class="s2">"16:9"</span>,
  <span class="s2">"image_size"</span>: <span class="s2">"4K"</span>
<span class="o">}</span>

<span class="c"># Alibaba: pixel dimensions + seed for reproducibility</span>
<span class="s2">"image_config"</span>: <span class="o">{</span>
  <span class="s2">"image_size"</span>: <span class="s2">"1024x1536"</span>,
  <span class="s2">"seed"</span>: 42
<span class="o">}</span>
</code></pre>

</div>



<h2>
  
  
  Wrapping Up
</h2>

<p>With LLM Gateway, image generation becomes provider-agnostic. Whether you're using the OpenAI SDK, Vercel AI SDK, or raw HTTP — you get one API, multiple providers, and the flexibility to switch models without changing your code.</p>

<p><strong>Resources:</strong></p>

<ul>
<li><a href="https://docs.llmgateway.io/features/image-generation" rel="noopener noreferrer">LLM Gateway Image Generation Docs</a></li>
<li><a href="https://llmgateway.io/models?filters=1&amp;imageGeneration=true" rel="noopener noreferrer">Browse available image models</a></li>
<li><a href="https://chat.llmgateway.io" rel="noopener noreferrer">Try it in the Playground</a></li>
<li><a href="https://www.npmjs.com/package/@llmgateway/ai-sdk-provider" rel="noopener noreferrer">AI SDK Provider on npm</a></li>
</ul>

<p><strong><a href="https://llmgateway.io/signup" rel="noopener noreferrer">Get started →</a></strong></p>

