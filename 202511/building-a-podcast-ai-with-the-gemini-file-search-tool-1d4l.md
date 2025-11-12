---
Title: Tutorial: Building a Podcast Knowledge Base with the Gemini File Search Tool
Description: 
Author: Mark McDonald
Date: 2025-11-12T22:05:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Recently we launched a new <a href="https://ai.google.dev/gemini-api/docs/file-search" rel="noopener noreferrer"><strong>File Search Tool</strong></a>, a fully managed RAG system built directly into the Gemini API that abstracts away the retrieval pipeline so you can focus on building. For all of the details, check out the <a href="https://blog.google/technology/developers/file-search-gemini-api/" rel="noopener noreferrer">blog post</a>, or read on for a tutorial <strong>from <a href="http://linktree.com/markmcd" rel="noopener noreferrer">Mark McDonald</a></strong>.</p>

<p>Imagine effortlessly recalling specific details from your favorite podcasts or getting a quick recap when returning to a long series' storyline. An AI conversation, equipped with the full context of your podcasts, makes these easy.</p>

<p>In this tutorial, we'll build a tool for this exact case. We'll create a Python app that ingests a podcast RSS feed, transcribes the episodes, and indexes them using the <a href="https://ai.google.dev/gemini-api/docs/file-search" rel="noopener noreferrer"><strong>File Search Tool</strong></a>. This allows us to ask natural language questions and get answers based on the actual content of the podcasts, complete with citations pointing back to the specific episodes.</p>

<h2>
  
  
  Overview of the Solution
</h2>

<p>The application consists of two main parts:</p>

<ol>
<li>
<strong>Ingestion (<code>ingest.py</code>):</strong> Downloads episodes, transcribes them, and uploads the transcripts to a File Search Store.
</li>
<li>
<strong>Querying (<code>query.py</code>):</strong> Takes a user question, searches the File Search Store, and generates an answer using Gemini.</li>
</ol>

<h2>
  
  
  Step 1: Create a file search store
</h2>

<p>A <strong>File Search Store</strong> is the container for scoping your documents. In this example, we are using a single store for all of our podcasts so we can search across them all at once.</p>

<p>First, get set up with the <a href="https://ai.google.dev/gemini-api/docs/libraries#python" rel="noopener noreferrer">Python SDK</a>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google</span> <span class="kn">import</span> <span class="n">genai</span>
<span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>

<span class="n">client</span> <span class="o">=</span> <span class="n">genai</span><span class="p">.</span><span class="nc">Client</span><span class="p">()</span>
</code></pre>

</div>


<p>To create a new store, we use <a href="https://github.com/markmcd/podcast-search/blob/100c4208c99779a0fcfc83f8e08c65d6d3dccb25/ingest.py#L70-L74" rel="noopener noreferrer"><code>client.file_search_stores.create</code></a>. We’ll use the optional display name to identify our podcast index.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">store</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">file_search_stores</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">config</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">display_name</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">My Podcast Store</span><span class="sh">'</span><span class="p">}</span>
<span class="p">)</span>
</code></pre>

</div>

<h2>
  
  
  Step 2: Transcribe episodes
</h2>

<p>To index the content, we need to turn the audio into text. We download the audio file and then use the <strong>Gemini 2.5 Flash-Lite</strong> model to transcribe it. We use Flash-Lite as it's extremely fast and cost-effective for this task.</p>

<p>In <code>ingest.py</code>, the <a href="https://github.com/markmcd/podcast-search/blob/100c4208c99779a0fcfc83f8e08c65d6d3dccb25/ingest.py#L40" rel="noopener noreferrer"><code>transcribe_audio</code></a> function handles this, and you can add any prompt direction to Gemini to help manage the quality of your transcripts, e.g. to skip the intro or label the speakers.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
      <span class="n">model</span><span class="o">=</span><span class="sh">'</span><span class="s">gemini-2.5-flash-lite</span><span class="sh">'</span><span class="p">,</span>
      <span class="n">contents</span><span class="o">=</span><span class="p">[</span>
        <span class="n">types</span><span class="p">.</span><span class="n">Part</span><span class="p">.</span><span class="nf">from_uri</span><span class="p">(</span>
          <span class="n">file_uri</span><span class="o">=</span><span class="n">audio_file</span><span class="p">.</span><span class="n">uri</span><span class="p">,</span>
          <span class="n">mime_type</span><span class="o">=</span><span class="n">audio_file</span><span class="p">.</span><span class="n">mime_type</span>
        <span class="p">),</span>
        <span class="sh">"</span><span class="s">Transcribe this audio. Output only the transcription. Label the speakers. Do not include any obvious ad-reads or promotional segments in the transcription (if unsure, leave them in).</span><span class="sh">"</span>
      <span class="p">]</span>
  <span class="p">)</span>
</code></pre>

</div>

<h2>
  
  
  Step 3: Upload transcripts with metadata
</h2>

<p>Once we have the transcript, we can upload it to our store. A powerful feature of the File Search tool is that you can provide <strong>custom metadata</strong> that can be used to filter at generation-time, allowing us to restrict the source data to a specific podcast or date range.</p>

<p>To upload a file, we use <a href="https://github.com/markmcd/podcast-search/blob/100c4208c99779a0fcfc83f8e08c65d6d3dccb25/ingest.py#L146-L153" rel="noopener noreferrer"><code>client.file_search_stores.upload_to_file_search_store</code></a>. This handles uploading the file content and attaches the custom metadata in the same call.</p>

<p>Here's a sample of how we prepare the metadata and upload the file in <code>ingest.py</code>. The full code adds a number of other fields.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">metadata</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">{</span><span class="sh">'</span><span class="s">key</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">string_value</span><span class="sh">'</span><span class="p">:</span> <span class="n">ep</span><span class="p">.</span><span class="n">title</span><span class="p">},</span>
    <span class="p">{</span><span class="sh">'</span><span class="s">key</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">podcast</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">string_value</span><span class="sh">'</span><span class="p">:</span> <span class="n">feed_info</span><span class="p">.</span><span class="n">title</span><span class="p">},</span>
<span class="p">]</span>

<span class="c1"># Bring any tags from the feed itself
</span><span class="k">if</span> <span class="sh">'</span><span class="s">tags</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">ep</span><span class="p">:</span>
    <span class="k">for</span> <span class="n">tag</span> <span class="ow">in</span> <span class="n">ep</span><span class="p">.</span><span class="n">tags</span><span class="p">:</span>
        <span class="n">metadata</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span><span class="sh">'</span><span class="s">key</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">tag</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">string_value</span><span class="sh">'</span><span class="p">:</span> <span class="n">tag</span><span class="p">.</span><span class="n">term</span><span class="p">})</span>

<span class="n">op</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">file_search_stores</span><span class="p">.</span><span class="nf">upload_to_file_search_store</span><span class="p">(</span>
    <span class="n">file_search_store_name</span><span class="o">=</span><span class="n">store_name</span><span class="p">,</span>
    <span class="nb">file</span><span class="o">=</span><span class="n">transcript_filename</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">'</span><span class="s">custom_metadata</span><span class="sh">'</span><span class="p">:</span> <span class="n">metadata</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">display_name</span><span class="sh">'</span><span class="p">:</span> <span class="n">ep</span><span class="p">.</span><span class="n">title</span>
    <span class="p">}</span>
<span class="p">)</span>
</code></pre>

</div>

<h2>
  
  
  Step 4: Query the store
</h2>

<p>Now for the fun part: asking questions!</p>

<p>To enable file search in a generation request, we pass the <code>FileSearch</code> tool that defines the file store to search, along with any filtering we need.</p>

<p>From <a href="https://github.com/markmcd/podcast-search/blob/100c4208c99779a0fcfc83f8e08c65d6d3dccb25/query.py#L44-L57" rel="noopener noreferrer"><code>query.py</code></a>:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">if</span> <span class="n">args</span><span class="p">.</span><span class="n">podcast</span><span class="p">:</span>
    <span class="n">metadata_filter</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">podcast = </span><span class="si">{</span><span class="n">args</span><span class="p">.</span><span class="n">podcast</span><span class="si">}</span><span class="sh">"</span>

<span class="n">file_search</span> <span class="o">=</span> <span class="n">types</span><span class="p">.</span><span class="nc">FileSearch</span><span class="p">(</span>
    <span class="n">file_search_store_names</span><span class="o">=</span><span class="p">[</span><span class="n">store</span><span class="p">.</span><span class="n">name</span><span class="p">],</span>
    <span class="n">metadata_filter</span><span class="o">=</span><span class="n">metadata_filter</span>  <span class="c1"># Optional filter
</span><span class="p">)</span>
<span class="n">tool</span> <span class="o">=</span> <span class="n">types</span><span class="p">.</span><span class="nc">Tool</span><span class="p">(</span><span class="n">file_search</span><span class="o">=</span><span class="n">file_search</span><span class="p">)</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">'</span><span class="s">gemini-2.5-flash</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">contents</span><span class="o">=</span><span class="n">question</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">tools</span><span class="o">=</span><span class="p">[</span><span class="n">tool</span><span class="p">]</span>
    <span class="p">)</span>
<span class="p">)</span>
</code></pre>

</div>


<p>When we call <code>client.models.generate_content</code> with this tool, Gemini will automatically search our store for relevant information to answer the user's question.</p>
<h2>
  
  
  Step 5: Display results and citations
</h2>

<p>The response from Gemini includes not just the answer, but also <strong>citations</strong> showing exactly which parts of the uploaded files were used.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Answer:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">text</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Citations:</span><span class="sh">"</span><span class="p">)</span>
<span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="n">chunk</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">candidates</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">grounding_metadata</span><span class="p">.</span><span class="n">grounding_chunks</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">chunk</span><span class="p">.</span><span class="n">retrieved_context</span><span class="p">:</span>
        <span class="n">title</span> <span class="o">=</span> <span class="n">chunk</span><span class="p">.</span><span class="n">retrieved_context</span><span class="p">.</span><span class="n">title</span> <span class="ow">or</span> <span class="sh">"</span><span class="s">Unknown Episode</span><span class="sh">"</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Citation </span><span class="si">{</span><span class="n">i</span><span class="o">+</span><span class="mi">1</span><span class="si">}</span><span class="s">:</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Episode: </span><span class="si">{</span><span class="n">title</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Text: </span><span class="si">{</span><span class="n">chunk</span><span class="p">.</span><span class="n">retrieved_context</span><span class="p">.</span><span class="n">text</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>


<p>This allows users to verify the answer and explore the source material further.</p>
<h2>
  
  
  Run the application
</h2>

<ol>
<li>
<strong>Ingest a podcast:</strong>
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python ingest.py <span class="s2">"https://feeds.example.com/podcast.rss"</span> <span class="nt">--limit</span> 5
</code></pre>

</div>


<p>This will download the last 5 episodes, transcribe them, and upload them to the "Podcasts" store.</p>

<ol>
<li>
<strong>Ask a question:</strong>
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python query.py <span class="s2">"Why are red delicious apples so bad?"</span> <span class="nt">--podcast</span><span class="o">=</span><span class="s2">"..."</span>
</code></pre>

</div>


<p>Gemini will retrieve relevant chunks from the indexed transcripts, pass them in the input context with the query, and provide an answer with citations.</p>
<h2>
  
  
  What next?
</h2>

<p>By using the Gemini File Search API, we've turned a collection of audio files into a rich, searchable knowledge base. We didn't have to worry about chunking, embedding, or setting up a vector database - the API handled it all. With the addition of metadata, we built a powerful search tool with minimal code.</p>

<p>For more content, check out:</p>

<ul>
<li>
<a href="https://ai.google.dev/gemini-api/docs/file-search" rel="noopener noreferrer"><strong>Official Documentation</strong></a><strong>:</strong> Dive deeper into the File Search API capabilities.
</li>
<li>
<a href="https://aistudio.google.com/apps/bundled/ask_the_manual" rel="noopener noreferrer"><strong>Demo Applet</strong></a><strong>:</strong> Try out a live demo of a similar application in Google AI Studio (or vibe-code your own!).</li>
</ul>

<p>This article was brought to you by <a href="http://linktree.com/markmcd" rel="noopener noreferrer">Mark McDonald</a>. Grab the code here:</p>


<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/markmcd" rel="noopener noreferrer">
        markmcd
      </a> / <a href="https://github.com/markmcd/podcast-search" rel="noopener noreferrer">
        podcast-search
      </a>
    </h2>
    <h3>
      Index and search podcasts using the Gemini API's File Search tools
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">Podcast Search with Gemini File Search API</h1>
</div>

<p>This project is a simple command-line tool that demonstrates how to build a searchable podcast knowledge base using the <a href="https://ai.google.dev/gemini-api/docs/file-search" rel="nofollow noopener noreferrer">Gemini File Search API</a>.</p>

<p>It consists of two main scripts:</p>

<ul>
<li>
<code>ingest.py</code>: Ingests a podcast RSS feed, downloads audio, transcribes episodes using the fast Gemini 2.5 Flash-Lite model, and uploads them to a File Search Store.</li>
<li>
<code>query.py</code>: Allows you to ask natural language questions about the ingested podcast content and receive grounded answers with citations.</li>
</ul>

<div class="markdown-heading">
<h2 class="heading-element">Prerequisites</h2>
</div>

<ul>
<li>Python 3.10+</li>
<li>A Gemini API key (get one from <a href="https://aistudio.google.com/app/apikey" rel="nofollow noopener noreferrer">Google AI Studio</a>).</li>
</ul>

<div class="markdown-heading">
<h2 class="heading-element">Setup</h2>
</div>

<ol>
<li>
<p>Create and activate a virtual environment:</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>python3 -m venv .venv
<span class="pl-c1">source</span> .venv/bin/activate</pre>

</div>
</li>
<li>
<p>Install dependencies:</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>pip install -r requirements.txt</pre>

</div>
</li>
<li>
<p>Create a <code>.env</code> file and add your API key:</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre><span class="pl-c1">echo</span> <span class="pl-s"><span class="pl-pds">"</span>GOOGLE_API_KEY=your_api_key_here<span class="pl-pds">"</span></span> <span class="pl-k">&gt;</span> .env</pre>

</div>
</li>
</ol>

<div class="markdown-heading">
<h2 class="heading-element">Usage</h2>

</div>

<div class="markdown-heading">
<h3 class="heading-element">1. Ingest a Podcast</h3>

</div>

<p>Run <code>ingest.py</code> with the RSS feed URL of the podcast you want to index…</p>
</div>


</div>
<br>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/markmcd/podcast-search" rel="noopener noreferrer">View on GitHub</a></div>
<br>
</div>
<br>
 

