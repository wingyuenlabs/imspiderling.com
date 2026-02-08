---
Title: How I Built an AI Product Photography Pipeline with 30+ Models (Next.js + Express + Replicate/FAL)
Description: 
Author: Tyler
Date: 2026-02-08T21:11:51.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been building <a href="https://pixelmotion.io" rel="noopener noreferrer">PixelMotion</a> — a SaaS that takes product photos and transforms them into enhanced images and AI-generated videos. Here's a deep dive into the technical architecture and the lessons I learned orchestrating 30+ AI models in production.</p>

<h2>
  
  
  The Problem
</h2>

<p>Ecommerce brands need high-quality product visuals but professional photography is expensive ($500-2000 per product). AI models like Flux, Stable Diffusion, and video generators like Kling, Sora, and Veo can now produce stunning results — but each model has different strengths, APIs, pricing, and failure modes.</p>

<p>I wanted to build a system where a user uploads a single product photo, and the platform handles everything: background removal, enhancement, upscaling, and even generating marketing videos.</p>

<h2>
  
  
  Architecture Overview
</h2>

<p>The stack:</p>

<ul>
<li>
<strong>Frontend</strong>: Next.js 15 (App Router) + TailwindCSS</li>
<li>
<strong>Backend</strong>: Express.js + TypeScript + PostgreSQL (Sequelize ORM)</li>
<li>
<strong>AI Providers</strong>: Replicate, FAL AI, OpenAI</li>
<li>
<strong>Storage</strong>: Google Cloud Storage</li>
<li>
<strong>Payments</strong>: Stripe (usage-based credits)
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User Upload → Website Scraper → AI Analysis → Model Selection → Enhancement/Generation → Storage → Delivery
</code></pre>

</div>



<h2>
  
  
  Key Technical Decisions
</h2>

<h3>
  
  
  1. Multi-Provider Model Orchestration
</h3>

<p>The biggest challenge was abstracting away the differences between AI providers. Replicate uses a prediction-based API with webhooks. FAL uses queue-based async processing. OpenAI has its own patterns.</p>

<p>I built a unified <code>llmService</code> that normalizes the interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Simplified model orchestration</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">llmService</span><span class="p">.</span><span class="nf">generate</span><span class="p">({</span>
  <span class="na">provider</span><span class="p">:</span> <span class="nx">model</span><span class="p">.</span><span class="nx">provider</span><span class="p">,</span> <span class="c1">// 'replicate' | 'fal' | 'openai'</span>
  <span class="na">model</span><span class="p">:</span> <span class="nx">model</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
  <span class="na">input</span><span class="p">:</span> <span class="nx">normalizedParams</span><span class="p">,</span>
  <span class="na">webhook</span><span class="p">:</span> <span class="nx">callbackUrl</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Each provider adapter handles its own retry logic, timeout behavior, and error mapping. This means adding a new model is just a config change — no service code modifications.</p>

<h3>
  
  
  2. Fallback Chains
</h3>

<p>AI models fail. A lot. Rate limits, cold starts, model updates that change output quality. I implemented fallback chains:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">ENHANCEMENT_CHAIN</span> <span class="o">=</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">flux-pro-v2</span><span class="dl">'</span><span class="p">,</span> <span class="na">provider</span><span class="p">:</span> <span class="dl">'</span><span class="s1">replicate</span><span class="dl">'</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">flux-pro</span><span class="dl">'</span><span class="p">,</span> <span class="na">provider</span><span class="p">:</span> <span class="dl">'</span><span class="s1">fal</span><span class="dl">'</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">stable-diffusion-xl</span><span class="dl">'</span><span class="p">,</span> <span class="na">provider</span><span class="p">:</span> <span class="dl">'</span><span class="s1">replicate</span><span class="dl">'</span> <span class="p">},</span>
<span class="p">];</span>
</code></pre>

</div>



<p>If the primary model fails or times out, the system automatically tries the next one. Users don't see errors — they just get results.</p>

<h3>
  
  
  3. Credit-Based Pricing
</h3>

<p>Different models have wildly different costs. Sora costs ~10x more than Kling per generation. Instead of flat subscriptions, I went with a credit system where each model costs a different number of credits:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Model</th>
<th>Credits</th>
<th>Actual Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Flux 2 Pro (photo)</td>
<td>2</td>
<td>~$0.05</td>
</tr>
<tr>
<td>Kling 1.6 (video)</td>
<td>5</td>
<td>~$0.13</td>
</tr>
<tr>
<td>Sora 2 (video)</td>
<td>20</td>
<td>~$0.50</td>
</tr>
<tr>
<td>Veo 3.1 (video)</td>
<td>25</td>
<td>~$0.65</td>
</tr>
</tbody>
</table></div>

<p>This lets users choose based on their budget and quality needs.</p>

<h3>
  
  
  4. Automated Product Intelligence
</h3>

<p>When a user connects their ecommerce store, the platform scrapes product data and uses GPT-4o to analyze:</p>

<ul>
<li>Product category and type</li>
<li>Target audience</li>
<li>Brand aesthetic</li>
<li>Optimal AI models for that product type</li>
</ul>

<p>This analysis feeds into prompt generation, so a luxury watch gets different treatment than a kitchen gadget.</p>

<h3>
  
  
  5. Async Job Queue with Polling
</h3>

<p>AI generation takes 30-120 seconds. I use a job queue pattern:</p>

<ol>
<li>User submits request → job created with <code>pending</code> status</li>
<li>Backend dispatches to AI provider</li>
<li>Frontend polls every 3 seconds via custom <code>usePolling</code> hook</li>
<li>Webhook or poll catches completion → status updated to <code>completed</code>
</li>
<li>Frontend displays result
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Frontend polling hook (simplified)</span>
<span class="kd">const</span> <span class="nx">useGenerationStatus</span> <span class="o">=</span> <span class="p">(</span><span class="nx">jobId</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">status</span><span class="p">,</span> <span class="nx">setStatus</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">pending</span><span class="dl">'</span><span class="p">);</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">interval</span> <span class="o">=</span> <span class="nf">setInterval</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">api</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="s2">`/jobs/</span><span class="p">${</span><span class="nx">jobId</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="nf">setStatus</span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">status</span><span class="p">);</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">completed</span><span class="dl">'</span><span class="p">)</span> <span class="nf">clearInterval</span><span class="p">(</span><span class="nx">interval</span><span class="p">);</span>
    <span class="p">},</span> <span class="mi">3000</span><span class="p">);</span>
    <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">clearInterval</span><span class="p">(</span><span class="nx">interval</span><span class="p">);</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">jobId</span><span class="p">]);</span>

  <span class="k">return</span> <span class="nx">status</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  Lessons Learned
</h2>

<p><strong>1. Don't trust AI model output blindly.</strong> Some models occasionally return blank images, corrupted files, or completely wrong outputs. Always validate outputs before serving to users.</p>

<p><strong>2. Cost management is critical.</strong> Early on, I burned through $200 in a weekend because I didn't have per-user rate limits. Now every generation checks credit balance first, and I have daily cost alerts.</p>

<p><strong>3. Prompt engineering &gt; model selection.</strong> A well-crafted prompt on a cheaper model often beats a bad prompt on an expensive one. I spent weeks refining prompts and built a versioned prompt system to track changes.</p>

<p><strong>4. Webhooks are unreliable.</strong> Both Replicate and FAL occasionally fail to deliver webhooks. Always implement polling as a fallback.</p>

<p><strong>5. Users don't care about the model.</strong> They care about the result. Auto-selecting the best model based on the product type improved satisfaction more than letting users choose manually.</p>

<h2>
  
  
  What's Next
</h2>

<p>Currently working on:</p>

<ul>
<li>Multi-photo video generation (combine multiple product angles into one video)</li>
<li>UGC-style video generation with AI avatars</li>
<li>Direct publishing to TikTok/YouTube from the platform</li>
</ul>

<p>If you're building with AI APIs and want to chat about multi-provider orchestration, fallback patterns, or credit systems — drop a comment. Happy to go deeper on any of these topics.</p>

<p>You can check out the live product at <a href="https://pixelmotion.io" rel="noopener noreferrer">pixelmotion.io</a>.</p>




<p><em>Built with Next.js 15, Express.js, PostgreSQL, Replicate, FAL AI, and OpenAI.</em></p>

