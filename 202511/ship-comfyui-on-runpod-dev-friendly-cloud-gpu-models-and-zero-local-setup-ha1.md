---
Title: Ship ComfyUI on RunPod (Dev-Friendly): Cloud GPU, models, and zero local setup
Description: 
Author: Prompting Pixels
Date: 2025-11-28T21:30:42.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What we’re building
</h2>

<p>A browser-based ComfyUI workstation running on a rented GPU (<a href="https://www.runpod.io/" rel="noopener noreferrer">RunPod</a>), preloaded with your favorite models and custom nodes, with no local installs. You’ll:</p>

<ul>
<li>Launch a GPU pod with the official ComfyUI template</li>
<li>Install models LoRA/custom nodes the fast way (copy/paste one-liner)</li>
<li>Restart ComfyUI cleanly and verify everything works</li>
<li>Pull images off the pod and keep costs in check</li>
</ul>

<p>If you like command-line control, I added optional CLI and debugging bits along the way.</p>

<h2>
  
  
  TL;DR (Quick Reference)
</h2>

<ul>
<li>Pick a GPU (3090 is great for learning; 5090 is fast if you need speed)</li>
<li>Use the RunPod ComfyUI template (no manual installs)</li>
<li>Enable the Web Terminal and paste a deployment one-liner from Prompting Pixels</li>
<li>Set Hugging Face and Civitai API tokens before running the script</li>
<li>Restart ComfyUI from Manager inside the UI</li>
<li>Download outputs via File Browser (port 8080)</li>
<li>Stop the pod when you’re done to avoid charges</li>
</ul>

<h2>
  
  
  The problem I was trying to solve
</h2>

<p>I wanted ComfyUI with good models and a clean environment without touching my local machine. My laptop can’t compete with a data center GPU, and I’m not babysitting CUDA installs or driver versions. RunPod + an official ComfyUI template + a model installer script is the shortest path I’ve found.</p>

<h2>
  
  
  Plan of attack
</h2>

<p>1) Provision a GPU pod on RunPod<br><br>
2) Select the official ComfyUI template<br><br>
3) Wait for services to come up (ComfyUI on port 8188)<br><br>
4) Use a one-liner to install models/custom nodes<br><br>
5) Restart ComfyUI and generate images<br><br>
6) Download results and pause billing</p>


<h2>
  
  
  1) Provision the GPU pod
</h2>

<p>Sign into RunPod and open Pods. Filter for an affordable but capable GPU.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9f8nr8hxxsejmwbxlp15.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9f8nr8hxxsejmwbxlp15.png" alt="RunPod console 'Deploy a Pod' Select an Instance screen with left sidebar highlighting Pods, VRAM filter slider, featured GPU cards (RTX 5090, A40, H200 SXM) and a large red arrow annotation pointing to the main panel." width="800" height="459"></a></p>

<ul>
<li>Starter pick: RTX 3090 (~$0.30–0.50/hr)</li>
<li>Faster iterations: RTX 5090 (~$0.75–0.90/hr)</li>
</ul>

<blockquote>
<p>💡 Pro tip: Don’t overspend on GPU at the start. You can always spin up a beefier pod later to re-run workflows faster.</p>
</blockquote>
<h3>
  
  
  Attach the right template
</h3>

<p>On the config screen, change the template to the official RunPod ComfyUI image.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxu9ongompjei6h9h2rg6.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxu9ongompjei6h9h2rg6.webp" alt="Runpod deploy UI showing Configure Deployment with pod name 'miserable_peach_manatee', Runpod PyTorch 2.8.0 template, red arrow and text 'Click " width="800" height="459"></a></p>

<p>Search “ComfyUI” and pick the RunPod-owned template (not a random user).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F381hek52ekweu3awz1dk.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F381hek52ekweu3awz1dk.webp" alt="Runpod modal 'Explore Pod Templates' showing a search for 'Comfyui', a grid of pod template cards and a red arrow labeled 'Select this one' pointing to the 'ComfyUI' card (runpod/comfyui:latest)." width="800" height="459"></a></p>

<p>Name your pod and deploy on-demand.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ferqlsik2rm6lxlf0sgbr.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ferqlsik2rm6lxlf0sgbr.webp" alt="RunPod deploy screen with ComfyUI template, GPU count slider, pricing cards (On-Demand $0.46/hr selected), red annotation and arrow pointing to purple 'Deploy On-Demand' button; pod shows 1x RTX 3090, 117GB RAM, 200GB disk." width="800" height="459"></a></p>

<blockquote>
<p>⚠️ Warning: Keep an eye on allocated disk size. Models are big. 100–200 GB leaves room for a couple of XL checkpoints + LoRAs + outputs.</p>
</blockquote>


<h2>
  
  
  2) Wait for services, then open ComfyUI
</h2>

<p>RunPod will boot your pod and wire up HTTP ports. You want ComfyUI on port 8188 to be “Ready” (green).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3z0cst2qwggw5fh6b59.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3z0cst2qwggw5fh6b59.webp" alt="Browser console showing Pod 'miserable_peach_manatee' details with HTTP Services: Port 8080 FileBrowser, Port 8188 ComfyUI (red arrow note), Port 8888 JupyterLab, and SSH key setup box" width="800" height="459"></a></p>

<p>Click “ComfyUI” to open the UI in a new tab.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr2y6bk85z53geltf7q67.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr2y6bk85z53geltf7q67.webp" alt="Dark ComfyUI Templates modal with search/filters, left sidebar of generation types and APIs, and a grid of image and video template cards." width="800" height="459"></a></p>

<blockquote>
<p>💡 Pro tip: JupyterLab (port 8888) and File Browser (port 8080) are also exposed. I use them for quick inspections/edits without SSH.</p>
</blockquote>


<h2>
  
  
  3) Install models and custom nodes (no manual path wrangling)
</h2>

<p>We’ll use a one-liner generator so you can pick models/nodes via checkboxes and paste a script once. Open:</p>

<ul>
<li>deploy.promptingpixels.com</li>
<li>Choose your base models (e.g., JuggernautXL, RealVisXL)</li>
<li>Add LoRAs/styles and useful nodes (UltimateSDUpscale is solid)</li>
<li>Select “RunPod” as the provider</li>
<li>Copy the generated one-liner</li>
</ul>

<p>Now in your pod’s Connect tab, enable the Web Terminal and open it:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8iqbe37qn2f1tj45v663.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8iqbe37qn2f1tj45v663.webp" alt="RunPod console for pod miserable_peach_manatee showing Connect tab with HTTP services (8080 FileBrowser, 8188 ComfyUI, 8888 JupyterLab), SSH key instructions, red text and arrow pointing to purple 'Enable Web Terminal' toggle labeled Running" width="800" height="459"></a></p>

<p>Paste the command you copied. It will look conceptually like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Example shape — use the exact one-liner from the site</span>
bash &lt;<span class="o">(</span>wget <span class="nt">-qO-</span> https://deploy.promptingpixels.com//api/script/cmijdjpcm000knikfe33dqbc2<span class="o">)</span> <span class="nt">--hf-token</span><span class="o">=</span>YOUR_HF_TOKEN
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftq7vs3hodm74bct06cu5.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftq7vs3hodm74bct06cu5.webp" alt="Browser window showing a terminal running a bash wget deploy script URL with redacted --hf-token and --civitai-token values highlighted in magenta" width="800" height="459"></a></p>

<blockquote>
<p>⚠️ Warning: Replace YOUR_HF_TOKEN and YOUR_CIVITAI_TOKEN with real API tokens.  </p>

<ul>
<li>Hugging Face: <a href="https://huggingface.co/settings/tokens" rel="noopener noreferrer">https://huggingface.co/settings/tokens</a>
</li>
<li>Civitai: <a href="https://civitai.com/user/account" rel="noopener noreferrer">https://civitai.com/user/account</a>
</li>
</ul>
</blockquote>

<p>Prefer not to paste tokens in plain text?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">HF_TOKEN</span><span class="o">=</span>hf_XXXXXXXXXXXXXXXXXXXXXXXXXXXX
<span class="nb">export </span><span class="nv">CIVITAI_TOKEN</span><span class="o">=</span>ct_XXXXXXXXXXXXXXXXXXXXXXXX
<span class="c"># Then paste the one-liner and change flags to:</span>
<span class="c"># ... --hf-token "$HF_TOKEN" --civitai-token "$CIVITAI_TOKEN"</span>
</code></pre>

</div>



<p>Go grab a drink; downloads can take a while.</p>




<h2>
  
  
  4) Restart ComfyUI cleanly
</h2>

<p>After the installer finishes, switch to your ComfyUI tab and use the Manager to reload nodes and models.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzmpi5kd0vjhxafqbo5js.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzmpi5kd0vjhxafqbo5js.webp" alt="ComfyUI Manager v3.37.1 modal; red arrow and 'Press Restart' label point at red Restart button to load custom nodes; right panel shows version links." width="800" height="459"></a></p>

<blockquote>
<p>💡 Pro tip: If you see “Bad Gateway” once, wait ~30s and refresh. The service is coming back up.</p>
</blockquote>

<p>At this point, models and nodes should show up in dropdowns and node pickers.</p>




<h2>
  
  
  5) Generate and export results
</h2>

<p>Create a simple workflow, run it, and confirm outputs appear in ComfyUI’s output directory.</p>

<p>To batch-download images, use File Browser (port 8080).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuj5h1es6lkjjubzeoz3v.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuj5h1es6lkjjubzeoz3v.webp" alt="RunPod console showing pod 'miserable_peach_manatee' Connect view listing FileBrowser 8080, ComfyUI, JupyterLab 8888, SSH key box and red arrows." width="800" height="459"></a></p>

<p>Default credentials for File Browser:</p>

<ul>
<li>Username: admin</li>
<li>Password: adminadmin12</li>
</ul>

<p>Navigate: runpod-slim → ComfyUI → output, then right-click to download or multi-select.</p>

<blockquote>
<p>💡 Pro tip: For automation, you can also zip the output folder and grab a single archive. Or sync to an object store if you want to get fancy.</p>
</blockquote>




<h2>
  
  
  Debugging and ops cheat sheet
</h2>

<p>When things go sideways, here’s what I actually run:</p>

<ul>
<li>GPU sanity:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  nvidia-smi
</code></pre>

</div>



<ul>
<li>Disk space:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  <span class="nb">df</span> <span class="nt">-h</span>
  <span class="nb">du</span> <span class="nt">-h</span> <span class="nt">--max-depth</span><span class="o">=</span>1 runpod-slim/ComfyUI/models | <span class="nb">sort</span> <span class="nt">-h</span>
</code></pre>

</div>



<ul>
<li>Memory headroom:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  free <span class="nt">-h</span>
</code></pre>

</div>



<ul>
<li>Permissions jank (rare, but happens with new files/folders):
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  <span class="nb">chmod</span> <span class="nt">-R</span> u+rwX,g+rwX runpod-slim/ComfyUI
</code></pre>

</div>



<ul>
<li>Quick reset if Manager restart isn’t enough:

<ul>
<li>Stop and start the pod from the RunPod dashboard</li>
</ul>


</li>

<li>Verify ports:

<ul>
<li>ComfyUI on 8188, File Browser on 8080, JupyterLab on 8888</li>
</ul>


</li>

</ul>

<blockquote>
<p>🧪 Tip for VRAM errors: Drop resolution, batch size, or disable high-memory nodes first. On 24GB cards, SDXL at 1024px usually needs conservative settings.</p>
</blockquote>




<h2>
  
  
  Gotchas (aka “wish I knew this earlier”)
</h2>

<ul>
<li>Model sizes add up fast. Plan 10–30 GB per XL checkpoint, plus LoRAs and VAE.</li>
<li>The first template boot can take a few minutes—don’t panic-refresh too early.</li>
<li>A “Ready” indicator for port 8188 is the source of truth. If it isn’t green, ComfyUI isn’t up yet.</li>
<li>After adding nodes, always restart via Manager before assuming “it didn’t install.”</li>
<li>Tokens matter: a bad Hugging Face or Civitai token silently causes partial downloads.</li>
<li>Billing is minute-by-minute. A forgotten running pod is an involuntary donation.</li>
</ul>




<h2>
  
  
  Keep costs under control
</h2>

<p>When you’re done, pause the pod to stop charges while preserving your data. Termination deletes everything—use it when you’re really done.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa2dr5e0ln8ll1fnbvm4o.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa2dr5e0ln8ll1fnbvm4o.webp" alt="Runpod Pods page showing pod 'miserable_peach_manatee', resource gauges, a dropdown menu with 'Stop Pod' highlighted by a red arrow and note to stop before terminating." width="800" height="459"></a></p>

<ul>
<li>Stop = pause billing, keep storage</li>
<li>Terminate = destroy pod and data</li>
</ul>

<blockquote>
<p>💡 Pro tip: Name pods clearly (e.g., comfyui-sdxl-playground) so you don’t forget what’s safe to shut down.</p>
</blockquote>




<h2>
  
  
  Extras for power users
</h2>

<ul>
<li>JupyterLab (8888) for quick scripts and file edits</li>
<li>SSH keys for terminal lovers (optional, Web Terminal works fine)</li>
<li>Volume planning: if you’re a model hoarder, allocate &gt;200 GB up front</li>
<li>CI-like behavior: keep a lightweight pod for experimenting, spin a faster one for final renders</li>
</ul>




<h2>
  
  
  Quick Reference
</h2>

<ul>
<li>GPU pick

<ul>
<li>3090 = economical learning</li>
<li>5090 = fast iteration</li>
</ul>


</li>

<li>Services

<ul>
<li>ComfyUI: port 8188</li>
<li>File Browser: port 8080 (admin/adminadmin12)</li>
<li>JupyterLab: port 8888</li>
</ul>


</li>

<li>Tokens

<ul>
<li>Hugging Face: <a href="https://huggingface.co/settings/tokens" rel="noopener noreferrer">https://huggingface.co/settings/tokens</a>
</li>
<li>Civitai: <a href="https://civitai.com/user/account" rel="noopener noreferrer">https://civitai.com/user/account</a>
</li>
</ul>


</li>

<li>Installer

<ul>
<li>Use deploy.promptingpixels.com, provider “RunPod”, copy the one-liner</li>
</ul>


</li>

<li>Common commands
</li>

</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  nvidia-smi
  <span class="nb">df</span> <span class="nt">-h</span>
  free <span class="nt">-h</span>
</code></pre>

</div>



<ul>
<li>Cost control

<ul>
<li>Stop pod when idle</li>
<li>Terminate only when you want a clean slate</li>
</ul>


</li>

</ul>




<h2>
  
  
  What’s next
</h2>

<ul>
<li>Add ControlNet and pose/edge preprocessors for surgical control</li>
<li>Wire up an S3 bucket to sync outputs automatically</li>
<li>Build a tiny HTTP service that triggers ComfyUI workflows (webhooks + queue)</li>
<li>Try LoRA training pods to create your own styles</li>
<li>Spin a second pod with a bigger GPU just for render days</li>
</ul>

<p>If you ship anything cool with this setup, share your workflow JSON and model picks—I love seeing what people make with a clean ComfyUI rig.</p>

