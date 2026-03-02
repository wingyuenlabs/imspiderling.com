---
Title: How I Added AI Image Search to a Marketplace Bot (And Why It Changed Everything)
Description: 
Author: david
Date: 2026-03-02T21:35:52.000Z
Robots: noindex,nofollow
Template: index
---
<p>My users were frustrated. Not in a "this feature doesn't work" way — in a "I literally don't know what words to use" way.</p>

<p>A buyer would join the marketplace bot, open the search, and just... stare. They knew exactly what they wanted. They'd seen it at a friend's place, or spotted it in a photo. But they couldn't find the right keywords. "Decorative thing"? "Round wooden thingy"? Every search returned garbage, or nothing.</p>

<p>That's the moment I decided to build image search. And it turned out to be one of the best decisions I made for the project.</p>

<h2>
  
  
  The Problem With Text Search in Marketplaces
</h2>

<p>Text search is great when users know the vocabulary. But in a secondhand or small-seller marketplace, that's rarely the case.</p>

<p>Sellers list things the way they think about them. Buyers search the way they think about them. Those two vocabularies almost never match perfectly. A seller lists "mid-century credenza." A buyer searches "brown dresser with legs." Even with fuzzy matching and morphological analysis (which I'd already implemented), there's a fundamental gap.</p>

<p>Photos don't have this problem. A photo of a thing <em>is</em> the thing. No translation required.</p>

<h2>
  
  
  What I Actually Built
</h2>

<p>The core idea is simple: when a buyer sends a photo of something they're looking for, the bot finds visually similar products from existing listings.</p>

<p>Here's the user flow:</p>

<ol>
<li>Buyer taps "Search by photo" in the bot</li>
<li>Sends any image — a photo they took, a screenshot, something from Pinterest</li>
<li>Bot processes it, searches the catalog, and returns the most visually similar listings</li>
<li>Buyer taps through results, finds what they want, contacts the seller</li>
</ol>

<p>That's it. No keywords. No categories to navigate. Just "I want something that looks like this."</p>

<h2>
  
  
  The Tech Stack (Without Getting Too Deep)
</h2>

<p>I'm using <strong>SigLIP</strong> — a vision-language model from Google — to convert images into vector embeddings. Think of an embedding as a list of ~1000 numbers that mathematically describes what an image looks like. Images of similar things have similar numbers.</p>

<p>When a seller uploads a product photo, the bot:</p>

<ol>
<li>Runs the image through SigLIP's vision encoder</li>
<li>Gets back a 1152-dimensional vector</li>
<li>Stores it in <strong>Qdrant</strong> (a vector database built for this exact use case)</li>
</ol>

<p>When a buyer searches by photo:</p>

<ol>
<li>Same process — their photo becomes a vector</li>
<li>Qdrant finds the closest stored vectors using cosine similarity</li>
<li>Returns the matching products, ranked by visual similarity</li>
</ol>

<p>The whole thing runs in under a second on modest hardware (I'm on a $9/month VPS with 2 CPUs and ~6GB RAM, for context).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Simplified version of what happens during search
</span><span class="k">async</span> <span class="k">def</span> <span class="nf">search_by_image</span><span class="p">(</span><span class="n">photo_bytes</span><span class="p">:</span> <span class="nb">bytes</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="n">Product</span><span class="p">]:</span>
    <span class="c1"># Convert photo to embedding
</span>    <span class="n">embedding</span> <span class="o">=</span> <span class="k">await</span> <span class="n">image_service</span><span class="p">.</span><span class="nf">encode_image</span><span class="p">(</span><span class="n">photo_bytes</span><span class="p">)</span>

    <span class="c1"># Find similar products in vector DB
</span>    <span class="n">results</span> <span class="o">=</span> <span class="k">await</span> <span class="n">qdrant_client</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span>
        <span class="n">collection_name</span><span class="o">=</span><span class="sh">"</span><span class="s">products</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">query_vector</span><span class="o">=</span><span class="n">embedding</span><span class="p">,</span>
        <span class="n">limit</span><span class="o">=</span><span class="mi">10</span><span class="p">,</span>
        <span class="n">score_threshold</span><span class="o">=</span><span class="mf">0.75</span>  <span class="c1"># Only return reasonably similar results
</span>    <span class="p">)</span>

    <span class="k">return</span> <span class="p">[</span><span class="k">await</span> <span class="nf">get_product</span><span class="p">(</span><span class="n">r</span><span class="p">.</span><span class="nb">id</span><span class="p">)</span> <span class="k">for</span> <span class="n">r</span> <span class="ow">in</span> <span class="n">results</span><span class="p">]</span>
</code></pre>

</div>



<h2>
  
  
  The "Why Does This Actually Work?" Part
</h2>

<p>SigLIP was trained on billions of image-text pairs. It learned to understand visual concepts at a semantic level — not just pixel patterns. So when you send a photo of a red ceramic vase, it doesn't just find other red ceramic vases by color matching. It finds things that are conceptually <em>vases</em>, even if they're different colors, shapes, or photographed differently.</p>

<p>This matters a lot in marketplace context. Sellers photograph products in wildly different conditions — different lighting, backgrounds, angles. Classic image similarity algorithms would fail here. Semantic embeddings handle it gracefully.</p>

<p>I also use the <strong>text encoder</strong> from SigLIP to power the regular text search and NSFW filtering — same model, different head. That's why you can search with text like "vintage lamp" and get reasonable results even if sellers didn't use those exact words.</p>

<h2>
  
  
  The ONNX Optimization That Made It Viable
</h2>

<p>Running SigLIP in production on a budget VPS isn't trivial. The original PyTorch models are huge and slow.</p>

<p>I converted both the vision and text encoders to ONNX format, then applied INT8 dynamic quantization using <code>onnxruntime</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">onnxruntime.quantization</span> <span class="kn">import</span> <span class="n">quantize_dynamic</span><span class="p">,</span> <span class="n">QuantType</span>

<span class="nf">quantize_dynamic</span><span class="p">(</span>
    <span class="n">model_input</span><span class="o">=</span><span class="sh">"</span><span class="s">vision_model.onnx</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">model_output</span><span class="o">=</span><span class="sh">"</span><span class="s">vision_int8.onnx</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">weight_type</span><span class="o">=</span><span class="n">QuantType</span><span class="p">.</span><span class="n">QUInt8</span><span class="p">,</span>
    <span class="n">op_types_to_quantize</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">MatMul</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Gemm</span><span class="sh">"</span><span class="p">]</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Results:</p>

<ul>
<li>Vision model: <strong>355MB → 97MB</strong> (3.7x smaller)</li>
<li>Text model: <strong>421MB → 176MB</strong> (2.4x smaller)</li>
<li>Combined RAM savings: <strong>~500MB</strong>
</li>
<li>Speed: <strong>2x faster</strong> inference</li>
</ul>

<p>I benchmarked the quantized models against FP32 on 30 real product photos. NSFW detection agreement was 100%. Category classification agreement was 93% (2 edge cases where "cookware" was classified as "furniture" — acceptable). The quality loss is negligible, the resource savings are massive.</p>

<h2>
  
  
  What Changed After Launching This
</h2>

<p>The honest answer: engagement patterns shifted noticeably.</p>

<p>Before image search, most users who couldn't find what they wanted just left. The bounce rate on the search flow was high. After launch, those same users had a fallback that actually worked.</p>

<p>More interestingly, image search surfaced <em>unexpected</em> results that users liked. Someone uploads a photo of a specific chair style and discovers a similar piece they'd never have thought to search for. That serendipity is hard to manufacture with keyword search.</p>

<p>Sellers noticed too. Products that previously had no text description beyond "nice chair" started getting found because the visual index doesn't care about descriptions.</p>

<h2>
  
  
  The Stuff That Was Harder Than Expected
</h2>

<p><strong>Thumbnail generation.</strong> When displaying search results, I needed consistent thumbnails. Sellers upload wildly different aspect ratios. Getting crops that looked good without cutting off the actual product required more iteration than I expected.</p>

<p><strong>Score thresholds.</strong> Setting the right similarity threshold is more art than science. Too strict, and you return zero results even when good matches exist. Too loose, and you show obviously irrelevant products. I ended up at 0.75 cosine similarity after testing with real data, but it took a while to get there.</p>

<p><strong>Cold start.</strong> Image search is useless with an empty product catalog. The feature became genuinely useful only after a few hundred indexed products. Building the feature before having that scale required some faith.</p>

<p><strong>NSFW filtering.</strong> You can't let users upload anything and have it shown to others unfiltered. I repurpose the same SigLIP text encoder to score images against safe/unsafe text prompts before indexing. It's not perfect, but it catches obvious cases without a separate moderation model.</p>

<h2>
  
  
  The Stack, For the Curious
</h2>

<ul>
<li>
<strong>Bot framework:</strong> aiogram 3 (Python async Telegram bot framework)</li>
<li>
<strong>Database:</strong> PostgreSQL + SQLAlchemy 2</li>
<li>
<strong>Vector search:</strong> Qdrant</li>
<li>
<strong>ML model:</strong> SigLIP Base (Google), converted to ONNX + INT8 quantized</li>
<li>
<strong>Text search:</strong> Elasticsearch with morphological analysis</li>
<li>
<strong>Caching:</strong> Redis</li>
<li>
<strong>Infra:</strong> Single VPS, Docker Compose</li>
</ul>

<p>The entire thing — bot, databases, ML inference, search — runs on that $9/month server. Cost efficiency matters when you're building something that hasn't proven revenue yet.</p>

<h2>
  
  
  Try It Now
</h2>

<p>If you want to see this in action, the bot is live at <a href="https://t.me/k4pi_bot" rel="noopener noreferrer">@k4pi_bot</a>.</p>

<p><strong>k4pi</strong> is an AI-powered marketplace bot for buying and selling on Telegram. You can list products, browse listings, and — yes — search by photo. Send it a picture of something you're looking for and watch it find visually similar items from real listings.</p>

<p><a href="https://t.me/k4pi_bot" rel="noopener noreferrer">Open @k4pi_bot on Telegram</a></p>

<p>It's free to use. If you're building something similar or just curious how the pieces fit together, feel free to reach out. The architecture decisions here weren't obvious at the start, and I'm happy to share what I learned.</p>




<p><em>Building in public. The mistakes were educational.</em></p>

