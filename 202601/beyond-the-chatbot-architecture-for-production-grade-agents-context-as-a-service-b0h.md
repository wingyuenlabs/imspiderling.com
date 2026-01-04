---
Title: Beyond the Chatbot: Architecture for Production-Grade Agents (Context as a Service)
Description: 
Author: Imran Siddique
Date: 2026-01-04T21:33:34.000Z
Robots: noindex,nofollow
Template: index
---
<p>We are past the "Hello World" phase of AI Agents.</p>

<p>The internet is flooded with tutorials on how to make an LLM call a weather API. But if you are building agents for enterprise production, you know that <strong>making the LLM "do things" is the easy part.</strong></p>

<p>The hard part is preventing it from doing <em>bad</em> things, understanding <em>what</em> it learned, and rendering the output in a way that isn't just a wall of text.</p>

<p>I’ve been working on an architecture I call <strong>Context as a Service (CaaS)</strong>. It decouples the "Brain" (LLM) from the "Infrastructure" (Tools/UI) using a set of deterministic layers.</p>

<p>Here is the architecture that turns a toy agent into a production system.</p>

<h2>
  
  
  1. The Logic Firewall (Constraint Engineering)
</h2>

<p><strong>The Problem:</strong> Prompt Engineering is fragile. Begging an LLM, <em>"Please do not drop the database,"</em> is not a security strategy.</p>

<p><strong>The Solution:</strong> A deterministic <strong>Constraint Engine</strong> that sits between the Agent and the Executor. The Agent generates a plan, but the Firewall approves it.</p>

<p>I implemented a <code>ConstraintEngine</code> class that intercepts plans before they touch the infrastructure. It uses regex and logic, not AI.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># constraint_engine.py
</span><span class="k">class</span> <span class="nc">SQLInjectionRule</span><span class="p">(</span><span class="n">ConstraintRule</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Detects dangerous SQL operations deterministically.</span><span class="sh">"""</span>
    <span class="n">DANGEROUS_PATTERNS</span> <span class="o">=</span> <span class="p">[</span>
        <span class="sa">r</span><span class="sh">'</span><span class="s">\bDROP\s+TABLE\b</span><span class="sh">'</span><span class="p">,</span>
        <span class="sa">r</span><span class="sh">'</span><span class="s">\bDELETE\s+FROM\b.*\bWHERE\s+1\s*=\s*1\b</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">]</span>

    <span class="k">def</span> <span class="nf">validate</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">plan</span><span class="p">):</span>
        <span class="n">query</span> <span class="o">=</span> <span class="n">plan</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">query</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>
        <span class="k">for</span> <span class="n">pattern</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">DANGEROUS_PATTERNS</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">re</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="n">pattern</span><span class="p">,</span> <span class="n">query</span><span class="p">,</span> <span class="n">re</span><span class="p">.</span><span class="n">IGNORECASE</span><span class="p">):</span>
                <span class="k">return</span> <span class="nc">ConstraintViolation</span><span class="p">(</span>
                    <span class="n">severity</span><span class="o">=</span><span class="n">ViolationSeverity</span><span class="p">.</span><span class="n">CRITICAL</span><span class="p">,</span>
                    <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">Dangerous SQL detected. Execution Blocked.</span><span class="sh">"</span>
                <span class="p">)</span>

</code></pre>

</div>



<p>The key insight: <strong>The Human builds the walls; the AI plays inside them.</strong></p>

<h2>
  
  
  2. The Wisdom Curator (Human-in-the-Loop)
</h2>

<p><strong>The Problem:</strong> You can't review 10,000 interactions a day. But you can't let the agent learn "bad habits" (like ignoring errors to seem successful) without oversight.</p>

<p><strong>The Solution:</strong> The <strong>Wisdom Curator</strong>. Instead of reviewing every log, we review <strong>Strategic Samples</strong> and <strong>Policy Violations</strong>.</p>

<p>My <code>WisdomCurator</code> tracks "Wisdom Updates" (new lessons the agent wants to save). If the agent tries to save a lesson like "Ignore 500 errors," the Curator catches it using a keyword blacklist.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># wisdom_curator.py
</span><span class="k">class</span> <span class="nc">PolicyViolationType</span><span class="p">(</span><span class="n">Enum</span><span class="p">):</span>
    <span class="n">HARMFUL_BEHAVIOR</span> <span class="o">=</span> <span class="sh">"</span><span class="s">harmful_behavior</span><span class="sh">"</span> <span class="c1"># e.g. "ignore error"
</span>    <span class="n">SECURITY_RISK</span> <span class="o">=</span> <span class="sh">"</span><span class="s">security_risk</span><span class="sh">"</span>       <span class="c1"># e.g. "disable auth"
</span>
<span class="k">def</span> <span class="nf">requires_policy_review</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">proposed_wisdom</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">bool</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Blocks auto-updates if they violate safety policy.</span><span class="sh">"""</span>
    <span class="k">for</span> <span class="n">pattern</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">policy_patterns</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">pattern</span> <span class="ow">in</span> <span class="n">proposed_wisdom</span><span class="p">.</span><span class="nf">lower</span><span class="p">():</span>
            <span class="k">return</span> <span class="bp">True</span> <span class="c1"># Human must approve
</span>    <span class="k">return</span> <span class="bp">False</span>

</code></pre>

</div>



<p>This shifts the human role from "Editor" (fixing grammar) to "Curator" (approving knowledge).</p>

<h2>
  
  
  3. Polymorphic Output (Just-in-Time UI)
</h2>

<p><strong>The Problem:</strong> Why do AI agents always reply with text? If I ask for sales data, I want a chart. If I ask for a bug fix, I want a diff.</p>

<p><strong>The Solution:</strong> <strong>Polymorphic Output</strong>. The agent generates data and an <code>InputContext</code>. The interface layer decides <em>how</em> to render it.</p>

<ul>
<li>
<strong>Context:</strong> <code>IDE</code> → <strong>Output:</strong> <code>GHOST_TEXT</code> (Autocomplete)</li>
<li>
<strong>Context:</strong> <code>DASHBOARD</code> → <strong>Output:</strong> <code>WIDGET</code> (React Component)</li>
<li>
<strong>Context:</strong> <code>CHAT</code> → <strong>Output:</strong> <code>TEXT</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># example_polymorphic_output.py
</span><span class="k">def</span> <span class="nf">scenario_telemetry_to_widget</span><span class="p">():</span>
    <span class="c1"># Input: Backend telemetry stream with high urgency
</span>    <span class="c1"># Context: Monitoring Dashboard
</span>
    <span class="n">response</span> <span class="o">=</span> <span class="n">engine</span><span class="p">.</span><span class="nf">generate_response</span><span class="p">(</span>
        <span class="n">data</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">metric</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latency</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">value</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">2000ms</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">trend</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">up</span><span class="sh">"</span><span class="p">},</span>
        <span class="n">input_context</span><span class="o">=</span><span class="n">InputContext</span><span class="p">.</span><span class="n">MONITORING</span><span class="p">,</span>
        <span class="n">urgency</span><span class="o">=</span><span class="mf">0.9</span>
    <span class="p">)</span>

    <span class="c1"># Result: A React Widget spec, NOT a text message.
</span>    <span class="c1"># Modality: OutputModality.DASHBOARD_WIDGET
</span>
</code></pre>

</div>



<p>If input can be anything (multimodal), output must be anything.</p>

<h2>
  
  
  4. Silent Signals (The Feedback Loop)
</h2>

<p><strong>The Problem:</strong> Users don't click "Thumbs Down." They just leave. Explicit feedback is a blind spot.</p>

<p><strong>The Solution:</strong> Capture <strong>Silent Signals</strong>.</p>

<ul>
<li>
<strong>Undo Signal:</strong> User hits <code>Ctrl+Z</code>? That’s a <strong>Critical Failure</strong>.</li>
<li>
<strong>Abandonment:</strong> User stops typing mid-flow? <strong>Engagement Failure</strong>.</li>
<li>
<strong>Acceptance:</strong> User copies code and tabs away? <strong>Success</strong>.</li>
</ul>

<p>We instrument the <code>DoerAgent</code> to emit these signals automatically.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># example_silent_signals.py
</span><span class="k">def</span> <span class="nf">on_undo_detected</span><span class="p">():</span>
    <span class="c1"># The loudest "Thumbs Down" possible
</span>    <span class="n">doer</span><span class="p">.</span><span class="nf">emit_undo_signal</span><span class="p">(</span>
        <span class="n">query</span><span class="o">=</span><span class="sh">"</span><span class="s">Delete temp files</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">agent_response</span><span class="o">=</span><span class="sh">"</span><span class="s">rm -rf /</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">undo_action</span><span class="o">=</span><span class="sh">"</span><span class="s">Ctrl+Z</span><span class="sh">"</span>
    <span class="p">)</span>
    <span class="c1"># Observer Agent immediately flags this as CRITICAL
</span>
</code></pre>

</div>



<p>We stop begging for feedback and start observing behavior.</p>

<h2>
  
  
  5. Evaluation Engineering (Eval-DD)
</h2>

<p><strong>The Problem:</strong> You can't write unit tests for an LLM. <code>assert response == "Hello"</code> fails if the AI says "Hi".</p>

<p><strong>The Solution:</strong> <strong>Eval-DD (Evaluation Driven Development)</strong>. We replace unit tests with <strong>Golden Datasets</strong> and <strong>Scoring Rubrics</strong>.</p>

<p>Instead of testing for exact matches, we score on dimensions:</p>

<ol>
<li>
<strong>Correctness</strong> (Did it solve it?)</li>
<li>
<strong>Tone</strong> (Was it rude?)</li>
<li>
<strong>Safety</strong> (Did it leak secrets?)
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># evaluation_engineering.py
</span><span class="n">rubric</span> <span class="o">=</span> <span class="nc">ScoringRubric</span><span class="p">(</span><span class="sh">"</span><span class="s">Customer Service</span><span class="sh">"</span><span class="p">)</span>
<span class="n">rubric</span><span class="p">.</span><span class="nf">add_criteria</span><span class="p">(</span><span class="sh">"</span><span class="s">correctness</span><span class="sh">"</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="mf">0.5</span><span class="p">,</span> <span class="n">evaluator</span><span class="o">=</span><span class="n">correctness_eval</span><span class="p">)</span>
<span class="n">rubric</span><span class="p">.</span><span class="nf">add_criteria</span><span class="p">(</span><span class="sh">"</span><span class="s">tone</span><span class="sh">"</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="mf">0.4</span><span class="p">,</span> <span class="n">evaluator</span><span class="o">=</span><span class="n">tone_eval</span><span class="p">)</span> <span class="c1"># Tone matters!
</span>
<span class="n">runner</span> <span class="o">=</span> <span class="nc">EvaluationRunner</span><span class="p">(</span><span class="n">dataset</span><span class="p">,</span> <span class="n">rubric</span><span class="p">,</span> <span class="n">agent</span><span class="p">)</span>
<span class="n">results</span> <span class="o">=</span> <span class="n">runner</span><span class="p">.</span><span class="nf">run</span><span class="p">()</span>

</code></pre>

</div>



<p>If the agent is the Coder, the Engineer is the Examiner.</p>

<h2>
  
  
  Conclusion
</h2>

<p>We are building <strong>Context as a Service</strong>.</p>

<p>This architecture acknowledges that the LLM is a probabilistic engine that needs a deterministic chassis to be safe, useful, and scalable.</p>

<ol>
<li>
<strong>Constraint Engine</strong> keeps it safe.</li>
<li>
<strong>Wisdom Curator</strong> keeps it smart.</li>
<li>
<strong>Polymorphic Output</strong> makes it usable.</li>
<li>
<strong>Silent Signals</strong> makes it learn.</li>
<li>
<strong>Eval-DD</strong> proves it works.</li>
</ol>

<p>Stop building chatbots. Start building architectures.</p>

