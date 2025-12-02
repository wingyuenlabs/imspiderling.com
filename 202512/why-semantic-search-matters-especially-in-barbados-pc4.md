---
Title: Why Semantic Search Matters... especially in Barbados!
Description: 
Author: Matt Hamilton
Date: 2025-12-02T21:31:43.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  How Semantic Search Finds Products That Keywords Miss
</h2>

<p>I've been building this product search engine for Barbados called "Yuh Gettin' Tru?" (Bajan for "Are you finding what you need?"), and I wanted to share something that perfectly illustrates why semantic search matters.</p>

<h2>
  
  
  The Problem with Traditional Search
</h2>

<p>I was testing the site against a local home store called Dwellings. They sell all sorts of home goods - furniture, kitchenware, decor. I searched for "hardwood cutting board" on their website and got... nothing. Zero results.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3okfj8re3vd329wdy3y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3okfj8re3vd329wdy3y.png" alt="Screenshot of Dwellings website showing no results for hardwood cutting board" width="800" height="488"></a></p>

<p>Now, here's the thing. They absolutely <em>do</em> sell hardwood cutting boards. They have these lovely acacia wood boards with handles. But their product database doesn't call them "hardwood cutting boards" - it calls them "Acacia Cutting Board with Handle".</p>

<p>Traditional keyword search sees "hardwood" and "acacia" as completely different strings. No overlap, no match, no results. The customer leaves, probably assumes the store doesn't stock what they need.</p>

<h2>
  
  
  What Semantic Search Does Differently
</h2>

<p>When you search for "hardwood cutting board" in Yuh Gettin' Tru?, the system understands that you're looking for a cutting board made from hard, durable wood. It doesn't just match keywords - it matches <em>meaning</em>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgrbo6wd7pkk40uew7a12.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgrbo6wd7pkk40uew7a12.png" alt="Screenshot of YuhGettinTru searching for hardwood cutting board and it matching an Acacia cutting board and saying why" width="800" height="774"></a></p>

<p>The same search returns those acacia cutting boards with a 90% match score. And here's the bit I'm particularly pleased with - it explains <em>why</em> it matched:</p>

<blockquote>
<p>"This is a cutting board made of acacia wood, which is a type of hardwood. It is a very good match for the query."</p>
</blockquote>

<p>The system knows that acacia is a hardwood. Not because someone manually tagged it, but because the embedding model (sentence-transformers/all-MiniLM-L6-v2) has learned the semantic relationship between these concepts.</p>

<h2>
  
  
  How It Works Under the Hood
</h2>

<p>The technical pipeline is quite satisfying to work on:</p>

<ol>
<li><p><strong>Vector embeddings</strong> - Each product gets converted into a 384-dimensional vector that captures its semantic meaning. "Acacia cutting board" and "hardwood cutting board" end up quite close in this vector space because they mean similar things.</p></li>
<li><p><strong>Similarity search</strong> - MongoDB Atlas Vector Search finds the closest matches in milliseconds, regardless of exact wording.</p></li>
<li><p><strong>LLM reranking</strong> - Gemini Flash takes the top candidates and scores each one for genuine relevance, filtering out false positives and generating those explanations.</p></li>
</ol>

<p>The whole process takes about 3-5 seconds, most of which is the LLM reranking. The vector search itself is lightning fast.</p>

<h2>
  
  
  Why This Matters for Caribbean Retail
</h2>

<p>Barbados has a fragmented retail landscape. Dozens of stores, each sourcing goods from different parts of the world - some from the US, some from the UK, some from Asia. This means product terminology varies wildly.</p>

<p>Take "cutlery" versus "flatware". If you're British (or Bajan, since we inherited British English), you call your knives and forks "cutlery". But many retailers here source from American suppliers, and their databases use "flatware" instead. Search for "cutlery" on some store websites and you get nothing. The products exist, they're just labelled differently.</p>

<p>Semantic search understands that cutlery and flatware mean the same thing. It bridges the gap between how customers think about products and how retailers happen to have labelled them.</p>

<h2>
  
  
  The Small Things
</h2>

<p>I think what I enjoy most about this project is how the small technical decisions compound into real user value. Choosing the right embedding model. Adding category extraction from breadcrumbs. Structuring the embedding text with labels like "product:" and "brand:" to give the model more context.</p>

<p>None of these are flashy features. But together, they mean a customer searching for "hardwood cutting board" actually finds the acacia board they're looking for.</p>

<p>The small things. They matter.</p>

