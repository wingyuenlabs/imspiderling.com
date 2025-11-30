---
Title: One-command ComfyUI on Cloud GPUs: A Practical, Repeatable Setup
Description: 
Author: Prompting Pixels
Date: 2025-11-30T21:59:33.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What we're building
</h2>

<p>A repeatable way to boot a cloud GPU (RunPod or Vast.ai), paste a single command, grab the exact ComfyUI version you want, auto-install your favorite custom nodes, and download models from Hugging Face/Civitai into the correct folders. No more “did I put that LoRA in the right place?” or “why is this template six months behind?”.</p>

<p>We’ll use a free script generator to produce the one-liner and show you how to tweak, debug, and extend it for your workflow.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo36qlsnlysusrn7o7oxo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo36qlsnlysusrn7o7oxo.png" alt="Prompting Pixels AI Launcher web page showing a welcome panel, provider radios (Vast.ai selected), a bash wget installation one-liner, purple 'Deploy on RunPod' and black 'Deploy on Vast.ai' buttons, and save/load/preset configuration buttons; contact email deploy@promptingpixels.com visible." width="800" height="520"></a></p>

<blockquote>
<p>💡 Pro tip: Time is literally money on cloud GPUs. Automating the boring parts pays for itself on the first run.</p>
</blockquote>




<h2>
  
  
  Why this was a pain (until now)
</h2>

<p>If you’ve tried to stand up ComfyUI on a fresh GPU instance, you’ve probably done some combination of:</p>

<ul>
<li><p>Opening a terminal, then manually git pulling ComfyUI to get a newer version</p></li>
<li><p>Downloading models piecemeal from Hugging Face or Civitai</p></li>
<li><p>Guessing model folder locations (checkpoints vs LoRA vs VAE)</p></li>
<li><p>Cloning custom nodes and hoping dependencies resolve</p></li>
<li><p>Restarting ComfyUI multiple times to make new nodes appear</p></li>
</ul>

<p>That’s too many manual steps, which means it’s slow, error-prone, and easy to forget when you come back a week later.</p>




<h2>
  
  
  The fix: generate a deployment command once, reuse forever
</h2>

<p>We’ll use the generator at:</p>

<ul>
<li>  <a href="https://deploy.promptingpixels.com/" rel="noopener noreferrer">https://deploy.promptingpixels.com/</a>
</li>
</ul>

<p>It outputs a one-line shell command that:</p>

<ul>
<li><p>Installs or updates ComfyUI to a specific version</p></li>
<li><p>Downloads your selected models into the correct ComfyUI subfolders</p></li>
<li><p>Installs custom nodes you choose from the ComfyUI registry</p></li>
<li><p>Adapts paths to your provider (RunPod/Vast.ai) or your local OS</p></li>
<li><p>Supports tokens for gated downloads</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ldnbe18zavqxs6aldv0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ldnbe18zavqxs6aldv0.png" alt="Dark-themed AI deployment web UI showing Installation Script with One-Liner tab, red arrow and text 'One line is all you need :-)', a bash wget command, 'Deploy on RunPod' and 'Deploy on Vast.ai' buttons, and configuration summary with Add Models button." width="800" height="520"></a></p>

<blockquote>
<p>🧭 Heads up: The generated command is provider-aware. Pick the right target before copying.</p>
</blockquote>




<h2>
  
  
  Hands-on: from blank GPU to ComfyUI, step-by-step
</h2>

<p>We’ll demonstrate with Vast.ai, but I’ll call out RunPod differences as we go.</p>

<h3>
  
  
  1) Launch a GPU instance
</h3>

<ul>
<li><p>Vast.ai: Use an image/template that includes a Jupyter Terminal or shell access.</p></li>
<li><p>RunPod: Either the ComfyUI template or a general-purpose image with CUDA.</p></li>
</ul>

<p>Open a terminal on the instance:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhf32tfnrowdhgwx1f9n8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhf32tfnrowdhgwx1f9n8.png" alt="Vast.ai Applications dashboard with tiles and blue 'Launch Application' buttons; a red arrow points to the 'Jupyter Terminal' tile." width="800" height="520"></a></p>

<h3>
  
  
  2) Generate your script
</h3>

<ul>
<li><p>Visit <a href="https://deploy.promptingpixels.com/" rel="noopener noreferrer">https://deploy.promptingpixels.com/</a></p></li>
<li><p>Choose App: ComfyUI</p></li>
<li><p>Pick the provider (Vast.ai or RunPod)</p></li>
<li>
<p>Add Models:</p>

<ul>
<li>  Search from Hugging Face or Civitai</li>
<li>  The generator will route each file to the correct ComfyUI directory</li>
</ul>


</li>

<li>

<p>Add Custom Nodes:</p>

<ul>
<li>  Search popular nodes (e.g., Impact Pack) and add them</li>
</ul>


</li>

<li><p>Optionally pin the ComfyUI version (handy for reproducible builds)</p></li>

</ul>

<blockquote>
<p>💡 Pro tip: Use presets to recreate environments from previous projects. Consistency saves debugging time.</p>
</blockquote>

<h3>
  
  
  3) Copy the one-liner and run it
</h3>

<p>Paste the generated command into your instance terminal. It typically looks like a wget/curl piping into bash. If you need tokens for gated models, export them first:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Optional: tokens for gated downloads</span>
<span class="nb">export </span><span class="nv">HF_TOKEN</span><span class="o">=</span>hf_your_read_token_here
<span class="nb">export </span><span class="nv">CIVITAI_TOKEN</span><span class="o">=</span>your_civitai_token_here

<span class="c"># Example shape of the generated command (yours will be specific)</span>
bash &lt;<span class="o">(</span>wget <span class="nt">-qO-</span> https://deploy.promptingpixels.com//api/script/cmim9aus50008n5vdgw3g00yv<span class="o">)</span> <span class="nt">--hf-token</span><span class="o">=</span>YOUR_HF_TOKEN
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnvzk07prjdvoyy6wuzbx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnvzk07prjdvoyy6wuzbx.png" alt="Jupyter terminal showing 'Activated conda/uv virtual environment at /venv/main' in green, red provisioning warnings, and a bash wget command with a redacted token; browser URL shows 'Not Secure' and an IP address" width="800" height="520"></a></p>

<p>Go grab a coffee. When it finishes, ComfyUI will be set up with your exact configuration.</p>

<h3>
  
  
  4) Launch ComfyUI
</h3>

<p>Start ComfyUI from your provider’s UI (or the app menu). If new nodes don’t show up in the menu, do a quick restart of the app.</p>




<h2>
  
  
  Verify and debug like a developer
</h2>

<p>I like to sanity-check a fresh environment with a few quick commands.</p>

<h3>
  
  
  Check the ComfyUI version you actually deployed
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">COMFYUI_ROOT</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">COMFYUI_ROOT</span><span class="k">:-</span><span class="p">/workspace/ComfyUI</span><span class="k">}</span><span class="s2">"</span>  <span class="c"># adjust per provider if needed</span>
git <span class="nt">-C</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">"</span> rev-parse <span class="nt">--short</span> HEAD
</code></pre>

</div>



<h3>
  
  
  Confirm models landed in the right folders
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> <span class="nt">-1</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">/models/checkpoints"</span> | <span class="nb">head
ls</span> <span class="nt">-1</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">/models/loras"</span> | <span class="nb">head
ls</span> <span class="nt">-1</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">/models/vae"</span> | <span class="nb">head</span>
</code></pre>

</div>



<h3>
  
  
  See which custom nodes got installed
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> <span class="nt">-1</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">/custom_nodes"</span> | <span class="nb">sort</span> | <span class="nb">head</span> <span class="nt">-n</span> 20
</code></pre>

</div>



<blockquote>
<p>🧪 Tip: If a node has Python deps, open its README. Some custom nodes require an extra pip install or a build tool. The generator handles common cases, but niche nodes can have surprises.</p>
</blockquote>




<h2>
  
  
  Provider-specific notes
</h2>

<ul>
<li>
<p>Vast.ai common ComfyUI path:</p>

<ul>
<li>  /workspace/ComfyUI</li>
</ul>


</li>

<li>

<p>RunPod common ComfyUI path:</p>

<ul>
<li>  /workspace/runpod-slim/ComfyUI</li>
</ul>


</li>

</ul>

<p>If you’re on a non-template image or your provider changed paths, set COMFYUI_ROOT manually and use the “Full Script” editor to update paths before running.</p>

<blockquote>
<p>⚠️ Warning: Custom nodes appear after ComfyUI restarts. If you installed nodes while the UI was running, restart the service/app.</p>
</blockquote>




<h2>
  
  
  Full script mode: for tinkerers and control freaks
</h2>

<p>Click “Full Script” in the generator to see and edit everything it plans to run. This is great when you want to:</p>

<ul>
<li><p>Pin exact commits for ComfyUI or nodes</p></li>
<li><p>Add extra pip packages</p></li>
<li><p>Change model destination directories</p></li>
<li><p>Integrate with a persistent volume</p></li>
<li><p>Insert health checks or post-install tests</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flp6wys2igdhiwi021g4f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flp6wys2igdhiwi021g4f.png" alt="Browser screenshot of a deployment UI with Vast.ai selected, dark code panel showing bash exports for HF and Civitai tokens, plus deploy buttons for RunPod and Vast.ai" width="800" height="520"></a></p>

<p>Example tweaks you might add:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Pin a specific ComfyUI commit</span>
git <span class="nt">-C</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">"</span> fetch <span class="nt">--all</span>
git <span class="nt">-C</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">"</span> checkout &lt;commit-or-tag&gt;

<span class="c"># Install extra Python deps required by a custom workflow</span>
<span class="nb">source</span> /venv/bin/activate 2&gt;/dev/null <span class="o">||</span> <span class="nb">true
</span>pip <span class="nb">install </span><span class="nv">xformers</span><span class="o">==</span>0.0.23 <span class="nv">safetensors</span><span class="o">==</span>0.4.3

<span class="c"># Verify GPU is visible</span>
nvidia-smi <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"No GPU found (driver/container mismatch?)"</span>
</code></pre>

</div>






<h2>
  
  
  Troubleshooting and “wish I knew this sooner”
</h2>

<ul>
<li>
<p>Hugging Face 403? You probably need a token for that model/repo.<br>
</p>
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">HF_TOKEN</span><span class="o">=</span>hf_xxx
</code></pre>

</li>
<li><p>Slow model downloads: instance egress can be limited. Consider smaller test models first to validate paths.</p></li>
<li><p>Not enough disk: large checkpoints can exceed ephemeral storage. Use a larger volume or a persistent disk.</p></li>
<li><p>Node missing in the menu: restart ComfyUI after node install/update.</p></li>
<li><p>CUDA mismatch errors: ensure your image, driver, and PyTorch stack align. Templates help, bare images can drift.</p></li>
<li><p>Case-sensitive paths: ComfyUI model folders are strict: “checkpoints”, “loras”, “vae”, etc.</p></li>
<li><p>Port blocked? Verify the provider exposes the ComfyUI port (often 8188) and the service is running.</p></li>
</ul>

<blockquote>
<p>🛠️ Debug pattern I use: tail logs while launching the UI to spot import errors.</p>


<pre class="highlight shell"><code>ps aux | <span class="nb">grep</span> <span class="nt">-i</span> comfy
<span class="c"># or check the provider's app logs panel if available</span>
</code></pre>

</blockquote>




<h2>
  
  
  Developer tips
</h2>

<ul>
<li><p>Use presets to create named environments for different workflows (e.g., “ControlNet Editing”, “Tiny SDXL Playground”).</p></li>
<li><p>Pin versions when collaborating so everyone runs the same stack.</p></li>
<li><p>For long downloads, wrap your terminal session in tmux/screen to avoid drops.</p></li>
<li><p>Cache model folders on a persistent volume to avoid re-downloading every session.</p></li>
<li><p>If you’re on Windows/macOS locally, point the generator to your ComfyUI path and generate a matching script for your OS.</p></li>
</ul>

<blockquote>
<p>💡 Pro tip: You can run the generator for local machines too. Set the install path and let it do the folder mapping for you.</p>
</blockquote>




<h2>
  
  
  Quick Reference (TL;DR)
</h2>

<ul>
<li><p>Generator: <a href="https://deploy.promptingpixels.com/" rel="noopener noreferrer">https://deploy.promptingpixels.com/</a></p></li>
<li><p>Workflow:</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Launch GPU instance (RunPod/Vast.ai) and open a terminal
2. In the generator, pick provider + ComfyUI version
3. Add models (HF/Civitai) and custom nodes
4. Copy the one-liner, set tokens if needed, paste into terminal
5. Launch ComfyUI; restart once to load new nodes
</code></pre>

</div>

<ul>
<li>
<p>Helpful env vars:<br>
</p>
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">HF_TOKEN</span><span class="o">=</span>hf_xxx
<span class="nb">export </span><span class="nv">CIVITAI_TOKEN</span><span class="o">=</span>xxx
<span class="nb">export </span><span class="nv">COMFYUI_ROOT</span><span class="o">=</span>/workspace/ComfyUI  <span class="c"># adjust if your layout differs</span>
</code></pre>

</li>
<li>
<p>Verify after install:<br>
</p>
<pre class="highlight shell"><code>git <span class="nt">-C</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">"</span> rev-parse <span class="nt">--short</span> HEAD
<span class="nb">ls</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">/models/checkpoints"</span> | <span class="nb">head
ls</span> <span class="s2">"</span><span class="nv">$COMFYUI_ROOT</span><span class="s2">/custom_nodes"</span> | <span class="nb">head</span>
</code></pre>

</li>
</ul>




<p>If you’ve got feature ideas or run into edge cases, the tool’s maintained and open to feedback: <a href="mailto:deploy@promptingpixels.com">deploy@promptingpixels.com</a></p>

<p>Happy building!</p>

