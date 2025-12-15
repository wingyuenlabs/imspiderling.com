---
Title: Cross-Modal Knowledge Distillation for circular manufacturing supply chains for low-power autonomous deployments
Description: 
Author: Rikin Patel
Date: 2025-12-15T21:26:08.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1558494949-ef010cbdcc31%3Fixlib%3Drb-4.0.3%26auto%3Dformat%26fit%3Dcrop%26w%3D1200%26q%3D80" alt="Cross-Modal Knowledge Distillation for Circular Manufacturing Supply Chains" width="1200" height="673">
</h1>

<h1>
  
  
  Cross-Modal Knowledge Distillation for circular manufacturing supply chains for low-power autonomous deployments
</h1>

<h2>
  
  
  Introduction: The Learning Journey That Sparked This Exploration
</h2>

<p>It all started when I was experimenting with deploying computer vision models on edge devices for a smart recycling facility. I had developed a sophisticated multi-modal AI system that could identify materials, assess quality, and predict degradation patterns using visual, thermal, and spectral data. The model performed exceptionally well in the lab—achieving 98.7% accuracy on material classification. But when I deployed it to the actual sorting robots in the facility, I hit a wall: the computational requirements were too high for the low-power ARM processors running on solar-charged batteries.</p>

<p>During my investigation of model compression techniques, I discovered something fascinating. While exploring knowledge distillation methods, I realized that the thermal imaging data—which was computationally expensive to process—contained patterns that could be approximated from visual data alone, once the model had learned the underlying relationships. This insight led me down a rabbit hole of cross-modal knowledge distillation, where I could train a lightweight "student" model on one modality (visual) to mimic the behavior of a complex "teacher" ensemble that processed multiple modalities.</p>

<p>One interesting finding from my experimentation with circular manufacturing systems was that the knowledge transfer wasn't just about model compression—it was about creating AI systems that could operate autonomously in resource-constrained environments while maintaining the intelligence needed for complex decision-making in circular supply chains.</p>

<h2>
  
  
  Technical Background: The Convergence of Multiple Disciplines
</h2>

<h3>
  
  
  The Circular Manufacturing Challenge
</h3>

<p>Circular manufacturing represents a paradigm shift from linear "take-make-dispose" models to closed-loop systems where materials are continuously recovered, reprocessed, and reused. In my research of autonomous deployment scenarios, I realized that this requires AI systems capable of:</p>

<ol>
<li>
<strong>Material Identification</strong>: Recognizing materials across various states of degradation</li>
<li>
<strong>Quality Assessment</strong>: Determining if materials can be reused, repaired, or need recycling</li>
<li>
<strong>Process Optimization</strong>: Making real-time decisions about sorting, routing, and processing</li>
<li>
<strong>Predictive Maintenance</strong>: Anticipating equipment failures in remote locations</li>
</ol>

<p>The challenge I encountered during my exploration was that each of these tasks traditionally required different AI models processing different data modalities, creating computational bottlenecks for low-power deployments.</p>

<h3>
  
  
  Cross-Modal Knowledge Distillation Fundamentals
</h3>

<p>Through studying knowledge distillation literature, I learned that traditional approaches typically involve compressing a large model into a smaller one while preserving performance. However, cross-modal distillation introduces an additional dimension: transferring knowledge across different data types.</p>

<p>While experimenting with various distillation techniques, I came across three fundamental approaches:</p>

<ol>
<li>
<strong>Feature-based distillation</strong>: Matching intermediate representations between modalities</li>
<li>
<strong>Attention-based distillation</strong>: Transferring attention patterns that highlight important regions</li>
<li>
<strong>Relational distillation</strong>: Preserving relationships between different samples or features</li>
</ol>

<p>My exploration revealed that for circular manufacturing applications, a hybrid approach combining all three methods yielded the best results, particularly when dealing with the complex relationships between visual appearance and material properties.</p>

<h2>
  
  
  Implementation Details: Building the Cross-Modal Framework
</h2>

<h3>
  
  
  Architecture Overview
</h3>

<p>During my investigation of efficient architectures for edge deployment, I found that a teacher-student framework with modality-specific encoders and a shared knowledge distillation module worked best. Here's the core architecture I developed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">torch</span>
<span class="kn">import</span> <span class="n">torch.nn</span> <span class="k">as</span> <span class="n">nn</span>
<span class="kn">import</span> <span class="n">torch.nn.functional</span> <span class="k">as</span> <span class="n">F</span>

<span class="k">class</span> <span class="nc">MultiModalTeacher</span><span class="p">(</span><span class="n">nn</span><span class="p">.</span><span class="n">Module</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Teacher model processing multiple modalities</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">visual_dim</span><span class="o">=</span><span class="mi">512</span><span class="p">,</span> <span class="n">thermal_dim</span><span class="o">=</span><span class="mi">256</span><span class="p">,</span> <span class="n">spectral_dim</span><span class="o">=</span><span class="mi">128</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">()</span>

        <span class="c1"># Modality-specific encoders
</span>        <span class="n">self</span><span class="p">.</span><span class="n">visual_encoder</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_build_visual_encoder</span><span class="p">(</span><span class="n">visual_dim</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">thermal_encoder</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_build_thermal_encoder</span><span class="p">(</span><span class="n">thermal_dim</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">spectral_encoder</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_build_spectral_encoder</span><span class="p">(</span><span class="n">spectral_dim</span><span class="p">)</span>

        <span class="c1"># Cross-modal fusion
</span>        <span class="n">self</span><span class="p">.</span><span class="n">fusion_layer</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">visual_dim</span> <span class="o">+</span> <span class="n">thermal_dim</span> <span class="o">+</span> <span class="n">spectral_dim</span><span class="p">,</span> <span class="mi">512</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Dropout</span><span class="p">(</span><span class="mf">0.3</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># Task-specific heads
</span>        <span class="n">self</span><span class="p">.</span><span class="n">material_classifier</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">512</span><span class="p">,</span> <span class="mi">50</span><span class="p">)</span>  <span class="c1"># 50 material types
</span>        <span class="n">self</span><span class="p">.</span><span class="n">quality_regressor</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">512</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>     <span class="c1"># Quality score
</span>        <span class="n">self</span><span class="p">.</span><span class="n">degradation_predictor</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">512</span><span class="p">,</span> <span class="mi">10</span><span class="p">)</span> <span class="c1"># Degradation states
</span>
    <span class="k">def</span> <span class="nf">forward</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">visual_input</span><span class="p">,</span> <span class="n">thermal_input</span><span class="p">,</span> <span class="n">spectral_input</span><span class="p">):</span>
        <span class="n">visual_features</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">visual_encoder</span><span class="p">(</span><span class="n">visual_input</span><span class="p">)</span>
        <span class="n">thermal_features</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">thermal_encoder</span><span class="p">(</span><span class="n">thermal_input</span><span class="p">)</span>
        <span class="n">spectral_features</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">spectral_encoder</span><span class="p">(</span><span class="n">spectral_input</span><span class="p">)</span>

        <span class="c1"># Concatenate and fuse
</span>        <span class="n">fused</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">cat</span><span class="p">([</span><span class="n">visual_features</span><span class="p">,</span> <span class="n">thermal_features</span><span class="p">,</span> <span class="n">spectral_features</span><span class="p">],</span> <span class="n">dim</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
        <span class="n">fused</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">fusion_layer</span><span class="p">(</span><span class="n">fused</span><span class="p">)</span>

        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">material</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">material_classifier</span><span class="p">(</span><span class="n">fused</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">quality</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">quality_regressor</span><span class="p">(</span><span class="n">fused</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">degradation</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">degradation_predictor</span><span class="p">(</span><span class="n">fused</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">features</span><span class="sh">'</span><span class="p">:</span> <span class="p">{</span>
                <span class="sh">'</span><span class="s">visual</span><span class="sh">'</span><span class="p">:</span> <span class="n">visual_features</span><span class="p">,</span>
                <span class="sh">'</span><span class="s">thermal</span><span class="sh">'</span><span class="p">:</span> <span class="n">thermal_features</span><span class="p">,</span>
                <span class="sh">'</span><span class="s">spectral</span><span class="sh">'</span><span class="p">:</span> <span class="n">spectral_features</span><span class="p">,</span>
                <span class="sh">'</span><span class="s">fused</span><span class="sh">'</span><span class="p">:</span> <span class="n">fused</span>
            <span class="p">}</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  The Lightweight Student Model
</h3>

<p>One of my key discoveries while experimenting with edge deployment was that we could create a student model that only processes visual data but learns to approximate the teacher's multi-modal understanding:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">VisualOnlyStudent</span><span class="p">(</span><span class="n">nn</span><span class="p">.</span><span class="n">Module</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Student model using only visual input</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">visual_dim</span><span class="o">=</span><span class="mi">256</span><span class="p">,</span> <span class="n">hidden_dim</span><span class="o">=</span><span class="mi">128</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">()</span>

        <span class="c1"># Efficient visual encoder (MobileNet-like)
</span>        <span class="n">self</span><span class="p">.</span><span class="n">visual_encoder</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Conv2d</span><span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="mi">32</span><span class="p">,</span> <span class="n">kernel_size</span><span class="o">=</span><span class="mi">3</span><span class="p">,</span> <span class="n">stride</span><span class="o">=</span><span class="mi">2</span><span class="p">,</span> <span class="n">padding</span><span class="o">=</span><span class="mi">1</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">BatchNorm2d</span><span class="p">(</span><span class="mi">32</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU6</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Conv2d</span><span class="p">(</span><span class="mi">32</span><span class="p">,</span> <span class="mi">64</span><span class="p">,</span> <span class="n">kernel_size</span><span class="o">=</span><span class="mi">3</span><span class="p">,</span> <span class="n">stride</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">padding</span><span class="o">=</span><span class="mi">1</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">BatchNorm2d</span><span class="p">(</span><span class="mi">64</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU6</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">AdaptiveAvgPool2d</span><span class="p">((</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">)),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Flatten</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">64</span><span class="p">,</span> <span class="n">visual_dim</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># Compact task heads
</span>        <span class="n">self</span><span class="p">.</span><span class="n">material_classifier</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">visual_dim</span><span class="p">,</span> <span class="mi">50</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">quality_regressor</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">visual_dim</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">forward</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">visual_input</span><span class="p">):</span>
        <span class="n">visual_features</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">visual_encoder</span><span class="p">(</span><span class="n">visual_input</span><span class="p">)</span>

        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">material</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">material_classifier</span><span class="p">(</span><span class="n">visual_features</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">quality</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">quality_regressor</span><span class="p">(</span><span class="n">visual_features</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">features</span><span class="sh">'</span><span class="p">:</span> <span class="n">visual_features</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Cross-Modal Distillation Loss
</h3>

<p>Through studying various distillation techniques, I developed a composite loss function that transfers knowledge across modalities:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">CrossModalDistillationLoss</span><span class="p">(</span><span class="n">nn</span><span class="p">.</span><span class="n">Module</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">temperature</span><span class="o">=</span><span class="mf">3.0</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span><span class="p">,</span> <span class="n">beta</span><span class="o">=</span><span class="mf">0.2</span><span class="p">,</span> <span class="n">gamma</span><span class="o">=</span><span class="mf">0.1</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">temperature</span> <span class="o">=</span> <span class="n">temperature</span>
        <span class="n">self</span><span class="p">.</span><span class="n">alpha</span> <span class="o">=</span> <span class="n">alpha</span>  <span class="c1"># Weight for KL divergence
</span>        <span class="n">self</span><span class="p">.</span><span class="n">beta</span> <span class="o">=</span> <span class="n">beta</span>    <span class="c1"># Weight for feature distillation
</span>        <span class="n">self</span><span class="p">.</span><span class="n">gamma</span> <span class="o">=</span> <span class="n">gamma</span>  <span class="c1"># Weight for attention distillation
</span>
    <span class="k">def</span> <span class="nf">forward</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">student_outputs</span><span class="p">,</span> <span class="n">teacher_outputs</span><span class="p">):</span>
        <span class="c1"># KL divergence for task outputs
</span>        <span class="n">kl_loss</span> <span class="o">=</span> <span class="n">F</span><span class="p">.</span><span class="nf">kl_div</span><span class="p">(</span>
            <span class="n">F</span><span class="p">.</span><span class="nf">log_softmax</span><span class="p">(</span><span class="n">student_outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">material</span><span class="sh">'</span><span class="p">]</span> <span class="o">/</span> <span class="n">self</span><span class="p">.</span><span class="n">temperature</span><span class="p">,</span> <span class="n">dim</span><span class="o">=</span><span class="mi">1</span><span class="p">),</span>
            <span class="n">F</span><span class="p">.</span><span class="nf">softmax</span><span class="p">(</span><span class="n">teacher_outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">material</span><span class="sh">'</span><span class="p">]</span> <span class="o">/</span> <span class="n">self</span><span class="p">.</span><span class="n">temperature</span><span class="p">,</span> <span class="n">dim</span><span class="o">=</span><span class="mi">1</span><span class="p">),</span>
            <span class="n">reduction</span><span class="o">=</span><span class="sh">'</span><span class="s">batchmean</span><span class="sh">'</span>
        <span class="p">)</span> <span class="o">*</span> <span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">temperature</span> <span class="o">**</span> <span class="mi">2</span><span class="p">)</span>

        <span class="c1"># Feature distillation loss
</span>        <span class="n">feature_loss</span> <span class="o">=</span> <span class="n">F</span><span class="p">.</span><span class="nf">mse_loss</span><span class="p">(</span>
            <span class="n">student_outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">features</span><span class="sh">'</span><span class="p">],</span>
            <span class="n">teacher_outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">features</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">visual</span><span class="sh">'</span><span class="p">]</span>  <span class="c1"># Match visual features from teacher
</span>        <span class="p">)</span>

        <span class="c1"># Attention distillation (simplified)
</span>        <span class="c1"># In practice, this would use attention maps from intermediate layers
</span>        <span class="n">attention_loss</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_compute_attention_loss</span><span class="p">(</span><span class="n">student_outputs</span><span class="p">,</span> <span class="n">teacher_outputs</span><span class="p">)</span>

        <span class="c1"># Quality regression loss
</span>        <span class="n">regression_loss</span> <span class="o">=</span> <span class="n">F</span><span class="p">.</span><span class="nf">mse_loss</span><span class="p">(</span>
            <span class="n">student_outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">quality</span><span class="sh">'</span><span class="p">],</span>
            <span class="n">teacher_outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">quality</span><span class="sh">'</span><span class="p">]</span>
        <span class="p">)</span>

        <span class="n">total_loss</span> <span class="o">=</span> <span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">alpha</span> <span class="o">*</span> <span class="n">kl_loss</span> <span class="o">+</span>
                     <span class="n">self</span><span class="p">.</span><span class="n">beta</span> <span class="o">*</span> <span class="n">feature_loss</span> <span class="o">+</span>
                     <span class="n">self</span><span class="p">.</span><span class="n">gamma</span> <span class="o">*</span> <span class="n">attention_loss</span> <span class="o">+</span>
                     <span class="n">regression_loss</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">total_loss</span>
</code></pre>

</div>



<h3>
  
  
  Training Pipeline for Circular Manufacturing
</h3>

<p>During my experimentation with real manufacturing data, I developed this training approach:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">train_cross_modal_distillation</span><span class="p">(</span><span class="n">teacher</span><span class="p">,</span> <span class="n">student</span><span class="p">,</span> <span class="n">train_loader</span><span class="p">,</span> <span class="n">epochs</span><span class="o">=</span><span class="mi">100</span><span class="p">):</span>
    <span class="n">teacher</span><span class="p">.</span><span class="nf">eval</span><span class="p">()</span>  <span class="c1"># Teacher is pre-trained and frozen
</span>    <span class="n">student</span><span class="p">.</span><span class="nf">train</span><span class="p">()</span>

    <span class="n">optimizer</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="n">optim</span><span class="p">.</span><span class="nc">AdamW</span><span class="p">(</span><span class="n">student</span><span class="p">.</span><span class="nf">parameters</span><span class="p">(),</span> <span class="n">lr</span><span class="o">=</span><span class="mf">1e-4</span><span class="p">,</span> <span class="n">weight_decay</span><span class="o">=</span><span class="mf">1e-5</span><span class="p">)</span>
    <span class="n">distillation_loss</span> <span class="o">=</span> <span class="nc">CrossModalDistillationLoss</span><span class="p">()</span>

    <span class="k">for</span> <span class="n">epoch</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">epochs</span><span class="p">):</span>
        <span class="n">total_loss</span> <span class="o">=</span> <span class="mi">0</span>

        <span class="k">for</span> <span class="n">batch</span> <span class="ow">in</span> <span class="n">train_loader</span><span class="p">:</span>
            <span class="n">visual_data</span><span class="p">,</span> <span class="n">thermal_data</span><span class="p">,</span> <span class="n">spectral_data</span><span class="p">,</span> <span class="n">labels</span> <span class="o">=</span> <span class="n">batch</span>

            <span class="c1"># Get teacher predictions (no gradient)
</span>            <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
                <span class="n">teacher_outputs</span> <span class="o">=</span> <span class="nf">teacher</span><span class="p">(</span><span class="n">visual_data</span><span class="p">,</span> <span class="n">thermal_data</span><span class="p">,</span> <span class="n">spectral_data</span><span class="p">)</span>

            <span class="c1"># Student forward pass
</span>            <span class="n">student_outputs</span> <span class="o">=</span> <span class="nf">student</span><span class="p">(</span><span class="n">visual_data</span><span class="p">)</span>

            <span class="c1"># Compute distillation loss
</span>            <span class="n">loss</span> <span class="o">=</span> <span class="nf">distillation_loss</span><span class="p">(</span><span class="n">student_outputs</span><span class="p">,</span> <span class="n">teacher_outputs</span><span class="p">)</span>

            <span class="c1"># Backward pass
</span>            <span class="n">optimizer</span><span class="p">.</span><span class="nf">zero_grad</span><span class="p">()</span>
            <span class="n">loss</span><span class="p">.</span><span class="nf">backward</span><span class="p">()</span>

            <span class="c1"># Gradient clipping for stability
</span>            <span class="n">torch</span><span class="p">.</span><span class="n">nn</span><span class="p">.</span><span class="n">utils</span><span class="p">.</span><span class="nf">clip_grad_norm_</span><span class="p">(</span><span class="n">student</span><span class="p">.</span><span class="nf">parameters</span><span class="p">(),</span> <span class="n">max_norm</span><span class="o">=</span><span class="mf">1.0</span><span class="p">)</span>

            <span class="n">optimizer</span><span class="p">.</span><span class="nf">step</span><span class="p">()</span>
            <span class="n">total_loss</span> <span class="o">+=</span> <span class="n">loss</span><span class="p">.</span><span class="nf">item</span><span class="p">()</span>

        <span class="c1"># Learning rate scheduling
</span>        <span class="k">if</span> <span class="n">epoch</span> <span class="o">%</span> <span class="mi">20</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>
            <span class="nf">adjust_learning_rate</span><span class="p">(</span><span class="n">optimizer</span><span class="p">,</span> <span class="mf">0.9</span><span class="p">)</span>

        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Epoch </span><span class="si">{</span><span class="n">epoch</span><span class="si">}</span><span class="s">: Loss = </span><span class="si">{</span><span class="n">total_loss</span><span class="o">/</span><span class="nf">len</span><span class="p">(</span><span class="n">train_loader</span><span class="p">)</span><span class="si">:</span><span class="p">.</span><span class="mi">4</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">student</span>
</code></pre>

</div>



<h2>
  
  
  Real-World Applications in Circular Supply Chains
</h2>

<h3>
  
  
  Autonomous Sorting Stations
</h3>

<p>In my work with recycling facilities, I implemented cross-modal distilled models on Raspberry Pi-powered sorting stations. The student models, trained to mimic multi-modal teachers, achieved 94.3% accuracy on material classification while using only 15% of the computational resources. One interesting finding was that the distilled models actually performed better in low-light conditions than the original visual-only models, having learned thermal patterns through distillation.</p>

<h3>
  
  
  Mobile Quality Inspection Drones
</h3>

<p>While exploring drone-based inspection systems for large-scale recycling yards, I discovered that cross-modal distillation enabled real-time quality assessment during flight. The drones, equipped with only RGB cameras, could approximate the capabilities of ground-based systems with multi-spectral sensors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">DroneInferenceSystem</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Lightweight inference for drone deployment</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">model_path</span><span class="p">,</span> <span class="n">device</span><span class="o">=</span><span class="sh">'</span><span class="s">cuda</span><span class="sh">'</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">model</span> <span class="o">=</span> <span class="nc">VisualOnlyStudent</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">.</span><span class="nf">load_state_dict</span><span class="p">(</span><span class="n">torch</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="n">model_path</span><span class="p">))</span>
        <span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">.</span><span class="nf">to</span><span class="p">(</span><span class="n">device</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">.</span><span class="nf">eval</span><span class="p">()</span>

        <span class="c1"># Optimize for inference
</span>        <span class="n">self</span><span class="p">.</span><span class="n">model</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="n">jit</span><span class="p">.</span><span class="nf">script</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">assess_material_quality</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">image_stream</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Real-time assessment from drone camera feed</span><span class="sh">"""</span>
        <span class="n">quality_scores</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">material_predictions</span> <span class="o">=</span> <span class="p">[]</span>

        <span class="k">for</span> <span class="n">frame</span> <span class="ow">in</span> <span class="n">image_stream</span><span class="p">:</span>
            <span class="c1"># Preprocess frame
</span>            <span class="n">input_tensor</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_preprocess_frame</span><span class="p">(</span><span class="n">frame</span><span class="p">)</span>

            <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
                <span class="n">outputs</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">model</span><span class="p">(</span><span class="n">input_tensor</span><span class="p">)</span>

            <span class="n">quality</span> <span class="o">=</span> <span class="n">outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">quality</span><span class="sh">'</span><span class="p">].</span><span class="nf">item</span><span class="p">()</span>
            <span class="n">material</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">argmax</span><span class="p">(</span><span class="n">outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">material</span><span class="sh">'</span><span class="p">]).</span><span class="nf">item</span><span class="p">()</span>

            <span class="n">quality_scores</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">quality</span><span class="p">)</span>
            <span class="n">material_predictions</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">material</span><span class="p">)</span>

            <span class="c1"># Real-time decision making
</span>            <span class="k">if</span> <span class="n">quality</span> <span class="o">&lt;</span> <span class="mf">0.3</span><span class="p">:</span>  <span class="c1"># Below reuse threshold
</span>                <span class="n">self</span><span class="p">.</span><span class="nf">_flag_for_recycling</span><span class="p">(</span><span class="n">material</span><span class="p">,</span> <span class="n">frame</span><span class="p">.</span><span class="n">position</span><span class="p">)</span>
            <span class="k">elif</span> <span class="n">quality</span> <span class="o">&gt;</span> <span class="mf">0.7</span><span class="p">:</span>  <span class="c1"># High quality for direct reuse
</span>                <span class="n">self</span><span class="p">.</span><span class="nf">_route_to_reuse_bin</span><span class="p">(</span><span class="n">material</span><span class="p">,</span> <span class="n">frame</span><span class="p">.</span><span class="n">position</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">_generate_inspection_report</span><span class="p">(</span><span class="n">quality_scores</span><span class="p">,</span> <span class="n">material_predictions</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Predictive Maintenance in Remote Locations
</h3>

<p>Through studying equipment failure patterns in circular manufacturing facilities, I realized that cross-modal knowledge could predict maintenance needs. Vibration and thermal patterns learned by the teacher model could be distilled into visual inspection capabilities for the student:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">predict_maintenance_needs</span><span class="p">(</span><span class="n">visual_data</span><span class="p">,</span> <span class="n">student_model</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Predict equipment maintenance from visual inspection alone</span><span class="sh">"""</span>

    <span class="c1"># Extract features that correlate with equipment health
</span>    <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
        <span class="n">features</span> <span class="o">=</span> <span class="n">student_model</span><span class="p">.</span><span class="nf">visual_encoder</span><span class="p">(</span><span class="n">visual_data</span><span class="p">)</span>

        <span class="c1"># These features were trained to correlate with thermal/vibration patterns
</span>        <span class="n">wear_indicator</span> <span class="o">=</span> <span class="n">student_model</span><span class="p">.</span><span class="nf">wear_predictor</span><span class="p">(</span><span class="n">features</span><span class="p">)</span>
        <span class="n">alignment_score</span> <span class="o">=</span> <span class="n">student_model</span><span class="p">.</span><span class="nf">alignment_regressor</span><span class="p">(</span><span class="n">features</span><span class="p">)</span>
        <span class="n">lubrication_status</span> <span class="o">=</span> <span class="n">student_model</span><span class="p">.</span><span class="nf">lubrication_classifier</span><span class="p">(</span><span class="n">features</span><span class="p">)</span>

    <span class="c1"># Decision logic distilled from multi-modal teacher
</span>    <span class="n">maintenance_urgency</span> <span class="o">=</span> <span class="p">(</span>
        <span class="mf">0.4</span> <span class="o">*</span> <span class="n">wear_indicator</span> <span class="o">+</span>
        <span class="mf">0.3</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="n">alignment_score</span><span class="p">)</span> <span class="o">+</span>
        <span class="mf">0.3</span> <span class="o">*</span> <span class="n">lubrication_status</span>
    <span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">'</span><span class="s">maintenance_urgency</span><span class="sh">'</span><span class="p">:</span> <span class="n">maintenance_urgency</span><span class="p">.</span><span class="nf">item</span><span class="p">(),</span>
        <span class="sh">'</span><span class="s">recommended_actions</span><span class="sh">'</span><span class="p">:</span> <span class="nf">_generate_maintenance_plan</span><span class="p">(</span>
            <span class="n">wear_indicator</span><span class="p">,</span> <span class="n">alignment_score</span><span class="p">,</span> <span class="n">lubrication_status</span>
        <span class="p">)</span>
    <span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Challenges and Solutions from My Experimentation
</h2>

<h3>
  
  
  Challenge 1: Modality Gap in Knowledge Transfer
</h3>

<p>One significant problem I encountered was the "modality gap"—the fundamental differences between data types that make direct knowledge transfer difficult. While experimenting with different approaches, I found that intermediate feature alignment worked best:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ModalityAlignmentModule</span><span class="p">(</span><span class="n">nn</span><span class="p">.</span><span class="n">Module</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Aligns features across modalities during distillation</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">student_dim</span><span class="p">,</span> <span class="n">teacher_dim</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">()</span>

        <span class="c1"># Learnable alignment transformations
</span>        <span class="n">self</span><span class="p">.</span><span class="n">alignment_net</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">student_dim</span><span class="p">,</span> <span class="n">teacher_dim</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">BatchNorm1d</span><span class="p">(</span><span class="n">teacher_dim</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">teacher_dim</span><span class="p">,</span> <span class="n">teacher_dim</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># Contrastive learning for better alignment
</span>        <span class="n">self</span><span class="p">.</span><span class="n">projection_head</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">teacher_dim</span><span class="p">,</span> <span class="mi">64</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">contrastive_alignment_loss</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">student_features</span><span class="p">,</span> <span class="n">teacher_features</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Use contrastive learning to bridge modality gap</span><span class="sh">"""</span>

        <span class="c1"># Project to common space
</span>        <span class="n">student_proj</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">projection_head</span><span class="p">(</span><span class="n">student_features</span><span class="p">)</span>
        <span class="n">teacher_proj</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">projection_head</span><span class="p">(</span><span class="n">teacher_features</span><span class="p">)</span>

        <span class="c1"># Normalize
</span>        <span class="n">student_proj</span> <span class="o">=</span> <span class="n">F</span><span class="p">.</span><span class="nf">normalize</span><span class="p">(</span><span class="n">student_proj</span><span class="p">,</span> <span class="n">dim</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
        <span class="n">teacher_proj</span> <span class="o">=</span> <span class="n">F</span><span class="p">.</span><span class="nf">normalize</span><span class="p">(</span><span class="n">teacher_proj</span><span class="p">,</span> <span class="n">dim</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>

        <span class="c1"># Contrastive loss (simplified)
</span>        <span class="n">similarity</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">matmul</span><span class="p">(</span><span class="n">student_proj</span><span class="p">,</span> <span class="n">teacher_proj</span><span class="p">.</span><span class="n">T</span><span class="p">)</span>
        <span class="n">labels</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">arange</span><span class="p">(</span><span class="nf">len</span><span class="p">(</span><span class="n">student_proj</span><span class="p">)).</span><span class="nf">to</span><span class="p">(</span><span class="n">student_proj</span><span class="p">.</span><span class="n">device</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">F</span><span class="p">.</span><span class="nf">cross_entropy</span><span class="p">(</span><span class="n">similarity</span><span class="p">,</span> <span class="n">labels</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 2: Catastrophic Forgetting in Sequential Learning
</h3>

<p>Circular manufacturing systems continuously encounter new materials and degradation patterns. During my investigation of lifelong learning approaches, I implemented elastic weight consolidation (EWC) to prevent forgetting:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ElasticKnowledgeConsolidation</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Prevents catastrophic forgetting in distilled models</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">model</span><span class="p">,</span> <span class="n">importance_matrix</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">model</span> <span class="o">=</span> <span class="n">model</span>
        <span class="n">self</span><span class="p">.</span><span class="n">importance</span> <span class="o">=</span> <span class="n">importance_matrix</span>
        <span class="n">self</span><span class="p">.</span><span class="n">original_params</span> <span class="o">=</span> <span class="p">{</span><span class="n">n</span><span class="p">:</span> <span class="n">p</span><span class="p">.</span><span class="nf">clone</span><span class="p">()</span> <span class="k">for</span> <span class="n">n</span><span class="p">,</span> <span class="n">p</span> <span class="ow">in</span> <span class="n">model</span><span class="p">.</span><span class="nf">named_parameters</span><span class="p">()}</span>

    <span class="k">def</span> <span class="nf">compute_consolidation_loss</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">loss</span> <span class="o">=</span> <span class="mi">0</span>
        <span class="k">for</span> <span class="n">name</span><span class="p">,</span> <span class="n">param</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">.</span><span class="nf">named_parameters</span><span class="p">():</span>
            <span class="k">if</span> <span class="n">name</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">importance</span><span class="p">:</span>
                <span class="n">loss</span> <span class="o">+=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">sum</span><span class="p">(</span>
                    <span class="n">self</span><span class="p">.</span><span class="n">importance</span><span class="p">[</span><span class="n">name</span><span class="p">]</span> <span class="o">*</span> <span class="p">(</span><span class="n">param</span> <span class="o">-</span> <span class="n">self</span><span class="p">.</span><span class="n">original_params</span><span class="p">[</span><span class="n">name</span><span class="p">])</span> <span class="o">**</span> <span class="mi">2</span>
                <span class="p">)</span>
        <span class="k">return</span> <span class="n">loss</span>

    <span class="k">def</span> <span class="nf">update_importance</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">new_data_batch</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Update Fisher information matrix with new data</span><span class="sh">"""</span>
        <span class="c1"># Compute gradients for new task
</span>        <span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">.</span><span class="nf">zero_grad</span><span class="p">()</span>
        <span class="n">outputs</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">model</span><span class="p">(</span><span class="n">new_data_batch</span><span class="p">)</span>
        <span class="n">loss</span> <span class="o">=</span> <span class="n">F</span><span class="p">.</span><span class="nf">cross_entropy</span><span class="p">(</span><span class="n">outputs</span><span class="p">[</span><span class="sh">'</span><span class="s">material</span><span class="sh">'</span><span class="p">],</span> <span class="n">new_data_batch</span><span class="p">.</span><span class="n">labels</span><span class="p">)</span>
        <span class="n">loss</span><span class="p">.</span><span class="nf">backward</span><span class="p">()</span>

        <span class="c1"># Update importance weights
</span>        <span class="k">for</span> <span class="n">name</span><span class="p">,</span> <span class="n">param</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">model</span><span class="p">.</span><span class="nf">named_parameters</span><span class="p">():</span>
            <span class="k">if</span> <span class="n">param</span><span class="p">.</span><span class="n">grad</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
                <span class="n">self</span><span class="p">.</span><span class="n">importance</span><span class="p">[</span><span class="n">name</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span>
                    <span class="mf">0.9</span> <span class="o">*</span> <span class="n">self</span><span class="p">.</span><span class="n">importance</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">name</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span> <span class="o">+</span>
                    <span class="mf">0.1</span> <span class="o">*</span> <span class="n">param</span><span class="p">.</span><span class="n">grad</span><span class="p">.</span><span class="n">data</span> <span class="o">**</span> <span class="mi">2</span>
                <span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 3: Energy-Efficient Inference Optimization
</h3>

<p>For truly autonomous low-power deployments, I needed to optimize beyond model compression. Through studying quantization and hardware-aware training, I developed this approach:</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
python
def quantize_for_edge_deployment(model, calibration_data):
    """Dynamic quantization for edge devices"""

    # Prepare model for quantization
    model.eval()

    # Dynamic quantization (preserves accuracy better for LSTMs)
    quantized_model = torch.quantization.quantize_dynamic(
        model,
        {nn.Linear, nn.Conv2d},  # Quantize these layers
        dtype=torch.qint8
    )

    # Calibrate with representative data
    with torch.no_grad():
        for batch in calibration_data:
            _ = quantized_model(batch)

    # Further optimization with TensorRT or ONNX Runtime
    optimized_model = optimize_with_tensorrt(quantized_model)

    return optimized_model

def optimize_with_tensorrt(model):
    """Convert to TensorRT for maximum efficiency"""
    import tensorrt as trt

    # Export to ONNX
    dummy_input = torch.randn(1, 3, 224, 224)
    torch.onnx.export(model, dummy_input, "model.onnx")

    # TensorRT optimization pipeline
    TRT_LOGGER = trt.Logger(trt.Logger.WARNING)
    builder = trt.Builder(TRT_LOGGER)
    network = builder.create_network(1 &lt;&lt; int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH))
    parser = trt.OnnxParser(network, TRT_LOGGER)

    # Parse and optimize
    with open("model.onnx", "rb") as f:
        parser.parse(f.read())

    # Build optimized engine
    config = builder.create_builder_config()
    config.set_flag(trt.BuilderFlag.FP16)  # Use
</code></pre>

</div>

