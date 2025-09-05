---
Title: RAG Evaluation in Java: A Comprehensive Guide
Description: 
Author: vishalmysore
Date: 2025-09-05T22:04:47.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Quick Introduction to RAG
</h2>

<p>Retrieval-Augmented Generation (RAG) is a powerful approach that combines document retrieval with Large Language Models (LLMs). By first retrieving relevant documents from a knowledge base and then using them to inform the LLM's response, RAG systems ensure more accurate, context-aware, and factual outputs while reducing hallucinations.</p>

<blockquote>
<p>Code for this article is <a href="https://github.com/vishalmysore/ragevalinjava" rel="noopener noreferrer">here</a></p>
</blockquote>

<h2>
  
  
  Why RAG Evaluation is Important
</h2>

<p>Evaluating RAG systems is crucial because:</p>

<ul>
<li>It helps ensure the accuracy and reliability of AI applications</li>
<li>It verifies that the right documents are being retrieved from the knowledge base</li>
<li>It confirms that LLM responses are faithful to source documents</li>
<li>It helps identify and minimize hallucinations and incorrect information</li>
<li>It enables continuous optimization of the system's performance</li>
</ul>

<h2>
  
  
  RAG Evaluation Parameters
</h2>

<h3>
  
  
  Core Retrieval Metrics
</h3>

<ol>
<li>
<p><strong>Precision (✅)</strong></p>

<ul>
<li>Definition: How many retrieved documents are actually relevant</li>
<li>Formula: <code>Precision = Relevant Retrieved / Total Retrieved</code>
</li>
<li>Use Case: Critical when wrong document retrieval is costly</li>
<li>Impact: Helps minimize irrelevant information in responses</li>
</ul>
</li>
<li>
<p><strong>Recall (✅)</strong></p>

<ul>
<li>Definition: Proportion of relevant documents retrieved from all possible ones</li>
<li>Formula: <code>Recall = Relevant Retrieved / All Relevant</code>
</li>
<li>Use Case: Important when missing important information is risky</li>
<li>Impact: Ensures comprehensive coverage of relevant information</li>
</ul>
</li>
<li>
<p><strong>F1 Score</strong></p>

<ul>
<li>Definition: Harmonic mean of precision and recall</li>
<li>Formula: <code>F1 = 2 * (Precision * Recall) / (Precision + Recall)</code>
</li>
<li>Use Case: When you need a single comprehensive metric</li>
<li>Impact: Provides balanced evaluation of retrieval performance</li>
</ul>
</li>
<li>
<p><strong>MRR (Mean Reciprocal Rank)</strong></p>

<ul>
<li>Definition: Position of the first relevant document</li>
<li>Formula: <code>1 / Rank</code> (averaged)</li>
<li>Use Case: Particularly important for Q&amp;A systems</li>
<li>Impact: Ensures most relevant documents appear first</li>
</ul>
</li>
<li>
<p><strong>nDCG (Normalized Discounted Cumulative Gain)</strong></p>

<ul>
<li>Definition: Quality of document ranking</li>
<li>Use Case: Especially relevant for vector DB + LLM pipelines</li>
<li>Impact: Measures how well the system ranks documents by relevance</li>
</ul>
</li>
<li>
<p><strong>Hit Rate / Recall@k</strong></p>

<ul>
<li>Definition: Presence of relevant documents in top k results</li>
<li>Formula: <code>(#Queries with ≥1 relevant doc in top k) / Total Queries</code>
</li>
<li>Use Case: Optimizing chunk sizes in RAG</li>
<li>Impact: Ensures relevant information is within the top results</li>
</ul>
</li>
</ol>

<h2>
  
  
  Implementation Example: Document Storage
</h2>

<p>The following Java code demonstrates how documents are stored in the RAG system using an A2A agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">StoreDocumentsWithA2A</span> <span class="o">{</span>
    <span class="kd">public</span> <span class="kd">static</span> <span class="kt">void</span> <span class="nf">main</span><span class="o">(</span><span class="nc">String</span><span class="o">[]</span> <span class="n">args</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">A2AAgent</span> <span class="n">agent</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">A2AAgent</span><span class="o">();</span>
        <span class="n">agent</span><span class="o">.</span><span class="na">connect</span><span class="o">(</span><span class="s">"http://localhost:7860"</span><span class="o">);</span>

        <span class="c1">// Store all ground truth documents in RAG</span>
        <span class="k">for</span> <span class="o">(</span><span class="nc">String</span> <span class="n">instructions</span> <span class="o">:</span> <span class="nc">GroundTruthData</span><span class="o">.</span><span class="na">GROUND_TRUTH_DOCS</span><span class="o">)</span> <span class="o">{</span>
            <span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"Storing instructions: {}"</span><span class="o">,</span> <span class="n">instructions</span><span class="o">);</span>
            <span class="nc">String</span> <span class="n">response</span> <span class="o">=</span> <span class="n">agent</span><span class="o">.</span><span class="na">remoteMethodCall</span><span class="o">(</span>
                <span class="s">"Store these instructions: "</span> <span class="o">+</span> <span class="n">instructions</span>
            <span class="o">).</span><span class="na">getTextResult</span><span class="o">();</span>
            <span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"Agent response: {}"</span><span class="o">,</span> <span class="n">response</span><span class="o">);</span>
        <span class="o">}</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This implementation:</p>

<ol>
<li>Creates an A2A agent instance</li>
<li>Connects to the local RAG server</li>
<li>Iterates through ground truth documents</li>
<li>Stores each document with logging for tracking</li>
</ol>

<h2>
  
  
  Document Retrieval
</h2>

<p>Documents can be retrieved using a REST endpoint:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GET http://localhost:7860/getDocuments?documentText=dishwasher
</code></pre>

</div>



<p>This endpoint:</p>

<ul>
<li>Accepts a query parameter <code>documentText</code>
</li>
<li>Returns relevant documents based on semantic similarity</li>
<li>Uses vector embeddings for matching</li>
<li>Supports natural language queries</li>
</ul>

<h2>
  
  
  RAG Evaluation Results
</h2>

<p>When evaluating the system with a query for "dishwasher", we obtained the following metrics:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>RAG Metrics:
- Precision: 0.2 (20% of retrieved documents were relevant)
- Recall: 0.011 (1.1% of all relevant documents were retrieved)
- F1 Score: 0.021 (Combined precision/recall performance)
- MRR: 0.0 (No relevant documents in top positions)
- nDCG: 0.0 (Poor ranking quality)
- Hit Rate: 0.0 (No relevant documents in top-k results)
</code></pre>

</div>



<h3>
  
  
  Analysis of Results:
</h3>

<ol>
<li>
<p><strong>Low Precision (0.2)</strong></p>

<ul>
<li>Only 20% of retrieved documents were relevant</li>
<li>Indicates high noise in retrieval results</li>
<li>Suggests need for better filtering</li>
</ul>
</li>
<li>
<p><strong>Very Low Recall (0.011)</strong></p>

<ul>
<li>System is missing 98.9% of relevant documents</li>
<li>Indicates potential issues with:

<ul>
<li>Document embedding quality</li>
<li>Similarity threshold settings</li>
<li>Query processing</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>Poor F1 Score (0.021)</strong></p>

<ul>
<li>Confirms overall suboptimal performance</li>
<li>Shows need for system-wide improvements</li>
</ul>
</li>
<li>
<p><strong>Zero Metrics (MRR, nDCG, Hit Rate)</strong></p>

<ul>
<li>Indicates serious ranking issues</li>
<li>Suggests need for:

<ul>
<li>Re-evaluation of embedding model</li>
<li>Adjustment of similarity thresholds</li>
<li>Review of document preprocessing</li>
<li>Optimization of ranking algorithm</li>
</ul>
</li>
</ul>
</li>
</ol>

<h3>
  
  
  Recommendations for Improvement:
</h3>

<ol>
<li>
<p><strong>Embedding Quality</strong></p>

<ul>
<li>Consider using domain-adapted embedding models</li>
<li>Experiment with different embedding dimensions</li>
</ul>
</li>
<li>
<p><strong>Retrieval Strategy</strong></p>

<ul>
<li>Implement hybrid retrieval (semantic + keyword)</li>
<li>Adjust similarity thresholds</li>
<li>Consider using multiple retrieval stages</li>
</ul>
</li>
<li>
<p><strong>Document Processing</strong></p>

<ul>
<li>Review document chunking strategy</li>
<li>Implement better text preprocessing</li>
<li>Consider adding metadata enrichment</li>
</ul>
</li>
<li>
<p><strong>System Optimization</strong></p>

<ul>
<li>Fine-tune vector store parameters</li>
<li>Implement results reranking</li>
<li>Add relevance feedback mechanisms</li>
</ul>
</li>
</ol>

<p>These results highlight the importance of continuous monitoring and iterative improvement in RAG systems. Regular evaluation using these metrics helps maintain system quality and guides optimization efforts.</p>

