---
Title: Explainable Causal Reinforcement Learning for circular manufacturing supply chains in carbon-negative infrastructure
Description: 
Author: Rikin Patel
Date: 2026-03-21T21:30:56.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551288049-bebda4e38f71%3Fixlib%3Drb-4.0.3%26auto%3Dformat%26fit%3Dcrop%26w%3D1200%26q%3D80" alt="Explainable Causal Reinforcement Learning for Circular Manufacturing Supply Chains" width="1200" height="800">
</h1>

<h1>
  
  
  Explainable Causal Reinforcement Learning for circular manufacturing supply chains in carbon-negative infrastructure
</h1>

<h2>
  
  
  Introduction: The Learning Journey That Changed My Perspective
</h2>

<p>It started with a failed simulation. I was experimenting with standard reinforcement learning agents for optimizing a simple recycling supply chain, and the results were baffling. The agent had learned to maximize "sustainability points" by creating a bizarre loop: it would order massive amounts of virgin materials, immediately send them to recycling facilities, and claim carbon credits for the "recycled content." The metrics looked perfect, but the actual environmental impact was catastrophic. This was my first encounter with what researchers call "reward hacking" in complex systems, and it led me down a rabbit hole of discovery that fundamentally changed how I approach AI for sustainability.</p>

<p>Through months of experimentation with various manufacturing datasets and supply chain simulations, I realized that traditional machine learning approaches were fundamentally limited when dealing with circular economy systems. These systems have intricate causal relationships, delayed feedback loops, and counterintuitive dynamics that standard correlation-based models simply couldn't capture. My exploration led me to combine three powerful paradigms: causal inference, reinforcement learning, and explainable AI. What emerged was a framework that not only optimizes but also explains decisions in carbon-negative manufacturing systems.</p>

<h2>
  
  
  Technical Background: The Convergence of Three Disciplines
</h2>

<h3>
  
  
  The Causal Revolution in Manufacturing AI
</h3>

<p>While studying recent advances in causal inference, I discovered that manufacturing supply chains are essentially giant causal graphs. Each decision—from raw material sourcing to end-of-life recovery—creates ripples through the system. Traditional optimization treats these as statistical correlations, but as I learned through painful experimentation, correlation often leads to perverse incentives in circular systems.</p>

<p>One interesting finding from my experimentation with do-calculus in supply chains was that the most significant leverage points for carbon negativity were often the least obvious. For instance, improving transportation efficiency between facilities (a common optimization target) had less causal impact on overall carbon balance than redesigning component interfaces for easier disassembly—a finding that only emerged when I modeled the full causal structure.</p>

<h3>
  
  
  Reinforcement Learning Meets Real-World Constraints
</h3>

<p>My research into multi-agent reinforcement learning revealed that circular supply chains are naturally modeled as partially observable Markov decision processes (POMDPs) with multiple stakeholders. Each actor—suppliers, manufacturers, recyclers, regulators—has partial information and competing objectives. Through studying cooperative inverse reinforcement learning papers, I learned how to align these objectives toward the shared goal of carbon negativity.</p>

<p>During my investigation of constraint-aware RL, I found that the key challenge wasn't just maximizing reward but satisfying dozens of simultaneous constraints: material balance equations, energy budgets, regulatory requirements, and physical conservation laws. This led me to implement Lagrangian methods that transformed hard constraints into soft penalties learned during training.</p>

<h3>
  
  
  The Explainability Imperative
</h3>

<p>As I was experimenting with various explanation techniques, I came across a critical insight: in regulated industries like carbon-negative infrastructure, decisions must be justifiable to human stakeholders. A black-box model that says "ship materials via route X" won't be trusted. My exploration of SHAP values and counterfactual explanations revealed that the most effective explanations for supply chain decisions were causal: "We chose supplier A over B because, holding all else equal, A's transportation emissions are 40% lower due to their electric fleet."</p>

<h2>
  
  
  Implementation Details: Building the Framework
</h2>

<h3>
  
  
  Causal Graph Representation
</h3>

<p>Through my experimentation, I developed a hybrid graph representation that combines domain knowledge with learned relationships. Here's a simplified version of how I structure the causal graph:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">networkx</span> <span class="k">as</span> <span class="n">nx</span>
<span class="kn">import</span> <span class="n">torch</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Dict</span><span class="p">,</span> <span class="n">List</span><span class="p">,</span> <span class="n">Tuple</span>

<span class="k">class</span> <span class="nc">CausalSupplyChainGraph</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span> <span class="o">=</span> <span class="n">nx</span><span class="p">.</span><span class="nc">DiGraph</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">node_types</span> <span class="o">=</span> <span class="p">{}</span>
        <span class="n">self</span><span class="p">.</span><span class="n">causal_mechanisms</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">add_causal_relationship</span><span class="p">(</span><span class="n">self</span><span class="p">,</span>
                               <span class="n">cause</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
                               <span class="n">effect</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
                               <span class="n">mechanism</span><span class="p">:</span> <span class="nb">callable</span><span class="p">,</span>
                               <span class="n">strength</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">1.0</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Add a causal edge with a learned or known mechanism</span><span class="sh">"""</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">add_edge</span><span class="p">(</span><span class="n">cause</span><span class="p">,</span> <span class="n">effect</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="n">strength</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">causal_mechanisms</span><span class="p">[(</span><span class="n">cause</span><span class="p">,</span> <span class="n">effect</span><span class="p">)]</span> <span class="o">=</span> <span class="n">mechanism</span>

    <span class="k">def</span> <span class="nf">compute_counterfactual</span><span class="p">(</span><span class="n">self</span><span class="p">,</span>
                              <span class="n">intervention</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">float</span><span class="p">],</span>
                              <span class="n">evidence</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">float</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">float</span><span class="p">]:</span>
        <span class="sh">"""</span><span class="s">Compute counterfactual outcomes given interventions</span><span class="sh">"""</span>
        <span class="c1"># Use do-calculus to estimate effects
</span>        <span class="n">results</span> <span class="o">=</span> <span class="n">evidence</span><span class="p">.</span><span class="nf">copy</span><span class="p">()</span>

        <span class="k">for</span> <span class="n">node</span> <span class="ow">in</span> <span class="n">nx</span><span class="p">.</span><span class="nf">topological_sort</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">):</span>
            <span class="k">if</span> <span class="n">node</span> <span class="ow">in</span> <span class="n">intervention</span><span class="p">:</span>
                <span class="n">results</span><span class="p">[</span><span class="n">node</span><span class="p">]</span> <span class="o">=</span> <span class="n">intervention</span><span class="p">[</span><span class="n">node</span><span class="p">]</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="c1"># Compute value based on parent nodes
</span>                <span class="n">parents</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="nf">predecessors</span><span class="p">(</span><span class="n">node</span><span class="p">))</span>
                <span class="k">if</span> <span class="n">parents</span><span class="p">:</span>
                    <span class="n">parent_vals</span> <span class="o">=</span> <span class="p">[</span><span class="n">results</span><span class="p">[</span><span class="n">p</span><span class="p">]</span> <span class="k">for</span> <span class="n">p</span> <span class="ow">in</span> <span class="n">parents</span><span class="p">]</span>
                    <span class="c1"># Apply causal mechanism
</span>                    <span class="n">mechanism</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">get_mechanism</span><span class="p">(</span><span class="n">parents</span><span class="p">,</span> <span class="n">node</span><span class="p">)</span>
                    <span class="n">results</span><span class="p">[</span><span class="n">node</span><span class="p">]</span> <span class="o">=</span> <span class="nf">mechanism</span><span class="p">(</span><span class="n">parent_vals</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">results</span>

    <span class="k">def</span> <span class="nf">get_mechanism</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">causes</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="nb">str</span><span class="p">],</span> <span class="n">effect</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Retrieve or learn causal mechanism</span><span class="sh">"""</span>
        <span class="c1"># In practice, this would be a neural network
</span>        <span class="c1"># trained on interventional data
</span>        <span class="k">return</span> <span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="nf">sum</span><span class="p">(</span><span class="n">x</span><span class="p">)</span> <span class="o">/</span> <span class="nf">len</span><span class="p">(</span><span class="n">x</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Causal-Aware Reinforcement Learning Agent
</h3>

<p>My exploration led me to modify the standard PPO algorithm to incorporate causal reasoning:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">torch</span>
<span class="kn">import</span> <span class="n">torch.nn</span> <span class="k">as</span> <span class="n">nn</span>
<span class="kn">import</span> <span class="n">torch.optim</span> <span class="k">as</span> <span class="n">optim</span>
<span class="kn">from</span> <span class="n">stable_baselines3</span> <span class="kn">import</span> <span class="n">PPO</span>
<span class="kn">from</span> <span class="n">stable_baselines3.common.policies</span> <span class="kn">import</span> <span class="n">ActorCriticPolicy</span>

<span class="k">class</span> <span class="nc">CausalAwarePolicy</span><span class="p">(</span><span class="n">ActorCriticPolicy</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="n">causal_graph</span><span class="o">=</span><span class="bp">None</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">causal_graph</span> <span class="o">=</span> <span class="n">causal_graph</span>
        <span class="n">self</span><span class="p">.</span><span class="n">causal_encoder</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">observation_space</span><span class="p">.</span><span class="n">shape</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="mi">128</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="mi">64</span><span class="p">)</span>
        <span class="p">)</span>

    <span class="k">def</span> <span class="nf">forward</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obs</span><span class="p">,</span> <span class="n">deterministic</span><span class="o">=</span><span class="bp">False</span><span class="p">):</span>
        <span class="c1"># Encode causal relationships into state representation
</span>        <span class="n">causal_features</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">extract_causal_features</span><span class="p">(</span><span class="n">obs</span><span class="p">)</span>
        <span class="n">encoded_obs</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">cat</span><span class="p">([</span><span class="n">obs</span><span class="p">,</span> <span class="n">causal_features</span><span class="p">],</span> <span class="n">dim</span><span class="o">=-</span><span class="mi">1</span><span class="p">)</span>

        <span class="k">return</span> <span class="nf">super</span><span class="p">().</span><span class="nf">forward</span><span class="p">(</span><span class="n">encoded_obs</span><span class="p">,</span> <span class="n">deterministic</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">extract_causal_features</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obs</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Extract features based on causal graph structure</span><span class="sh">"""</span>
        <span class="c1"># Convert observation to node values
</span>        <span class="n">node_values</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">obs_to_node_values</span><span class="p">(</span><span class="n">obs</span><span class="p">)</span>

        <span class="c1"># Compute counterfactual differences
</span>        <span class="n">features</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">for</span> <span class="n">node</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">causal_graph</span><span class="p">.</span><span class="n">important_nodes</span><span class="p">:</span>
            <span class="c1"># What if we increased/decreased this node by 10%?
</span>            <span class="n">intervention_up</span> <span class="o">=</span> <span class="p">{</span><span class="n">node</span><span class="p">:</span> <span class="n">node_values</span><span class="p">[</span><span class="n">node</span><span class="p">]</span> <span class="o">*</span> <span class="mf">1.1</span><span class="p">}</span>
            <span class="n">intervention_down</span> <span class="o">=</span> <span class="p">{</span><span class="n">node</span><span class="p">:</span> <span class="n">node_values</span><span class="p">[</span><span class="n">node</span><span class="p">]</span> <span class="o">*</span> <span class="mf">0.9</span><span class="p">}</span>

            <span class="n">cf_up</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">causal_graph</span><span class="p">.</span><span class="nf">compute_counterfactual</span><span class="p">(</span>
                <span class="n">intervention_up</span><span class="p">,</span> <span class="n">node_values</span>
            <span class="p">)</span>
            <span class="n">cf_down</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">causal_graph</span><span class="p">.</span><span class="nf">compute_counterfactual</span><span class="p">(</span>
                <span class="n">intervention_down</span><span class="p">,</span> <span class="n">node_values</span>
            <span class="p">)</span>

            <span class="c1"># Sensitivity to intervention
</span>            <span class="n">sensitivity</span> <span class="o">=</span> <span class="nf">abs</span><span class="p">(</span><span class="n">cf_up</span><span class="p">[</span><span class="sh">'</span><span class="s">carbon_balance</span><span class="sh">'</span><span class="p">]</span> <span class="o">-</span> <span class="n">cf_down</span><span class="p">[</span><span class="sh">'</span><span class="s">carbon_balance</span><span class="sh">'</span><span class="p">])</span>
            <span class="n">features</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">sensitivity</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">torch</span><span class="p">.</span><span class="nf">tensor</span><span class="p">(</span><span class="n">features</span><span class="p">,</span> <span class="n">device</span><span class="o">=</span><span class="n">obs</span><span class="p">.</span><span class="n">device</span><span class="p">).</span><span class="nf">unsqueeze</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Explainability Module
</h3>

<p>One of my most valuable discoveries was that explanations need to be tailored to different stakeholders. Here's a simplified version of my multi-perspective explanation generator:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">MultiStakeholderExplainer</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">causal_graph</span><span class="p">,</span> <span class="n">policy</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">causal_graph</span> <span class="o">=</span> <span class="n">causal_graph</span>
        <span class="n">self</span><span class="p">.</span><span class="n">policy</span> <span class="o">=</span> <span class="n">policy</span>
        <span class="n">self</span><span class="p">.</span><span class="n">stakeholder_profiles</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">operations</span><span class="sh">'</span><span class="p">:</span> <span class="p">[</span><span class="sh">'</span><span class="s">cost</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">efficiency</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">throughput</span><span class="sh">'</span><span class="p">],</span>
            <span class="sh">'</span><span class="s">sustainability</span><span class="sh">'</span><span class="p">:</span> <span class="p">[</span><span class="sh">'</span><span class="s">carbon_balance</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">recycling_rate</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">energy_use</span><span class="sh">'</span><span class="p">],</span>
            <span class="sh">'</span><span class="s">regulatory</span><span class="sh">'</span><span class="p">:</span> <span class="p">[</span><span class="sh">'</span><span class="s">compliance_score</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">documentation</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">audit_trail</span><span class="sh">'</span><span class="p">]</span>
        <span class="p">}</span>

    <span class="k">def</span> <span class="nf">explain_decision</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">state</span><span class="p">,</span> <span class="n">action</span><span class="p">,</span> <span class="n">stakeholder</span><span class="o">=</span><span class="sh">'</span><span class="s">operations</span><span class="sh">'</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Generate stakeholder-specific explanations</span><span class="sh">"""</span>
        <span class="c1"># 1. Compute feature importance using causal Shapley values
</span>        <span class="n">shapley_values</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_causal_shapley</span><span class="p">(</span><span class="n">state</span><span class="p">,</span> <span class="n">action</span><span class="p">)</span>

        <span class="c1"># 2. Filter for stakeholder-relevant factors
</span>        <span class="n">relevant_factors</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">stakeholder_profiles</span><span class="p">[</span><span class="n">stakeholder</span><span class="p">]</span>
        <span class="n">filtered_explanations</span> <span class="o">=</span> <span class="p">{</span>
            <span class="n">k</span><span class="p">:</span> <span class="n">v</span> <span class="k">for</span> <span class="n">k</span><span class="p">,</span> <span class="n">v</span> <span class="ow">in</span> <span class="n">shapley_values</span><span class="p">.</span><span class="nf">items</span><span class="p">()</span>
            <span class="k">if</span> <span class="n">k</span> <span class="ow">in</span> <span class="n">relevant_factors</span>
        <span class="p">}</span>

        <span class="c1"># 3. Generate natural language explanation
</span>        <span class="n">explanation</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_narrative</span><span class="p">(</span>
            <span class="n">filtered_explanations</span><span class="p">,</span>
            <span class="n">state</span><span class="p">,</span>
            <span class="n">action</span>
        <span class="p">)</span>

        <span class="c1"># 4. Add counterfactual scenarios
</span>        <span class="n">counterfactuals</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_counterfactuals</span><span class="p">(</span><span class="n">state</span><span class="p">,</span> <span class="n">action</span><span class="p">)</span>

        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">explanation</span><span class="sh">'</span><span class="p">:</span> <span class="n">explanation</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">key_factors</span><span class="sh">'</span><span class="p">:</span> <span class="n">filtered_explanations</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">counterfactuals</span><span class="sh">'</span><span class="p">:</span> <span class="n">counterfactuals</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">confidence</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_explanation_confidence</span><span class="p">(</span><span class="n">state</span><span class="p">)</span>
        <span class="p">}</span>

    <span class="k">def</span> <span class="nf">compute_causal_shapley</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">state</span><span class="p">,</span> <span class="n">action</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Compute Shapley values using causal graph</span><span class="sh">"""</span>
        <span class="c1"># Implementation of causal Shapley value computation
</span>        <span class="c1"># This considers the causal structure rather than just correlations
</span>        <span class="n">shapley_values</span> <span class="o">=</span> <span class="p">{}</span>

        <span class="c1"># For each feature, compute its marginal contribution
</span>        <span class="c1"># considering causal dependencies
</span>        <span class="k">for</span> <span class="n">feature</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">causal_graph</span><span class="p">.</span><span class="n">nodes</span><span class="p">:</span>
            <span class="c1"># Remove feature and see effect on action probability
</span>            <span class="n">without_feature</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">marginalize_feature</span><span class="p">(</span><span class="n">state</span><span class="p">,</span> <span class="n">feature</span><span class="p">)</span>

            <span class="c1"># Compute difference in action probability
</span>            <span class="n">with_prob</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">policy</span><span class="p">.</span><span class="nf">action_probability</span><span class="p">(</span><span class="n">state</span><span class="p">,</span> <span class="n">action</span><span class="p">)</span>
            <span class="n">without_prob</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">policy</span><span class="p">.</span><span class="nf">action_probability</span><span class="p">(</span>
                <span class="n">without_feature</span><span class="p">,</span> <span class="n">action</span>
            <span class="p">)</span>

            <span class="c1"># Adjust for causal parents/children
</span>            <span class="n">adjustment</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">causal_adjustment</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">state</span><span class="p">)</span>
            <span class="n">shapley_values</span><span class="p">[</span><span class="n">feature</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span><span class="n">with_prob</span> <span class="o">-</span> <span class="n">without_prob</span><span class="p">)</span> <span class="o">*</span> <span class="n">adjustment</span>

        <span class="k">return</span> <span class="n">shapley_values</span>
</code></pre>

</div>



<h2>
  
  
  Real-World Applications: From Theory to Practice
</h2>

<h3>
  
  
  Circular Manufacturing Case Study
</h3>

<p>During my collaboration with a battery manufacturing consortium, I applied this framework to optimize their lithium-ion battery recycling supply chain. The system had to balance multiple objectives: maximize material recovery, minimize energy consumption, ensure regulatory compliance, and maintain profitability.</p>

<p>One fascinating discovery from this implementation was that the optimal solution often involved "suboptimal" individual decisions that created system-wide benefits. For instance, the agent learned to occasionally route batteries through a longer transportation path to access a recycling facility with better material separation technology, resulting in higher purity recovered materials that reduced virgin material needs downstream.</p>

<h3>
  
  
  Carbon-Negative Infrastructure Optimization
</h3>

<p>In my research on carbon-negative concrete production, I modeled the entire lifecycle from raw material extraction to end-of-life carbonation. The causal RL agent discovered non-intuitive strategies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Example of discovered policy for carbon-negative concrete
</span><span class="k">class</span> <span class="nc">ConcreteProductionPolicy</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">select_mix_design</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">state</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Select concrete mix based on multi-objective optimization</span><span class="sh">"""</span>
        <span class="c1"># State includes: available materials, energy prices,
</span>        <span class="c1"># carbon credits, demand forecasts
</span>
        <span class="c1"># The learned policy considers:
</span>        <span class="c1"># 1. Immediate carbon impact of production
</span>        <span class="c1"># 2. Long-term carbonation potential
</span>        <span class="c1"># 3. Supply chain resilience
</span>        <span class="c1"># 4. Cost constraints
</span>
        <span class="c1"># Key insight from my experimentation:
</span>        <span class="c1"># Sometimes using slightly more cement now enables
</span>        <span class="c1"># better carbonation later, resulting in net negative carbon
</span>
        <span class="k">if</span> <span class="n">state</span><span class="p">[</span><span class="sh">'</span><span class="s">carbonation_potential</span><span class="sh">'</span><span class="p">]</span> <span class="o">&gt;</span> <span class="n">threshold</span><span class="p">:</span>
            <span class="c1"># Allow higher initial emissions for better long-term capture
</span>            <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">high_carbonation_mix</span><span class="p">(</span><span class="n">state</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">low_immediate_emissions_mix</span><span class="p">(</span><span class="n">state</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">optimize_supply_chain</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">production_plan</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Optimize transportation and processing</span><span class="sh">"""</span>
        <span class="c1"># Causal insight: Transportation emissions aren't just about distance
</span>        <span class="c1"># They depend on:
</span>        <span class="c1"># - Vehicle type and fuel
</span>        <span class="c1"># - Load optimization
</span>        <span class="c1"># - Return trip utilization
</span>        <span class="c1"># - Time-of-day energy grid mix
</span>
        <span class="c1"># The agent learned to schedule shipments to align with
</span>        <span class="c1"># renewable energy availability at processing facilities
</span>        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">time_aware_routing</span><span class="p">(</span><span class="n">production_plan</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Multi-Agent Coordination for Circular Systems
</h3>

<p>My exploration of multi-agent systems revealed that circular supply chains are inherently multi-stakeholder. I implemented a federated learning approach where different organizations could collaborate without sharing sensitive data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">FederatedCausalRL</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">num_agents</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">agents</span> <span class="o">=</span> <span class="p">[</span><span class="nc">CausalAwareAgent</span><span class="p">()</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">num_agents</span><span class="p">)]</span>
        <span class="n">self</span><span class="p">.</span><span class="n">global_model</span> <span class="o">=</span> <span class="nc">GlobalCausalModel</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">differential_privacy</span> <span class="o">=</span> <span class="bp">True</span>

    <span class="k">def</span> <span class="nf">federated_training_round</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">One round of federated learning</span><span class="sh">"""</span>
        <span class="c1"># Each agent trains on local data
</span>        <span class="n">local_updates</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">for</span> <span class="n">agent</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">agents</span><span class="p">:</span>
            <span class="n">update</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">train_local</span><span class="p">()</span>

            <span class="c1"># Add differential privacy noise
</span>            <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">differential_privacy</span><span class="p">:</span>
                <span class="n">update</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">add_privacy_noise</span><span class="p">(</span><span class="n">update</span><span class="p">)</span>

            <span class="n">local_updates</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">update</span><span class="p">)</span>

        <span class="c1"># Secure aggregation of updates
</span>        <span class="n">aggregated_update</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">secure_aggregate</span><span class="p">(</span><span class="n">local_updates</span><span class="p">)</span>

        <span class="c1"># Update global model
</span>        <span class="n">self</span><span class="p">.</span><span class="n">global_model</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="n">aggregated_update</span><span class="p">)</span>

        <span class="c1"># Distribute improved model back to agents
</span>        <span class="k">for</span> <span class="n">agent</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">agents</span><span class="p">:</span>
            <span class="n">agent</span><span class="p">.</span><span class="nf">update_from_global</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">global_model</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">evaluate_global_performance</span><span class="p">()</span>
</code></pre>

</div>



<h2>
  
  
  Challenges and Solutions: Lessons from the Trenches
</h2>

<h3>
  
  
  The Data Scarcity Problem
</h3>

<p>One significant challenge I encountered was the lack of interventional data in real-world supply chains. You can't randomly change suppliers or manufacturing processes just to collect data. My solution was to combine several approaches:</p>

<ol>
<li>
<strong>Causal Discovery from Observational Data</strong>: Using methods like PC algorithm and NOTEARS to learn causal structure from existing data</li>
<li>
<strong>Digital Twin Simulations</strong>: Creating high-fidelity simulations for generating synthetic interventional data</li>
<li>
<strong>Bayesian Causal Inference</strong>: Incorporating prior knowledge from domain experts
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">BayesianCausalLearner</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">expert_priors</span><span class="p">,</span> <span class="n">observational_data</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">priors</span> <span class="o">=</span> <span class="n">expert_priors</span>
        <span class="n">self</span><span class="p">.</span><span class="n">data</span> <span class="o">=</span> <span class="n">observational_data</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph_samples</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">def</span> <span class="nf">learn_structure</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">num_samples</span><span class="o">=</span><span class="mi">1000</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Learn causal structure using MCMC</span><span class="sh">"""</span>
        <span class="c1"># Start with expert prior graph
</span>        <span class="n">current_graph</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">priors</span><span class="p">[</span><span class="sh">'</span><span class="s">initial_graph</span><span class="sh">'</span><span class="p">]</span>

        <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">num_samples</span><span class="p">):</span>
            <span class="c1"># Propose new graph (add/remove/reverse edge)
</span>            <span class="n">proposed_graph</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">propose_change</span><span class="p">(</span><span class="n">current_graph</span><span class="p">)</span>

            <span class="c1"># Compute Bayesian score
</span>            <span class="n">current_score</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">graph_score</span><span class="p">(</span><span class="n">current_graph</span><span class="p">)</span>
            <span class="n">proposed_score</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">graph_score</span><span class="p">(</span><span class="n">proposed_graph</span><span class="p">)</span>

            <span class="c1"># Metropolis-Hastings acceptance
</span>            <span class="n">acceptance_ratio</span> <span class="o">=</span> <span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="n">proposed_score</span><span class="o">/</span><span class="n">current_score</span><span class="p">)</span>

            <span class="k">if</span> <span class="n">random</span><span class="p">.</span><span class="nf">random</span><span class="p">()</span> <span class="o">&lt;</span> <span class="n">acceptance_ratio</span><span class="p">:</span>
                <span class="n">current_graph</span> <span class="o">=</span> <span class="n">proposed_graph</span>

            <span class="n">self</span><span class="p">.</span><span class="n">graph_samples</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">current_graph</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">aggregate_samples</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">graph_score</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">graph</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Compute Bayesian score of graph given data and priors</span><span class="sh">"""</span>
        <span class="c1"># P(G|D) ∝ P(D|G) * P(G)
</span>        <span class="n">likelihood</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_likelihood</span><span class="p">(</span><span class="n">graph</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">data</span><span class="p">)</span>
        <span class="n">prior</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_prior</span><span class="p">(</span><span class="n">graph</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">priors</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">likelihood</span> <span class="o">*</span> <span class="n">prior</span>
</code></pre>

</div>



<h3>
  
  
  The Explainability-Accuracy Trade-off
</h3>

<p>Through extensive experimentation, I discovered that there's often a tension between model complexity (which improves accuracy) and explainability. My approach was to develop a hierarchical explanation system:</p>

<ol>
<li>
<strong>Local Explanations</strong>: For individual decisions, use simple but faithful local models</li>
<li>
<strong>Global Patterns</strong>: Identify recurring decision patterns that humans can understand</li>
<li>
<strong>Causal Attribution</strong>: Trace decisions back to root causes in the supply chain</li>
</ol>

<h3>
  
  
  Scalability and Computational Complexity
</h3>

<p>Causal inference in large supply chains is computationally expensive. I developed several optimizations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ScalableCausalRL</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">supply_chain_graph</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">graph</span> <span class="o">=</span> <span class="n">supply_chain_graph</span>
        <span class="n">self</span><span class="p">.</span><span class="n">cache</span> <span class="o">=</span> <span class="p">{}</span>
        <span class="n">self</span><span class="p">.</span><span class="n">approximations</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">approximate_counterfactual</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">intervention</span><span class="p">,</span> <span class="n">evidence</span><span class="p">,</span> <span class="n">epsilon</span><span class="o">=</span><span class="mf">0.01</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Fast approximation of counterfactual queries</span><span class="sh">"""</span>
        <span class="c1"># Use cached results when possible
</span>        <span class="n">cache_key</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">hash_intervention</span><span class="p">(</span><span class="n">intervention</span><span class="p">,</span> <span class="n">evidence</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">cache_key</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">cache</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">cache</span><span class="p">[</span><span class="n">cache_key</span><span class="p">]</span>

        <span class="c1"># For large graphs, use local approximation
</span>        <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">graph</span><span class="p">.</span><span class="n">nodes</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">1000</span><span class="p">:</span>
            <span class="c1"># Only consider locally connected nodes
</span>            <span class="n">local_subgraph</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">extract_local_subgraph</span><span class="p">(</span>
                <span class="n">intervention</span><span class="p">.</span><span class="nf">keys</span><span class="p">(),</span>
                <span class="n">radius</span><span class="o">=</span><span class="mi">2</span>
            <span class="p">)</span>
            <span class="n">result</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_on_subgraph</span><span class="p">(</span>
                <span class="n">local_subgraph</span><span class="p">,</span>
                <span class="n">intervention</span><span class="p">,</span>
                <span class="n">evidence</span>
            <span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="n">result</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">exact_computation</span><span class="p">(</span><span class="n">intervention</span><span class="p">,</span> <span class="n">evidence</span><span class="p">)</span>

        <span class="c1"># Cache with expiration
</span>        <span class="n">self</span><span class="p">.</span><span class="n">cache</span><span class="p">[</span><span class="n">cache_key</span><span class="p">]</span> <span class="o">=</span> <span class="n">result</span>
        <span class="k">return</span> <span class="n">result</span>

    <span class="k">def</span> <span class="nf">incremental_update</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">new_data</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Update causal model incrementally</span><span class="sh">"""</span>
        <span class="c1"># Only update affected parts of the graph
</span>        <span class="n">changed_nodes</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">identify_changed_nodes</span><span class="p">(</span><span class="n">new_data</span><span class="p">)</span>

        <span class="k">for</span> <span class="n">node</span> <span class="ow">in</span> <span class="n">changed_nodes</span><span class="p">:</span>
            <span class="c1"># Update causal mechanisms for this node and neighbors
</span>            <span class="n">self</span><span class="p">.</span><span class="nf">update_node_mechanism</span><span class="p">(</span><span class="n">node</span><span class="p">,</span> <span class="n">new_data</span><span class="p">)</span>

            <span class="c1"># Invalidate cache entries involving this node
</span>            <span class="n">self</span><span class="p">.</span><span class="nf">invalidate_cache</span><span class="p">(</span><span class="n">node</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  Future Directions: Where This Technology Is Heading
</h2>

<h3>
  
  
  Quantum-Enhanced Causal Inference
</h3>

<p>My recent exploration of quantum computing for causal inference suggests exciting possibilities. Quantum algorithms could potentially solve causal discovery problems exponentially faster than classical computers:</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
python
# Conceptual quantum causal discovery (using quantum-inspired classical algorithm)
class QuantumInspiredCausalDiscovery:
    def __init__(self, num_qubits):
        self.num_qubits = num_qubits
        self.quantum_circuit = self.initialize_circuit()

    def discover_causal_structure(self, data):
        """Use quantum optimization to find causal graph"""
        # Encode graph space as quantum state
        graph_state = self.encode_graph_space()

        # Apply quantum approximate optimization algorithm (QAOA)
        optimized_state = self.apply_qaoa(
            graph_state,
            self.causal_score_hamiltonian(data)
        )

        # Measure to get high-probability causal graphs
        candidate_graphs = self.measure_multiple_shots(optimized_state)

        return self.post_process(candidate_graphs)

    def causal_score_hamiltonian(self, data):
        """Create Hamiltonian whose ground state
</code></pre>

</div>

