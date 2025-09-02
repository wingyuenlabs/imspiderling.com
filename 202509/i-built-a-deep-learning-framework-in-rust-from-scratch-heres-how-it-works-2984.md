---
Title: I Built a Deep Learning Framework in Rust from Scratch. Here’s How It Works.
Description: 
Author: Pavel
Date: 2025-09-02T21:37:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've just published <code>RustyASG</code> to <a href="https://crates.io/crates/rustyasg" rel="noopener noreferrer">crates.io</a>, a deep learning framework I built from the ground up in Rust. The project's goal was to explore a graph-based architecture, moving away from the eager execution model common in frameworks like PyTorch.</p>

<p>This post is a technical breakdown of its architecture, core design, and the most critical bug I had to fix to make it work correctly.</p>

<h3>
  
  
  The Core Idea: Define-then-Run
</h3>

<p>RustyASG does not compute operations immediately. Instead, every operation constructs a node in an <strong>Abstract Semantic Graph (ASG)</strong>. This graph serves as a complete blueprint of the entire computation before any numbers are crunched.</p>

<p>Once the graph is fully defined, it can be:</p>

<ol>
<li> <strong>Analyzed:</strong> Statically infer tensor shapes to catch dimension mismatches before execution.</li>
<li> <strong>Differentiated:</strong> Automatically generate a new graph that calculates the gradients of any node.</li>
<li> <strong>Executed:</strong> Run the optimized graph on a backend, such as a CPU or GPU.</li>
</ol>

<p>This approach provides a high degree of control and enables global optimizations like kernel fusion and static memory planning.</p>

<h3>
  
  
  Architecture Overview
</h3>

<p>The framework's components are strictly separated. This flowchart shows how the pieces fit together:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdzeus8ypu8ru341z565p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdzeus8ypu8ru341z565p.png" alt=" " width="800" height="1200"></a></p>

<ul>
<li>  <strong><code>tensor</code>:</strong> A lightweight, symbolic handle to a node in the graph. All operations on it modify the graph.</li>
<li>  <strong><code>asg</code>:</strong> The core data structures that define the graph (<code>Node</code>, <code>NodeType</code>). This layer only <em>describes</em> computation.</li>
<li>  <strong><code>autograd</code>:</strong> The reverse-mode automatic differentiation engine. It transforms a forward-pass graph into a new graph that computes gradients.</li>
<li>  <strong><code>runtime</code>:</strong> Contains the <code>Backend</code> trait and its concrete implementations for CPU (<code>ndarray</code>) and GPU (<code>wgpu</code>).</li>
</ul>

<h3>
  
  
  Getting Started: Training a Transformer Block
</h3>

<p>An example is included to demonstrate a full training loop on a Transformer block.</p>

<ol>
<li>
<p><strong>Clone the repository:</strong><br>
</p>
<pre class="highlight shell"><code>git clone https://github.com/Xzdes/RustyAsg.git
<span class="nb">cd </span>RustyAsg
</code></pre>

</li>
<li>
<p><strong>Run the demo (GPU is the default backend):</strong><br>
</p>
<pre class="highlight shell"><code>cargo run <span class="nt">--example</span> transformer_demo <span class="nt">--release</span>
</code></pre>


<p>To use the CPU, set the <code>use_gpu</code> flag to <code>false</code> in <code>examples/transformer_demo.rs</code>.</p>
</li>
</ol>

<p>The output shows the training loss decreasing, which confirms that the entire stack—from graph construction to backpropagation on the GPU—is functioning correctly.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nt">---</span> TRAINING LOOP START <span class="nt">---</span>
Epoch: 1 , Loss: 3.081630
Epoch: 2 , Loss: 2.840065
...
Epoch: 15, Loss: 0.982813
<span class="nt">---</span> TRAINING FINISHED IN 1.34s <span class="nt">---</span>
</code></pre>

</div>



<h3>
  
  
  The Hardest Bug: A Flaw in the Gradient Logic
</h3>

<p>The framework appeared stable until I implemented a gradient checker. The tests compared the analytical gradients produced by the <code>autograd</code> engine against numerical estimates. A key test, which emulated a <code>LayerNorm</code> operation, was consistently failing with an error margin of nearly 60%.</p>

<p>The root cause was a subtle but critical flaw in the backpropagation logic for division, specifically when <strong>broadcasting</strong> was involved.</p>

<p>When a vector like <code>[10, 20]</code> is divided by a scalar <code>2</code>, the result is <code>[5, 10]</code>. During backpropagation, the gradient for the scalar <code>2</code> receives contributions from <em>both</em> elements of the output. Therefore, the incoming gradients must be <strong>summed</strong> to produce the final, correct gradient for the original scalar.</p>

<p>My autograd implementation for <code>Add</code> and <code>Subtract</code> handled this, but I had overlooked it for <code>Divide</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="c1">// The incorrect autograd logic for the divisor 'b'</span>
<span class="c1">// grad_b = ... // Calculation was mathematically correct but missed a step.</span>
<span class="c1">// self.accumulate_grad(b_id, grad_b)?;</span>

<span class="c1">// The corrected logic</span>
<span class="k">let</span> <span class="n">b_shape</span> <span class="o">=</span> <span class="k">self</span><span class="py">.source_asg</span><span class="nf">.get_node</span><span class="p">(</span><span class="n">b_id</span><span class="p">)</span><span class="o">?</span><span class="py">.shape</span><span class="nf">.as_ref</span><span class="p">()</span><span class="nf">.unwrap</span><span class="p">()</span><span class="nf">.clone</span><span class="p">();</span>
<span class="c1">// ... calculate grad_b ...</span>
<span class="k">if</span> <span class="n">b_shape</span> <span class="o">!=</span> <span class="n">grad_shape</span> <span class="p">{</span>
    <span class="c1">// If 'b' was broadcast, its gradient must be summed to match its original shape.</span>
    <span class="n">grad_b</span> <span class="o">=</span> <span class="k">self</span><span class="py">.grad_asg</span><span class="nf">.add_node</span><span class="p">(</span><span class="nb">None</span><span class="p">,</span> <span class="nn">NodeType</span><span class="p">::</span><span class="nf">Sum</span><span class="p">(</span><span class="n">grad_b</span><span class="p">));</span>
<span class="p">}</span>
<span class="k">self</span><span class="nf">.accumulate_grad</span><span class="p">(</span><span class="n">b_id</span><span class="p">,</span> <span class="n">grad_b</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>
</code></pre>

</div>



<p>This conditional sum was the missing piece. Adding it fixed the <code>LayerNorm</code> test and validated the entire <code>autograd</code> engine.</p>

<h3>
  
  
  What's Next
</h3>

<p>The foundation is now stable. The immediate roadmap is focused on expanding core functionality:</p>

<ul>
<li>  Implement gradients for <code>MatrixMultiply</code> and <code>Softmax</code>.</li>
<li>  Add more optimizers, starting with <code>AdamW</code>.</li>
<li>  Develop a memory-recycling buffer allocator for the <code>wgpu</code> backend to improve performance.</li>
<li>  Implement model serialization to save and load trained weights.</li>
</ul>

<p>Contributions are welcome. Please feel free to open Issues or Pull Requests.</p>

<p>You can find the project at:</p>

<ul>
<li>  <strong>GitHub:</strong> <a href="https://github.com/Xzdes/RustyAsg" rel="noopener noreferrer">https://github.com/Xzdes/RustyAsg</a>
</li>
<li>  <strong>Crates.io:</strong> <a href="https://crates.io/crates/rustyasg" rel="noopener noreferrer">https://crates.io/crates/rustyasg</a>
</li>
</ul>

