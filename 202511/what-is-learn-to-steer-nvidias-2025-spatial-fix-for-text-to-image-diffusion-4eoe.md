---
Title: What Is Learn-to-Steer? NVIDIA’s 2025 Spatial Fix for Text-to-Image Diffusion
Description: 
Author: David Evans
Date: 2025-11-19T21:56:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>Text-to-image diffusion models have become the workhorses of generative imaging. They can paint photorealistic scenes, mimic art styles, and blend concepts in ways that were science fiction a few years ago. Yet they stumble embarrassingly on a skill that even small children master: basic spatial reasoning.</p>

<p>Ask a state-of-the-art model for “a dog to the right of a teddy bear” and you often get:</p>

<ul>
<li>The dog on the left</li>
<li>One of the objects missing</li>
<li>Or a bizarre hybrid where dog and teddy are fused into a single creature</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F49rtb08366xdl284o4z0.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F49rtb08366xdl284o4z0.jpg" alt=" " width="800" height="532"></a></p>

<p>These failures become more severe for unusual compositions like “a giraffe above an airplane”. Traditional fixes range from expensive fine-tuning to brittle, hand-written loss functions at inference time—but both options come with significant downsides.</p>

<p>NVIDIA’s Learn-to-Steer framework (accepted to WACV 2026) proposes a different path: instead of hard-coding spatial rules or retraining the entire model, it learns a data-driven objective that can “steer” diffusion at inference time. The method reads the model’s own cross-attention maps, trains a lightweight classifier to detect spatial relations, and then uses that classifier’s gradient as a learned loss to nudge the generation towards layouts that match the prompt.</p>

<p>In this blog, we’ll unpack:</p>

<ul>
<li>What makes spatial reasoning so fragile in current diffusion models</li>
<li>How Learn-to-Steer learns spatial constraints from the model itself</li>
<li>How it steers images during generation without changing model weights</li>
<li>The top gains on spatial benchmarks like GenEval and T2I-CompBench</li>
<li>The trade-offs in compute cost and generality, and what this implies for future generative systems</li>
</ul>

<h1>
  
  
  Why Spatial Reasoning Fails in Text-to-Image Diffusion
</h1>

<h2>
  
  
  What Makes Spatial Relations So Difficult for Diffusion Models?
</h2>

<p>Modern diffusion models (e.g., Stable Diffusion, Flux) are excellent at what should appear in an image—objects, styles, textures—but much less reliable at where those objects should be.</p>

<p>Several factors contribute:</p>

<h3>
  
  
  Weak supervision of spatial language
</h3>

<ul>
<li>Training data rarely comes with precise annotations like “object A is left of object B”.
</li>
<li>Captions often describe content loosely, so phrases like “on top of” or “to the right of” are under-specified.</li>
</ul>

<h3>
  
  
  Entangled visual concepts
</h3>

<ul>
<li>When two objects frequently co-occur, models may treat them as a single visual blob.</li>
<li>This leads to object fusion, where a “cat on a bookshelf” becomes a cat-bookshelf chimera.</li>
</ul>

<h3>
  
  
  Benchmark saturation without spatial coverage
</h3>

<ul>
<li>Many standard text-to-image benchmarks emphasize realism and style, not relational accuracy.</li>
<li>Models can score highly while still being spatially confused.</li>
</ul>

<p>Empirical studies confirm three recurring failure modes on spatial benchmarks:</p>

<ul>
<li>Incorrect placement: Objects appear in the wrong relative position.</li>
<li>Missing entities: One or more requested objects never appear.</li>
<li>Merged entities: Two objects get mashed into a single, incoherent form.</li>
</ul>

<p>The model “knows” the objects you asked for, but it doesn’t reliably understand where to place them.</p>

<h1>
  
  
  Why Fine-Tuning and Handcrafted Losses Are Not Enough
</h1>

<p>Two broad strategies have tried to patch this gap:</p>

<h2>
  
  
  Fine-tuning for spatial awareness
</h2>

<ul>
<li>Retrain the diffusion model on datasets with explicit layouts or spatial annotations.</li>
<li>Methods like COMPASS show that this can significantly improve spatial accuracy.</li>
<li>But this comes at a cost: expensive retraining, sensitivity to dataset bias, and often regressions in other capabilities such as color fidelity or counting.</li>
</ul>

<h2>
  
  
  Handcrafted test-time losses
</h2>

<ul>
<li>At inference, inject extra loss terms that penalize spatial errors (e.g., overlapping activation maps, incorrect ordering).</li>
<li>These losses must be manually designed to approximate relations like “left of” or “above”.</li>
<li>In practice, these heuristics are fragile, often over-fitting simple cases and failing on more complex layouts.</li>
</ul>

<p>In short, we’ve lacked a solution that is:</p>

<ul>
<li>Data-driven rather than rule-based</li>
<li>Plug-and-play at inference time (no full retraining)</li>
<li>Targeted enough to improve spatial reasoning without damaging other strengths</li>
</ul>

<p>This is where Learn-to-Steer enters.</p>

<h1>
  
  
  How Learn-to-Steer Works: Data-Driven Steering at Inference
</h1>

<h2>
  
  
  How Cross-Attention Maps Provide a Spatial Signal
</h2>

<p>During diffusion, at each denoising step, the model computes cross-attention maps that connect text tokens to image regions. For a prompt like “a dog to the right of a teddy bear”, you can think of:</p>

<ul>
<li>One set of attention maps for “dog”</li>
<li>Another set for “teddy bear”</li>
<li>Additional context around words like “right” or “of”</li>
</ul>

<p>These maps form a rich, high-dimensional signal describing where in the image the model currently believes each word should manifest. Prior work has used cross-attention to locate objects or edit images; Learn-to-Steer goes further by treating them as a feature space in which spatial relations can be learned.</p>

<h2>
  
  
  How a Relation Classifier Becomes a Learned Loss
</h2>

<p>The core idea of Learn-to-Steer is to train a small relation classifier that takes cross-attention maps for two objects and predicts the spatial relation between them (left-of, right-of, above, below, etc.).</p>

<p>The pipeline looks like this:</p>

<h3>
  
  
  Collect supervision
</h3>

<ul>
<li>Use images where the true relation between object A and object B is known (from datasets like GQA and synthetic layouts).</li>
<li>For each image, invert it through the diffusion model with a descriptive prompt to recover cross-attention maps for the relevant tokens.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi9dbjsdc4c8yjz2r88k4.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi9dbjsdc4c8yjz2r88k4.jpg" alt=" " width="800" height="446"></a></p>

<h3>
  
  
  Train a classifier on attention patterns
</h3>

<ul>
<li>Input: attention maps for object A and object B.</li>
<li>Output: predicted relation (e.g., “A is left of B”).</li>
</ul>

<p>Naively, however, this leads to a subtle but serious issue: relation leakage.</p>

<h2>
  
  
  How Dual Inversion Solves the “Relation Leakage” Problem
</h2>

<p>If you always invert images with a correct prompt (e.g., “a dog to the left of a cat”), hints about the word “left” can leak into the attention patterns. A naïve classifier might then “cheat” by reading out linguistic artefacts instead of learning genuine visual geometry.</p>

<p>To prevent this, Learn-to-Steer uses a dual inversion strategy:</p>

<ul>
<li>For each image with a true relation (say, dog left of cat), create two prompts:

<ul>
<li>A positive prompt with the correct relation (“dog to the left of a cat”).</li>
<li>A negative prompt with an incorrect relation (“dog above a cat”).</li>
</ul>


</li>

<li>Run inversion with both prompts, obtaining two sets of attention maps.</li>

<li>Label both sets with the true relation (left-of), because that is what the image actually depicts.</li>

</ul>

<p>The classifier sees pairs of attention maps that share the same underlying geometry but differ in the relation words used in the prompt. To succeed, it must ignore the unreliable linguistic cue and zero in on the geometric evidence in the attention patterns. This breaks the leakage shortcut and yields a classifier that actually understands “left-of” in terms of where things appear in the model’s internal vision.</p>

<p>To improve robustness, NVIDIA combines:</p>

<ul>
<li>Real images (complex, natural scenes)</li>
<li>Synthetic images (simpler, cleaner attention patterns akin to generation scenarios)</li>
</ul>

<h1>
  
  
  How Learn-to-Steer Guides Images During Generation
</h1>

<h2>
  
  
  Step-by-Step: From Prompt to Steered Latent
</h2>

<p>Once the relation classifier is trained, Learn-to-Steer uses it at inference time as a learned objective:</p>

<h3>
  
  
  Parse the spatial prompt
</h3>

<ul>
<li>Extract subject, relation, and object from the text (e.g., subject = dog, relation = right-of, object = teddy bear).</li>
</ul>

<h3>
  
  
  Run diffusion as usual—but with checkpoints
</h3>

<ul>
<li>As the model denoises latent noise into an image, periodically extract cross-attention maps for the subject and object tokens.</li>
</ul>

<h3>
  
  
  Evaluate spatial correctness
</h3>

<ul>
<li>Feed these maps into the relation classifier, which outputs a probability distribution over relations.</li>
<li>Compare this distribution to the desired relation from the prompt, and compute a loss (e.g., cross-entropy).</li>
</ul>

<h3>
  
  
  Backpropagate into the latent
</h3>

<ul>
<li>Compute the gradient of this loss with respect to the latent representation at that timestep.</li>
<li>Nudge the latent in the direction that increases the classifier’s confidence in the correct relation.</li>
</ul>

<h3>
  
  
  Continue the diffusion process
</h3>

<ul>
<li>Let the denoising proceed from the adjusted latent.</li>
<li>Repeat this steering a number of times (often during the earlier half of the diffusion steps).</li>
</ul>

<h2>
  
  
  Support for Multiple Architectures and Relations
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F578a9bjc7gmtemh0jbsj.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F578a9bjc7gmtemh0jbsj.jpg" alt=" " width="800" height="477"></a></p>

<p>A key advantage of Learn-to-Steer is that it’s architecture-agnostic:</p>

<ul>
<li>It has been demonstrated on both UNet-based models (like Stable Diffusion 1.4/2.1) and MMDiT-style models (like Flux).</li>
<li>The only requirement is access to a text-image alignment signal (cross-attention or similar).</li>
</ul>

<p>It can also handle prompts with multiple constraints, such as:</p>

<p>“A frog above a sneaker below a teapot.”</p>

<p>Here, Learn-to-Steer alternates attention between relations:</p>

<ul>
<li>At one timestep, optimize the frog–sneaker relation.</li>
<li>At another, optimize the sneaker–teapot relation.</li>
</ul>

