---
Title: Explainable Causal Reinforcement Learning for precision oncology clinical workflows under real-time policy constraints
Description: 
Author: Rikin Patel
Date: 2026-03-17T21:43:05.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1559757148-5c350d0d3c56%3Fixlib%3Drb-4.0.3%26auto%3Dformat%26fit%3Dcrop%26w%3D1200%26q%3D80" alt="Explainable Causal Reinforcement Learning for Precision Oncology" width="1200" height="675">
</h1>

<h1>
  
  
  Explainable Causal Reinforcement Learning for precision oncology clinical workflows under real-time policy constraints
</h1>

<h2>
  
  
  A Personal Journey into the Heart of AI-Driven Medicine
</h2>

<p>My journey into this fascinating intersection of AI and oncology began during a late-night research session about three years ago. I was experimenting with standard reinforcement learning (RL) for treatment optimization when I encountered a particularly troubling case study. The model had recommended a treatment regimen that, statistically, should have worked—but the patient experienced severe adverse effects. While exploring the model's decision-making process, I discovered a fundamental limitation: traditional RL could identify correlations between treatments and outcomes, but it couldn't distinguish causation from mere association. This realization sent me down a rabbit hole of causal inference literature, structural equation modeling, and eventually to the frontier of explainable AI.</p>

<p>In my research of precision oncology workflows, I realized that clinicians weren't just asking "what treatment works?" but "why does this treatment work for this specific patient?" and "what would happen if we tried something different?" These questions demanded more than predictive accuracy—they required causal understanding and transparent reasoning. Through studying dozens of clinical trials and electronic health records, I learned that oncology decisions operate under severe real-time constraints: treatment windows measured in days, rapidly changing patient conditions, and the constant pressure of tumor progression.</p>

<h2>
  
  
  Technical Background: The Convergence of Three Disciplines
</h2>

<h3>
  
  
  The Causal Revolution in Machine Learning
</h3>

<p>While exploring causal inference frameworks, I discovered that Judea Pearl's ladder of causation provides the perfect conceptual framework for medical AI. Most machine learning operates on the first rung—association. We observe that treatment A correlates with outcome B. Causal models ascend to the second rung—intervention. They answer "what if" questions: what would happen if we administered treatment A? The highest rung, counterfactuals, addresses what would have happened had we chosen a different treatment.</p>

<p>One interesting finding from my experimentation with do-calculus was that many medical "best practices" were actually causal relationships waiting to be formalized. For instance, the relationship between chemotherapy dosage and white blood cell count isn't just correlational—it's fundamentally causal, with clear directionality and potential confounders.</p>

<h3>
  
  
  Reinforcement Learning with Policy Constraints
</h3>

<p>During my investigation of constrained RL, I found that oncology presents unique challenges. Unlike games where agents can explore freely, medical RL must operate within ethical and clinical boundaries. Real-time policy constraints aren't just optimization boundaries—they're hard requirements derived from pharmacokinetics, toxicity thresholds, and clinical guidelines.</p>

<p>As I was experimenting with constrained policy optimization, I came across the fundamental tension between exploration and safety. In oncology, exploration doesn't mean trying random treatments—it means carefully navigating the space of evidence-based options while respecting individual patient tolerances.</p>

<h3>
  
  
  Explainability as a Clinical Necessity
</h3>

<p>My exploration of explainable AI in medical contexts revealed that clinicians don't just want feature importance scores. They need causal explanations: "The model recommends reducing the dosage because your recent liver function tests show decreased metabolic capacity, which would otherwise lead to toxic accumulation based on these pharmacokinetic simulations."</p>

<h2>
  
  
  Implementation Details: Building an Explainable Causal RL System
</h2>

<h3>
  
  
  Causal Model Representation
</h3>

<p>Through studying structural causal models (SCMs), I developed a hybrid approach that combines domain knowledge with data-driven discovery. Here's a simplified representation of how we structure the causal graph for oncology decision-making:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">networkx</span> <span class="k">as</span> <span class="n">nx</span>
<span class="kn">import</span> <span class="n">pandas</span> <span class="k">as</span> <span class="n">pd</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Dict</span><span class="p">,</span> <span class="n">List</span><span class="p">,</span> <span class="n">Tuple</span>

<span class="k">class</span> <span class="nc">OncologyCausalGraph</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">patient_data</span><span class="p">:</span> <span class="n">pd</span><span class="p">.</span><span class="n">DataFrame</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Initialize causal graph for oncology decision-making</span><span class="sh">"""</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span> <span class="o">=</span> <span class="n">nx</span><span class="p">.</span><span class="nc">DiGraph</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">patient_data</span> <span class="o">=</span> <span class="n">patient_data</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">_build_base_structure</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">_build_base_structure</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Build the fundamental causal relationships in oncology</span><span class="sh">"""</span>
        <span class="c1"># Treatment nodes
</span>        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">chemotherapy_dose</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">treatment</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">constraints</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">min</span><span class="sh">"</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="sh">"</span><span class="s">max</span><span class="sh">"</span><span class="p">:</span> <span class="mi">100</span><span class="p">})</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">immunotherapy</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">treatment</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">constraints</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">binary</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span><span class="p">})</span>

        <span class="c1"># Patient state nodes
</span>        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">tumor_burden</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">state</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">measurement_unit</span><span class="o">=</span><span class="sh">"</span><span class="s">mm^3</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">immune_response</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">state</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">measurement_unit</span><span class="o">=</span><span class="sh">"</span><span class="s">cell_count</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Physiological nodes
</span>        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">liver_function</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">physiology</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">constraints</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">normal_range</span><span class="sh">"</span><span class="p">:</span> <span class="p">(</span><span class="mi">40</span><span class="p">,</span> <span class="mi">120</span><span class="p">)})</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">kidney_function</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">physiology</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Outcome nodes
</span>        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">toxicity</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">outcome</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">severity_levels</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">mild</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">moderate</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">severe</span><span class="sh">"</span><span class="p">])</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_node</span><span class="p">(</span><span class="sh">"</span><span class="s">progression_free_survival</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">node_type</span><span class="o">=</span><span class="sh">"</span><span class="s">outcome</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">measurement_unit</span><span class="o">=</span><span class="sh">"</span><span class="s">days</span><span class="sh">"</span><span class="p">)</span>

        <span class="c1"># Causal relationships (edges with effect sizes and confidence)
</span>        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_edge</span><span class="p">(</span><span class="sh">"</span><span class="s">chemotherapy_dose</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">tumor_burden</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">effect_type</span><span class="o">=</span><span class="sh">"</span><span class="s">negative</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">confidence</span><span class="o">=</span><span class="mf">0.85</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_edge</span><span class="p">(</span><span class="sh">"</span><span class="s">chemotherapy_dose</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">immune_response</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">effect_type</span><span class="o">=</span><span class="sh">"</span><span class="s">negative</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">confidence</span><span class="o">=</span><span class="mf">0.75</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_edge</span><span class="p">(</span><span class="sh">"</span><span class="s">chemotherapy_dose</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">toxicity</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">effect_type</span><span class="o">=</span><span class="sh">"</span><span class="s">positive</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">confidence</span><span class="o">=</span><span class="mf">0.90</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_edge</span><span class="p">(</span><span class="sh">"</span><span class="s">immunotherapy</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">immune_response</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">effect_type</span><span class="o">=</span><span class="sh">"</span><span class="s">positive</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">confidence</span><span class="o">=</span><span class="mf">0.80</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_edge</span><span class="p">(</span><span class="sh">"</span><span class="s">liver_function</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">chemotherapy_dose</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">effect_type</span><span class="o">=</span><span class="sh">"</span><span class="s">moderator</span><span class="sh">"</span><span class="p">,</span>
                           <span class="n">confidence</span><span class="o">=</span><span class="mf">0.95</span><span class="p">)</span>  <span class="c1"># Liver function affects dose tolerance
</span>
    <span class="k">def</span> <span class="nf">estimate_causal_effect</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">treatment</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">outcome</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
                              <span class="n">adjustment_set</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="nb">str</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Estimate causal effect using backdoor criterion</span><span class="sh">"""</span>
        <span class="c1"># Simplified implementation - in practice would use do-calculus
</span>        <span class="c1"># or double machine learning
</span>        <span class="n">treatment_data</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">patient_data</span><span class="p">[</span><span class="n">treatment</span><span class="p">]</span>
        <span class="n">outcome_data</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">patient_data</span><span class="p">[</span><span class="n">outcome</span><span class="p">]</span>

        <span class="c1"># Adjust for confounders
</span>        <span class="k">if</span> <span class="n">adjustment_set</span><span class="p">:</span>
            <span class="n">confounders</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">patient_data</span><span class="p">[</span><span class="n">adjustment_set</span><span class="p">]</span>
            <span class="c1"># Use linear regression for simplicity (real implementation would be more robust)
</span>            <span class="kn">from</span> <span class="n">sklearn.linear_model</span> <span class="kn">import</span> <span class="n">LinearRegression</span>
            <span class="n">model</span> <span class="o">=</span> <span class="nc">LinearRegression</span><span class="p">()</span>
            <span class="n">model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">confounders</span><span class="p">,</span> <span class="n">treatment_data</span><span class="p">)</span>
            <span class="n">treatment_residuals</span> <span class="o">=</span> <span class="n">treatment_data</span> <span class="o">-</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">confounders</span><span class="p">)</span>

            <span class="n">model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">confounders</span><span class="p">,</span> <span class="n">outcome_data</span><span class="p">)</span>
            <span class="n">outcome_residuals</span> <span class="o">=</span> <span class="n">outcome_data</span> <span class="o">-</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">confounders</span><span class="p">)</span>

            <span class="n">effect</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">corrcoef</span><span class="p">(</span><span class="n">treatment_residuals</span><span class="p">,</span> <span class="n">outcome_residuals</span><span class="p">)[</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">]</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="n">effect</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">corrcoef</span><span class="p">(</span><span class="n">treatment_data</span><span class="p">,</span> <span class="n">outcome_data</span><span class="p">)[</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">]</span>

        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">treatment</span><span class="sh">"</span><span class="p">:</span> <span class="n">treatment</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">outcome</span><span class="sh">"</span><span class="p">:</span> <span class="n">outcome</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">causal_effect</span><span class="sh">"</span><span class="p">:</span> <span class="n">effect</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">adjustment_set</span><span class="sh">"</span><span class="p">:</span> <span class="n">adjustment_set</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">interpretation</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">_interpret_effect</span><span class="p">(</span><span class="n">effect</span><span class="p">,</span> <span class="n">treatment</span><span class="p">,</span> <span class="n">outcome</span><span class="p">)</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Constrained Reinforcement Learning Implementation
</h3>

<p>One of the most challenging aspects I encountered was implementing real-time policy constraints. While learning about constrained Markov decision processes (CMDPs), I observed that oncology constraints are often dynamic—they change based on patient response.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">torch</span>
<span class="kn">import</span> <span class="n">torch.nn</span> <span class="k">as</span> <span class="n">nn</span>
<span class="kn">import</span> <span class="n">torch.optim</span> <span class="k">as</span> <span class="n">optim</span>
<span class="kn">from</span> <span class="n">collections</span> <span class="kn">import</span> <span class="n">deque</span>
<span class="kn">import</span> <span class="n">random</span>

<span class="k">class</span> <span class="nc">ConstrainedOncologyAgent</span><span class="p">(</span><span class="n">nn</span><span class="p">.</span><span class="n">Module</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">state_dim</span><span class="p">:</span> <span class="nb">int</span><span class="p">,</span> <span class="n">action_dim</span><span class="p">:</span> <span class="nb">int</span><span class="p">,</span> <span class="n">constraint_dim</span><span class="p">:</span> <span class="nb">int</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">state_dim</span> <span class="o">=</span> <span class="n">state_dim</span>
        <span class="n">self</span><span class="p">.</span><span class="n">action_dim</span> <span class="o">=</span> <span class="n">action_dim</span>
        <span class="n">self</span><span class="p">.</span><span class="n">constraint_dim</span> <span class="o">=</span> <span class="n">constraint_dim</span>

        <span class="c1"># Policy network
</span>        <span class="n">self</span><span class="p">.</span><span class="n">policy_net</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">state_dim</span><span class="p">,</span> <span class="mi">128</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="mi">128</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="n">action_dim</span> <span class="o">*</span> <span class="mi">2</span><span class="p">)</span>  <span class="c1"># Mean and log_std for each action
</span>        <span class="p">)</span>

        <span class="c1"># Value network
</span>        <span class="n">self</span><span class="p">.</span><span class="n">value_net</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">state_dim</span><span class="p">,</span> <span class="mi">128</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="mi">128</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># Constraint network - predicts constraint violations
</span>        <span class="n">self</span><span class="p">.</span><span class="n">constraint_net</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">state_dim</span> <span class="o">+</span> <span class="n">action_dim</span><span class="p">,</span> <span class="mi">64</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">64</span><span class="p">,</span> <span class="n">constraint_dim</span><span class="p">)</span>
        <span class="p">)</span>

    <span class="k">def</span> <span class="nf">forward</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">state</span><span class="p">:</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Tuple</span><span class="p">[</span><span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">,</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">]:</span>
        <span class="sh">"""</span><span class="s">Forward pass through policy network</span><span class="sh">"""</span>
        <span class="n">policy_output</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">policy_net</span><span class="p">(</span><span class="n">state</span><span class="p">)</span>
        <span class="n">mean</span> <span class="o">=</span> <span class="n">policy_output</span><span class="p">[:,</span> <span class="p">:</span><span class="n">self</span><span class="p">.</span><span class="n">action_dim</span><span class="p">]</span>
        <span class="n">log_std</span> <span class="o">=</span> <span class="n">policy_output</span><span class="p">[:,</span> <span class="n">self</span><span class="p">.</span><span class="n">action_dim</span><span class="p">:]</span>
        <span class="n">std</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">exp</span><span class="p">(</span><span class="n">log_std</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">mean</span><span class="p">,</span> <span class="n">std</span>

    <span class="k">def</span> <span class="nf">get_action</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">state</span><span class="p">:</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">,</span>
                   <span class="n">current_constraints</span><span class="p">:</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Sample action with constraint satisfaction</span><span class="sh">"""</span>
        <span class="n">mean</span><span class="p">,</span> <span class="n">std</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">forward</span><span class="p">(</span><span class="n">state</span><span class="p">)</span>
        <span class="n">normal_dist</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="n">distributions</span><span class="p">.</span><span class="nc">Normal</span><span class="p">(</span><span class="n">mean</span><span class="p">,</span> <span class="n">std</span><span class="p">)</span>

        <span class="c1"># Sample candidate actions
</span>        <span class="n">candidate_actions</span> <span class="o">=</span> <span class="n">normal_dist</span><span class="p">.</span><span class="nf">sample</span><span class="p">((</span><span class="mi">100</span><span class="p">,))</span>  <span class="c1"># Sample 100 candidates
</span>
        <span class="c1"># Evaluate constraints for each candidate
</span>        <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
            <span class="n">constraint_violations</span> <span class="o">=</span> <span class="p">[]</span>
            <span class="k">for</span> <span class="n">action</span> <span class="ow">in</span> <span class="n">candidate_actions</span><span class="p">:</span>
                <span class="c1"># Predict constraint violations
</span>                <span class="n">state_action</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">cat</span><span class="p">([</span><span class="n">state</span><span class="p">,</span> <span class="n">action</span><span class="p">.</span><span class="nf">unsqueeze</span><span class="p">(</span><span class="mi">0</span><span class="p">)],</span> <span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>
                <span class="n">violations</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">constraint_net</span><span class="p">(</span><span class="n">state_action</span><span class="p">)</span>
                <span class="n">constraint_violations</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">violations</span><span class="p">)</span>

            <span class="n">constraint_violations</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">stack</span><span class="p">(</span><span class="n">constraint_violations</span><span class="p">)</span>

            <span class="c1"># Find actions that satisfy all constraints
</span>            <span class="n">constraint_satisfied</span> <span class="o">=</span> <span class="p">(</span><span class="n">constraint_violations</span> <span class="o">&lt;=</span> <span class="n">current_constraints</span><span class="p">).</span><span class="nf">all</span><span class="p">(</span><span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>

            <span class="k">if</span> <span class="n">constraint_satisfied</span><span class="p">.</span><span class="nf">any</span><span class="p">():</span>
                <span class="c1"># Choose best action among those satisfying constraints
</span>                <span class="n">valid_indices</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">where</span><span class="p">(</span><span class="n">constraint_satisfied</span><span class="p">)[</span><span class="mi">0</span><span class="p">]</span>
                <span class="c1"># Evaluate expected value for valid actions
</span>                <span class="n">state_repeated</span> <span class="o">=</span> <span class="n">state</span><span class="p">.</span><span class="nf">repeat</span><span class="p">(</span><span class="nf">len</span><span class="p">(</span><span class="n">valid_indices</span><span class="p">),</span> <span class="mi">1</span><span class="p">)</span>
                <span class="n">valid_actions</span> <span class="o">=</span> <span class="n">candidate_actions</span><span class="p">[</span><span class="n">valid_indices</span><span class="p">]</span>
                <span class="n">values</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">value_net</span><span class="p">(</span><span class="n">state_repeated</span><span class="p">)</span>
                <span class="n">best_idx</span> <span class="o">=</span> <span class="n">valid_indices</span><span class="p">[</span><span class="n">torch</span><span class="p">.</span><span class="nf">argmax</span><span class="p">(</span><span class="n">values</span><span class="p">)]</span>
                <span class="k">return</span> <span class="n">candidate_actions</span><span class="p">[</span><span class="n">best_idx</span><span class="p">]</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="c1"># No action satisfies all constraints - choose least violating
</span>                <span class="n">total_violations</span> <span class="o">=</span> <span class="n">constraint_violations</span><span class="p">.</span><span class="nf">sum</span><span class="p">(</span><span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>
                <span class="n">best_idx</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">argmin</span><span class="p">(</span><span class="n">total_violations</span><span class="p">)</span>
                <span class="k">return</span> <span class="n">candidate_actions</span><span class="p">[</span><span class="n">best_idx</span><span class="p">]</span>

<span class="k">class</span> <span class="nc">ConstrainedPPO</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">agent</span><span class="p">:</span> <span class="n">ConstrainedOncologyAgent</span><span class="p">,</span>
                 <span class="n">constraint_tolerance</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">0.1</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">agent</span> <span class="o">=</span> <span class="n">agent</span>
        <span class="n">self</span><span class="p">.</span><span class="n">constraint_tolerance</span> <span class="o">=</span> <span class="n">constraint_tolerance</span>
        <span class="n">self</span><span class="p">.</span><span class="n">optimizer</span> <span class="o">=</span> <span class="n">optim</span><span class="p">.</span><span class="nc">Adam</span><span class="p">(</span><span class="n">agent</span><span class="p">.</span><span class="nf">parameters</span><span class="p">(),</span> <span class="n">lr</span><span class="o">=</span><span class="mf">3e-4</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">memory</span> <span class="o">=</span> <span class="nf">deque</span><span class="p">(</span><span class="n">maxlen</span><span class="o">=</span><span class="mi">10000</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">update</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">states</span><span class="p">,</span> <span class="n">actions</span><span class="p">,</span> <span class="n">rewards</span><span class="p">,</span> <span class="n">next_states</span><span class="p">,</span>
               <span class="n">constraints</span><span class="p">,</span> <span class="n">constraint_violations</span><span class="p">,</span> <span class="n">dones</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Update policy with constraint-aware loss</span><span class="sh">"""</span>
        <span class="n">states</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nc">FloatTensor</span><span class="p">(</span><span class="n">states</span><span class="p">)</span>
        <span class="n">actions</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nc">FloatTensor</span><span class="p">(</span><span class="n">actions</span><span class="p">)</span>
        <span class="n">rewards</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nc">FloatTensor</span><span class="p">(</span><span class="n">rewards</span><span class="p">)</span>
        <span class="n">next_states</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nc">FloatTensor</span><span class="p">(</span><span class="n">next_states</span><span class="p">)</span>
        <span class="n">constraints</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nc">FloatTensor</span><span class="p">(</span><span class="n">constraints</span><span class="p">)</span>
        <span class="n">constraint_violations</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nc">FloatTensor</span><span class="p">(</span><span class="n">constraint_violations</span><span class="p">)</span>

        <span class="c1"># Calculate advantages
</span>        <span class="k">with</span> <span class="n">torch</span><span class="p">.</span><span class="nf">no_grad</span><span class="p">():</span>
            <span class="n">values</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">agent</span><span class="p">.</span><span class="nf">value_net</span><span class="p">(</span><span class="n">states</span><span class="p">)</span>
            <span class="n">next_values</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">agent</span><span class="p">.</span><span class="nf">value_net</span><span class="p">(</span><span class="n">next_states</span><span class="p">)</span>
            <span class="n">advantages</span> <span class="o">=</span> <span class="n">rewards</span> <span class="o">+</span> <span class="mf">0.99</span> <span class="o">*</span> <span class="n">next_values</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="n">dones</span><span class="p">)</span> <span class="o">-</span> <span class="n">values</span>

        <span class="c1"># Policy loss with constraint penalty
</span>        <span class="n">mean</span><span class="p">,</span> <span class="n">std</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">agent</span><span class="p">(</span><span class="n">states</span><span class="p">)</span>
        <span class="n">normal_dist</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="n">distributions</span><span class="p">.</span><span class="nc">Normal</span><span class="p">(</span><span class="n">mean</span><span class="p">,</span> <span class="n">std</span><span class="p">)</span>
        <span class="n">log_probs</span> <span class="o">=</span> <span class="n">normal_dist</span><span class="p">.</span><span class="nf">log_prob</span><span class="p">(</span><span class="n">actions</span><span class="p">).</span><span class="nf">sum</span><span class="p">(</span><span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>

        <span class="c1"># Constraint violation penalty
</span>        <span class="n">constraint_penalty</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">relu</span><span class="p">(</span><span class="n">constraint_violations</span> <span class="o">-</span> <span class="n">constraints</span><span class="p">)</span>
        <span class="n">penalty_weight</span> <span class="o">=</span> <span class="mf">10.0</span>  <span class="c1"># Hyperparameter tuned through experimentation
</span>
        <span class="c1"># Combined loss
</span>        <span class="n">policy_loss</span> <span class="o">=</span> <span class="o">-</span><span class="p">(</span><span class="n">log_probs</span> <span class="o">*</span> <span class="n">advantages</span><span class="p">).</span><span class="nf">mean</span><span class="p">()</span>
        <span class="n">constraint_loss</span> <span class="o">=</span> <span class="n">penalty_weight</span> <span class="o">*</span> <span class="n">constraint_penalty</span><span class="p">.</span><span class="nf">mean</span><span class="p">()</span>
        <span class="n">total_loss</span> <span class="o">=</span> <span class="n">policy_loss</span> <span class="o">+</span> <span class="n">constraint_loss</span>

        <span class="n">self</span><span class="p">.</span><span class="n">optimizer</span><span class="p">.</span><span class="nf">zero_grad</span><span class="p">()</span>
        <span class="n">total_loss</span><span class="p">.</span><span class="nf">backward</span><span class="p">()</span>
        <span class="n">torch</span><span class="p">.</span><span class="n">nn</span><span class="p">.</span><span class="n">utils</span><span class="p">.</span><span class="nf">clip_grad_norm_</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">agent</span><span class="p">.</span><span class="nf">parameters</span><span class="p">(),</span> <span class="mf">0.5</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">optimizer</span><span class="p">.</span><span class="nf">step</span><span class="p">()</span>
</code></pre>

</div>



<h3>
  
  
  Explainability Module
</h3>

<p>During my experimentation with explanation generation, I found that clinicians respond best to explanations that mirror clinical reasoning patterns. While exploring various explanation techniques, I developed a hybrid approach:</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
python
from typing import Dict, List, Any
import numpy as np
from dataclasses import dataclass

@dataclass
class ClinicalExplanation:
    recommendation: str
    confidence: float
    primary_reasoning: List[str]
    alternative_options: List[Dict[str, Any]]
    counterfactual_scenarios: List[Dict[str, Any]]
    evidence_level: str  # "strong", "moderate", "weak"
    uncertainty_acknowledgement: str

class CausalExplanationGenerator:
    def __init__(self, causal_graph: OncologyCausalGraph,
                 treatment_history: pd.DataFrame):
        self.causal_graph = causal_graph
        self.treatment_history = treatment_history
        self.explanation_templates = self._load_templates()

    def generate_explanation(self, current_state: Dict[str, float],
                            recommended_action: Dict[str, float],
                            alternative_actions: List[Dict[str, float]]) -&gt; ClinicalExplanation:
        """Generate clinical explanation for treatment recommendation"""

        # 1. Identify primary causal pathways
        primary_pathways = self._identify_causal_pathways(
            current_state, recommended_action
        )

        # 2. Estimate effect sizes
        effect_sizes = self._estimate_effects(
            current_state, recommended_action, primary_pathways
        )

        # 3. Generate counterfactuals
        counterfactuals = self._generate_counterfactuals(
            current_state, recommended_action, alternative_actions
        )

        # 4. Assess evidence strength
        evidence_strength = self._assess_evidence(
            primary_pathways, effect_sizes
        )

        # 5. Compose explanation
        explanation_text = self._compose_explanation(
            primary_pathways, effect_sizes, counterfactuals, evidence_strength
        )

        return ClinicalExplanation(
            recommendation=self._format_recommendation(recommended_action),
            confidence=self._calculate_confidence(effect_sizes),
            primary_reasoning=primary_pathways[:3],  # Top 3 pathways
            alternative_options=alternative_actions[:2],  # Top 2 alternatives
            counterfactual_scenarios=counterfactuals,
            evidence_level=evidence_strength,
            uncertainty_acknowledgement=self._acknowledge_uncertainties(
                effect_sizes, counterfactuals
            )
        )

    def _identify_causal_pathways(self, state: Dict, action: Dict) -&gt; List[str]:
        """Identify the most relevant causal pathways for this decision"""
        pathways = []

        # Analyze treatment -&gt; outcome pathways
        for treatment, dose in action.items():
            if dose &gt; 0:  # Active treatment
                # Find outcomes affected by this treatment
                for outcome in self.causal_graph.graph.nodes:
                    if outcome.startswith("progression") or outcome == "toxicity":
                        # Check if there's a causal path
                        try:
                            paths = list(nx.all_simple_paths(
                                self.causal_graph.graph,
                                source=treatment,
                                target=outcome,
                                cutoff=3  # Maximum path length
                            ))
                            if paths:
                                for path in paths[:2]:  # Take up to 2 shortest paths
                                    pathway_desc = self._describe_pathway(path)
                                    pathways.append(pathway_desc)
                        except:
                            continue

        # Sort by pathway strength (simplified)
        return sorted(pathways, key=lambda x: len(x), reverse=True)[:5]

    def _estimate_effects(self, state: Dict, action: Dict,
                         pathways: List[str]) -&gt; Dict[str, float]:
        """Estimate effect sizes for each causal pathway"""
        effects = {}

        # Simplified effect estimation
        # In practice, this would use the causal model's do-calculus
        for pathway in pathways:
            # Extract treatment and outcome from pathway description
            components = pathway.split(" → ")
            if len(components) &gt;= 2:
                treatment = components[0]
                outcome = components[-1]

                # Use the causal graph's effect estimation
                adjustment_set = self._identify_adjustment_set(treatment, outcome)
                effect_result = self.causal_graph.estimate_causal_effect(
                    treatment, outcome, adjustment_set
                )
                effects[pathway] = effect_result["causal_effect"]

        return effects

    def _generate_counterfactuals(self, state: Dict,
                                 chosen_action: Dict,
                                 alternatives: List[Dict]) -&gt; List[Dict]:
        """Generate counterfactual scenarios for alternative treatments"""
        counterfactuals = []

        for alt_action in alternatives[:3]:  # Consider top 3 alternatives
            scenario = {
                "alternative_action": alt_action,
                "expected_outcomes": {},
                "comparison_to_chosen": {},
                "risk_assessment": {}
            }

            # Estimate outcomes for alternative (simplified)
            for outcome in ["progression_free_survival", "toxicity"]:
                # This would use the causal model for proper counterfactual inference
                alt_outcome = self._estimate_counterfactual_outcome(
                    state, alt_action, outcome
                )
                chosen_outcome = self._estimate_counterfactual_outcome(
                    state, chosen_action, outcome
                )

                scenario["expected_outcomes"][outcome] = alt_outcome
                scenario["comparison_to_chosen"][outcome] = (
                    alt_outcome - chosen_outcome
                )

            counterfactuals.append(scenario)
</code></pre>

</div>

