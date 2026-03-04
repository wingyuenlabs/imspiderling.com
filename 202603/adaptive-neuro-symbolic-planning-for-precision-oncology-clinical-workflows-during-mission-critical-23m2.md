---
Title: Adaptive Neuro-Symbolic Planning for precision oncology clinical workflows during mission-critical recovery windows
Description: 
Author: Rikin Patel
Date: 2026-03-04T21:40:09.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1559757148-5c350d0d3c56%3Fixlib%3Drb-4.0.3%26auto%3Dformat%26fit%3Dcrop%26w%3D1200%26q%3D80" alt="Adaptive Neuro-Symbolic Planning for Precision Oncology" width="1200" height="675">
</h1>

<h1>
  
  
  Adaptive Neuro-Symbolic Planning for precision oncology clinical workflows during mission-critical recovery windows
</h1>

<h2>
  
  
  Introduction: A Learning Journey at the Intersection of AI and Medicine
</h2>

<p>My journey into this specialized field began not in a hospital, but in a research lab where I was experimenting with reinforcement learning for robotic path planning. While exploring multi-agent systems for warehouse automation, I stumbled upon a paper about treatment scheduling in oncology that fundamentally changed my perspective. The parallels were striking: both involved optimizing sequences of actions under uncertainty with critical timing constraints. However, the stakes in oncology were incomparably higher—human lives hung in the balance during what clinicians call "mission-critical recovery windows."</p>

<p>Through studying clinical oncology workflows, I realized that current AI approaches were treating cancer treatment as either a pure classification problem (which drug works?) or a pure optimization problem (which schedule is optimal?). Neither captured the complex interplay between symbolic clinical reasoning (protocols, guidelines, logical constraints) and sub-symbolic pattern recognition (tumor response patterns, toxicity predictions, biomarker correlations).</p>

<p>One interesting finding from my experimentation with hybrid AI systems was that purely neural approaches could suggest biologically plausible but clinically impossible treatment sequences, while purely symbolic systems couldn't adapt to the unique complexities of individual patient responses. This realization led me down a year-long exploration of neuro-symbolic AI, where I discovered that the most promising path forward lay in adaptive planning systems that could reason both statistically and logically about cancer treatment.</p>

<h2>
  
  
  Technical Background: The Dual Nature of Clinical Decision-Making
</h2>

<p>Precision oncology represents one of the most challenging domains for AI systems. During my investigation of clinical workflows, I found that oncologists operate in two complementary cognitive modes:</p>

<ol>
<li>
<strong>Symbolic reasoning</strong>: Applying clinical guidelines, checking contraindications, following protocol logic</li>
<li>
<strong>Sub-symbolic pattern recognition</strong>: Interpreting imaging results, recognizing subtle response patterns, predicting toxicity from complex biomarker combinations</li>
</ol>

<p>Traditional AI approaches have struggled with this duality. While exploring deep learning for medical applications, I discovered that neural networks excel at the pattern recognition aspects but lack explainability and struggle with logical constraints. Conversely, expert systems handle the symbolic reasoning well but can't learn from data or adapt to novel situations.</p>

<h3>
  
  
  The Mission-Critical Recovery Window Concept
</h3>

<p>Through studying clinical oncology literature, I learned that "mission-critical recovery windows" refer to specific time periods where:</p>

<ul>
<li>Treatment decisions must be made within narrow time constraints (often 24-72 hours)</li>
<li>Multiple interdependent decisions must be coordinated (imaging, labs, pharmacy, administration)</li>
<li>Patient physiology is in a particular state (e.g., neutrophil recovery, organ function thresholds)</li>
<li>The consequences of suboptimal timing are severe (reduced efficacy, increased toxicity, missed opportunities)</li>
</ul>

<p>My exploration of these windows revealed they're not fixed intervals but adaptive temporal constructs that depend on individual patient factors, treatment history, and institutional constraints.</p>

<h2>
  
  
  Implementation Details: Building an Adaptive Neuro-Symbolic Planner
</h2>

<h3>
  
  
  Core Architecture Design
</h3>

<p>During my experimentation with hybrid architectures, I developed a three-layer system that combines neural perception with symbolic reasoning:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">torch</span>
<span class="kn">import</span> <span class="n">torch.nn</span> <span class="k">as</span> <span class="n">nn</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Dict</span><span class="p">,</span> <span class="n">List</span><span class="p">,</span> <span class="n">Tuple</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">from</span> <span class="n">dataclasses</span> <span class="kn">import</span> <span class="n">dataclass</span>
<span class="kn">from</span> <span class="n">enum</span> <span class="kn">import</span> <span class="n">Enum</span>

<span class="k">class</span> <span class="nc">ClinicalState</span><span class="p">(</span><span class="n">Enum</span><span class="p">):</span>
    <span class="n">PRE_TREATMENT</span> <span class="o">=</span> <span class="sh">"</span><span class="s">pre_treatment</span><span class="sh">"</span>
    <span class="n">ACTIVE_TREATMENT</span> <span class="o">=</span> <span class="sh">"</span><span class="s">active_treatment</span><span class="sh">"</span>
    <span class="n">RECOVERY_WINDOW</span> <span class="o">=</span> <span class="sh">"</span><span class="s">recovery_window</span><span class="sh">"</span>
    <span class="n">TOXICITY_MANAGEMENT</span> <span class="o">=</span> <span class="sh">"</span><span class="s">toxicity_management</span><span class="sh">"</span>
    <span class="n">TREATMENT_HOLD</span> <span class="o">=</span> <span class="sh">"</span><span class="s">treatment_hold</span><span class="sh">"</span>

<span class="nd">@dataclass</span>
<span class="k">class</span> <span class="nc">PatientState</span><span class="p">:</span>
    <span class="n">biomarkers</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">float</span><span class="p">]</span>
    <span class="n">toxicity_scores</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">float</span><span class="p">]</span>
    <span class="n">treatment_history</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="n">Dict</span><span class="p">]</span>
    <span class="n">time_since_last_treatment</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">clinical_state</span><span class="p">:</span> <span class="n">ClinicalState</span>
    <span class="n">recovery_milestones</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">bool</span><span class="p">]</span>

<span class="k">class</span> <span class="nc">NeuralPerceptionModule</span><span class="p">(</span><span class="n">nn</span><span class="p">.</span><span class="n">Module</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Learns to extract clinical patterns from multimodal data</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">input_dim</span><span class="p">:</span> <span class="nb">int</span><span class="p">,</span> <span class="n">hidden_dim</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">256</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">encoder</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">input_dim</span><span class="p">,</span> <span class="n">hidden_dim</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Dropout</span><span class="p">(</span><span class="mf">0.3</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">hidden_dim</span><span class="p">,</span> <span class="n">hidden_dim</span> <span class="o">//</span> <span class="mi">2</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="n">hidden_dim</span> <span class="o">//</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">128</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># Multi-head attention for temporal patterns
</span>        <span class="n">self</span><span class="p">.</span><span class="n">temporal_attention</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">MultiheadAttention</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="n">num_heads</span><span class="o">=</span><span class="mi">8</span><span class="p">,</span> <span class="n">dropout</span><span class="o">=</span><span class="mf">0.1</span><span class="p">)</span>

        <span class="c1"># Toxicity prediction heads
</span>        <span class="n">self</span><span class="p">.</span><span class="n">toxicity_predictor</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">128</span><span class="p">,</span> <span class="mi">10</span><span class="p">)</span>  <span class="c1"># 10 common toxicities
</span>
    <span class="k">def</span> <span class="nf">forward</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">patient_data</span><span class="p">:</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span><span class="p">,</span> <span class="n">temporal_mask</span><span class="p">:</span> <span class="n">torch</span><span class="p">.</span><span class="n">Tensor</span> <span class="o">=</span> <span class="bp">None</span><span class="p">):</span>
        <span class="n">encoded</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">encoder</span><span class="p">(</span><span class="n">patient_data</span><span class="p">)</span>

        <span class="c1"># Reshape for temporal attention if sequence data
</span>        <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">encoded</span><span class="p">.</span><span class="n">shape</span><span class="p">)</span> <span class="o">==</span> <span class="mi">2</span><span class="p">:</span>
            <span class="n">encoded</span> <span class="o">=</span> <span class="n">encoded</span><span class="p">.</span><span class="nf">unsqueeze</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>

        <span class="n">attended</span><span class="p">,</span> <span class="n">_</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">temporal_attention</span><span class="p">(</span><span class="n">encoded</span><span class="p">,</span> <span class="n">encoded</span><span class="p">,</span> <span class="n">encoded</span><span class="p">,</span>
                                            <span class="n">key_padding_mask</span><span class="o">=</span><span class="n">temporal_mask</span><span class="p">)</span>

        <span class="n">toxicity_probs</span> <span class="o">=</span> <span class="n">torch</span><span class="p">.</span><span class="nf">sigmoid</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="nf">toxicity_predictor</span><span class="p">(</span><span class="n">attended</span><span class="p">.</span><span class="nf">mean</span><span class="p">(</span><span class="n">dim</span><span class="o">=</span><span class="mi">1</span><span class="p">)))</span>
        <span class="k">return</span> <span class="n">attended</span><span class="p">,</span> <span class="n">toxicity_probs</span>
</code></pre>

</div>



<h3>
  
  
  Symbolic Reasoning Layer
</h3>

<p>One of my key discoveries while building this system was that clinical guidelines could be represented as temporal logic formulas. Through studying formal methods, I learned to encode clinical protocols as constraint satisfaction problems:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">z3</span> <span class="kn">import</span> <span class="n">Solver</span><span class="p">,</span> <span class="n">Real</span><span class="p">,</span> <span class="n">Bool</span><span class="p">,</span> <span class="n">And</span><span class="p">,</span> <span class="n">Or</span><span class="p">,</span> <span class="n">Not</span><span class="p">,</span> <span class="n">Implies</span><span class="p">,</span> <span class="n">sat</span>
<span class="kn">import</span> <span class="n">datetime</span>

<span class="k">class</span> <span class="nc">ClinicalConstraintSolver</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Symbolic reasoning layer encoding clinical guidelines as constraints</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">solver</span> <span class="o">=</span> <span class="nc">Solver</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">constraints</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">self</span><span class="p">.</span><span class="n">variables</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">add_recovery_constraints</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">patient_state</span><span class="p">:</span> <span class="n">PatientState</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Encode recovery window constraints based on clinical guidelines</span><span class="sh">"""</span>

        <span class="c1"># Define symbolic variables
</span>        <span class="n">t_next_treatment</span> <span class="o">=</span> <span class="nc">Real</span><span class="p">(</span><span class="sh">'</span><span class="s">t_next_treatment</span><span class="sh">'</span><span class="p">)</span>
        <span class="n">neutrophil_count</span> <span class="o">=</span> <span class="nc">Real</span><span class="p">(</span><span class="sh">'</span><span class="s">neutrophil_count</span><span class="sh">'</span><span class="p">)</span>
        <span class="n">platelet_count</span> <span class="o">=</span> <span class="nc">Real</span><span class="p">(</span><span class="sh">'</span><span class="s">platelet_count</span><span class="sh">'</span><span class="p">)</span>
        <span class="n">organ_function_ok</span> <span class="o">=</span> <span class="nc">Bool</span><span class="p">(</span><span class="sh">'</span><span class="s">organ_function_ok</span><span class="sh">'</span><span class="p">)</span>

        <span class="c1"># Standard recovery constraints (based on NCCN guidelines)
</span>        <span class="n">self</span><span class="p">.</span><span class="n">constraints</span><span class="p">.</span><span class="nf">extend</span><span class="p">([</span>
            <span class="n">neutrophil_count</span> <span class="o">&gt;=</span> <span class="mf">1.5</span><span class="p">,</span>  <span class="c1"># ANC ≥ 1500/mm³
</span>            <span class="n">platelet_count</span> <span class="o">&gt;=</span> <span class="mi">100</span><span class="p">,</span>    <span class="c1"># Platelets ≥ 100,000/mm³
</span>            <span class="n">organ_function_ok</span> <span class="o">==</span> <span class="bp">True</span><span class="p">,</span>

            <span class="c1"># Time-based constraints for specific agents
</span>            <span class="nc">Implies</span><span class="p">(</span>
                <span class="n">patient_state</span><span class="p">.</span><span class="n">treatment_history</span><span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">][</span><span class="sh">'</span><span class="s">agent</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">cisplatin</span><span class="sh">'</span><span class="p">,</span>
                <span class="n">t_next_treatment</span> <span class="o">&gt;=</span> <span class="n">patient_state</span><span class="p">.</span><span class="n">time_since_last_treatment</span> <span class="o">+</span> <span class="mi">21</span>  <span class="c1"># 3-week cycle
</span>            <span class="p">),</span>

            <span class="c1"># Sequential therapy constraints
</span>            <span class="nc">Implies</span><span class="p">(</span>
                <span class="n">patient_state</span><span class="p">.</span><span class="n">clinical_state</span> <span class="o">==</span> <span class="n">ClinicalState</span><span class="p">.</span><span class="n">RECOVERY_WINDOW</span><span class="p">,</span>
                <span class="nc">And</span><span class="p">([</span>
                    <span class="n">self</span><span class="p">.</span><span class="nf">check_recovery_milestones</span><span class="p">(</span><span class="n">patient_state</span><span class="p">),</span>
                    <span class="n">self</span><span class="p">.</span><span class="nf">validate_timing_constraints</span><span class="p">(</span><span class="n">patient_state</span><span class="p">)</span>
                <span class="p">])</span>
            <span class="p">)</span>
        <span class="p">])</span>

    <span class="k">def</span> <span class="nf">solve_optimal_timing</span><span class="p">(</span><span class="n">self</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Find optimal treatment timing satisfying all constraints</span><span class="sh">"""</span>
        <span class="k">for</span> <span class="n">constraint</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">constraints</span><span class="p">:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">solver</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="n">constraint</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">solver</span><span class="p">.</span><span class="nf">check</span><span class="p">()</span> <span class="o">==</span> <span class="n">sat</span><span class="p">:</span>
            <span class="n">model</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">solver</span><span class="p">.</span><span class="nf">model</span><span class="p">()</span>
            <span class="k">return</span> <span class="p">{</span>
                <span class="sh">'</span><span class="s">next_treatment_time</span><span class="sh">'</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">model</span><span class="p">[</span><span class="n">t_next_treatment</span><span class="p">].</span><span class="nf">as_fraction</span><span class="p">()),</span>
                <span class="sh">'</span><span class="s">feasible</span><span class="sh">'</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span>
                <span class="sh">'</span><span class="s">constraints_satisfied</span><span class="sh">'</span><span class="p">:</span> <span class="nf">len</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">constraints</span><span class="p">)</span>
            <span class="p">}</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="p">{</span><span class="sh">'</span><span class="s">feasible</span><span class="sh">'</span><span class="p">:</span> <span class="bp">False</span><span class="p">,</span> <span class="sh">'</span><span class="s">conflicting_constraints</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">get_unsat_core</span><span class="p">()}</span>
</code></pre>

</div>



<h3>
  
  
  Adaptive Planning with Monte Carlo Tree Search
</h3>

<p>While exploring planning algorithms, I discovered that Monte Carlo Tree Search (MCTS) could be adapted for clinical decision-making by incorporating both neural value estimates and symbolic constraints:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ClinicalMCTSNode</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Node in the treatment planning search tree</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">patient_state</span><span class="p">:</span> <span class="n">PatientState</span><span class="p">,</span> <span class="n">action</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span> <span class="n">parent</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">patient_state</span> <span class="o">=</span> <span class="n">patient_state</span>
        <span class="n">self</span><span class="p">.</span><span class="n">action</span> <span class="o">=</span> <span class="n">action</span>
        <span class="n">self</span><span class="p">.</span><span class="n">parent</span> <span class="o">=</span> <span class="n">parent</span>
        <span class="n">self</span><span class="p">.</span><span class="n">children</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">self</span><span class="p">.</span><span class="n">visits</span> <span class="o">=</span> <span class="mi">0</span>
        <span class="n">self</span><span class="p">.</span><span class="n">value</span> <span class="o">=</span> <span class="mf">0.0</span>
        <span class="n">self</span><span class="p">.</span><span class="n">untried_actions</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">get_possible_actions</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">get_possible_actions</span><span class="p">(</span><span class="n">self</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">List</span><span class="p">[</span><span class="nb">str</span><span class="p">]:</span>
        <span class="sh">"""</span><span class="s">Generate clinically feasible actions using symbolic reasoning</span><span class="sh">"""</span>
        <span class="n">feasible_actions</span> <span class="o">=</span> <span class="p">[]</span>

        <span class="c1"># Check clinical guidelines symbolically
</span>        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="nf">is_recovery_complete</span><span class="p">():</span>
            <span class="n">feasible_actions</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sh">'</span><span class="s">administer_next_cycle</span><span class="sh">'</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="nf">has_significant_toxicity</span><span class="p">():</span>
            <span class="n">feasible_actions</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sh">'</span><span class="s">dose_reduce</span><span class="sh">'</span><span class="p">)</span>
            <span class="n">feasible_actions</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sh">'</span><span class="s">treatment_hold</span><span class="sh">'</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="nf">is_progression_detected</span><span class="p">():</span>
            <span class="n">feasible_actions</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sh">'</span><span class="s">switch_regimen</span><span class="sh">'</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">feasible_actions</span>

    <span class="k">def</span> <span class="nf">expand</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Expand node by trying an untried action</span><span class="sh">"""</span>
        <span class="n">action</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">untried_actions</span><span class="p">.</span><span class="nf">pop</span><span class="p">()</span>
        <span class="n">next_state</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">simulate_action</span><span class="p">(</span><span class="n">action</span><span class="p">)</span>
        <span class="n">child_node</span> <span class="o">=</span> <span class="nc">ClinicalMCTSNode</span><span class="p">(</span><span class="n">next_state</span><span class="p">,</span> <span class="n">action</span><span class="p">,</span> <span class="n">self</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">children</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">child_node</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">child_node</span>

    <span class="k">def</span> <span class="nf">simulate_action</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">action</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">PatientState</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Simulate action outcome using neural predictors</span><span class="sh">"""</span>
        <span class="c1"># Use neural network to predict next state
</span>        <span class="n">predicted_outcomes</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">neural_predictor</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span>
            <span class="n">self</span><span class="p">.</span><span class="n">patient_state</span><span class="p">,</span>
            <span class="n">action</span>
        <span class="p">)</span>

        <span class="c1"># Apply symbolic constraints to filter impossible outcomes
</span>        <span class="n">feasible_outcomes</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">apply_clinical_constraints</span><span class="p">(</span><span class="n">predicted_outcomes</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="nf">select_most_likely_outcome</span><span class="p">(</span><span class="n">feasible_outcomes</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">AdaptiveTreatmentPlanner</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Main planner combining neural and symbolic reasoning</span><span class="sh">"""</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">neural_module</span><span class="p">:</span> <span class="n">NeuralPerceptionModule</span><span class="p">,</span>
                 <span class="n">symbolic_solver</span><span class="p">:</span> <span class="n">ClinicalConstraintSolver</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">neural_module</span> <span class="o">=</span> <span class="n">neural_module</span>
        <span class="n">self</span><span class="p">.</span><span class="n">symbolic_solver</span> <span class="o">=</span> <span class="n">symbolic_solver</span>
        <span class="n">self</span><span class="p">.</span><span class="n">mcts</span> <span class="o">=</span> <span class="nc">ClinicalMCTS</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">plan_treatment_sequence</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">patient_state</span><span class="p">:</span> <span class="n">PatientState</span><span class="p">,</span>
                               <span class="n">horizon</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">4</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">List</span><span class="p">[</span><span class="n">Dict</span><span class="p">]:</span>
        <span class="sh">"""</span><span class="s">Generate optimal treatment sequence for recovery window</span><span class="sh">"""</span>

        <span class="n">plans</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">current_state</span> <span class="o">=</span> <span class="n">patient_state</span>

        <span class="k">for</span> <span class="n">step</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">horizon</span><span class="p">):</span>
            <span class="c1"># Neural prediction of outcomes
</span>            <span class="n">neural_predictions</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">neural_module</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">current_state</span><span class="p">)</span>

            <span class="c1"># Symbolic validation of predictions
</span>            <span class="n">validated_predictions</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">symbolic_solver</span><span class="p">.</span><span class="nf">validate</span><span class="p">(</span>
                <span class="n">neural_predictions</span><span class="p">,</span>
                <span class="n">current_state</span>
            <span class="p">)</span>

            <span class="c1"># MCTS for optimal action selection
</span>            <span class="n">best_action</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">mcts</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span>
                <span class="n">current_state</span><span class="p">,</span>
                <span class="n">validated_predictions</span><span class="p">,</span>
                <span class="n">simulations</span><span class="o">=</span><span class="mi">1000</span>
            <span class="p">)</span>

            <span class="c1"># Generate execution plan with timing
</span>            <span class="n">execution_plan</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_execution_plan</span><span class="p">(</span>
                <span class="n">best_action</span><span class="p">,</span>
                <span class="n">current_state</span>
            <span class="p">)</span>

            <span class="n">plans</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">execution_plan</span><span class="p">)</span>
            <span class="n">current_state</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">simulate_next_state</span><span class="p">(</span><span class="n">current_state</span><span class="p">,</span> <span class="n">best_action</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">plans</span>
</code></pre>

</div>



<h2>
  
  
  Real-World Applications: From Theory to Clinical Practice
</h2>

<h3>
  
  
  Case Study: Managing CAR-T Cell Therapy Recovery
</h3>

<p>During my research into immunotherapy workflows, I focused on CAR-T cell therapy—a treatment with exceptionally narrow recovery windows. Through studying clinical cases, I learned that cytokine release syndrome (CRS) and neurotoxicity require precise timing of interventions like tocilizumab or steroids.</p>

<p>One implementation I developed was a real-time monitoring system:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">CAR_TRecoveryMonitor</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Real-time monitoring for CAR-T recovery windows</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">crs_risk_predictor</span> <span class="o">=</span> <span class="n">nn</span><span class="p">.</span><span class="nc">Sequential</span><span class="p">(</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">8</span><span class="p">,</span> <span class="mi">32</span><span class="p">),</span>  <span class="c1"># 8 biomarkers including IL-6, ferritin, CRP
</span>            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">32</span><span class="p">,</span> <span class="mi">16</span><span class="p">),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">ReLU</span><span class="p">(),</span>
            <span class="n">nn</span><span class="p">.</span><span class="nc">Linear</span><span class="p">(</span><span class="mi">16</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="n">self</span><span class="p">.</span><span class="n">toxicity_escalation_rules</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">grade1</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">monitor_q4h</span><span class="sh">'</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">grade2</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">consider_tocilizumab</span><span class="sh">'</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">grade3</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">administer_tocilizumab</span><span class="sh">'</span><span class="p">,</span>
            <span class="sh">'</span><span class="s">grade4</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">tocilizumab_plus_steroids_icu</span><span class="sh">'</span>
        <span class="p">}</span>

    <span class="k">def</span> <span class="nf">monitor_recovery_window</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">patient_stream</span><span class="p">:</span> <span class="n">DataStream</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Continuous monitoring during critical recovery period</span><span class="sh">"""</span>
        <span class="n">alerts</span> <span class="o">=</span> <span class="p">[]</span>

        <span class="k">while</span> <span class="n">patient_stream</span><span class="p">.</span><span class="n">active</span><span class="p">:</span>
            <span class="n">current_data</span> <span class="o">=</span> <span class="n">patient_stream</span><span class="p">.</span><span class="nf">get_latest</span><span class="p">()</span>

            <span class="c1"># Neural prediction of CRS risk
</span>            <span class="n">crs_risk</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">crs_risk_predictor</span><span class="p">(</span>
                <span class="n">torch</span><span class="p">.</span><span class="nf">tensor</span><span class="p">(</span><span class="n">current_data</span><span class="p">[</span><span class="sh">'</span><span class="s">biomarkers</span><span class="sh">'</span><span class="p">])</span>
            <span class="p">).</span><span class="nf">item</span><span class="p">()</span>

            <span class="c1"># Symbolic rule checking
</span>            <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="nf">check_escalation_criteria</span><span class="p">(</span><span class="n">current_data</span><span class="p">):</span>
                <span class="n">recommended_action</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">toxicity_escalation_rules</span><span class="p">[</span>
                    <span class="n">current_data</span><span class="p">[</span><span class="sh">'</span><span class="s">toxicity_grade</span><span class="sh">'</span><span class="p">]</span>
                <span class="p">]</span>

                <span class="c1"># Temporal constraint checking
</span>                <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="nf">validate_timing</span><span class="p">(</span><span class="n">current_data</span><span class="p">,</span> <span class="n">recommended_action</span><span class="p">):</span>
                    <span class="n">alerts</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                        <span class="sh">'</span><span class="s">timestamp</span><span class="sh">'</span><span class="p">:</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span>
                        <span class="sh">'</span><span class="s">risk_score</span><span class="sh">'</span><span class="p">:</span> <span class="n">crs_risk</span><span class="p">,</span>
                        <span class="sh">'</span><span class="s">recommended_action</span><span class="sh">'</span><span class="p">:</span> <span class="n">recommended_action</span><span class="p">,</span>
                        <span class="sh">'</span><span class="s">time_critical</span><span class="sh">'</span><span class="p">:</span> <span class="bp">True</span> <span class="k">if</span> <span class="n">crs_risk</span> <span class="o">&gt;</span> <span class="mf">0.7</span> <span class="k">else</span> <span class="bp">False</span>
                    <span class="p">})</span>

            <span class="c1"># Adaptive sampling based on risk
</span>            <span class="n">sampling_interval</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">adaptive_sampling_interval</span><span class="p">(</span><span class="n">crs_risk</span><span class="p">)</span>
            <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">sampling_interval</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">alerts</span>
</code></pre>

</div>



<h3>
  
  
  Integration with Hospital Systems
</h3>

<p>Through my experimentation with healthcare IT integration, I discovered several critical implementation patterns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ClinicalWorkflowIntegrator</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Integrates planner with hospital EHR and scheduling systems</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">generate_workflow_orchestration</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">treatment_plan</span><span class="p">:</span> <span class="n">Dict</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Convert treatment plan into executable clinical workflow</span><span class="sh">"""</span>

        <span class="n">workflow</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">pharmacy_orders</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_pharmacy_orders</span><span class="p">(</span><span class="n">treatment_plan</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">lab_requisitions</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_lab_schedule</span><span class="p">(</span><span class="n">treatment_plan</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">imaging_requests</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_imaging_schedule</span><span class="p">(</span><span class="n">treatment_plan</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">nursing_tasks</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_nursing_protocol</span><span class="p">(</span><span class="n">treatment_plan</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">provider_notifications</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_alert_schedule</span><span class="p">(</span><span class="n">treatment_plan</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">patient_instructions</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">generate_patient_instructions</span><span class="p">(</span><span class="n">treatment_plan</span><span class="p">)</span>
        <span class="p">}</span>

        <span class="c1"># Add temporal constraints between tasks
</span>        <span class="n">workflow</span><span class="p">[</span><span class="sh">'</span><span class="s">temporal_constraints</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">extract_timing_constraints</span><span class="p">(</span>
            <span class="n">treatment_plan</span><span class="p">[</span><span class="sh">'</span><span class="s">symbolic_constraints</span><span class="sh">'</span><span class="p">]</span>
        <span class="p">)</span>

        <span class="k">return</span> <span class="n">workflow</span>

    <span class="k">def</span> <span class="nf">execute_adaptive_schedule</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">workflow</span><span class="p">:</span> <span class="n">Dict</span><span class="p">,</span>
                                 <span class="n">real_time_updates</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="n">Dict</span><span class="p">]):</span>
        <span class="sh">"""</span><span class="s">Dynamically adjust schedule based on real-time data</span><span class="sh">"""</span>

        <span class="n">adjusted_schedule</span> <span class="o">=</span> <span class="n">workflow</span><span class="p">.</span><span class="nf">copy</span><span class="p">()</span>

        <span class="k">for</span> <span class="n">update</span> <span class="ow">in</span> <span class="n">real_time_updates</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">update</span><span class="p">[</span><span class="sh">'</span><span class="s">type</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">lab_result</span><span class="sh">'</span><span class="p">:</span>
                <span class="c1"># Re-evaluate timing based on new results
</span>                <span class="n">new_timing</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">recalculate_timing</span><span class="p">(</span>
                    <span class="n">update</span><span class="p">[</span><span class="sh">'</span><span class="s">value</span><span class="sh">'</span><span class="p">],</span>
                    <span class="n">workflow</span><span class="p">[</span><span class="sh">'</span><span class="s">temporal_constraints</span><span class="sh">'</span><span class="p">]</span>
                <span class="p">)</span>
                <span class="n">adjusted_schedule</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">reschedule_tasks</span><span class="p">(</span>
                    <span class="n">adjusted_schedule</span><span class="p">,</span>
                    <span class="n">new_timing</span>
                <span class="p">)</span>

            <span class="k">elif</span> <span class="n">update</span><span class="p">[</span><span class="sh">'</span><span class="s">type</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">resource_constraint</span><span class="sh">'</span><span class="p">:</span>
                <span class="c1"># Handle resource availability issues
</span>                <span class="n">adjusted_schedule</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">reallocate_resources</span><span class="p">(</span>
                    <span class="n">adjusted_schedule</span><span class="p">,</span>
                    <span class="n">update</span><span class="p">[</span><span class="sh">'</span><span class="s">constraints</span><span class="sh">'</span><span class="p">]</span>
                <span class="p">)</span>

        <span class="k">return</span> <span class="n">adjusted_schedule</span>
</code></pre>

</div>



<h2>
  
  
  Challenges and Solutions: Lessons from Implementation
</h2>

<h3>
  
  
  Challenge 1: Handling Uncertain and Incomplete Data
</h3>

<p>During my experimentation with real clinical data, I encountered significant missingness and measurement error. One solution I developed was a probabilistic neuro-symbolic layer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ProbabilisticNeuroSymbolicLayer</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Handles uncertainty in both neural and symbolic components</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">bayesian_nn</span> <span class="o">=</span> <span class="nc">BayesianNeuralNetwork</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="n">probabilistic_logic</span> <span class="o">=</span> <span class="nc">MarkovLogicNetwork</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">reason_under_uncertainty</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">evidence</span><span class="p">:</span> <span class="n">Dict</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Perform joint inference over neural and symbolic variables</span><span class="sh">"""</span>

        <span class="c1"># Neural predictions with uncertainty estimates
</span>        <span class="n">neural_predictions</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">bayesian_nn</span><span class="p">.</span><span class="nf">predict_with_uncertainty</span><span class="p">(</span><span class="n">evidence</span><span class="p">)</span>

        <span class="c1"># Symbolic reasoning with soft constraints
</span>        <span class="n">symbolic_inference</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">probabilistic_logic</span><span class="p">.</span><span class="nf">infer</span><span class="p">(</span>
            <span class="n">evidence</span><span class="p">,</span>
            <span class="n">neural_predictions</span><span class="p">[</span><span class="sh">'</span><span class="s">mean</span><span class="sh">'</span><span class="p">],</span>
            <span class="n">neural_predictions</span><span class="p">[</span><span class="sh">'</span><span class="s">variance</span><span class="sh">'</span><span class="p">]</span>
        <span class="p">)</span>

        <span class="c1"># Fuse predictions using probabilistic graphical model
</span>        <span class="n">joint_distribution</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">build_joint_distribution</span><span class="p">(</span>
            <span class="n">neural_predictions</span><span class="p">,</span>
            <span class="n">symbolic_inference</span>
        <span class="p">)</span>

        <span class="c1"># Sample treatment plans from joint distribution
</span>        <span class="n">feasible_plans</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">sample_feasible_plans</span><span class="p">(</span>
            <span class="n">joint_distribution</span><span class="p">,</span>
            <span class="n">n_samples</span><span class="o">=</span><span class="mi">1000</span>
        <span class="p">)</span>

        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">optimal_plan</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">select_robust_plan</span><span class="p">(</span><span class="n">feasible_plans</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">confidence_interval</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_confidence_intervals</span><span class="p">(</span><span class="n">feasible_plans</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">risk_estimates</span><span class="sh">'</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="nf">compute_risk_metrics</span><span class="p">(</span><span class="n">feasible_plans</span><span class="p">)</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 2: Real-Time Adaptation to Changing Conditions
</h3>

<p>While testing in simulated environments, I found that recovery windows could shift dramatically based on individual patient responses. My solution involved continuous learning:</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
python
class ContinuousAdaptationModule:
    """Enables planner to adapt to individual patient trajectories"""

    def __init__(self):
        self.online_learner = OnlineNeuralNetwork()
        self.constraint_updater = DynamicConstraintManager()
        self.experience_replay = ClinicalExperienceBuffer()

    def adapt_to_patient_response(self, observed_outcome: Dict,
                                 predicted_outcome: Dict):
        """Update models based on prediction errors"""

        # Compute prediction error
        error = self.compute_multiobjective_error(
            observed_outcome,
            predicted_outcome
        )

        # Update neural models online
        if error['neural_component'] &gt; self.thresholds['neural']:
            self.online_learner.update(
                self.experience_replay.sample_batch(),
                observed_outcome
            )

        # Update symbolic constraints
        if error['symbolic_component'] &gt; self.thresholds['symbolic']:
            new_constraints = self.constraint_updater.learn_constraints(
                observed_outcome,
                self.experience_replay.get_trajectory()
            )
            self.symbolic_solver.add_constraints(new_constraints)

        # Update recovery window definitions
        self.update_recovery_window_definition(
            observed_outcome['re
</code></pre>

</div>

