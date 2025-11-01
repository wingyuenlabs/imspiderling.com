---
Title: Semantic search with embeddings in PHP: a hands-on guide using Neuron AI and Ollama
Description: 
Author: Roberto B.
Date: 2025-11-01T21:55:17.000Z
Robots: noindex,nofollow
Template: index
---
<p>Imagine you run an e-commerce or content website. A user searches for “holiday gifts”, but your product catalog includes titles like “Christmas stocking”, “December sale”, or “winter celebration bundle”.</p>

<p>A traditional keyword search will struggle to find these items if the exact words don’t match. This is where <strong>embeddings</strong> come in.</p>

<p>Embeddings let your system search <strong>by meaning</strong> rather than just matching words. By converting text into numerical vectors that represent semantic meaning, embeddings enable the identification of similar items even when they use different words.</p>

<p>Everyday use cases include:</p>

<ul>
<li>Semantic search (searching by meaning)</li>
<li>Recommendation systems (finding similar products or documents)</li>
<li>Duplicate detection (different phrasings, same idea)</li>
<li>Chatbots and intent classification</li>
</ul>

<h2>
  
  
  How embeddings help
</h2>

<p>Think of embeddings as <strong>coordinates on a meaning map</strong>.<br>
Each phrase or word becomes a point in a high-dimensional space, where nearby points represent similar meanings.</p>

<p>Large language models learn to produce these vectors by analyzing <strong>huge training datasets</strong>; essentially, they observe how words and phrases are used <strong>in context</strong> across billions of examples.</p>

<p>If two words often appear in <strong>similar contexts</strong> or play similar <strong>semantic roles</strong>, their embeddings tend to be close in that space. For example, “Christmas” and “December festivity” frequently occur in similar kinds of sentences about holidays, family, gifts, and celebrations, so their numerical representations are close together.</p>

<p>Each embedding is a long array of numbers (a vector).<br>
When we compare two embeddings using a <strong>similarity or distance metric</strong> (such as cosine similarity), we can quantify how closely their meanings align. The closer the vectors, the more semantically related the terms are.</p>

<p>This is why embeddings can recognize that certain words or phrases can often be <strong>used interchangeably</strong> or belong to the same conceptual cluster, even if they don’t share exact words.</p>
<h2>
  
  
  Why we’ll use single words (and when to prefer sentences)
</h2>

<p>In this article, we’ll simplify the concept by embedding <strong>single words</strong>.<br><br>
It’s easier to see how similarity works between a few simple terms.</p>

<p>However, in real-world systems, you’ll usually embed <strong>sentences or short descriptions</strong>. Why?</p>

<ul>
<li>
<strong>Context adds meaning</strong>: “Apple” could mean a fruit or a company; context resolves that.</li>
<li>
<strong>Sentence embeddings are more precise</strong> because they use word relationships and grammar.</li>
<li>
<strong>Phrases can invert meaning</strong>: “not good” vs. “good”. Individual words often miss that nuance.</li>
</ul>

<p>When word vectors can fail vs. sentence vectors:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Situation</th>
<th>Word-level Limitations</th>
<th>Sentence-level Strength</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ambiguous words (“bank”)</td>
<td>Can’t know which meaning</td>
<td>Sentence disambiguates</td>
</tr>
<tr>
<td>Negation (“not good”)</td>
<td>Misses combined meaning</td>
<td>Captures context</td>
</tr>
<tr>
<td>Intent (“how do I return my order?”)</td>
<td>Too sparse</td>
<td>Rich semantic info</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  Tools used: PHP, Neuron AI, and Ollama
</h2>

<p>We’ll use <strong>PHP</strong> to demonstrate the concept because it’s easy to run and integrate with Neuron AI’s RAG tools.</p>

<p><strong>Neuron AI</strong> provides convenient wrappers for embeddings, vector similarity, and RAG (Retrieval-Augmented Generation).</p>

<p><strong>Ollama</strong> runs the embedding model locally, for example, <code>nomic-embed-text</code>.</p>
<h2>
  
  
  Installation Steps
</h2>

<p>Ensure you have PHP 8 or later and Composer installed. Then run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>composer require neuron-core/neuron-ai
</code></pre>

</div>



<p>You’ll also need to have <strong>Ollama</strong> running locally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ollama serve
ollama pull nomic-embed-text:latest
</code></pre>

</div>



<h2>
  
  
  Introducing the example
</h2>

<p>We’ll compare the term <strong>“Christmas”</strong> with a list of related and unrelated words.</p>

<p><strong>Reference word:</strong>  <code>Christmas</code></p>

<p><strong>Words to compare:</strong></p>

<ul>
<li>Christmas</li>
<li>December festivity</li>
<li>New Year</li>
<li>Easter</li>
<li>Car</li>
<li>Bicycle</li>
<li>Banana</li>
</ul>

<p><strong>Expectation:</strong> the similarity between “Christmas” and “December festivity” should be high, while unrelated words like “Banana” should be very low.</p>

<h2>
  
  
  The PHP code
</h2>

<p>Here’s the complete example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="kn">use</span> <span class="nc">NeuronAI\RAG\Embeddings\OllamaEmbeddingsProvider</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">NeuronAI\RAG\VectorSimilarity</span><span class="p">;</span>

<span class="k">require</span> <span class="s2">"vendor/autoload.php"</span><span class="p">;</span>

<span class="c1">// Create provider (configure API key and model)</span>
<span class="nv">$provider</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">OllamaEmbeddingsProvider</span><span class="p">(</span>
    <span class="n">url</span><span class="o">:</span> <span class="s2">"http://localhost:11434/api"</span><span class="p">,</span>
    <span class="n">model</span><span class="o">:</span> <span class="s2">"nomic-embed-text"</span><span class="p">,</span>
<span class="p">);</span>

<span class="nv">$referenceWord</span> <span class="o">=</span> <span class="s2">"Christmas"</span><span class="p">;</span>

<span class="nv">$wordList</span> <span class="o">=</span> <span class="p">[</span>
    <span class="s2">"Christmas"</span><span class="p">,</span>
    <span class="s2">"December festivity"</span><span class="p">,</span>
    <span class="s2">"New Year"</span><span class="p">,</span>
    <span class="s2">"Easter"</span><span class="p">,</span>
    <span class="s2">"Car"</span><span class="p">,</span>
    <span class="s2">"Bicycle"</span><span class="p">,</span>
    <span class="s2">"Banana"</span><span class="p">,</span>
<span class="p">];</span>

<span class="k">echo</span> <span class="s2">"Calculating embed"</span> <span class="mf">.</span> <span class="kc">PHP_EOL</span><span class="p">;</span>
<span class="nv">$referenceEmbedding</span> <span class="o">=</span> <span class="nv">$provider</span><span class="o">-&gt;</span><span class="nf">embedText</span><span class="p">(</span><span class="nv">$referenceWord</span><span class="p">);</span>

<span class="nv">$embeds</span> <span class="o">=</span> <span class="p">[];</span>
<span class="k">foreach</span> <span class="p">(</span><span class="nv">$wordList</span> <span class="k">as</span> <span class="nv">$word</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">echo</span> <span class="s2">"Calculating embed"</span> <span class="mf">.</span> <span class="kc">PHP_EOL</span><span class="p">;</span>
    <span class="nv">$embeds</span><span class="p">[</span><span class="nv">$word</span><span class="p">]</span> <span class="o">=</span> <span class="nv">$provider</span><span class="o">-&gt;</span><span class="nf">embedText</span><span class="p">(</span><span class="nv">$word</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">echo</span> <span class="s2">"Vector similarity"</span> <span class="mf">.</span> <span class="kc">PHP_EOL</span><span class="p">;</span>
<span class="k">foreach</span> <span class="p">(</span><span class="nv">$embeds</span> <span class="k">as</span> <span class="nv">$word</span> <span class="o">=&gt;</span> <span class="nv">$embedding</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">echo</span> <span class="s2">"sim(</span><span class="nv">$referenceWord</span><span class="s2">, </span><span class="nv">$word</span><span class="s2">) = "</span> <span class="mf">.</span>
        <span class="nc">VectorSimilarity</span><span class="o">::</span><span class="nf">cosineSimilarity</span><span class="p">(</span><span class="nv">$referenceEmbedding</span><span class="p">,</span> <span class="nv">$embedding</span><span class="p">)</span> <span class="mf">.</span>
        <span class="kc">PHP_EOL</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Step-by-step explanation
</h2>

<h3>
  
  
  1. Import dependencies
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">require</span> <span class="s2">"vendor/autoload.php"</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">NeuronAI\RAG\Embeddings\OllamaEmbeddingsProvider</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">NeuronAI\RAG\VectorSimilarity</span><span class="p">;</span>
</code></pre>

</div>



<p>This loads Composer dependencies and brings in the classes for embeddings and similarity computation.</p>

<h3>
  
  
  2. Create the provider
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$provider</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">OllamaEmbeddingsProvider</span><span class="p">(</span>
    <span class="n">url</span><span class="o">:</span> <span class="s2">"http://localhost:11434/api"</span><span class="p">,</span>
    <span class="n">model</span><span class="o">:</span> <span class="s2">"nomic-embed-text"</span><span class="p">,</span>
<span class="p">);</span>
</code></pre>

</div>



<p>This initialized the PHP Neuron AI provider to your local Ollama API using the <code>nomic-embed-text</code> model for generating embeddings.</p>

<p>If you want to use a different provider, for example, you want to use Gemini instead of Ollama, you can set up a different provider:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$provider</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">GeminiEmbeddingsProvider</span><span class="p">(</span>
    <span class="n">key</span><span class="o">:</span> <span class="s2">"your-api-key"</span><span class="p">,</span>
    <span class="n">model</span><span class="o">:</span> <span class="s2">"gemini-embedding-001"</span><span class="p">,</span> <span class="c1">//</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  3. Set reference and word list
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$referenceWord</span> <span class="o">=</span> <span class="s2">"Christmas"</span><span class="p">;</span>
<span class="nv">$wordList</span> <span class="o">=</span> <span class="p">[</span><span class="mf">...</span><span class="p">];</span>
</code></pre>

</div>



<p>Defines the base term to compare against and the list of words to analyze.</p>

<h3>
  
  
  4. Generate embeddings
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$referenceEmbedding</span> <span class="o">=</span> <span class="nv">$provider</span><span class="o">-&gt;</span><span class="nf">embedText</span><span class="p">(</span><span class="nv">$referenceWord</span><span class="p">);</span>
<span class="k">foreach</span> <span class="p">(</span><span class="nv">$wordList</span> <span class="k">as</span> <span class="nv">$word</span><span class="p">)</span> <span class="p">{</span>
    <span class="nv">$embeds</span><span class="p">[</span><span class="nv">$word</span><span class="p">]</span> <span class="o">=</span> <span class="nv">$provider</span><span class="o">-&gt;</span><span class="nf">embedText</span><span class="p">(</span><span class="nv">$word</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Each word is transformed into a high-dimensional numeric vector that represents its meaning.</p>

<h3>
  
  
  5. Compute similarities
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">foreach</span> <span class="p">(</span><span class="nv">$embeds</span> <span class="k">as</span> <span class="nv">$word</span> <span class="o">=&gt;</span> <span class="nv">$embedding</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">echo</span> <span class="s2">"sim(</span><span class="nv">$referenceWord</span><span class="s2">, </span><span class="nv">$word</span><span class="s2">) = "</span> <span class="mf">.</span>
        <span class="nc">VectorSimilarity</span><span class="o">::</span><span class="nf">cosineSimilarity</span><span class="p">(</span><span class="nv">$referenceEmbedding</span><span class="p">,</span> <span class="nv">$embedding</span><span class="p">)</span> <span class="mf">.</span>
        <span class="kc">PHP_EOL</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Uses cosine similarity (provided and implemented by Neuron AI) to measure how close each vector is to the reference.</p>

<h2>
  
  
  Sample output
</h2>

<p>Your actual numbers will vary, but you can expect something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Calculating embed
Calculating embed
Calculating embed
Calculating embed
Calculating embed
Calculating embed
Calculating embed
Calculating embed
Vector similarity
sim(Christmas, Christmas) = 1
sim(Christmas, December festivity) = 0.80044420060232
sim(Christmas, New Year) = 0.62543668707038
sim(Christmas, Easter) = 0.64751234202926
sim(Christmas, Car) = 0.4313590955453
sim(Christmas, Bicycle) = 0.38693581208508
sim(Christmas, Banana) = 0.41243323374531
</code></pre>

</div>



<p>Values close to <strong>1.0</strong> mean strong similarity (same semantic concept).<br>
Lower values indicate weaker or no semantic relation.</p>

<h2>
  
  
  Recap: when to use embeddings
</h2>

<p><strong>Use embeddings when:</strong></p>

<ul>
<li>You want to find related content, even if it uses different words.</li>
<li>You’re building a recommendation system or semantic search.</li>
<li>You need clustering or deduplication by meaning.</li>
</ul>

<p><strong>Avoid embeddings when:</strong></p>

<ul>
<li>You only need exact keyword matching.</li>
<li>The dataset is small and rule-based matching is easier.</li>
</ul>

<h2>
  
  
  Next steps
</h2>

<p>Try these enhancements:</p>

<ul>
<li>Replace single words with <strong>product descriptions</strong> or <strong>FAQ questions/answers</strong>.</li>
<li>Store vectors in a <strong>vector database</strong>, such as Pinecone or Weaviate (Neuron AI provides you with several ready-to-use interfaces against several vector databases).</li>
<li>Visualize embeddings using <strong>Python + PCA/UMAP</strong> to see the clusters.</li>
<li>Experiment with different embedding models (<a href="https://ollama.com/search?c=embedding" rel="noopener noreferrer">https://ollama.com/search?c=embedding</a>).</li>
</ul>

<h2>
  
  
  Final checklist
</h2>

<p>Before running:</p>

<ol>
<li>Start the Ollama server (<code>ollama serve</code>).</li>
<li>Pull the model: <code>ollama pull nomic-embed-text</code>.</li>
<li>Install Neuron AI with Composer.</li>
<li>Run the script: <code>php embeddings-demo.php</code>.</li>
</ol>

<p>Once it runs, you’ll see numbers that <em>measure meaning</em>, your first taste of true <strong>semantic search</strong>.</p>

