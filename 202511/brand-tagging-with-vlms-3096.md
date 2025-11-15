---
Title: Brand Tagging with VLMs
Description: 
Author: Ville Vesilehto
Date: 2025-11-15T20:58:39.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  TL;DR
</h3>

<p>Build a two-stage logo pipeline:</p>

<ol>
<li><p>Retrieval - generate image embeddings for small crops and match against a logo dictionary with FAISS cosine search. Use SigLIP-2 (NaFlex) so logos are not distorted and small marks still pop.</p></li>
<li><p>Verification - for top matches, ask LLaVA-OneVision-1.5 a strict JSON question ("Is this the X logo?") and accept only high-confidence "yes". It's a good model sir.</p></li>
</ol>

<p>A bit longer post this time around.</p>

<h3>
  
  
  Intro
</h3>

<p>Brand tagging in real-world video is hard: logos are tiny, partly occluded, moving, and often appear on textured backgrounds. A practical approach is a two-stage pipeline: </p>

<ul>
<li>First retrieve likely logo crops with a fast contrastive image encoder.</li>
<li>Then verify each candidate with a vision-language model (VLM) that can read text and reason about shapes and context. Each candidate is a (frame, bbox, brand, retrieval_score) record that we pass to the VLM.</li>
</ul>

<p>This tutorial combines a modern image-text encoder (SigLIP-2, NaFlex variant) for high-recall retrieval with a VLM from the LLaVA family for precise, structured yes/no verification.</p>

<p>You could train a <a href="https://en.wikipedia.org/wiki/You_Only_Look_Once" rel="noopener noreferrer">YOLO-style detector</a> for specific brands. This post, however, focuses on a more flexible "embedding + VLM" approach that adapts quickly to new logos without retraining. Some might find this fun.</p>

<h3>
  
  
  Research
</h3>

<p>Contrastive encoders such as CLIP/SigLIP produce embeddings where similar visuals are close. FAISS makes nearest-neighbor search over many references instantaneous. You can treat logo search as nearest-neighbor lookup instead of training a custom detector.</p>

<p>Verification reduces false positives. A VLM can explicitly answer "Is this the Red Bull logo?" and justify the decision, improving precision on lookalikes, partial views, and blur.</p>

<p>Pointers to the underlying research for the initiated:</p>

<ul>
<li><a href="https://arxiv.org/abs/2103.00020" rel="noopener noreferrer">CLIP: Learning Transferable Visual Models From Natural Language Supervision (arXiv)</a></li>
<li><a href="https://arxiv.org/abs/2303.15343" rel="noopener noreferrer">SigLIP: Sigmoid Loss for Language-Image Pre-Training (arXiv)</a></li>
<li><a href="https://arxiv.org/abs/2304.08485" rel="noopener noreferrer">LLaVA: Visual Instruction Tuning (arXiv)</a></li>
<li><a href="https://arxiv.org/abs/1702.08734" rel="noopener noreferrer">FAISS: Billion-scale similarity search with GPUs (arXiv)</a></li>
</ul>

<h3>
  
  
  Setup
</h3>

<p>You’ll need to be comfortable with basic Python and PyTorch. No prior experience with FAISS or VLMs required.</p>

<p>Hardware/software requirements:</p>

<ul>
<li>At least one NVIDIA GPU (at least 32 GB VRAM recommended)</li>
<li>CUDA 12.x, Python 3.10–3.12</li>
<li>Disk: 5–10 GB (models + caches)</li>
<li>Tools: ffmpeg for frames</li>
</ul>

<p>This tutorial was built and tested on a single NVIDIA H200.</p>

<p>Prepare the Python environment as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>python3 -m venv .venv &amp;&amp; source .venv/bin/activate
pip install -U torch torchvision torchaudio \
  "transformers&gt;=4.45" accelerate pillow opencv-python faiss-cpu \
  numpy pydantic polars
</code></pre>

</div>



<p>For simplicity we'll use <a href="https://pypi.org/project/faiss-cpu/" rel="noopener noreferrer">ƒaiss-cpu</a> as it's fine at this scale. It's a single logo after all. GPU for the models.</p>

<p>Example video (Creative Commons):</p>

<ul>
<li>Title: Red Bull Racing Pit Stop Practice (2015), 44 s, 1920x1080.</li>
<li>License: CC BY-SA 4.0 - attribute ProtoplasmaKid / Wikimedia Commons.</li>
<li>It has tiny, moving logos, occlusion, uniforms, car bodywork, pit rig and acts as a great stress-test.</li>
</ul>

<h3>
  
  
  Architecture
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>frames (2–4 FPS) ──▶ crops (multi-scale grid)
         │
         ▼
 [Stage 1] Retrieval (SigLIP-2 image features + FAISS cosine)
         │      └─ top-K per frame/brand with heuristics
         ▼
 [Stage 2] VLM verification (LLaVA-OneVision-1.5 JSON verdict)
         │
         ▼
    JSONL evidence
</code></pre>

</div>



<p>Models used in this post:</p>

<ul>
<li><a href="https://huggingface.co/google/siglip2-base-patch16-naflex" rel="noopener noreferrer">SigLIP-2: google/siglip2-base-patch16-naflex</a></li>
<li><a href="https://huggingface.co/lmms-lab/LLaVA-OneVision-1.5-8B-Instruct" rel="noopener noreferrer">LLaVA-OneVision-1.5-8B-Instruct</a></li>
</ul>

<p>SigLIP-2 reports better zero-shot and retrieval performance than prior SigLIP/CLIP models on public benchmarks. NaFlex means the encoder resizes each crop to a grid of flexible patches instead of forcing a fixed square, so long thin logos don't get squashed.</p>

<p>OneVision-1.5 is an open VLM family. Its card reports strong benchmark leads vs other open models.</p>

<p>Both are Apache-2.0 licensed.</p>

<p>So let's do it!</p>

<h3>
  
  
  Quickstart
</h3>

<p>1) Grab the video:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>wget https://upload.wikimedia.org/wikipedia/commons/b/ba/Red_Bull_Racing_Pit_Stop_Practice.webm
</code></pre>

</div>



<p>Then extract frames. Two FPS, scaled down to 1280x720:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> frames
ffmpeg <span class="nt">-i</span> <span class="s2">"Red_Bull_Racing_Pit_Stop_Practice.webm"</span> <span class="nt">-vf</span> <span class="s2">"fps=2,scale=1280:-1:flags=lanczos"</span> <span class="nt">-q</span>:v 3 frames/f_%06d.jpg
</code></pre>

</div>



<p>This yields 88 JPG files.</p>

<p>2) Prepare a logo dictionary</p>

<p>In this example we're only interested in the Red Bull logo. Create a <code>logos</code> directory you want to use. Grab the logo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> logos
wget <span class="nt">-P</span> logos https://upload.wikimedia.org/wikipedia/fi/a/a5/Red_Bull_logo.png
</code></pre>

</div>



<p><strong>Disclaimer:</strong> This tutorial is for educational purposes only and is not affiliated with or endorsed by Red Bull. Red Bull is a registered trademark of Red Bull GmbH. It's a decent energy drink though.</p>

<p>3) Build the logo index (FAISS, SigLIP-2)</p>

<p>Save to <code>build_logo_index.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span><span class="p">,</span> <span class="n">faiss</span><span class="p">,</span> <span class="n">torch</span>
<span class="kn">from</span> <span class="n">pathlib</span> <span class="kn">import</span> <span class="n">Path</span>
<span class="kn">from</span> <span class="n">PIL</span> <span class="kn">import</span> <span class="n">Image</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">from</span> <span class="n">transformers</span> <span class="kn">import</span> <span class="n">AutoModel</span><span class="p">,</span> <span class="n">AutoProcessor</span>

<span class="n">MODEL_ID</span> <span class="o">=</span> <span class="sh">"</span><span class="s">google/siglip2-base-patch16-naflex</span><span class="sh">"</span>  <span class="c1"># NaFlex = native aspect ratio
</span><span class="n">OUT_DIR</span> <span class="o">=</span> <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">artifacts</span><span class="sh">"</span><span class="p">);</span> <span class="n">OUT_DIR</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="n">exist_ok</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">parents</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">embed_images</span><span class="p">(</span><span class="n">paths</span><span class="p">,</span> <span class="n">model</span><span class="p">,</span> <span class="n">proc</span><span class="p">,</span> <span class="n">batch</span><span class="o">=</span><span class="mi">16</span><span class="p">):</span>
    <span class="n">imgs</span> <span class="o">=</span> <span class="p">[</span><span class="n">Image</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">p</span><span class="p">).</span><span class="nf">convert</span><span class="p">(</span><span class="sh">"</span><span class="s">RGB</span><span class="sh">"</span><span class="p">)</span> <span class="k">for</span> <span class="n">p</span> <span class="ow">in</span> <span class="n">paths</span><span class="p">]</span>
    <span class="n">feats</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nf">len</span><span class="p">(</span><span class="n">imgs</span><span class="p">),</span> <span class="n">batch</span><span class="p">):</span>
        <span class="n">chunk</span> <span class="o">=</span> <span class="n">imgs</span><span class="p">[</span><span class="n">i</span><span class="p">:</span><span class="n">i</span><span class="o">+</span><span class="n">batch</span><span class="p">]</span>
        <span class="n">inputs</span> <span class="o">=</span> <span class="nf">proc</span><span class="p">(</span><span class="n">images</span><span class="o">=</span><span class="n">chunk</span><span class="p">,</span> <span class="n">return_tensors</span><span class="o">=</span><span class="sh">"</span><span class="s">pt</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">inputs</span> <span class="o">=</span> <span class="p">{</span><span class="n">k</span><span class="p">:</span> <span class="n">v</span><span class="p">.</span><span class="nf">to</span><span class="p">(</span><span class="n">model</span><span class="p">.</span><span class="n">device</span><span class="p">)</span> <span class="k">for</span> <span class="n">k</span><span class="p">,</span> <span class="n">v</span> <span class="ow">in</span> <span class="n">inputs</span><span class="p">.</span><span class="nf">items</span><span class="p">()}</span>
        <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
            <span class="n">f</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">get_image_features</span><span class="p">(</span><span class="o">**</span><span class="n">inputs</span><span class="p">)</span>  <span class="c1"># (B, d)
</span>            <span class="n">f</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="n">nn</span><span class="p">.</span><span class="n">functional</span><span class="p">.</span><span class="nf">normalize</span><span class="p">(</span><span class="n">f</span><span class="p">,</span> <span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>
        <span class="n">feats</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">f</span><span class="p">.</span><span class="nf">cpu</span><span class="p">())</span>
    <span class="k">return</span> <span class="n">torch</span><span class="p">.</span><span class="nf">cat</span><span class="p">(</span><span class="n">feats</span><span class="p">,</span> <span class="n">dim</span><span class="o">=</span><span class="mi">0</span><span class="p">).</span><span class="nf">numpy</span><span class="p">()</span>

<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">AutoModel</span><span class="p">.</span><span class="nf">from_pretrained</span><span class="p">(</span><span class="n">MODEL_ID</span><span class="p">,</span> <span class="n">dtype</span><span class="o">=</span><span class="n">torch</span><span class="p">.</span><span class="n">float32</span><span class="p">,</span>
                                      <span class="n">device_map</span><span class="o">=</span><span class="sh">"</span><span class="s">auto</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">proc</span>  <span class="o">=</span> <span class="n">AutoProcessor</span><span class="p">.</span><span class="nf">from_pretrained</span><span class="p">(</span><span class="n">MODEL_ID</span><span class="p">)</span>

    <span class="n">logo_paths</span> <span class="o">=</span> <span class="nf">sorted</span><span class="p">(</span><span class="nf">list</span><span class="p">(</span><span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">logos</span><span class="sh">"</span><span class="p">).</span><span class="nf">glob</span><span class="p">(</span><span class="sh">"</span><span class="s">*.*</span><span class="sh">"</span><span class="p">)))</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">logo_paths</span><span class="p">:</span>
        <span class="k">raise</span> <span class="nc">SystemExit</span><span class="p">(</span><span class="sh">"</span><span class="s">No logo files found in </span><span class="sh">'</span><span class="s">logos</span><span class="sh">'</span><span class="s"> directory.</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">brands</span> <span class="o">=</span> <span class="p">[</span><span class="n">p</span><span class="p">.</span><span class="n">stem</span> <span class="k">for</span> <span class="n">p</span> <span class="ow">in</span> <span class="n">logo_paths</span><span class="p">]</span>
    <span class="n">vecs</span> <span class="o">=</span> <span class="nf">embed_images</span><span class="p">(</span><span class="n">logo_paths</span><span class="p">,</span> <span class="n">model</span><span class="p">,</span> <span class="n">proc</span><span class="p">)</span>
    <span class="n">d</span> <span class="o">=</span> <span class="n">vecs</span><span class="p">.</span><span class="n">shape</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span>
    <span class="n">index</span> <span class="o">=</span> <span class="n">faiss</span><span class="p">.</span><span class="nc">IndexFlatIP</span><span class="p">(</span><span class="n">d</span><span class="p">);</span> <span class="n">index</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="n">vecs</span><span class="p">)</span>

    <span class="n">faiss</span><span class="p">.</span><span class="nf">write_index</span><span class="p">(</span><span class="n">index</span><span class="p">,</span> <span class="nf">str</span><span class="p">(</span><span class="n">OUT_DIR</span> <span class="o">/</span> <span class="sh">"</span><span class="s">logos.faiss</span><span class="sh">"</span><span class="p">))</span>
    <span class="p">(</span><span class="n">OUT_DIR</span> <span class="o">/</span> <span class="sh">"</span><span class="s">logos_meta.json</span><span class="sh">"</span><span class="p">).</span><span class="nf">write_text</span><span class="p">(</span>
        <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">({</span><span class="sh">"</span><span class="s">brands</span><span class="sh">"</span><span class="p">:</span> <span class="n">brands</span><span class="p">,</span> <span class="sh">"</span><span class="s">files</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="nf">str</span><span class="p">(</span><span class="n">p</span><span class="p">)</span> <span class="k">for</span> <span class="n">p</span> <span class="ow">in</span> <span class="n">logo_paths</span><span class="p">]},</span>
                   <span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span>
    <span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Indexed </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">brands</span><span class="p">)</span><span class="si">}</span><span class="s"> brands into </span><span class="si">{</span><span class="n">OUT_DIR</span><span class="o">/</span><span class="sh">'</span><span class="s">logos.faiss</span><span class="sh">'</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>
</code></pre>

</div>



<p>SigLIP-2 provides get_image_features and NaFlex dynamic resizing to minimize distortion on non-square inputs — useful for narrow/wide logos.</p>

<p>Run the tool:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>python build_logo_index.py
...
Indexed 1 brands into artifacts/logos.faiss
</code></pre>

</div>



<p>￼<br>
4) Generate crops + Stage-1 retrieval</p>

<p>Save the following to <code>retrieve.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span><span class="p">,</span> <span class="n">math</span><span class="p">,</span> <span class="n">os</span><span class="p">,</span> <span class="n">faiss</span><span class="p">,</span> <span class="n">torch</span>
<span class="kn">from</span> <span class="n">pathlib</span> <span class="kn">import</span> <span class="n">Path</span>
<span class="kn">from</span> <span class="n">PIL</span> <span class="kn">import</span> <span class="n">Image</span><span class="p">,</span> <span class="n">ImageDraw</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">from</span> <span class="n">transformers</span> <span class="kn">import</span> <span class="n">AutoModel</span><span class="p">,</span> <span class="n">AutoProcessor</span>

<span class="n">EMB_ID</span> <span class="o">=</span> <span class="sh">"</span><span class="s">google/siglip2-base-patch16-naflex</span><span class="sh">"</span>
<span class="n">ART</span> <span class="o">=</span> <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">artifacts</span><span class="sh">"</span><span class="p">);</span> <span class="n">ART</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="n">exist_ok</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="n">SIZES</span> <span class="o">=</span> <span class="p">[</span><span class="mi">192</span><span class="p">,</span> <span class="mi">256</span><span class="p">,</span> <span class="mi">320</span><span class="p">]</span>     <span class="c1"># square windows
</span><span class="n">STRIDE</span> <span class="o">=</span> <span class="mf">0.5</span>                <span class="c1"># 50% overlap
</span><span class="n">TOPK</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">TOPK</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">3</span><span class="sh">"</span><span class="p">))</span>
<span class="n">COSINE_TH</span> <span class="o">=</span> <span class="nf">float</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">COSINE_TH</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">0.7</span><span class="sh">"</span><span class="p">))</span>  <span class="c1"># keep candidates above this
</span>
<span class="k">def</span> <span class="nf">_metric_type</span><span class="p">(</span><span class="n">index</span><span class="p">):</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">index</span><span class="p">.</span><span class="n">metric_type</span>
    <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
        <span class="k">return</span> <span class="bp">None</span>

<span class="k">def</span> <span class="nf">_l2_to_cosine</span><span class="p">(</span><span class="n">d</span><span class="p">:</span> <span class="n">np</span><span class="p">.</span><span class="n">ndarray</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">np</span><span class="p">.</span><span class="n">ndarray</span><span class="p">:</span>
    <span class="c1"># If vectors are L2-normalized, cos = 1 - 0.5 * ||a - b||^2
</span>    <span class="k">return</span> <span class="mf">1.0</span> <span class="o">-</span> <span class="mf">0.5</span> <span class="o">*</span> <span class="n">d</span>

<span class="k">def</span> <span class="nf">_as_float32</span><span class="p">(</span><span class="n">x</span><span class="p">:</span> <span class="n">np</span><span class="p">.</span><span class="n">ndarray</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">np</span><span class="p">.</span><span class="n">ndarray</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">x</span><span class="p">.</span><span class="n">dtype</span> <span class="o">!=</span> <span class="n">np</span><span class="p">.</span><span class="n">float32</span><span class="p">:</span>
        <span class="n">x</span> <span class="o">=</span> <span class="n">x</span><span class="p">.</span><span class="nf">astype</span><span class="p">(</span><span class="n">np</span><span class="p">.</span><span class="n">float32</span><span class="p">,</span> <span class="n">copy</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">np</span><span class="p">.</span><span class="nf">ascontiguousarray</span><span class="p">(</span><span class="n">x</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">grid_crops</span><span class="p">(</span><span class="n">im</span><span class="p">:</span> <span class="n">Image</span><span class="p">.</span><span class="n">Image</span><span class="p">):</span>
    <span class="n">W</span><span class="p">,</span> <span class="n">H</span> <span class="o">=</span> <span class="n">im</span><span class="p">.</span><span class="n">size</span>
    <span class="k">for</span> <span class="n">s</span> <span class="ow">in</span> <span class="n">SIZES</span><span class="p">:</span>
        <span class="n">step</span> <span class="o">=</span> <span class="nf">max</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nf">int</span><span class="p">(</span><span class="n">s</span> <span class="o">*</span> <span class="n">STRIDE</span><span class="p">))</span>
        <span class="k">for</span> <span class="n">y</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nf">max</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="n">H</span> <span class="o">-</span> <span class="n">s</span> <span class="o">+</span> <span class="mi">1</span><span class="p">),</span> <span class="n">step</span><span class="p">):</span>
            <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nf">max</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="n">W</span> <span class="o">-</span> <span class="n">s</span> <span class="o">+</span> <span class="mi">1</span><span class="p">),</span> <span class="n">step</span><span class="p">):</span>
                <span class="nf">yield </span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">s</span><span class="p">,</span> <span class="n">s</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">embed</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="n">proc</span><span class="p">,</span> <span class="n">pil_list</span><span class="p">,</span> <span class="n">bs</span><span class="o">=</span><span class="mi">24</span><span class="p">):</span>
    <span class="n">out</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nf">len</span><span class="p">(</span><span class="n">pil_list</span><span class="p">),</span> <span class="n">bs</span><span class="p">):</span>
        <span class="n">chunk</span> <span class="o">=</span> <span class="n">pil_list</span><span class="p">[</span><span class="n">i</span><span class="p">:</span><span class="n">i</span><span class="o">+</span><span class="n">bs</span><span class="p">]</span>
        <span class="n">inp</span> <span class="o">=</span> <span class="nf">proc</span><span class="p">(</span><span class="n">images</span><span class="o">=</span><span class="n">chunk</span><span class="p">,</span> <span class="n">return_tensors</span><span class="o">=</span><span class="sh">"</span><span class="s">pt</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">inp</span> <span class="o">=</span> <span class="p">{</span><span class="n">k</span><span class="p">:</span> <span class="n">v</span><span class="p">.</span><span class="nf">to</span><span class="p">(</span><span class="n">model</span><span class="p">.</span><span class="n">device</span><span class="p">)</span> <span class="k">for</span> <span class="n">k</span><span class="p">,</span> <span class="n">v</span> <span class="ow">in</span> <span class="n">inp</span><span class="p">.</span><span class="nf">items</span><span class="p">()}</span>
        <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
            <span class="n">f</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">get_image_features</span><span class="p">(</span><span class="o">**</span><span class="n">inp</span><span class="p">)</span>
            <span class="n">f</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="n">nn</span><span class="p">.</span><span class="n">functional</span><span class="p">.</span><span class="nf">normalize</span><span class="p">(</span><span class="n">f</span><span class="p">,</span> <span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>
        <span class="n">out</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">f</span><span class="p">.</span><span class="nf">detach</span><span class="p">().</span><span class="nf">to</span><span class="p">(</span><span class="n">torch</span><span class="p">.</span><span class="n">float32</span><span class="p">).</span><span class="nf">cpu</span><span class="p">())</span>
    <span class="n">vecs</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">cat</span><span class="p">(</span><span class="n">out</span><span class="p">,</span> <span class="n">dim</span><span class="o">=</span><span class="mi">0</span><span class="p">).</span><span class="nf">numpy</span><span class="p">()</span>
    <span class="k">return</span> <span class="nf">_as_float32</span><span class="p">(</span><span class="n">vecs</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="c1"># load logo index
</span>    <span class="n">index</span> <span class="o">=</span> <span class="n">faiss</span><span class="p">.</span><span class="nf">read_index</span><span class="p">(</span><span class="nf">str</span><span class="p">(</span><span class="n">ART</span> <span class="o">/</span> <span class="sh">"</span><span class="s">logos.faiss</span><span class="sh">"</span><span class="p">))</span>
    <span class="n">meta</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">((</span><span class="n">ART</span> <span class="o">/</span> <span class="sh">"</span><span class="s">logos_meta.json</span><span class="sh">"</span><span class="p">).</span><span class="nf">read_text</span><span class="p">())</span>
    <span class="n">brands</span> <span class="o">=</span> <span class="n">meta</span><span class="p">[</span><span class="sh">"</span><span class="s">brands</span><span class="sh">"</span><span class="p">]</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[retrieve] index ntotal=</span><span class="si">{</span><span class="n">index</span><span class="p">.</span><span class="n">ntotal</span><span class="si">}</span><span class="s">, brands=</span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">brands</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
        <span class="k">pass</span>

    <span class="c1"># load embedder
</span>    <span class="n">model</span> <span class="o">=</span> <span class="n">AutoModel</span><span class="p">.</span><span class="nf">from_pretrained</span><span class="p">(</span><span class="n">EMB_ID</span><span class="p">,</span> <span class="n">dtype</span><span class="o">=</span><span class="n">torch</span><span class="p">.</span><span class="n">float32</span><span class="p">,</span> <span class="n">device_map</span><span class="o">=</span><span class="sh">"</span><span class="s">auto</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">proc</span>  <span class="o">=</span> <span class="n">AutoProcessor</span><span class="p">.</span><span class="nf">from_pretrained</span><span class="p">(</span><span class="n">EMB_ID</span><span class="p">)</span>

    <span class="n">frames</span> <span class="o">=</span> <span class="nf">sorted</span><span class="p">(</span><span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">frames</span><span class="sh">"</span><span class="p">).</span><span class="nf">glob</span><span class="p">(</span><span class="sh">"</span><span class="s">f_*.jpg</span><span class="sh">"</span><span class="p">))</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[retrieve] frames found=</span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">frames</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">out</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="n">dump_all</span> <span class="o">=</span> <span class="nf">bool</span><span class="p">(</span><span class="nf">int</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">DEBUG_DUMP_ALL</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">0</span><span class="sh">"</span><span class="p">)))</span>
    <span class="n">dbg_all</span> <span class="o">=</span> <span class="p">[]</span> <span class="k">if</span> <span class="n">dump_all</span> <span class="k">else</span> <span class="bp">None</span>
    <span class="n">debug_draw</span> <span class="o">=</span> <span class="n">dump_all</span> <span class="ow">and</span> <span class="nf">bool</span><span class="p">(</span><span class="nf">int</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">DEBUG_DRAW</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">0</span><span class="sh">"</span><span class="p">)))</span>
    <span class="n">debug_draw_dir</span> <span class="o">=</span> <span class="n">ART</span> <span class="o">/</span> <span class="sh">"</span><span class="s">debug_vis</span><span class="sh">"</span>
    <span class="k">if</span> <span class="n">debug_draw</span><span class="p">:</span>
        <span class="n">debug_draw_dir</span><span class="p">.</span><span class="nf">mkdir</span><span class="p">(</span><span class="n">parents</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">exist_ok</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">debug_draw_th</span> <span class="o">=</span> <span class="nf">float</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">DEBUG_DRAW_TH</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">-1.0</span><span class="sh">"</span><span class="p">))</span>
    <span class="n">debug_draw_max</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">DEBUG_DRAW_MAX</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">0</span><span class="sh">"</span><span class="p">))</span>  <span class="c1"># 0 = unlimited
</span>    <span class="n">best</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">:</span> <span class="o">-</span><span class="mf">1.0</span><span class="p">,</span> <span class="sh">"</span><span class="s">frame</span><span class="sh">"</span><span class="p">:</span> <span class="bp">None</span><span class="p">,</span> <span class="sh">"</span><span class="s">brand</span><span class="sh">"</span><span class="p">:</span> <span class="bp">None</span><span class="p">,</span> <span class="sh">"</span><span class="s">bbox</span><span class="sh">"</span><span class="p">:</span> <span class="bp">None</span><span class="p">}</span>

    <span class="k">for</span> <span class="n">fpath</span> <span class="ow">in</span> <span class="n">frames</span><span class="p">:</span>
        <span class="n">im</span> <span class="o">=</span> <span class="n">Image</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="n">fpath</span><span class="p">).</span><span class="nf">convert</span><span class="p">(</span><span class="sh">"</span><span class="s">RGB</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">boxes</span><span class="p">,</span> <span class="n">crops</span> <span class="o">=</span> <span class="p">[],</span> <span class="p">[]</span>
        <span class="nf">for </span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">h</span><span class="p">)</span> <span class="ow">in</span> <span class="nf">grid_crops</span><span class="p">(</span><span class="n">im</span><span class="p">):</span>
            <span class="n">boxes</span><span class="p">.</span><span class="nf">append</span><span class="p">((</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">h</span><span class="p">))</span>
            <span class="n">crops</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">im</span><span class="p">.</span><span class="nf">crop</span><span class="p">((</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">x</span><span class="o">+</span><span class="n">w</span><span class="p">,</span> <span class="n">y</span><span class="o">+</span><span class="n">h</span><span class="p">)))</span>

        <span class="k">if</span> <span class="ow">not</span> <span class="n">crops</span><span class="p">:</span> <span class="k">continue</span>
        <span class="n">vecs</span> <span class="o">=</span> <span class="nf">embed</span><span class="p">(</span><span class="n">model</span><span class="p">,</span> <span class="n">proc</span><span class="p">,</span> <span class="n">crops</span><span class="p">)</span>
        <span class="n">D</span><span class="p">,</span> <span class="n">I</span> <span class="o">=</span> <span class="n">index</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="nf">_as_float32</span><span class="p">(</span><span class="n">vecs</span><span class="p">),</span> <span class="n">TOPK</span><span class="p">)</span>

        <span class="n">mt</span> <span class="o">=</span> <span class="nf">_metric_type</span><span class="p">(</span><span class="n">index</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">mt</span> <span class="o">==</span> <span class="nf">getattr</span><span class="p">(</span><span class="n">faiss</span><span class="p">,</span> <span class="sh">"</span><span class="s">METRIC_L2</span><span class="sh">"</span><span class="p">,</span> <span class="mi">1</span><span class="p">):</span>
            <span class="n">scores_mat</span> <span class="o">=</span> <span class="nf">_l2_to_cosine</span><span class="p">(</span><span class="n">D</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="n">scores_mat</span> <span class="o">=</span> <span class="n">D</span>

        <span class="c1"># collect frame-local debug matches as well
</span>        <span class="n">frame_dbg</span> <span class="o">=</span> <span class="p">[]</span> <span class="k">if</span> <span class="n">dump_all</span> <span class="k">else</span> <span class="bp">None</span>
        <span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="p">(</span><span class="n">scores</span><span class="p">,</span> <span class="n">ids</span><span class="p">)</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="nf">zip</span><span class="p">(</span><span class="n">scores_mat</span><span class="p">,</span> <span class="n">I</span><span class="p">)):</span>
            <span class="k">if</span> <span class="n">dump_all</span><span class="p">:</span>
                <span class="k">for</span> <span class="n">r</span><span class="p">,</span> <span class="p">(</span><span class="n">score</span><span class="p">,</span> <span class="n">idx</span><span class="p">)</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="nf">zip</span><span class="p">(</span><span class="n">scores</span><span class="p">.</span><span class="nf">tolist</span><span class="p">(),</span> <span class="n">ids</span><span class="p">.</span><span class="nf">tolist</span><span class="p">())):</span>
                    <span class="n">rec</span> <span class="o">=</span> <span class="p">{</span>
                        <span class="sh">"</span><span class="s">frame</span><span class="sh">"</span><span class="p">:</span> <span class="n">fpath</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
                        <span class="sh">"</span><span class="s">bbox</span><span class="sh">"</span><span class="p">:</span> <span class="n">boxes</span><span class="p">[</span><span class="n">i</span><span class="p">],</span>
                        <span class="sh">"</span><span class="s">rank</span><span class="sh">"</span><span class="p">:</span> <span class="nf">int</span><span class="p">(</span><span class="n">r</span><span class="p">),</span>
                        <span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">score</span><span class="p">),</span>
                        <span class="sh">"</span><span class="s">brand</span><span class="sh">"</span><span class="p">:</span> <span class="n">brands</span><span class="p">[</span><span class="n">idx</span><span class="p">]</span>
                    <span class="p">}</span>
                    <span class="n">dbg_all</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">rec</span><span class="p">)</span>
                    <span class="k">if</span> <span class="n">frame_dbg</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
                        <span class="n">frame_dbg</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">rec</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">score</span><span class="p">,</span> <span class="n">idx</span> <span class="ow">in</span> <span class="nf">zip</span><span class="p">(</span><span class="n">scores</span><span class="p">,</span> <span class="n">ids</span><span class="p">):</span>
                <span class="k">if</span> <span class="n">score</span> <span class="o">&gt;</span> <span class="n">best</span><span class="p">[</span><span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">]:</span>
                    <span class="n">best</span><span class="p">.</span><span class="nf">update</span><span class="p">({</span>
                        <span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">score</span><span class="p">),</span>
                        <span class="sh">"</span><span class="s">frame</span><span class="sh">"</span><span class="p">:</span> <span class="n">fpath</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
                        <span class="sh">"</span><span class="s">brand</span><span class="sh">"</span><span class="p">:</span> <span class="n">brands</span><span class="p">[</span><span class="n">idx</span><span class="p">],</span>
                        <span class="sh">"</span><span class="s">bbox</span><span class="sh">"</span><span class="p">:</span> <span class="n">boxes</span><span class="p">[</span><span class="n">i</span><span class="p">]</span>
                    <span class="p">})</span>
                <span class="k">if</span> <span class="n">score</span> <span class="o">&lt;</span> <span class="n">COSINE_TH</span><span class="p">:</span>
                    <span class="k">continue</span>
                <span class="n">bx</span> <span class="o">=</span> <span class="n">boxes</span><span class="p">[</span><span class="n">i</span><span class="p">]</span>
                <span class="n">out</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                    <span class="sh">"</span><span class="s">frame</span><span class="sh">"</span><span class="p">:</span> <span class="n">fpath</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
                    <span class="sh">"</span><span class="s">bbox</span><span class="sh">"</span><span class="p">:</span> <span class="n">bx</span><span class="p">,</span>
                    <span class="sh">"</span><span class="s">score_retr</span><span class="sh">"</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">score</span><span class="p">),</span>
                    <span class="sh">"</span><span class="s">brand</span><span class="sh">"</span><span class="p">:</span> <span class="n">brands</span><span class="p">[</span><span class="n">idx</span><span class="p">]</span>
                <span class="p">})</span>

        <span class="c1"># draw annotations for this frame if requested
</span>        <span class="k">if</span> <span class="n">debug_draw</span> <span class="ow">and</span> <span class="n">frame_dbg</span><span class="p">:</span>
            <span class="c1"># sort by score desc
</span>            <span class="n">frame_dbg</span><span class="p">.</span><span class="nf">sort</span><span class="p">(</span><span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">r</span><span class="p">:</span> <span class="n">r</span><span class="p">[</span><span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">],</span> <span class="n">reverse</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
            <span class="k">if</span> <span class="n">debug_draw_max</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
                <span class="n">frame_dbg</span> <span class="o">=</span> <span class="n">frame_dbg</span><span class="p">[:</span><span class="n">debug_draw_max</span><span class="p">]</span>
            <span class="n">canvas</span> <span class="o">=</span> <span class="n">im</span><span class="p">.</span><span class="nf">copy</span><span class="p">()</span>
            <span class="n">draw</span> <span class="o">=</span> <span class="n">ImageDraw</span><span class="p">.</span><span class="nc">Draw</span><span class="p">(</span><span class="n">canvas</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">rec</span> <span class="ow">in</span> <span class="n">frame_dbg</span><span class="p">:</span>
                <span class="k">if</span> <span class="n">rec</span><span class="p">[</span><span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">]</span> <span class="o">&lt;</span> <span class="n">debug_draw_th</span><span class="p">:</span>
                    <span class="k">continue</span>
                <span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">h</span> <span class="o">=</span> <span class="n">rec</span><span class="p">[</span><span class="sh">"</span><span class="s">bbox</span><span class="sh">"</span><span class="p">]</span>
                <span class="n">x2</span><span class="p">,</span> <span class="n">y2</span> <span class="o">=</span> <span class="n">x</span> <span class="o">+</span> <span class="n">w</span><span class="p">,</span> <span class="n">y</span> <span class="o">+</span> <span class="n">h</span>
                <span class="n">color</span> <span class="o">=</span> <span class="p">(</span><span class="mi">255</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span>
                <span class="n">draw</span><span class="p">.</span><span class="nf">rectangle</span><span class="p">([</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">x2</span><span class="p">,</span> <span class="n">y2</span><span class="p">],</span> <span class="n">outline</span><span class="o">=</span><span class="n">color</span><span class="p">,</span> <span class="n">width</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span>
                <span class="n">label</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">rec</span><span class="p">[</span><span class="sh">'</span><span class="s">brand</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">rec</span><span class="p">[</span><span class="sh">'</span><span class="s">score</span><span class="sh">'</span><span class="p">]</span><span class="si">:</span><span class="p">.</span><span class="mi">3</span><span class="n">f</span><span class="si">}</span><span class="s">#</span><span class="si">{</span><span class="n">rec</span><span class="p">[</span><span class="sh">'</span><span class="s">rank</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span>
                <span class="c1"># simple text; if background needed, draw a small filled box then text
</span>                <span class="n">draw</span><span class="p">.</span><span class="nf">text</span><span class="p">((</span><span class="n">x</span> <span class="o">+</span> <span class="mi">3</span><span class="p">,</span> <span class="n">y</span> <span class="o">+</span> <span class="mi">3</span><span class="p">),</span> <span class="n">label</span><span class="p">,</span> <span class="n">fill</span><span class="o">=</span><span class="n">color</span><span class="p">)</span>
            <span class="n">out_path</span> <span class="o">=</span> <span class="n">debug_draw_dir</span> <span class="o">/</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">fpath</span><span class="p">.</span><span class="n">stem</span><span class="si">}</span><span class="s">_debug.jpg</span><span class="sh">"</span>
            <span class="n">canvas</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="n">out_path</span><span class="p">,</span> <span class="n">quality</span><span class="o">=</span><span class="mi">90</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[retrieve] wrote debug visualization → </span><span class="si">{</span><span class="n">out_path</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">candidates.jsonl</span><span class="sh">"</span><span class="p">).</span><span class="nf">write_text</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">x</span><span class="p">)</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="n">out</span><span class="p">))</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">wrote </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">out</span><span class="p">)</span><span class="si">}</span><span class="s"> retrieval candidates → candidates.jsonl</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">out</span> <span class="ow">and</span> <span class="n">best</span><span class="p">[</span><span class="sh">"</span><span class="s">frame</span><span class="sh">"</span><span class="p">]</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[retrieve] no candidates above threshold </span><span class="si">{</span><span class="n">COSINE_TH</span><span class="si">}</span><span class="s">. </span><span class="sh">"</span>
              <span class="sa">f</span><span class="sh">"</span><span class="s">Best observed: score=</span><span class="si">{</span><span class="n">best</span><span class="p">[</span><span class="sh">'</span><span class="s">score</span><span class="sh">'</span><span class="p">]</span><span class="si">:</span><span class="p">.</span><span class="mi">3</span><span class="n">f</span><span class="si">}</span><span class="s">, frame=</span><span class="si">{</span><span class="n">best</span><span class="p">[</span><span class="sh">'</span><span class="s">frame</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">, </span><span class="sh">"</span>
              <span class="sa">f</span><span class="sh">"</span><span class="s">brand=</span><span class="si">{</span><span class="n">best</span><span class="p">[</span><span class="sh">'</span><span class="s">brand</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">, bbox=</span><span class="si">{</span><span class="n">best</span><span class="p">[</span><span class="sh">'</span><span class="s">bbox</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> — consider lowering COSINE_TH.</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">dump_all</span> <span class="ow">and</span> <span class="n">dbg_all</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
        <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">debug_matches.jsonl</span><span class="sh">"</span><span class="p">).</span><span class="nf">write_text</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">x</span><span class="p">)</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="n">dbg_all</span><span class="p">))</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[retrieve] wrote </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">dbg_all</span><span class="p">)</span><span class="si">}</span><span class="s"> raw matches → debug_matches.jsonl (DEBUG_DUMP_ALL=1)</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>
</code></pre>

</div>



<p>This is quite a few things so breaking it down:</p>

<ul>
<li>Crops: multi-scale grid over each frame using <code>SIZES</code> and <code>STRIDE</code>, yielding square patches and their <code>bbox</code> tuples.</li>
<li>Embeddings: SigLIP‑2 <code>get_image_features</code> on each crop batch, then L2‑normalize features.</li>
<li>Search: FAISS over the logo index; uses inner‑product on normalized vectors (cosine). If the index is L2, we convert to cosine (<code>1 − 0.5·L2²</code>).</li>
<li>Types: FAISS expects contiguous <code>float32</code>; embeddings are cast/contiguous before <code>index.search</code>.</li>
<li>Thresholds: keep top‑<code>TOPK</code> per crop, then filter by <code>COSINE_TH</code>. You can override at runtime:

<ul>
<li><code>TOPK=10 COSINE_TH=0.65 python retrieve.py</code></li>
</ul>


</li>

<li>Debugging:

<ul>
<li>
<code>DEBUG_DUMP_ALL=1</code> writes every raw match to <code>debug_matches.jsonl</code> (ranked with scores).</li>
<li>
<code>DEBUG_DRAW=1</code> also saves <code>artifacts/debug_vis/*_debug.jpg</code> with [brand score#rank] boxes.</li>
<li>Optional: <code>DEBUG_DRAW_TH=0.3</code> (only draw ≥ threshold), <code>DEBUG_DRAW_MAX=200</code> (cap boxes).</li>
</ul>


</li>

<li>Outputs: filtered candidates land in <code>candidates.jsonl</code> and are fed to the verifier stage.</li>

<li>Knobs to tune recall/precision: crop sizes, stride, <code>TOPK</code>, <code>COSINE_TH</code>. For higher recall, increase sizes or <code>TOPK</code>; for precision, raise the threshold and later add a margin or temporal smoothing.</li>

</ul>

<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ python retrieve.py
[retrieve] index ntotal=1, brands=1
[retrieve] frames found=88
wrote 25 retrieval candidates → candidates.jsonl
</code></pre>

</div>



<p>Depending on the source material this probably needs tuning for recall/precision. The debugging knobs are quite nice. Here's an example frame from the video showing how it looks like on frame #7:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fprpqziqgwabqskgollfn.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fprpqziqgwabqskgollfn.jpg" alt="Example frame with SigLIP-2 retrieval overlay: Red Bull logo detected at cosine 0.70 (rank #0) on the car sidepod" width="800" height="450"></a></p>

<p><em>Frame from Red Bull Racing Pit Stop Practice (2015), ProtoplasmaKid / Wikimedia Commons / CC BY-SA 4.0.</em></p>

<p>5) VLM verification</p>

<p>Save the following to <code>verify.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span><span class="p">,</span> <span class="n">torch</span><span class="p">,</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">pathlib</span> <span class="kn">import</span> <span class="n">Path</span>
<span class="kn">from</span> <span class="n">PIL</span> <span class="kn">import</span> <span class="n">Image</span>
<span class="kn">from</span> <span class="n">transformers</span> <span class="kn">import</span> <span class="n">AutoProcessor</span><span class="p">,</span> <span class="n">AutoModelForCausalLM</span>

<span class="n">VLM_ID</span> <span class="o">=</span> <span class="sh">"</span><span class="s">lmms-lab/LLaVA-OneVision-1.5-8B-Instruct</span><span class="sh">"</span>

<span class="n">SYSTEM</span> <span class="o">=</span> <span class="p">(</span>
  <span class="sh">"</span><span class="s">You are a logo verification API. </span><span class="sh">"</span>
  <span class="sh">"</span><span class="s">Given an image crop and a target brand, answer in strict JSON with no extra text.</span><span class="sh">"</span>
<span class="p">)</span>

<span class="k">def</span> <span class="nf">build_user_prompt</span><span class="p">(</span><span class="n">brand</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="nf">return </span><span class="p">(</span>
      <span class="sh">"</span><span class="s">Task: Verify whether the crop contains the specified brand</span><span class="sh">'</span><span class="s">s logo.</span><span class="se">\n</span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="s">Brand: </span><span class="si">{</span><span class="n">brand</span><span class="si">}</span><span class="se">\n\n</span><span class="sh">"</span>
      <span class="sh">"</span><span class="s">Return JSON only:</span><span class="se">\n</span><span class="sh">"</span>
      <span class="sh">"</span><span class="s">{</span><span class="se">\n</span><span class="sh">"</span>
      <span class="sh">'</span><span class="s">  </span><span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">yes</span><span class="sh">"</span><span class="s"> | </span><span class="sh">"</span><span class="s">no</span><span class="sh">"</span><span class="s">,</span><span class="se">\n</span><span class="sh">'</span>
      <span class="sh">'</span><span class="s">  </span><span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="s">: 0.0-1.0,</span><span class="se">\n</span><span class="sh">'</span>
      <span class="sh">'</span><span class="s">  </span><span class="sh">"</span><span class="s">visual_cues</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">short, literal cues proving the verdict (colors/shapes/text)</span><span class="sh">"</span><span class="se">\n</span><span class="sh">'</span>
      <span class="sh">"</span><span class="s">}</span><span class="se">\n</span><span class="sh">"</span>
      <span class="sh">"</span><span class="s">Rules: output only the JSON object; no prose before or after. </span><span class="sh">"</span>
      <span class="sh">"</span><span class="s">Be literal; do not speculate; base confidence on how clearly the logo is visible.</span><span class="sh">"</span>
    <span class="p">)</span>

<span class="k">def</span> <span class="nf">_normalize_quotes</span><span class="p">(</span><span class="n">s</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="c1"># Replace smart quotes with ASCII equivalents
</span>    <span class="nf">return </span><span class="p">(</span>
        <span class="n">s</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sh">"</span><span class="se">\u201c</span><span class="sh">"</span><span class="p">,</span> <span class="sh">'"'</span><span class="p">).</span><span class="nf">replace</span><span class="p">(</span><span class="sh">"</span><span class="se">\u201d</span><span class="sh">"</span><span class="p">,</span> <span class="sh">'"'</span><span class="p">)</span>
         <span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sh">"</span><span class="se">\u2018</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"'"</span><span class="p">).</span><span class="nf">replace</span><span class="p">(</span><span class="sh">"</span><span class="se">\u2019</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"'"</span><span class="p">)</span>
    <span class="p">)</span>

<span class="k">def</span> <span class="nf">_extract_json_object</span><span class="p">(</span><span class="n">text</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="n">text</span> <span class="o">=</span> <span class="nf">_normalize_quotes</span><span class="p">(</span><span class="n">text</span><span class="p">)</span>
    <span class="c1"># Find first balanced {...} block; handle braces within strings
</span>    <span class="n">in_str</span> <span class="o">=</span> <span class="bp">False</span>
    <span class="n">escape</span> <span class="o">=</span> <span class="bp">False</span>
    <span class="n">depth</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="n">start</span> <span class="o">=</span> <span class="bp">None</span>
    <span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="n">ch</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="n">text</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">ch</span> <span class="o">==</span> <span class="sh">"</span><span class="se">\\</span><span class="sh">"</span> <span class="ow">and</span> <span class="ow">not</span> <span class="n">escape</span><span class="p">:</span>
            <span class="n">escape</span> <span class="o">=</span> <span class="bp">True</span>
            <span class="k">continue</span>
        <span class="k">if</span> <span class="n">ch</span> <span class="o">==</span> <span class="sh">'"'</span> <span class="ow">and</span> <span class="ow">not</span> <span class="n">escape</span><span class="p">:</span>
            <span class="n">in_str</span> <span class="o">=</span> <span class="ow">not</span> <span class="n">in_str</span>
        <span class="n">escape</span> <span class="o">=</span> <span class="bp">False</span>
        <span class="k">if</span> <span class="n">in_str</span><span class="p">:</span>
            <span class="k">continue</span>
        <span class="k">if</span> <span class="n">ch</span> <span class="o">==</span> <span class="sh">"</span><span class="s">{</span><span class="sh">"</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">depth</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>
                <span class="n">start</span> <span class="o">=</span> <span class="n">i</span>
            <span class="n">depth</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="k">elif</span> <span class="n">ch</span> <span class="o">==</span> <span class="sh">"</span><span class="s">}</span><span class="sh">"</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">depth</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
                <span class="n">depth</span> <span class="o">-=</span> <span class="mi">1</span>
                <span class="k">if</span> <span class="n">depth</span> <span class="o">==</span> <span class="mi">0</span> <span class="ow">and</span> <span class="n">start</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
                    <span class="n">candidate</span> <span class="o">=</span> <span class="n">text</span><span class="p">[</span><span class="n">start</span><span class="p">:</span><span class="n">i</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>
                    <span class="k">try</span><span class="p">:</span>
                        <span class="n">j</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">candidate</span><span class="p">)</span>
                        <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">j</span><span class="p">,</span> <span class="nb">dict</span><span class="p">)</span> <span class="ow">and</span> <span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">j</span> <span class="ow">and</span> <span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">j</span><span class="p">:</span>
                            <span class="k">return</span> <span class="n">j</span>
                    <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
                        <span class="k">pass</span>
    <span class="k">return</span> <span class="bp">None</span>

<span class="k">def</span> <span class="nf">run_once</span><span class="p">(</span><span class="n">proc</span><span class="p">,</span> <span class="n">model</span><span class="p">,</span> <span class="n">crop</span><span class="p">:</span> <span class="n">Image</span><span class="p">.</span><span class="n">Image</span><span class="p">,</span> <span class="n">brand</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">max_new</span><span class="o">=</span><span class="mi">128</span><span class="p">):</span>
    <span class="n">msgs</span> <span class="o">=</span> <span class="p">[</span>
        <span class="p">{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span><span class="sh">"</span><span class="s">system</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">SYSTEM</span><span class="p">},</span>
        <span class="p">{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span><span class="sh">"</span><span class="s">image</span><span class="sh">"</span><span class="p">},</span> <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="nf">build_user_prompt</span><span class="p">(</span><span class="n">brand</span><span class="p">)}]}</span>
    <span class="p">]</span>
    <span class="n">text</span> <span class="o">=</span> <span class="n">proc</span><span class="p">.</span><span class="nf">apply_chat_template</span><span class="p">(</span><span class="n">msgs</span><span class="p">,</span> <span class="n">tokenize</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span> <span class="n">add_generation_prompt</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">inputs</span> <span class="o">=</span> <span class="nf">proc</span><span class="p">(</span><span class="n">text</span><span class="o">=</span><span class="p">[</span><span class="n">text</span><span class="p">],</span> <span class="n">images</span><span class="o">=</span><span class="p">[</span><span class="n">crop</span><span class="p">],</span> <span class="n">padding</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">return_tensors</span><span class="o">=</span><span class="sh">"</span><span class="s">pt</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">inputs</span> <span class="o">=</span> <span class="p">{</span><span class="n">k</span><span class="p">:</span> <span class="p">(</span><span class="n">v</span><span class="p">.</span><span class="nf">to</span><span class="p">(</span><span class="n">model</span><span class="p">.</span><span class="n">device</span><span class="p">)</span> <span class="k">if</span> <span class="nf">hasattr</span><span class="p">(</span><span class="n">v</span><span class="p">,</span><span class="sh">"</span><span class="s">to</span><span class="sh">"</span><span class="p">)</span> <span class="k">else</span> <span class="n">v</span><span class="p">)</span> <span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="ow">in</span> <span class="n">inputs</span><span class="p">.</span><span class="nf">items</span><span class="p">()}</span>
    <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">inference_mode</span><span class="p">():</span>
        <span class="n">ids</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">generate</span><span class="p">(</span><span class="o">**</span><span class="n">inputs</span><span class="p">,</span> <span class="n">max_new_tokens</span><span class="o">=</span><span class="n">max_new</span><span class="p">,</span> <span class="n">temperature</span><span class="o">=</span><span class="mf">0.0</span><span class="p">,</span> <span class="n">do_sample</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
    <span class="n">out</span> <span class="o">=</span> <span class="n">proc</span><span class="p">.</span><span class="nf">batch_decode</span><span class="p">(</span><span class="n">ids</span><span class="p">,</span> <span class="n">skip_special_tokens</span><span class="o">=</span><span class="bp">True</span><span class="p">)[</span><span class="mi">0</span><span class="p">]</span>
    <span class="n">j</span> <span class="o">=</span> <span class="nf">_extract_json_object</span><span class="p">(</span><span class="n">out</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">j</span> <span class="ow">is</span> <span class="bp">None</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">DEBUG_VLM</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">0</span><span class="sh">"</span><span class="p">)</span> <span class="o">==</span> <span class="sh">"</span><span class="s">1</span><span class="sh">"</span><span class="p">:</span>
            <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">vlm_raw.txt</span><span class="sh">"</span><span class="p">).</span><span class="nf">write_text</span><span class="p">(</span><span class="n">out</span><span class="p">)</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span><span class="p">:</span><span class="sh">"</span><span class="s">no</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">:</span><span class="mf">0.0</span><span class="p">,</span> <span class="sh">"</span><span class="s">visual_cues</span><span class="sh">"</span><span class="p">:</span><span class="sh">"</span><span class="s">parse_error</span><span class="sh">"</span><span class="p">}</span>
    <span class="c1"># Coerce fields
</span>    <span class="n">verdict</span> <span class="o">=</span> <span class="nf">str</span><span class="p">(</span><span class="n">j</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">no</span><span class="sh">"</span><span class="p">)).</span><span class="nf">strip</span><span class="p">().</span><span class="nf">lower</span><span class="p">()</span>
    <span class="k">if</span> <span class="n">verdict</span> <span class="ow">not</span> <span class="ow">in</span> <span class="p">(</span><span class="sh">"</span><span class="s">yes</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">no</span><span class="sh">"</span><span class="p">):</span>
        <span class="n">verdict</span> <span class="o">=</span> <span class="sh">"</span><span class="s">no</span><span class="sh">"</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">conf</span> <span class="o">=</span> <span class="nf">float</span><span class="p">(</span><span class="n">j</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">))</span>
    <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
        <span class="n">conf</span> <span class="o">=</span> <span class="mf">0.0</span>
    <span class="n">cues</span> <span class="o">=</span> <span class="nf">str</span><span class="p">(</span><span class="n">j</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">visual_cues</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">))</span>
    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span><span class="p">:</span> <span class="n">verdict</span><span class="p">,</span> <span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">:</span> <span class="n">conf</span><span class="p">,</span> <span class="sh">"</span><span class="s">visual_cues</span><span class="sh">"</span><span class="p">:</span> <span class="n">cues</span><span class="p">}</span>

<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">proc</span> <span class="o">=</span> <span class="n">AutoProcessor</span><span class="p">.</span><span class="nf">from_pretrained</span><span class="p">(</span><span class="n">VLM_ID</span><span class="p">,</span> <span class="n">trust_remote_code</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">AutoModelForCausalLM</span><span class="p">.</span><span class="nf">from_pretrained</span><span class="p">(</span>
        <span class="n">VLM_ID</span><span class="p">,</span> <span class="n">dtype</span><span class="o">=</span><span class="n">torch</span><span class="p">.</span><span class="n">float32</span><span class="p">,</span> <span class="n">device_map</span><span class="o">=</span><span class="sh">"</span><span class="s">auto</span><span class="sh">"</span><span class="p">,</span> <span class="n">trust_remote_code</span><span class="o">=</span><span class="bp">True</span>
    <span class="p">)</span>
    <span class="n">results</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">candidates.jsonl</span><span class="sh">"</span><span class="p">).</span><span class="nf">read_text</span><span class="p">().</span><span class="nf">splitlines</span><span class="p">():</span>
        <span class="n">c</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">line</span><span class="p">)</span>
        <span class="n">im</span> <span class="o">=</span> <span class="n">Image</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">frames</span><span class="sh">"</span><span class="p">)</span><span class="o">/</span><span class="n">c</span><span class="p">[</span><span class="sh">"</span><span class="s">frame</span><span class="sh">"</span><span class="p">]).</span><span class="nf">convert</span><span class="p">(</span><span class="sh">"</span><span class="s">RGB</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="p">,</span><span class="n">w</span><span class="p">,</span><span class="n">h</span> <span class="o">=</span> <span class="n">c</span><span class="p">[</span><span class="sh">"</span><span class="s">bbox</span><span class="sh">"</span><span class="p">]</span>
        <span class="n">crop</span> <span class="o">=</span> <span class="n">im</span><span class="p">.</span><span class="nf">crop</span><span class="p">((</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="p">,</span><span class="n">x</span><span class="o">+</span><span class="n">w</span><span class="p">,</span><span class="n">y</span><span class="o">+</span><span class="n">h</span><span class="p">))</span>
        <span class="n">j</span> <span class="o">=</span> <span class="nf">run_once</span><span class="p">(</span><span class="n">proc</span><span class="p">,</span> <span class="n">model</span><span class="p">,</span> <span class="n">crop</span><span class="p">,</span> <span class="n">c</span><span class="p">[</span><span class="sh">"</span><span class="s">brand</span><span class="sh">"</span><span class="p">])</span>
        <span class="n">results</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
          <span class="o">**</span><span class="n">c</span><span class="p">,</span>
          <span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span><span class="p">:</span> <span class="n">j</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">verdict</span><span class="sh">"</span><span class="p">,</span><span class="sh">"</span><span class="s">no</span><span class="sh">"</span><span class="p">),</span>
          <span class="sh">"</span><span class="s">confidence_vlm</span><span class="sh">"</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">j</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">,</span><span class="mf">0.0</span><span class="p">)),</span>
          <span class="sh">"</span><span class="s">rationale</span><span class="sh">"</span><span class="p">:</span> <span class="n">j</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">visual_cues</span><span class="sh">"</span><span class="p">,</span><span class="sh">""</span><span class="p">)</span>
        <span class="p">})</span>
    <span class="nc">Path</span><span class="p">(</span><span class="sh">"</span><span class="s">detections.jsonl</span><span class="sh">"</span><span class="p">).</span><span class="nf">write_text</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">r</span><span class="p">,</span> <span class="n">ensure_ascii</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span> <span class="k">for</span> <span class="n">r</span> <span class="ow">in</span> <span class="n">results</span><span class="p">))</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">wrote detections.jsonl</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>
</code></pre>

</div>



<p>Again, breaking it down:</p>

<ul>
<li>Inputs: <code>candidates.jsonl</code> entries with <code>frame</code>, <code>bbox</code>, <code>score_retr</code>, <code>brand</code>.</li>
<li>Crop: open the frame and <code>im.crop(bbox)</code> per candidate.</li>
<li>Prompting: one system + one user message; user contains the target brand and an explicit "JSON only" schema.</li>
<li>Generation: <code>temperature=0.0</code>, <code>do_sample=False</code>, <code>max_new_tokens=128</code> for deterministic outputs.</li>
<li>Parsing: balanced-brace JSON extraction with smart-quote normalization; on failure returns <code>parse_error</code>. Set <code>DEBUG_VLM=1</code> to dump raw text to <code>vlm_raw.txt</code>.</li>
<li>Output: writes <code>detections.jsonl</code> with <code>verdict</code>, <code>confidence_vlm</code>, <code>rationale</code> merged onto each candidate.</li>
<li>Cost control: keep retrieval strict (lower <code>TOPK</code>, higher <code>COSINE_TH</code>) to limit VLM calls as that's the main runtime driver.</li>
</ul>

<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ python verify.py
wrote detections.jsonl
</code></pre>

</div>



<p>And what does <code>detections.jsonl</code> look like? It has items like this (one per line):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "frame": "f_000007.jpg",
  "bbox": [
    384,
    288,
    192,
    192
  ],
  "score_retr": 0.7020304203033447,
  "brand": "Red_Bull_logo",
  "verdict": "yes",
  "confidence_vlm": 0.95,
  "rationale": "red bull charging bull silhouette, red and yellow colors"
}
</code></pre>

</div>



<p>Here 0.702 is the cosine similarity between the crop and the logo embedding.</p>

<h3>
  
  
  Conclusion
</h3>

<p>I hope this showcases how a simple two-stage recipe - SigLIP-2 retrieval + VLM verification - can turn a semi-noisy video into reviewable brand evidence. I do want to mention that a pipeline like this is meant as a powerful filter, not an oracle. Human in the loop needed.</p>

<p>On this specific 44-second clip, with the thresholds above, I get N true positives, M false positives, and 0 missed clear logos (subjective visual check).</p>

<p>That being said, this is probably not a production-ready detector. We trade some accuracy and runtime for simplicity and transparency. Some improvement ideas I had in mind:</p>

<ul>
<li>
<code>float32</code> dtype could be replaced with lower precision, like <code>bfloat16</code>.</li>
<li>Add a margin filter (top1 − top2 ≥ 0.15) and simple temporal smoothing. Confirm across 2–3 adjacent frames.</li>
<li>Add a lightweight OCR gate for texty marks to backstop retrieval.</li>
<li>Calibrate thresholds per brand, as some logos need higher <code>COSINE_TH</code> or <code>TH_VLM</code>.</li>
<li>Expand the logo dictionary and try multi-resolution templates (flat vs curved surfaces).</li>
<li>Speed/scale: use FAISS IVF/PQ for larger dictionaries.</li>
<li>Quantize the VLM or batch crops.</li>
<li>Maybe consider a second encoder for consensus (SigLIP‑2 + CLIP) to reduce lookalikes.</li>
<li>For 4K frames, consider either down-scaling first or increasing STRIDE (e.g. 0.75) to avoid generating tens of thousands of crops per frame.</li>
</ul>

<p>As always, credits: thanks to the SigLIP-2 authors &amp; maintainers and the LLaVA-OneVision team for open releases. And attribution for the example video: ProtoplasmaKid / Wikimedia Commons / CC BY-SA 4.0.  ￼</p>

