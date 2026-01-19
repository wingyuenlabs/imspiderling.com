---
Title: I Built an Automated SLM Fine-Tuning Engine with Python and Unsloth 🚀
Description: 
Author: Eshan Roy (eshanized)
Date: 2026-01-19T21:15:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>Fine-tuning Small Language Models (SLMs) like Llama 3.2 or Phi-4 is easier than ever thanks to tools like Unsloth. But the <em>setup</em> is still a pain. You have to find the right Colab notebook, format your data into JSONL, debug generic errors, and figure out which model actually fits your use case.</p>

<p>I wanted to fix that friction. So I built <a href="https://github.com/eshanized/slmgen" rel="noopener noreferrer"><strong>SLMGen</strong></a>, an open-source tool that automates the entire process.</p>

<p><strong>Your Data → Best Model → Matched. One notebook. Zero setup.</strong></p>

<p>Here's how I built it and how it works under the hood.</p>

<h2>
  
  
  🏗️ The Architecture
</h2>

<p>SLMGen is a full-stack application:</p>

<ul>
<li>
<strong>Backend:</strong> Python 3.11 with FastAPI (because Python is the language of AI).</li>
<li>
<strong>Frontend:</strong> Next.js 16 (for a snappy, modern UI).</li>
<li>
<strong>Authentication:</strong> Supabase.</li>
<li>
<strong>Engine:</strong> A custom Recommendation Engine &amp; Notebook Generator.</li>
</ul>

<p>The core magic happens in <code>libslmgen</code>, the Python backend. Let's dive into the cool parts.</p>

<h2>
  
  
  🧠 1. The Recommendation Engine
</h2>

<p>Most people don't know which model to pick. "Should I use Phi-4 or Llama 3.2? Is Mistral 7B too big for my edge device?"</p>

<p>I built a <strong>100-point scoring system</strong> to mathematically answer that question. It analyzes your dataset and intent to pick the perfect model.</p>

<p>Here is the scoring logic from <code>core/recommender.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">get_recommendations</span><span class="p">(</span><span class="n">task</span><span class="p">,</span> <span class="n">deployment</span><span class="p">,</span> <span class="n">stats</span><span class="p">,</span> <span class="n">characteristics</span><span class="p">):</span>
    <span class="c1"># ...
</span>    <span class="c1"># 50 points: Does the model fit the task? (e.g. Code vs Creative Writing)
</span>    <span class="n">task_score</span> <span class="o">=</span> <span class="nf">_score_task_fit</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="n">task</span><span class="p">)</span>

    <span class="c1"># 30 points: Does it fit the hardware? (e.g. Edge vs Cloud)
</span>    <span class="n">deploy_score</span> <span class="o">=</span> <span class="nf">_score_deployment_fit</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="n">deployment</span><span class="p">)</span>

    <span class="c1"># 20 points: Does the data match the model strengths? (e.g. Multilingual -&gt; Qwen)
</span>    <span class="n">data_score</span> <span class="o">=</span> <span class="nf">_score_data_fit</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="n">stats</span><span class="p">,</span> <span class="n">characteristics</span><span class="p">)</span>

    <span class="n">total</span> <span class="o">=</span> <span class="n">task_score</span> <span class="o">+</span> <span class="n">deploy_score</span> <span class="o">+</span> <span class="n">data_score</span>
    <span class="c1"># ...
</span></code></pre>

</div>



<p>It doesn't just guess. It looks at your <strong>data</strong> features:</p>

<ul>
<li>
<strong>Multilingual?</strong> → Bonus for Qwen 2.5 (it's great at non-English).</li>
<li>
<strong>JSON Output?</strong> → Bonus for Phi-4 or Qwen (strong structured output).</li>
<li>
<strong>Edge Deployment?</strong> → Big penalty for 7B models, big bonus for Gemma 2B or SmolLM.</li>
</ul>

<h2>
  
  
  🔍 2. Dataset Intelligence
</h2>

<p>Before you train, you should know what you're training on. SLMGen scans your JSONL file for:</p>

<ul>
<li>
<strong>Quality Issues:</strong> Duplicates, empty responses, short inputs.</li>
<li>
<strong>Personality:</strong> Is your data "Formal &amp; Expert" or "Casual &amp; Helpful"?</li>
<li>
<strong>Hallucination Risk:</strong> It scans for vague language ("might," "probably") vs. grounded language ("according to," numbers/dates).</li>
</ul>

<p>This is done via heuristic analysis in <code>core/personality.py</code> and <code>core/risk.py</code>. It's not an LLM judging an LLM—it's fast, deterministic signal processing on text.</p>

<h2>
  
  
  🪄 3. Self-Contained Notebooks
</h2>

<p>This is my favorite feature. Usually, to run a Colab notebook, you have to:</p>

<ol>
<li>Download a notebook.</li>
<li>Upload your JSONL file to Colab.</li>
<li>Fix the file path in the code.</li>
</ol>

<p>SLMGen removes steps 2 and 3. It <strong>embeds your dataset directly into the notebook</strong> as a base64 string.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># From core/notebook.py
</span><span class="k">def</span> <span class="nf">generate_notebook</span><span class="p">(</span><span class="n">dataset_jsonl</span><span class="p">,</span> <span class="p">...):</span>
    <span class="c1"># Encode dataset
</span>    <span class="n">dataset_b64</span> <span class="o">=</span> <span class="n">base64</span><span class="p">.</span><span class="nf">b64encode</span><span class="p">(</span><span class="n">dataset_jsonl</span><span class="p">.</span><span class="nf">encode</span><span class="p">()).</span><span class="nf">decode</span><span class="p">()</span>

    <span class="c1"># Generate Python code cell
</span>    <span class="n">code_cell</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"""</span><span class="s">
    import base64
    import json

    DATASET_B64 = </span><span class="sh">"</span><span class="si">{</span><span class="n">dataset_b64</span><span class="si">}</span><span class="sh">"</span><span class="s">

    # Decode on the fly
    dataset_str = base64.b64decode(DATASET_B64).decode()
    raw_data = [json.loads(line) for line in dataset_str.splitlines()]
    </span><span class="sh">"""</span>
    <span class="c1"># ...
</span></code></pre>

</div>



<p>When you open the notebook, you just click "Run All". No file uploads. No path errors. It just works.</p>

<h2>
  
  
  ⚡ 4. Unsloth Integration
</h2>

<p>For the actual training, I rely on <a href="https://github.com/unslothai/unsloth" rel="noopener noreferrer">Unsloth</a>. It makes fine-tuning 2x faster and uses 70% less memory.</p>

<p>SLMGen automatically configures the correct LoRA target modules for every supported model structure (Llama, Phi, Gemma, etc.), so you never face a "layer not found" error.</p>

<h2>
  
  
  🌟 Supported Models
</h2>

<p>Right now, SLMGen supports 11 top-tier SLMs:</p>

<ul>
<li>
<strong>Phi-4 Mini</strong> (Reasoning powerhouse)</li>
<li>
<strong>Llama 3.2</strong> (1B/3B)</li>
<li>
<strong>Gemma 2</strong> (2B)</li>
<li><strong>Mistral 7B</strong></li>
<li>
<strong>Qwen 2.5</strong> (Best for coding/multilingual)</li>
<li>And more...</li>
</ul>

<h2>
  
  
  💻 Try it out
</h2>

<p>The project is fully open source. You can run it locally or deploy it yourself.</p>

<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/eshanized/slmgen" rel="noopener noreferrer">https://github.com/eshanized/slmgen</a>
</li>
<li>
<strong>Live Demo:</strong> <a href="https://slmgen.vercel.app" rel="noopener noreferrer">https://slmgen.vercel.app</a>
</li>
</ul>

<p>I'd love to hear your feedback! If this helps you fine-tune your first model, drop a star on the repo! ⭐</p>

