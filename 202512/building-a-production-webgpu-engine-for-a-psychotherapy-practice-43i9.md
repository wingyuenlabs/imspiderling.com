---
Title: Building a Production WebGPU Engine... for a psychotherapy practice?
Description: 
Author: x0101010011
Date: 2025-12-23T21:36:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Challenge: Where Noise Becomes Flow
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi5pscs3lx1tokb874zag.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi5pscs3lx1tokb874zag.jpeg" alt="Flow field as psyche metaphor" width="800" height="525"></a></p>

<p>When building the digital presence for <strong>Therapy Warsaw</strong>, we faced an unusual requirement. We didn't want stock photos or static illustrations. We wanted something that felt <em>alive</em>—a generative texture that was always changing, but never demanding attention.</p>

<p>The visual metaphor was simple: complex patterns finding clarity. A field of noise, slowly organizing itself into coherent, flowing lines.</p>

<p>The technical requirements were less simple:</p>

<ol>
<li> <strong>Organic &amp; Dense:</strong> ~10,000 interacting particles.</li>
<li> <strong>Performance Critical:</strong> 60FPS on mobile while users scroll.</li>
<li> <strong>Resilient:</strong> Must work on 10-year-old laptops (WebGL2) <em>and</em> bleeding-edge devices (WebGPU).</li>
<li> <strong>Framework-Free:</strong> No React, no Three.js. Just controlled, fluid logic.</li>
</ol>

<p>Here is how we built a dual-stack <strong>WebGPU + WebGL2</strong> engine to solve this.</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63oqxksctvg5y7kvbaph.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63oqxksctvg5y7kvbaph.jpeg" alt="Physics transitions" width="800" height="525"></a></p>

<h2>
  
  
  Architecture: Keeping the UI Responsive
</h2>

<p>The first rule of heavy graphics on the web: <strong>Get off the main thread.</strong></p>

<p>We strictly separated the application:</p>

<ul>
<li>  <strong>Main Thread:</strong> DOM, accessibility, routing, UI state.</li>
<li>  <strong>Worker Thread:</strong> Physics, geometry generation, rendering via <code>OffscreenCanvas</code>.</li>
</ul>

<p>Even if the physics simulation hiccups, page scrolling stays smooth. Communication happens via a dedicated messaging system that syncs visual "Presets" (colors, speed, turbulence) without blocking.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// main.js</span>
<span class="kd">const</span> <span class="nx">worker</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Worker</span><span class="p">(</span><span class="k">new</span> <span class="nc">URL</span><span class="p">(</span><span class="dl">'</span><span class="s1">./worker.js</span><span class="dl">'</span><span class="p">,</span> <span class="k">import</span><span class="p">.</span><span class="nx">meta</span><span class="p">.</span><span class="nx">url</span><span class="p">),</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">module</span><span class="dl">'</span> <span class="p">});</span>
<span class="kd">const</span> <span class="nx">offscreen</span> <span class="o">=</span> <span class="nx">canvas</span><span class="p">.</span><span class="nf">transferControlToOffscreen</span><span class="p">();</span>

<span class="c1">// Hand ownership to the worker</span>
<span class="nx">worker</span><span class="p">.</span><span class="nf">postMessage</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">init</span><span class="dl">'</span><span class="p">,</span> <span class="na">canvas</span><span class="p">:</span> <span class="nx">offscreen</span> <span class="p">},</span> <span class="p">[</span><span class="nx">offscreen</span><span class="p">]);</span>
</code></pre>

</div>






<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdka9baaalwq2kk9vb1i4.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdka9baaalwq2kk9vb1i4.jpeg" alt="#genart" width="800" height="525"></a></p>

<h2>
  
  
  Why WebGPU? (And Why We Still Needed WebGL2)
</h2>

<p>We started with <strong>WebGPU</strong> because Compute Shaders are a natural fit for particle systems.</p>

<h3>
  
  
  The WebGPU Pipeline
</h3>

<p>We use <strong>Compute Shaders</strong> for the heavy lifting:</p>

<ol>
<li> <strong>Map Pass:</strong> Generates noise textures (Burn, Density, Void maps).</li>
<li> <strong>Flow Pass:</strong> Calculates the vector field.</li>
<li> <strong>Life Pass:</strong> Updates particle ages and handles resets.</li>
<li> <strong>Physics Pass:</strong> Moves particles based on flow vectors.</li>
</ol>

<p>The key performance win: avoiding CPU-GPU round trips. The entire simulation stays on the GPU.</p>

<h3>
  
  
  The WebGL2 Fallback
</h3>

<p>WebGPU support is growing but not universal. We <em>had</em> to support WebGL2—but we didn't want a "dumb" fallback.</p>

<p>To achieve feature parity without destroying the CPU, we used <strong>Transform Feedback</strong>. This allows WebGL2 to update particle positions in the Vertex Shader and write them back to a buffer, mimicking compute shaders.</p>




<h2>
  
  
  The Spring Physics System
</h2>

<p>When a user navigates between pages, the visualization morphs:</p>

<ul>
<li>  Colors shift (e.g., warm orange → deep blue).</li>
<li>  Chaos decreases or increases.</li>
<li>  Speed adjusts.</li>
</ul>

<p>We couldn't just <code>lerp</code> these values; it looks robotic. We implemented a <strong>Critical Damping Spring System</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">updateSpring</span><span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">target</span><span class="p">,</span> <span class="nx">dt</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">tension</span> <span class="o">=</span> <span class="mi">120</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">friction</span> <span class="o">=</span> <span class="mi">20</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">displacement</span> <span class="o">=</span> <span class="nx">target</span> <span class="o">-</span> <span class="nx">state</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">force</span> <span class="o">=</span> <span class="nx">tension</span> <span class="o">*</span> <span class="nx">displacement</span> <span class="o">-</span> <span class="nx">friction</span> <span class="o">*</span> <span class="nx">state</span><span class="p">.</span><span class="nx">velocity</span><span class="p">;</span>

    <span class="nx">state</span><span class="p">.</span><span class="nx">velocity</span> <span class="o">+=</span> <span class="nx">force</span> <span class="o">*</span> <span class="nx">dt</span><span class="p">;</span>
    <span class="nx">state</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">state</span><span class="p">.</span><span class="nx">velocity</span> <span class="o">*</span> <span class="nx">dt</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Every frame, we update ~20 spring-driven parameters and upload them to a <strong>Uniform Buffer Object (UBO)</strong>. The result: transitions that feel physical, not computed.</p>




<h2>
  
  
  Optimization: Procedural Vertex Integration
</h2>

<p>Rendering thick lines usually means generating 2 triangles (6 vertices) per segment. For long trails, that's expensive memory bandwidth.</p>

<p>Our approach: store <strong>only the head position</strong> of each line.</p>

<p>Inside the Vertex Shader, we run a <code>for</code> loop (~60 iterations) to re-trace the path <em>backwards</em> through the flow field, reconstructing the trail on the fly.</p>

<ul>
<li>  <strong>Pros:</strong> Massive bandwidth reduction (1 point per line, not thousands of vertices).</li>
<li>  <strong>Cons:</strong> Higher ALU cost per vertex.</li>
</ul>

<p>On modern GPUs, ALU is cheap; bandwidth is expensive. This trade-off let us render thousands of long, smooth trails on mobile.</p>




<h2>
  
  
  The Result
</h2>

<p>The result is <a href="https://therapywarsaw.com" rel="noopener noreferrer">therapywarsaw.com</a>—a site where the background is a living simulation, a quiet texture that reflects the nature of the work.</p>

<p>The engine is open source:</p>

<p><strong>Repo:</strong> <a href="https://github.com/23x2/generative-flow-field" rel="noopener noreferrer">github.com/23x2/generative-flow-field</a></p>




<p><em>Questions about the shader pipeline or Transform Feedback? Ask below.</em></p>

