---
Title: Building ClipSearch: AI-Powered Product Sourcing for Interior Designers
Description: 
Author: Ben Stone
Date: 2026-01-20T21:58:22.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building ClipSearch: AI-Powered Product Sourcing for Interior Designers
</h1>

<p>I just launched <a href="https://designshelf.biz" rel="noopener noreferrer">ClipSearch</a>, a product sourcing tool for interior designers. This post covers the technical architecture, interesting challenges, and what I learned building a Chrome extension + web app with AI-powered search.</p>

<h2>
  
  
  The Problem Space
</h2>

<p>Interior designers browse hundreds of products across dozens of websites every week. They need to:</p>

<ul>
<li>Save products from any e-commerce site</li>
<li>Organize items by project/client</li>
<li>Find products later (often by vague descriptions)</li>
<li>Present curated selections to clients</li>
</ul>

<p>Traditional solutions (bookmarks, screenshots, Pinterest) break down at scale.</p>

<h2>
  
  
  Tech Stack
</h2>

<p><strong>Frontend:</strong></p>

<ul>
<li>Next.js 14 (App Router)</li>
<li>TypeScript</li>
<li>Tailwind CSS</li>
<li>Shadcn/ui components</li>
</ul>

<p><strong>Backend:</strong></p>

<ul>
<li>Supabase (PostgreSQL + Auth)</li>
<li>pgvector extension for semantic search</li>
<li>OpenAI API (text-embedding-3-small model)</li>
</ul>

<p><strong>Chrome Extension:</strong></p>

<ul>
<li>Manifest V3</li>
<li>Content scripts for product scraping</li>
<li>Background service worker for API calls</li>
</ul>

<p><strong>Deployment:</strong></p>

<ul>
<li>Fly.io (Next.js app)</li>
<li>Supabase cloud (database)</li>
<li>Chrome Web Store (extension)</li>
</ul>

<h2>
  
  
  Architecture Overview
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────┐
│ Chrome Extension│ ──┐
└─────────────────┘   │
                      ├──&gt; Next.js API Routes
┌─────────────────┐   │         │
│   Web App       │ ──┘         │
└─────────────────┘             ▼
                         ┌──────────────┐
                         │  Supabase    │
                         │  PostgreSQL  │
                         │  + pgvector  │
                         └──────────────┘
                                │
                                ▼
                         ┌──────────────┐
                         │  OpenAI API  │
                         │  Embeddings  │
                         └──────────────┘
</code></pre>

</div>



<h2>
  
  
  Key Features &amp; Implementation
</h2>

<h3>
  
  
  1. Chrome Extension Product Clipping
</h3>

<p>The extension uses content scripts to extract product information from any website:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// content-script.ts</span>
<span class="kd">function</span> <span class="nf">extractProductInfo</span><span class="p">():</span> <span class="nx">ProductData</span> <span class="p">{</span>
  <span class="c1">// Try Open Graph meta tags first</span>
  <span class="kd">const</span> <span class="nx">ogImage</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">meta[property="og:image"]</span><span class="dl">'</span><span class="p">)?.</span><span class="nx">content</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">ogTitle</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">meta[property="og:title"]</span><span class="dl">'</span><span class="p">)?.</span><span class="nx">content</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">ogPrice</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">meta[property="og:price:amount"]</span><span class="dl">'</span><span class="p">)?.</span><span class="nx">content</span><span class="p">;</span>

  <span class="c1">// Fallback: heuristic detection</span>
  <span class="kd">const</span> <span class="nx">priceElement</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">[class*="price"], [id*="price"]</span><span class="dl">'</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">titleElement</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">h1, [class*="product-title"]</span><span class="dl">'</span><span class="p">);</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">url</span><span class="p">:</span> <span class="nb">window</span><span class="p">.</span><span class="nx">location</span><span class="p">.</span><span class="nx">href</span><span class="p">,</span>
    <span class="na">title</span><span class="p">:</span> <span class="nx">ogTitle</span> <span class="o">||</span> <span class="nx">titleElement</span><span class="p">?.</span><span class="nx">textContent</span> <span class="o">||</span> <span class="nb">document</span><span class="p">.</span><span class="nx">title</span><span class="p">,</span>
    <span class="na">price</span><span class="p">:</span> <span class="nx">ogPrice</span> <span class="o">||</span> <span class="nf">extractPriceFromText</span><span class="p">(</span><span class="nx">priceElement</span><span class="p">?.</span><span class="nx">textContent</span><span class="p">),</span>
    <span class="na">imageUrl</span><span class="p">:</span> <span class="nx">ogImage</span> <span class="o">||</span> <span class="nf">findLargestImage</span><span class="p">(),</span>
    <span class="na">source</span><span class="p">:</span> <span class="k">new</span> <span class="nc">URL</span><span class="p">(</span><span class="nb">window</span><span class="p">.</span><span class="nx">location</span><span class="p">.</span><span class="nx">href</span><span class="p">).</span><span class="nx">hostname</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Challenge:</strong> Different e-commerce sites structure their HTML differently. Solution: Multi-layered extraction (Open Graph → schema.org → heuristics).</p>

<h3>
  
  
  2. AI Semantic Search
</h3>

<p>Users can search by describing items ("find mid-century credenzas under $2000") rather than exact keywords.</p>

<p><strong>Implementation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// app/api/search/route.ts</span>
<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">POST</span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">query</span><span class="p">,</span> <span class="nx">userId</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">req</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

  <span class="c1">// Generate embedding for search query</span>
  <span class="kd">const</span> <span class="nx">embedding</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">openai</span><span class="p">.</span><span class="nx">embeddings</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
    <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text-embedding-3-small</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">input</span><span class="p">:</span> <span class="nx">query</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="c1">// pgvector similarity search</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span><span class="p">,</span> <span class="nx">error</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">supabase</span><span class="p">.</span><span class="nf">rpc</span><span class="p">(</span><span class="dl">'</span><span class="s1">search_clips</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">query_embedding</span><span class="p">:</span> <span class="nx">embedding</span><span class="p">.</span><span class="nx">data</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">embedding</span><span class="p">,</span>
    <span class="na">match_threshold</span><span class="p">:</span> <span class="mf">0.7</span><span class="p">,</span>
    <span class="na">match_count</span><span class="p">:</span> <span class="mi">20</span><span class="p">,</span>
    <span class="na">user_id</span><span class="p">:</span> <span class="nx">userId</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="nx">Response</span><span class="p">.</span><span class="nf">json</span><span class="p">(</span><span class="nx">data</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Database function:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">FUNCTION</span> <span class="n">search_clips</span><span class="p">(</span>
  <span class="n">query_embedding</span> <span class="n">vector</span><span class="p">(</span><span class="mi">1536</span><span class="p">),</span>
  <span class="n">match_threshold</span> <span class="nb">float</span><span class="p">,</span>
  <span class="n">match_count</span> <span class="nb">int</span><span class="p">,</span>
  <span class="n">user_id</span> <span class="n">uuid</span>
<span class="p">)</span>
<span class="k">RETURNS</span> <span class="k">TABLE</span> <span class="p">(</span>
  <span class="n">id</span> <span class="n">uuid</span><span class="p">,</span>
  <span class="n">title</span> <span class="nb">text</span><span class="p">,</span>
  <span class="n">similarity</span> <span class="nb">float</span>
<span class="p">)</span>
<span class="k">LANGUAGE</span> <span class="n">plpgsql</span>
<span class="k">AS</span> <span class="err">$$</span>
<span class="k">BEGIN</span>
  <span class="k">RETURN</span> <span class="n">QUERY</span>
  <span class="k">SELECT</span>
    <span class="n">clips</span><span class="p">.</span><span class="n">id</span><span class="p">,</span>
    <span class="n">clips</span><span class="p">.</span><span class="n">title</span><span class="p">,</span>
    <span class="mi">1</span> <span class="o">-</span> <span class="p">(</span><span class="n">clips</span><span class="p">.</span><span class="n">embedding</span> <span class="o">&lt;=&gt;</span> <span class="n">query_embedding</span><span class="p">)</span> <span class="k">as</span> <span class="n">similarity</span>
  <span class="k">FROM</span> <span class="n">clips</span>
  <span class="k">WHERE</span> <span class="n">clips</span><span class="p">.</span><span class="n">user_id</span> <span class="o">=</span> <span class="n">user_id</span>
    <span class="k">AND</span> <span class="mi">1</span> <span class="o">-</span> <span class="p">(</span><span class="n">clips</span><span class="p">.</span><span class="n">embedding</span> <span class="o">&lt;=&gt;</span> <span class="n">query_embedding</span><span class="p">)</span> <span class="o">&gt;</span> <span class="n">match_threshold</span>
  <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">clips</span><span class="p">.</span><span class="n">embedding</span> <span class="o">&lt;=&gt;</span> <span class="n">query_embedding</span>
  <span class="k">LIMIT</span> <span class="n">match_count</span><span class="p">;</span>
<span class="k">END</span><span class="p">;</span>
<span class="err">$$</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Why pgvector?</strong></p>

<ul>
<li>Native PostgreSQL extension (no separate vector DB)</li>
<li>Scales to millions of vectors</li>
<li>Cosine similarity operator (<code>&lt;=&gt;</code>) is fast</li>
<li>Works seamlessly with existing relational data</li>
</ul>

<h3>
  
  
  3. Visual Image Search
</h3>

<p>Users can upload inspiration photos and find similar items they've saved.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Using CLIP-like approach</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">visualSearch</span><span class="p">(</span><span class="nx">imageFile</span><span class="p">:</span> <span class="nx">File</span><span class="p">,</span> <span class="nx">userId</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Convert image to embedding</span>
  <span class="kd">const</span> <span class="nx">imageEmbedding</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">openai</span><span class="p">.</span><span class="nx">embeddings</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
    <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text-embedding-3-small</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">input</span><span class="p">:</span> <span class="k">await</span> <span class="nf">imageToBase64</span><span class="p">(</span><span class="nx">imageFile</span><span class="p">),</span>
  <span class="p">});</span>

  <span class="c1">// Same pgvector search, but with image embedding</span>
  <span class="k">return</span> <span class="nx">supabase</span><span class="p">.</span><span class="nf">rpc</span><span class="p">(</span><span class="dl">'</span><span class="s1">search_clips</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">query_embedding</span><span class="p">:</span> <span class="nx">imageEmbedding</span><span class="p">.</span><span class="nx">data</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">embedding</span><span class="p">,</span>
    <span class="na">match_threshold</span><span class="p">:</span> <span class="mf">0.6</span><span class="p">,</span> <span class="c1">// Lower threshold for visual similarity</span>
    <span class="na">match_count</span><span class="p">:</span> <span class="mi">30</span><span class="p">,</span>
    <span class="na">user_id</span><span class="p">:</span> <span class="nx">userId</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  4. Extension &lt;-&gt; Web App Authentication
</h3>

<p><strong>Challenge:</strong> Share authentication state between Chrome extension and web app.</p>

<p><strong>Solution:</strong> Token-based flow<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Extension: popup.tsx</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">authenticate</span><span class="p">()</span> <span class="p">{</span>
  <span class="c1">// Open web app for OAuth</span>
  <span class="kd">const</span> <span class="nx">tab</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">chrome</span><span class="p">.</span><span class="nx">tabs</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
    <span class="na">url</span><span class="p">:</span> <span class="s2">`</span><span class="p">${</span><span class="nx">WEB_APP_URL</span><span class="p">}</span><span class="s2">/extension-auth`</span>
  <span class="p">});</span>

  <span class="c1">// Listen for auth token</span>
  <span class="nx">chrome</span><span class="p">.</span><span class="nx">runtime</span><span class="p">.</span><span class="nx">onMessage</span><span class="p">.</span><span class="nf">addListener</span><span class="p">((</span><span class="nx">message</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">message</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">AUTH_TOKEN</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// Store token in extension storage</span>
      <span class="nx">chrome</span><span class="p">.</span><span class="nx">storage</span><span class="p">.</span><span class="nx">local</span><span class="p">.</span><span class="nf">set</span><span class="p">({</span> <span class="na">authToken</span><span class="p">:</span> <span class="nx">message</span><span class="p">.</span><span class="nx">token</span> <span class="p">});</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="c1">// Web app: app/extension-auth/page.tsx</span>
<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">ExtensionAuth</span><span class="p">()</span> <span class="p">{</span>
  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">supabase</span><span class="p">.</span><span class="nx">auth</span><span class="p">.</span><span class="nf">getSession</span><span class="p">();</span>

    <span class="c1">// Send token to extension</span>
    <span class="k">if </span><span class="p">(</span><span class="nb">window</span><span class="p">.</span><span class="nx">opener</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">chrome</span><span class="p">.</span><span class="nx">runtime</span><span class="p">.</span><span class="nf">sendMessage</span><span class="p">({</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">AUTH_TOKEN</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">token</span><span class="p">:</span> <span class="nx">session</span><span class="p">.</span><span class="nx">access_token</span>
      <span class="p">});</span>
      <span class="nb">window</span><span class="p">.</span><span class="nf">close</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">},</span> <span class="p">[]);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  5. Price Tracking
</h3>

<p>Background job checks saved products for price changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Cron job (runs daily)</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">checkPriceChanges</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">clips</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">supabase</span>
    <span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">clips</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">select</span><span class="p">(</span><span class="dl">'</span><span class="s1">*</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">not</span><span class="p">(</span><span class="dl">'</span><span class="s1">url</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">is</span><span class="dl">'</span><span class="p">,</span> <span class="kc">null</span><span class="p">);</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">clip</span> <span class="k">of</span> <span class="nx">clips</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">currentPrice</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">scrapePrice</span><span class="p">(</span><span class="nx">clip</span><span class="p">.</span><span class="nx">url</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">currentPrice</span> <span class="o">&amp;&amp;</span> <span class="nx">currentPrice</span> <span class="o">!==</span> <span class="nx">clip</span><span class="p">.</span><span class="nx">price</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">supabase</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">price_changes</span><span class="dl">'</span><span class="p">).</span><span class="nf">insert</span><span class="p">({</span>
        <span class="na">clip_id</span><span class="p">:</span> <span class="nx">clip</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
        <span class="na">old_price</span><span class="p">:</span> <span class="nx">clip</span><span class="p">.</span><span class="nx">price</span><span class="p">,</span>
        <span class="na">new_price</span><span class="p">:</span> <span class="nx">currentPrice</span><span class="p">,</span>
        <span class="na">changed_at</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">()</span>
      <span class="p">});</span>

      <span class="c1">// Update clip price</span>
      <span class="k">await</span> <span class="nx">supabase</span>
        <span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">clips</span><span class="dl">'</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">update</span><span class="p">({</span> <span class="na">price</span><span class="p">:</span> <span class="nx">currentPrice</span> <span class="p">})</span>
        <span class="p">.</span><span class="nf">eq</span><span class="p">(</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">,</span> <span class="nx">clip</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Interesting Challenges
</h2>

<h3>
  
  
  1. Embedding Generation at Scale
</h3>

<p><strong>Problem:</strong> Generating embeddings for every clipped product is expensive (API costs).</p>

<p><strong>Solution:</strong></p>

<ul>
<li>Cache embeddings aggressively</li>
<li>Batch embedding generation (up to 100 texts per API call)</li>
<li>Only regenerate if title/description changes significantly
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">async</span> <span class="kd">function</span> <span class="nf">generateEmbeddings</span><span class="p">(</span><span class="nx">clips</span><span class="p">:</span> <span class="nx">Clip</span><span class="p">[])</span> <span class="p">{</span>
  <span class="c1">// Batch up to 100 clips</span>
  <span class="kd">const</span> <span class="nx">batches</span> <span class="o">=</span> <span class="nf">chunk</span><span class="p">(</span><span class="nx">clips</span><span class="p">,</span> <span class="mi">100</span><span class="p">);</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">batch</span> <span class="k">of</span> <span class="nx">batches</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">texts</span> <span class="o">=</span> <span class="nx">batch</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">c</span> <span class="o">=&gt;</span> <span class="s2">`</span><span class="p">${</span><span class="nx">c</span><span class="p">.</span><span class="nx">title</span><span class="p">}</span><span class="s2"> </span><span class="p">${</span><span class="nx">c</span><span class="p">.</span><span class="nx">description</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">openai</span><span class="p">.</span><span class="nx">embeddings</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
      <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">text-embedding-3-small</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">input</span><span class="p">:</span> <span class="nx">texts</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="c1">// Update all clips in batch</span>
    <span class="k">await</span> <span class="nx">supabase</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">clips</span><span class="dl">'</span><span class="p">).</span><span class="nf">upsert</span><span class="p">(</span>
      <span class="nx">batch</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">clip</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
        <span class="p">...</span><span class="nx">clip</span><span class="p">,</span>
        <span class="na">embedding</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">embedding</span>
      <span class="p">}))</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Chrome Extension Content Security Policy
</h3>

<p><strong>Problem:</strong> Manifest V3's strict CSP blocks inline scripts and <code>eval()</code>.</p>

<p><strong>Solution:</strong></p>

<ul>
<li>Move all logic to service workers</li>
<li>Use message passing for communication</li>
<li>Pre-compile templates (no runtime JSX in content scripts)</li>
</ul>

<h3>
  
  
  3. Handling Authentication Across Contexts
</h3>

<p><strong>Problem:</strong> Supabase session cookies don't work across extension and web app.</p>

<p><strong>Solution:</strong> Token-based auth with secure storage:</p>

<ul>
<li>Web app uses cookie-based sessions (httpOnly, secure)</li>
<li>Extension uses token-based auth (stored in chrome.storage.local)</li>
<li>Tokens are short-lived (1 hour) with refresh mechanism</li>
</ul>

<h2>
  
  
  Performance Optimizations
</h2>

<h3>
  
  
  Database Indexing
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Critical indexes for performance</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_clips_user_id</span> <span class="k">ON</span> <span class="n">clips</span><span class="p">(</span><span class="n">user_id</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_clips_embedding</span> <span class="k">ON</span> <span class="n">clips</span> <span class="k">USING</span> <span class="n">ivfflat</span> <span class="p">(</span><span class="n">embedding</span> <span class="n">vector_cosine_ops</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_clips_created_at</span> <span class="k">ON</span> <span class="n">clips</span><span class="p">(</span><span class="n">created_at</span> <span class="k">DESC</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Next.js Optimizations
</h3>

<ul>
<li>Server Components for initial page loads</li>
<li>Dynamic imports for heavy components</li>
<li>Image optimization with next/image</li>
<li>Route caching with <code>revalidate</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// app/dashboard/page.tsx</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">revalidate</span> <span class="o">=</span> <span class="mi">60</span><span class="p">;</span> <span class="c1">// Cache for 60 seconds</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">Dashboard</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">clips</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">getClips</span><span class="p">();</span> <span class="c1">// Server-side fetch</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">Suspense</span> <span class="nx">fallback</span><span class="o">=</span><span class="p">{</span><span class="o">&lt;</span><span class="nx">ClipsSkeleton</span> <span class="o">/&gt;</span><span class="p">}</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">ClipGrid</span> <span class="nx">clips</span><span class="o">=</span><span class="p">{</span><span class="nx">clips</span><span class="p">}</span> <span class="sr">/</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/Suspense</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Lessons Learned
</h2>

<h3>
  
  
  1. Start with Constraints
</h3>

<p>I initially wanted to support image recognition for all products on the web. Scoping down to "help designers organize what they already find" made the MVP achievable.</p>

<h3>
  
  
  2. pgvector is Production-Ready
</h3>

<p>I was skeptical about using PostgreSQL for vector search instead of Pinecone/Weaviate. pgvector has been rock-solid and eliminates operational complexity.</p>

<h3>
  
  
  3. Chrome Extension Distribution is Slow
</h3>

<p>The Chrome Web Store review process takes 3-7 days. Plan accordingly for updates.</p>

<h3>
  
  
  4. Pricing is Hard
</h3>

<p>Choosing $24/month was based on competitive analysis, but I'm still not sure if it's optimal. B2B tools can charge more; productivity tools need volume.</p>

<h2>
  
  
  What's Next
</h2>

<p><strong>Planned features:</strong></p>

<ul>
<li>Bulk import from Pinterest/bookmarks</li>
<li>Shared folders for team collaboration</li>
<li>Integration with design software (Mood boards, CAD tools)</li>
<li>Mobile app (React Native)</li>
</ul>

<p><strong>Scaling considerations:</strong></p>

<ul>
<li>Move to dedicated vector DB if &gt;1M clips</li>
<li>Add caching layer (Redis) for search results</li>
<li>Implement rate limiting for API routes</li>
</ul>

<h2>
  
  
  Try It Out
</h2>

<p>ClipSearch is live at <a href="https://designshelf.biz" rel="noopener noreferrer">https://designshelf.biz</a></p>

<p><strong>Free tier includes:</strong></p>

<ul>
<li>150 clips/month</li>
<li>AI semantic search</li>
<li>Visual image search</li>
<li>Folder organization</li>
</ul>

<p>Would love feedback from the dev community, especially on:</p>

<ul>
<li>The extension architecture</li>
<li>Vector search performance</li>
<li>Pricing strategy</li>
</ul>

<h2>
  
  
  Code Snippets &amp; Resources
</h2>

<p>Full tech stack:</p>

<ul>
<li><a href="https://nextjs.org/docs" rel="noopener noreferrer">Next.js 14 Docs</a></li>
<li><a href="https://supabase.com/docs/guides/ai/vector-columns" rel="noopener noreferrer">Supabase pgvector Guide</a></li>
<li><a href="https://developer.chrome.com/docs/extensions/mv3/" rel="noopener noreferrer">Chrome Extension Manifest V3</a></li>
<li><a href="https://platform.openai.com/docs/guides/embeddings" rel="noopener noreferrer">OpenAI Embeddings</a></li>
</ul>




<p>Questions? Drop them in the comments or reach out! Happy to discuss any of the technical decisions in more detail.</p>

<p><em>Built with ☕ and TypeScript</em></p>

