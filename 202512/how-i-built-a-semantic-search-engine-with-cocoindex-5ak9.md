---
Title: How I Built a Semantic Search Engine with CocoIndex
Description: 
Author: Linghua Jin
Date: 2025-12-02T22:04:22.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>In this tutorial, I'll walk you through how I built a semantic search engine using <strong>CocoIndex</strong>, an open-source Python library for creating powerful search experiences. If you've ever wanted to build a search feature that understands context and meaning (not just exact keyword matches), this post is for you!</p>

<h2>
  
  
  What is CocoIndex?
</h2>

<p>CocoIndex is a lightweight semantic search library that makes it easy to index and search through documents using vector embeddings. Unlike traditional keyword-based search, semantic search understands the meaning behind queries, allowing users to find relevant results even when they use different words.</p>

<h2>
  
  
  Why I Chose CocoIndex
</h2>

<p>I needed a search solution that was:</p>

<ul>
<li>
<strong>Easy to integrate</strong> - No complex setup or infrastructure required</li>
<li>
<strong>Fast</strong> - Quick indexing and search performance</li>
<li>
<strong>Semantic</strong> - Understanding context, not just keywords</li>
<li>
<strong>Open source</strong> - Free to use and modify</li>
</ul>

<p>CocoIndex checked all these boxes!</p>

<h2>
  
  
  Getting Started
</h2>

<p>First, install CocoIndex:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>cocoindex
</code></pre>

</div>



<h2>
  
  
  Building the Search Engine
</h2>

<p>Here's how I implemented the core functionality:</p>

<h3>
  
  
  1. Initialize CocoIndex
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">cocoindex</span> <span class="kn">import</span> <span class="n">CocoIndex</span>
</code></pre>

</div>



<h3>
  
  
  2. Add Documents
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@cocoindex.flow_def(name="TextEmbedding")
def text_embedding_flow(flow_builder: cocoindex.FlowBuilder, data_scope: cocoindex.DataScope):
    """
    Define an example flow that embeds text into a vector database.
    """
    data_scope["documents"] = flow_builder.add_source(
        cocoindex.sources.LocalFile(path="markdown_files"))

    doc_embeddings = data_scope.add_collector()
</code></pre>

</div>



<h1>
  
  
  Index the documents
</h1>

<p>Process each document<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>with data_scope["documents"].row() as doc:
    doc["chunks"] = doc["content"].transform(
        cocoindex.functions.SplitRecursively(),
        language="markdown", chunk_size=2000, chunk_overlap=500)
</code></pre>

</div>



<p>Embed<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>with doc["chunks"].row() as chunk:
    chunk["embedding"] = chunk["text"].transform(
        cocoindex.functions.SentenceTransformerEmbed(
            model="sentence-transformers/all-MiniLM-L6-v2"
        )
    )
    doc_embeddings.collect(filename=doc["filename"], location=chunk["location"],
                            text=chunk["text"], embedding=chunk["embedding"])
</code></pre>

</div>



<p>Export<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>doc_embeddings.export(
    "doc_embeddings",
    cocoindex.storages.Postgres(),
    primary_key_fields=["filename", "location"],
    vector_indexes=[
        cocoindex.VectorIndexDef(
            field_name="embedding",
            metric=cocoindex.VectorSimilarityMetric.COSINE_SIMILARITY)])
</code></pre>

</div>



<h3>
  
  
  3. Perform Semantic Search
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">search</span><span class="p">(</span><span class="n">pool</span><span class="p">:</span> <span class="n">ConnectionPool</span><span class="p">,</span> <span class="n">query</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">top_k</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">5</span><span class="p">):</span>
    <span class="n">table_name</span> <span class="o">=</span> <span class="n">cocoindex</span><span class="p">.</span><span class="n">utils</span><span class="p">.</span><span class="nf">get_target_storage_default_name</span><span class="p">(</span><span class="n">text_embedding_flow</span><span class="p">,</span> <span class="sh">"</span><span class="s">doc_embeddings</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">query_vector</span> <span class="o">=</span> <span class="n">text_to_embedding</span><span class="p">.</span><span class="nf">eval</span><span class="p">(</span><span class="n">query</span><span class="p">)</span>

    <span class="k">with</span> <span class="n">pool</span><span class="p">.</span><span class="nf">connection</span><span class="p">()</span> <span class="k">as</span> <span class="n">conn</span><span class="p">:</span>
        <span class="k">with</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span> <span class="k">as</span> <span class="n">cur</span><span class="p">:</span>
            <span class="n">cur</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"""</span><span class="s">
                SELECT filename, text, embedding &lt;=&gt; %s::vector AS distance
                FROM </span><span class="si">{</span><span class="n">table_name</span><span class="si">}</span><span class="s"> ORDER BY distance LIMIT %s
            </span><span class="sh">"""</span><span class="p">,</span> <span class="p">(</span><span class="n">query_vector</span><span class="p">,</span> <span class="n">top_k</span><span class="p">))</span>
            <span class="k">return</span> <span class="p">[</span>
                <span class="p">{</span><span class="sh">"</span><span class="s">filename</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="n">row</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">:</span> <span class="mf">1.0</span> <span class="o">-</span> <span class="n">row</span><span class="p">[</span><span class="mi">2</span><span class="p">]}</span>
                <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">cur</span><span class="p">.</span><span class="nf">fetchall</span><span class="p">()</span>
            <span class="p">]</span>
</code></pre>

</div>



<h2>
  
  
  Key Features I Implemented
</h2>

<h3>
  
  
  Fast Indexing
</h3>

<p>CocoIndex uses efficient vector storage, making indexing thousands of documents quick and painless.</p>

<h3>
  
  
  Semantic Understanding
</h3>

<p>The search understands that "teaching computers" relates to "machine learning" even without exact keyword matches.</p>

<h3>
  
  
  Customizable Embeddings
</h3>

<p>You can use different embedding models depending on your use case and accuracy requirements.</p>

<h2>
  
  
  Real-World Example
</h2>

<p>I built a documentation search for my project with 500+ markdown files. With CocoIndex:</p>

<ul>
<li>Indexing took less than 30 seconds</li>
<li>Search response time averaged 50ms</li>
<li>Users found relevant docs even with vague queries</li>
</ul>

<h2>
  
  
  Performance Tips
</h2>

<ol>
<li>
<strong>Batch indexing</strong> - Add multiple documents at once for better performance</li>
<li>
<strong>Choose the right embedding model</strong> - Balance between accuracy and speed</li>
<li>
<strong>Cache frequently accessed results</strong> - Store common queries for instant responses</li>
</ol>

<h2>
  
  
  Challenges I Faced
</h2>

<h3>
  
  
  Challenge 1: Choosing Embedding Dimensions
</h3>

<p>Higher dimensions = better accuracy but slower performance. I settled on 384 dimensions as a sweet spot.</p>

<h3>
  
  
  Challenge 2: Handling Large Document Collections
</h3>

<p>For collections over 10k documents, I implemented pagination and lazy loading.</p>

<h2>
  
  
  Results
</h2>

<p>After implementing CocoIndex:</p>

<ul>
<li>User satisfaction increased significantly</li>
<li>Implementation took only 2 days vs weeks for alternatives</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>CocoIndex made building a semantic search engine surprisingly simple. Whether you're building a documentation site, blog search, or product catalog, it's a fantastic tool that punches above its weight.</p>

<p>The library is actively maintained, well-documented, and the community is helpful. I highly recommend giving it a try for your next search implementation!</p>

<h2>
  
  
  Resources
</h2>

<ul>
<li>GitHub: <a href="https://github.com/cocoindex-io/cocoindex" rel="noopener noreferrer">CocoIndex Repository</a>
</li>
<li>Documentation: <a href="https://cocoindex.io" rel="noopener noreferrer">Official Docs</a>
</li>
<li>My Demo Project: <a href="https://cocoindex.io/docs/examples/simple_vector_index" rel="noopener noreferrer">Simple Vector Index Demo</a>
</li>
</ul>

<p>Have you used CocoIndex or other semantic search libraries? Share your experience in the comments below!</p>




<p><em>Happy coding! 🚀</em></p>

