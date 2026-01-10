---
Title: Minions: Building Self-Learning AI Agent Swarms That Actually Ship Code
Description: 
Author: S.P.C. Technology LTD.
Date: 2026-01-10T21:19:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>What if your development team included AI agents that could learn from their mistakes, teach each other new skills, and autonomously fix failing tests at 3 AM?</p>

<p>That's exactly what we built with <strong>Minions</strong> - an open-source, event-driven framework for orchestrating autonomous AI agents. And yes, they're named after the yellow creatures from the movies.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcgiu2ousapfum3kv6yi3.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcgiu2ousapfum3kv6yi3.jpg" alt="Minions | AI That Dreams" width="800" height="457"></a></p>

<h2>
  
  
  The Problem We Solved
</h2>

<p>Most AI coding assistants are reactive - you ask, they answer. But real development workflows are complex:</p>

<ul>
<li>Tests fail at random times</li>
<li>Multiple systems need to stay in sync</li>
<li>Security vulnerabilities need constant monitoring</li>
<li>Documentation drifts from code</li>
<li>New team members need onboarding</li>
</ul>

<p>We wanted AI that works <em>with</em> us, not just <em>for</em> us. AI that:</p>

<ol>
<li>
<strong>Monitors</strong> - Watches for events and reacts autonomously</li>
<li>
<strong>Learns</strong> - Gets better at fixing problems over time</li>
<li>
<strong>Collaborates</strong> - Agents teaching agents, sharing knowledge</li>
<li>
<strong>Self-heals</strong> - Automatically fixes issues without human intervention</li>
</ol>

<h2>
  
  
  Meet the Minions
</h2>

<p>Our framework includes 26 specialized agents, each named after characters from the Despicable Me universe:</p>

<h3>
  
  
  The Core Team
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Agent</th>
<th>Character</th>
<th>Superpower</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Gru</strong></td>
<td>The mastermind</td>
<td>Web interface &amp; conversational AI</td>
</tr>
<tr>
<td><strong>Dr. Nefario</strong></td>
<td>The inventor</td>
<td>Converts requirements to execution plans</td>
</tr>
<tr>
<td><strong>Silas</strong></td>
<td>Resource allocator</td>
<td>Project connections &amp; framework detection</td>
</tr>
<tr>
<td><strong>Lucy</strong></td>
<td>Self-improvement engine</td>
<td>Autonomous completion loops</td>
</tr>
<tr>
<td><strong>Tom</strong></td>
<td>Vigilant security</td>
<td>STRIDE threat modeling &amp; vulnerability scanning</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  The Specialists
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Agent</th>
<th>Character</th>
<th>Superpower</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Bob</strong></td>
<td>The tester</td>
<td>Multi-platform testing &amp; mutation analysis</td>
</tr>
<tr>
<td><strong>Kevin</strong></td>
<td>Performance guru</td>
<td>Profiling &amp; bottleneck detection</td>
</tr>
<tr>
<td><strong>Dave</strong></td>
<td>Database wizard</td>
<td>Schema design &amp; query optimization</td>
</tr>
<tr>
<td><strong>Stuart</strong></td>
<td>Backend builder</td>
<td>Routes, models, services, middleware</td>
</tr>
<tr>
<td><strong>Agnes</strong></td>
<td>Frontend crafter</td>
<td>Components, hooks, stores, forms</td>
</tr>
<tr>
<td><strong>Otto</strong></td>
<td>Flutter specialist</td>
<td>Widgets, blocs, pages, localization</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Architecture: Event-Driven Everything
</h2>

<p>The secret sauce is our event-driven architecture. Instead of rigid pipelines, agents communicate through 80+ event types:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌────────────────────────────────────────────────────────────────────────┐
│                         CLIENT INTERFACE                                │
│    Gru (Web UI)  →  Dr. Nefario (Planner)  →  Tom (Security)           │
├────────────────────────────────────────────────────────────────────────┤
│                         ORCHESTRATION                                   │
│    Orchestrator  →  DependencyGraph  →  AutonomousLoopManager          │
├────────────────────────────────────────────────────────────────────────┤
│                      SPECIALIZED AGENTS                                 │
│    Tester  │  Docker  │  GitHub  │  Database  │  Performance           │
├────────────────────────────────────────────────────────────────────────┤
│                      LEARNING &amp; EVOLUTION                               │
│    KnowledgeBrain  │  ReinforcementLearner  │  DynamicSkillGenerator   │
├────────────────────────────────────────────────────────────────────────┤
│                     FOUNDATION SERVICES                                 │
│    EventBus  │  MemoryStore  │  StateMachine  │  CircuitBreaker        │
└────────────────────────────────────────────────────────────────────────┘
</code></pre>

</div>



<p>Here's how simple it is to wire agents together:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">initializeMinions</span><span class="p">,</span> <span class="nx">createAgent</span><span class="p">,</span> <span class="nx">EventTypes</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">minions</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="p">{</span> <span class="nx">orchestrator</span><span class="p">,</span> <span class="nx">eventBus</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">initializeMinions</span><span class="p">();</span>

<span class="c1">// Create agents</span>
<span class="kd">const</span> <span class="nx">fetchAgent</span> <span class="o">=</span> <span class="nf">createAgent</span><span class="p">({</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">fetch-agent</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">execute</span><span class="p">:</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchFromAPI</span><span class="p">();</span>
    <span class="nx">eventBus</span><span class="p">.</span><span class="nf">publish</span><span class="p">(</span><span class="nx">EventTypes</span><span class="p">.</span><span class="nx">CODE_GENERATED</span><span class="p">,</span> <span class="p">{</span> <span class="na">files</span><span class="p">:</span> <span class="nx">data</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">processAgent</span> <span class="o">=</span> <span class="nf">createAgent</span><span class="p">({</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">process-agent</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">execute</span><span class="p">:</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// This runs AFTER fetch-agent completes</span>
    <span class="k">await</span> <span class="nf">processData</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// Register with dependencies</span>
<span class="nx">orchestrator</span><span class="p">.</span><span class="nf">registerAgent</span><span class="p">(</span><span class="dl">'</span><span class="s1">fetch-agent</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">fetchAgent</span><span class="p">,</span> <span class="p">[]);</span>
<span class="nx">orchestrator</span><span class="p">.</span><span class="nf">registerAgent</span><span class="p">(</span><span class="dl">'</span><span class="s1">process-agent</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">processAgent</span><span class="p">,</span> <span class="p">[</span><span class="dl">'</span><span class="s1">fetch-agent</span><span class="dl">'</span><span class="p">]);</span>

<span class="c1">// Execute in dependency order</span>
<span class="k">await</span> <span class="nx">orchestrator</span><span class="p">.</span><span class="nf">execute</span><span class="p">();</span>
</code></pre>

</div>



<h2>
  
  
  The Magic: Self-Learning Agents
</h2>

<p>Here's where it gets interesting. Minions don't just execute - they <strong>learn</strong>.</p>

<h3>
  
  
  Reinforcement Learning
</h3>

<p>Every time an agent succeeds or fails, it learns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">getReinforcementLearner</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">minions</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">learner</span> <span class="o">=</span> <span class="nf">getReinforcementLearner</span><span class="p">();</span>

<span class="c1">// Agent fixes a null reference error</span>
<span class="k">await</span> <span class="nx">learner</span><span class="p">.</span><span class="nf">recordExperience</span><span class="p">({</span>
  <span class="na">state</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test_failure_null_reference</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">action</span><span class="p">:</span> <span class="dl">'</span><span class="s1">apply_optional_chaining</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">reward</span><span class="p">:</span> <span class="mf">1.0</span><span class="p">,</span>  <span class="c1">// SUCCESS!</span>
  <span class="na">nextState</span><span class="p">:</span> <span class="dl">'</span><span class="s1">tests_passing</span><span class="dl">'</span>
<span class="p">});</span>

<span class="c1">// Next time it sees a similar error:</span>
<span class="kd">const</span> <span class="nx">bestAction</span> <span class="o">=</span> <span class="nx">learner</span><span class="p">.</span><span class="nf">selectAction</span><span class="p">(</span><span class="dl">'</span><span class="s1">test_failure_null_reference</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// Returns: 'apply_optional_chaining' (learned from experience)</span>
</code></pre>

</div>



<p>The learner uses Q-learning with Thompson sampling:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Q(s,a) ← Q(s,a) + α[r + γ·max_a'Q(s',a') - Q(s,a)]
</code></pre>

</div>



<p>Translation: It balances exploration (trying new things) with exploitation (using what works).</p>

<h3>
  
  
  Knowledge Brain: Collective Intelligence
</h3>

<p>Agents share knowledge through a distributed brain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">getKnowledgeBrain</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">minions</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">brain</span> <span class="o">=</span> <span class="nf">getKnowledgeBrain</span><span class="p">();</span>

<span class="c1">// Stuart (backend agent) learns a pattern</span>
<span class="k">await</span> <span class="nx">brain</span><span class="p">.</span><span class="nf">store</span><span class="p">({</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CODE_PATTERN</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">topic</span><span class="p">:</span> <span class="dl">'</span><span class="s1">null-check-fix</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">content</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">pattern</span><span class="p">:</span> <span class="dl">'</span><span class="s1">optional-chaining</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">obj?.nested?.value</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">effectiveness</span><span class="p">:</span> <span class="mf">0.94</span>
  <span class="p">},</span>
  <span class="na">confidence</span><span class="p">:</span> <span class="mf">0.95</span>
<span class="p">});</span>

<span class="c1">// Agnes (frontend agent) queries for similar patterns</span>
<span class="kd">const</span> <span class="nx">patterns</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">brain</span><span class="p">.</span><span class="nf">query</span><span class="p">({</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CODE_PATTERN</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">similarity</span><span class="p">:</span> <span class="dl">'</span><span class="s1">null reference error fix</span><span class="dl">'</span>
<span class="p">});</span>
<span class="c1">// Gets Stuart's pattern - knowledge shared!</span>
</code></pre>

</div>



<h3>
  
  
  Dynamic Skill Generation
</h3>

<p>The wildest part? Agents can <strong>generate new skills</strong> from patterns they observe:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">getDynamicSkillGenerator</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">minions</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">skillGen</span> <span class="o">=</span> <span class="nf">getDynamicSkillGenerator</span><span class="p">();</span>

<span class="c1">// System detects recurring pattern</span>
<span class="kd">const</span> <span class="nx">skill</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">skillGen</span><span class="p">.</span><span class="nf">generateSkill</span><span class="p">({</span>
  <span class="na">pattern</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">async-error-wrapper</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Wrap async functions with try-catch and logging</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">examples</span><span class="p">:</span> <span class="p">[</span><span class="cm">/* observed patterns */</span><span class="p">]</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// Canary deploy to 10% of requests</span>
<span class="k">await</span> <span class="nx">skillGen</span><span class="p">.</span><span class="nf">deployCanary</span><span class="p">(</span><span class="nx">skill</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="p">{</span> <span class="na">percentage</span><span class="p">:</span> <span class="mi">10</span> <span class="p">});</span>

<span class="c1">// If metrics look good, promote to production</span>
<span class="k">await</span> <span class="nx">skillGen</span><span class="p">.</span><span class="nf">approveSkill</span><span class="p">(</span><span class="nx">skill</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
</code></pre>

</div>



<p>The skill generation pipeline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Pattern Detected → LLM Generates Skill → Sandboxed Testing → Canary Deploy → A/B Test → Production
</code></pre>

</div>



<h2>
  
  
  Autonomous Fix Loops
</h2>

<p>When tests fail, Minions spring into action:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">getAutonomousLoopManager</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">minions</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">loopManager</span> <span class="o">=</span> <span class="nf">getAutonomousLoopManager</span><span class="p">();</span>

<span class="c1">// Register which agents handle which failures</span>
<span class="nx">loopManager</span><span class="p">.</span><span class="nf">registerAgentMatcher</span><span class="p">((</span><span class="nx">failure</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">failure</span><span class="p">.</span><span class="nx">file</span><span class="p">?.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/</span><span class="dl">'</span><span class="p">))</span> <span class="k">return</span> <span class="dl">'</span><span class="s1">backend-agent</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">failure</span><span class="p">.</span><span class="nx">file</span><span class="p">?.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">/components/</span><span class="dl">'</span><span class="p">))</span> <span class="k">return</span> <span class="dl">'</span><span class="s1">frontend-agent</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">failure</span><span class="p">.</span><span class="nx">file</span><span class="p">?.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">.test.</span><span class="dl">'</span><span class="p">))</span> <span class="k">return</span> <span class="dl">'</span><span class="s1">tester-agent</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
<span class="p">});</span>

<span class="c1">// Loop activates automatically on TESTS_FAILED events</span>
<span class="c1">// 1. Analyzes failure</span>
<span class="c1">// 2. Creates checkpoint</span>
<span class="c1">// 3. Applies fix (Tier 1: patterns, Tier 2: agent-specific)</span>
<span class="c1">// 4. Re-runs tests</span>
<span class="c1">// 5. Commits or rollbacks</span>
</code></pre>

</div>



<p>The system creates Git checkpoints before every fix attempt, so failures can be safely rolled back.</p>

<h2>
  
  
  Getting Started in 60 Seconds
</h2>

<h3>
  
  
  Docker (Recommended)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/Inweb-eg/minions.git
<span class="nb">cd </span>minions/docker
docker compose up <span class="nt">-d</span>
docker <span class="nb">exec </span>minions-ollama ollama pull deepseek-coder:6.7b
docker restart minions

<span class="c"># Open http://localhost:2505</span>
</code></pre>

</div>



<h3>
  
  
  Node.js
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/Inweb-eg/minions.git
<span class="nb">cd </span>minions
npm run <span class="nb">install</span>:all
node index.js <span class="nt">--gru</span>
</code></pre>

</div>



<p>You'll see Gru's web interface - a conversational AI that helps you plan and execute projects:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2emhq6atwce9vrv371ql.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2emhq6atwce9vrv371ql.jpg" alt="Gru Interface" width="800" height="450"></a></p>

<h2>
  
  
  Real-World Example: Building a REST API
</h2>

<p>Let's say you want to build a user authentication API. Here's how Minions handles it:</p>

<p><strong>1. Talk to Gru</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>You: "I need a REST API for user authentication with JWT tokens"
Gru: "I'll help you build that. Let me scan your project first..."
</code></pre>

</div>



<p><strong>2. Dr. Nefario Creates the Plan</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"tasks"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"backend"</span><span class="p">,</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"generate-model"</span><span class="p">,</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"User"</span><span class="w"> </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"backend"</span><span class="p">,</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"generate-route"</span><span class="p">,</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"auth"</span><span class="w"> </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"backend"</span><span class="p">,</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"generate-middleware"</span><span class="p">,</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"jwt"</span><span class="w"> </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"testing"</span><span class="p">,</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"generate-tests"</span><span class="p">,</span><span class="w"> </span><span class="nl">"target"</span><span class="p">:</span><span class="w"> </span><span class="s2">"auth"</span><span class="w"> </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"dependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"auth-route"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"user-model"</span><span class="p">,</span><span class="w"> </span><span class="s2">"jwt-middleware"</span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>3. Tom Validates Security</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✓ JWT secret not hardcoded
✓ Password hashing detected (bcrypt)
✓ Rate limiting recommended → Added to plan
✓ No SQL injection vulnerabilities
</code></pre>

</div>



<p><strong>4. Agents Execute in Order</strong></p>

<ul>
<li>Stuart generates User model, auth routes, JWT middleware</li>
<li>Bob generates and runs tests</li>
<li>If tests fail → AutoFixer attempts repair</li>
<li>If still failing → Stuart gets specific feedback</li>
</ul>

<p><strong>5. Lucy Ensures Completion</strong></p>

<ul>
<li>Detects gaps: "Missing password reset endpoint"</li>
<li>Generates additional tasks</li>
<li>Loops until 100% complete or max iterations</li>
</ul>

<h2>
  
  
  The Event System
</h2>

<p>Everything communicates through events. Here are some key ones:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Agent Lifecycle</span>
<span class="nx">AgentEvents</span><span class="p">.</span><span class="nx">AGENT_STARTED</span>
<span class="nx">AgentEvents</span><span class="p">.</span><span class="nx">AGENT_COMPLETED</span>
<span class="nx">AgentEvents</span><span class="p">.</span><span class="nx">AGENT_FAILED</span>

<span class="c1">// Testing</span>
<span class="nx">TestEvents</span><span class="p">.</span><span class="nx">TESTS_STARTED</span>
<span class="nx">TestEvents</span><span class="p">.</span><span class="nx">TESTS_COMPLETED</span>
<span class="nx">TestEvents</span><span class="p">.</span><span class="nx">TESTS_FAILED</span>  <span class="c1">// Triggers autonomous fix loop</span>

<span class="c1">// Security (Tom)</span>
<span class="nx">SecurityEvents</span><span class="p">.</span><span class="nx">VULNERABILITY_FOUND</span>
<span class="nx">SecurityEvents</span><span class="p">.</span><span class="nx">RISK_IDENTIFIED</span>
<span class="nx">SecurityEvents</span><span class="p">.</span><span class="nx">THREAT_MITIGATED</span>

<span class="c1">// Learning</span>
<span class="nx">LearningEvents</span><span class="p">.</span><span class="nx">PATTERN_DETECTED</span>
<span class="nx">LearningEvents</span><span class="p">.</span><span class="nx">SKILL_GENERATED</span>
<span class="nx">LearningEvents</span><span class="p">.</span><span class="nx">POLICY_UPDATED</span>
</code></pre>

</div>



<p>Subscribe to any event:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">eventBus</span><span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">EventTypes</span><span class="p">.</span><span class="nx">TESTS_FAILED</span><span class="p">,</span> <span class="dl">'</span><span class="s1">my-handler</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Tests failed in </span><span class="p">${</span><span class="nx">data</span><span class="p">.</span><span class="nx">file</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Failures: </span><span class="p">${</span><span class="nx">data</span><span class="p">.</span><span class="nx">failures</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Built-in Skills
</h2>

<p>Agents come with pre-built skills you can use directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Auto-fix test failures</span>
<span class="kd">const</span> <span class="nx">fixer</span> <span class="o">=</span> <span class="nf">getAutoFixer</span><span class="p">({</span> <span class="na">projectRoot</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/path/to/project</span><span class="dl">'</span> <span class="p">});</span>
<span class="k">await</span> <span class="nx">fixer</span><span class="p">.</span><span class="nf">handleTestFailure</span><span class="p">({</span> <span class="nx">testOutput</span><span class="p">,</span> <span class="nx">failedTests</span> <span class="p">});</span>

<span class="c1">// Code review</span>
<span class="kd">const</span> <span class="nx">reviewer</span> <span class="o">=</span> <span class="nf">getCodeReviewer</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">review</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">reviewer</span><span class="p">.</span><span class="nf">review</span><span class="p">(</span><span class="dl">'</span><span class="s1">src/api.js</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// { qualityScore: 85, issues: [...], suggestions: [...] }</span>

<span class="c1">// Security scanning</span>
<span class="kd">const</span> <span class="nx">scanner</span> <span class="o">=</span> <span class="nf">getSecurityScanner</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">scanner</span><span class="p">.</span><span class="nf">scan</span><span class="p">(</span><span class="dl">'</span><span class="s1">/path/to/project</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// Detects: SQL injection, XSS, hardcoded secrets, weak auth</span>

<span class="c1">// Dependency analysis</span>
<span class="kd">const</span> <span class="nx">analyzer</span> <span class="o">=</span> <span class="nf">getDependencyAnalyzer</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">deps</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">analyzer</span><span class="p">.</span><span class="nf">analyze</span><span class="p">(</span><span class="dl">'</span><span class="s1">/path/to/project</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// { outdated: [...], vulnerabilities: [...], unused: [...] }</span>
</code></pre>

</div>



<h2>
  
  
  The Learning Dashboard
</h2>

<p>Access the Learning Control Center at <code>http://localhost:2505/evolve</code>:</p>

<ul>
<li>
<strong>RL Policy Viewer</strong> - See Q-values for state-action pairs</li>
<li>
<strong>Skill Manager</strong> - Approve/reject generated skills</li>
<li>
<strong>A/B Test Monitor</strong> - Track skill experiments</li>
<li>
<strong>Teaching Sessions</strong> - Watch agents teach each other</li>
<li>
<strong>Pattern Detection</strong> - See recurring behaviors
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GET  /api/learning/stats          # Learning statistics
GET  /api/learning/skills         # Generated skills
GET  /api/learning/policy         # RL Q-table
POST /api/learning/skills/:id/approve  # Approve skill for production
POST /api/learning/teaching/start      # Start teaching session
</code></pre>

</div>



<h2>
  
  
  Cross-Agent Teaching
</h2>

<p>This is my favorite feature. Agents can teach each other:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">getCrossAgentTeacher</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">minions</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">teacher</span> <span class="o">=</span> <span class="nf">getCrossAgentTeacher</span><span class="p">();</span>

<span class="c1">// Stuart (backend expert) creates curriculum</span>
<span class="kd">const</span> <span class="nx">curriculum</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">teacher</span><span class="p">.</span><span class="nf">createCurriculum</span><span class="p">({</span>
  <span class="na">skill</span><span class="p">:</span> <span class="dl">'</span><span class="s1">input-validation</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">levels</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Basic</span><span class="dl">'</span><span class="p">,</span> <span class="na">topics</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">null-checks</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">type-guards</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Advanced</span><span class="dl">'</span><span class="p">,</span> <span class="na">topics</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">schema-validation</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">sanitization</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">});</span>

<span class="c1">// Agnes (frontend agent) learns from Stuart</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">teacher</span><span class="p">.</span><span class="nf">startTeachingSession</span><span class="p">({</span>
  <span class="na">fromAgent</span><span class="p">:</span> <span class="dl">'</span><span class="s1">stuart-backend</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">toAgent</span><span class="p">:</span> <span class="dl">'</span><span class="s1">agnes-frontend</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">skill</span><span class="p">:</span> <span class="dl">'</span><span class="s1">input-validation</span><span class="dl">'</span>
<span class="p">});</span>

<span class="c1">// Track mastery</span>
<span class="kd">const</span> <span class="nx">competencies</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">teacher</span><span class="p">.</span><span class="nf">getAgentCompetencies</span><span class="p">(</span><span class="dl">'</span><span class="s1">agnes-frontend</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// { 'input-validation': { level: 'Advanced', score: 0.92 } }</span>
</code></pre>

</div>



<h2>
  
  
  Why We Built This
</h2>

<p>I've been building AI-assisted development tools for years, and I kept running into the same problems:</p>

<ol>
<li>
<strong>One-shot AI</strong> - Ask a question, get an answer, context lost</li>
<li>
<strong>No memory</strong> - Same mistakes repeated endlessly</li>
<li>
<strong>No collaboration</strong> - AI tools don't talk to each other</li>
<li>
<strong>No autonomy</strong> - Still requires constant human oversight</li>
</ol>

<p>Minions addresses all of these:</p>

<ul>
<li>
<strong>Persistent memory</strong> via SQLite-backed MemoryStore</li>
<li>
<strong>Learning from experience</strong> via Reinforcement Learning</li>
<li>
<strong>Agent collaboration</strong> via EventBus and KnowledgeBrain</li>
<li>
<strong>Autonomous operation</strong> via event-driven architecture</li>
</ul>

<h2>
  
  
  Performance &amp; Scalability
</h2>

<p>Some numbers from our testing:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Event throughput</td>
<td>~1000/min</td>
</tr>
<tr>
<td>Agent execution</td>
<td>50ms - 5min</td>
</tr>
<tr>
<td>Memory (typical)</td>
<td>15-35 MB</td>
</tr>
<tr>
<td>Max concurrent agents</td>
<td>10-20</td>
</tr>
<tr>
<td>Event history</td>
<td>1000 events</td>
</tr>
</tbody>
</table></div>

<p>The framework is designed for development workflows, not production inference. It's optimized for correctness and learning, not raw speed.</p>

<h2>
  
  
  What's Next
</h2>

<p>We're actively working on:</p>

<ul>
<li>
<strong>Multi-project orchestration</strong> - Minions managing microservices</li>
<li>
<strong>Custom LLM adapters</strong> - Beyond Ollama/Gemini</li>
<li>
<strong>Visual workflow builder</strong> - Drag-and-drop agent configuration</li>
<li>
<strong>Plugin marketplace</strong> - Community-contributed agents</li>
<li>
<strong>Cloud deployment</strong> - Managed Minions service</li>
</ul>

<h2>
  
  
  Get Involved
</h2>

<p>Minions is open source and we'd love your contributions:</p>

<ul>
<li>
<strong>GitHub</strong>: <a href="https://github.com/Inweb-eg/minions" rel="noopener noreferrer">github.com/Inweb-eg/minions</a>
</li>
<li>
<strong>Documentation</strong>: Comprehensive docs in <code>/docs</code>
</li>
<li>
<strong>Discord</strong>: Join our community (link in repo)</li>
</ul>

<p>Star the repo if you find it interesting!</p>

<h2>
  
  
  Try It Now
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Clone</span>
git clone https://github.com/Inweb-eg/minions.git
<span class="nb">cd </span>minions

<span class="c"># Install</span>
npm run <span class="nb">install</span>:all

<span class="c"># Run tests</span>
npm <span class="nb">test</span>

<span class="c"># Start Gru</span>
node index.js <span class="nt">--gru</span>

<span class="c"># Visit http://localhost:2505</span>
</code></pre>

</div>



<p>Or with Docker:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>docker
docker compose up <span class="nt">-d</span>
docker <span class="nb">exec </span>minions-ollama ollama pull deepseek-coder:6.7b
docker restart minions
</code></pre>

</div>






<h2>
  
  
  TL;DR
</h2>

<p><strong>Minions</strong> is an open-source framework for building AI agent swarms that:</p>

<ul>
<li>Learn from experience (Q-learning)</li>
<li>Share knowledge (KnowledgeBrain)</li>
<li>Generate new skills (DynamicSkillGenerator)</li>
<li>Teach each other (CrossAgentTeacher)</li>
<li>Fix tests autonomously (AutonomousLoopManager)</li>
<li>Roll back on failure (RollbackManager)</li>
</ul>

<p>26 specialized agents. 80+ event types. One goal: AI that actually helps you ship code.</p>




<p><em>Built with love by Kareem Hussein at Inweb Software Solutions</em></p>

<p>What would you build with self-learning AI agents? Let me know in the comments!</p>

