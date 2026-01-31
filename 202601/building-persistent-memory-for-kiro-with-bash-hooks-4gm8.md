---
Title: Building Persistent Memory for Kiro with Bash Hooks
Description: 
Author: Volodymyr Marynychev
Date: 2026-01-31T22:00:01.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Making Kiro learn from every session</em></p>




<h2>
  
  
  The Problem
</h2>

<p>A few months into using Kiro and Kiro CLI daily, I wanted to review my week.</p>

<p>What problems did I solve? What approaches worked? What did I learn?</p>

<p>Kiro has conversation persistence: you can resume chats, save and load sessions. That's useful. But it's not what I was looking for.</p>

<p>I didn't want to scroll through old conversations. I wanted the <em>insights</em> extracted. The patterns. The solutions that worked and why.</p>

<p>Tuesday's debugging session taught me something about Terraform state locks. But that lesson was buried in a 200-message conversation. Next time I hit the same issue, would I remember to search for it? Would I even remember it happened?</p>

<p>Kiro gives you great infrastructure: agents, hooks, steering files. But no system for capturing what you learn and surfacing it when relevant.</p>

<p>I started asking: What if I built that layer on top of what Kiro already provides?</p>




<h2>
  
  
  The Idea
</h2>

<p>I found Daniel Miessler's <a href="https://github.com/danielmiessler/pai" rel="noopener noreferrer">PAI</a> project - Personal AI Infrastructure for Claude. It clicked.</p>

<p>The idea: an AI assistant that learns from your work and applies that knowledge to future tasks.</p>

<p>What if I could build something similar for Kiro? A layer that:</p>

<ul>
<li>Captures learnings as they happen</li>
<li>Knows my projects and priorities
</li>
<li>Follows a consistent problem-solving approach</li>
</ul>

<p>Not a smarter model. A system that learns.</p>

<p>Kiro already has the building blocks: hooks that run at key moments, steering files for persistent context, agent configurations. I just needed to wire them together.</p>




<h2>
  
  
  What I Built
</h2>

<p>PILOT is an agent for Kiro that adds three things:</p>

<p><strong>1. The Algorithm</strong></p>

<p>Every task follows seven phases:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>OBSERVE → THINK → PLAN → BUILD → EXECUTE → VERIFY → LEARN
</code></pre>

</div>



<p>Not revolutionary. It's how experienced engineers already work. But making it explicit means the AI follows it consistently.</p>

<p>The key insight: Define success criteria <em>before</em> executing. Most people skip this. They try something, vaguely check if it worked, move on.</p>

<p><strong>2. Memory</strong></p>

<p>PILOT captures solutions, but only after verification confirms they work.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>~/.pilot/learnings/
├── 2026-01-15_terraform-state-lock-fix.md
├── 2026-01-14_lambda-timeout-optimization.md
└── 2026-01-12_git-merge-conflict-pattern.md
</code></pre>

</div>



<p>Next time you hit a similar problem, PILOT surfaces the past solution.</p>

<p>For semantic search over learnings, PILOT uses Kiro's <code>/knowledge</code> feature. Without it, PILOT still works but uses keyword matching instead.</p>

<p>Unverified solutions are guesses. Verified solutions are knowledge.</p>

<p><strong>3. Identity</strong></p>

<p>Optional markdown files that give PILOT context about you:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>~/.pilot/identity/
├── MISSION.md       # What you're building
├── GOALS.md         # Current objectives
├── PROJECTS.md      # Active work
└── STRATEGIES.md    # Approaches that work for you
</code></pre>

</div>



<p>You don't have to fill these manually. PILOT observes your work and gradually populates them. The more you use it, the more context it captures: projects you work on, challenges you face, strategies that work for you.</p>

<p>Without identity: "Here's how to fix this Lambda timeout"</p>

<p>With identity: "Given your focus on cost optimization, consider this approach you used last month..."</p>




<h2>
  
  
  How It Works
</h2>

<p>PILOT uses four Kiro features: hooks, resources, agents, and the experimental knowledge base.</p>

<p><strong>Resources</strong> load context into the agent. Kiro supports three types:</p>

<ul>
<li>
<code>file://</code> resources load directly at startup (identity files like MISSION.md, GOALS.md)</li>
<li>
<code>skill://</code> resources load on-demand when relevant (the algorithm, principles)</li>
<li>
<code>knowledgeBase</code> resources enable semantic search over indexed content</li>
</ul>

<p>PILOT indexes your learnings folder as a knowledge base. When you ask about something, it can find related past solutions even if the keywords don't match exactly.</p>

<p>This keeps context lean. The agent sees your mission and goals immediately, but only loads the full algorithm documentation when it needs to reference it.</p>

<p><strong>Hooks</strong> run scripts at key moments. Kiro provides the trigger points, PILOT provides the scripts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────┐
│                      Kiro Session                           │
│                                                             │
│   Kiro Event              PILOT Script                      │
│   ──────────              ────────────                      │
│                                                             │
│   agentSpawn        →     agent-spawn.sh                    │
│                           Load identity, past learnings     │
│                                                             │
│   userPromptSubmit  →     user-prompt-submit.sh             │
│                           Search relevant patterns          │
│                                                             │
│   preToolUse        →     pre-tool-use.sh                   │
│                           Validate before execution         │
│                                                             │
│   [Tool executes]                                           │
│                                                             │
│   postToolUse       →     post-tool-use.sh                  │
│                           Capture results                   │
│                                                             │
│   stop              →     stop.sh                           │
│                           Archive session, save learnings   │
│                                                             │
└─────────────────────────────────────────────────────────────┘

 Kiro provides:          PILOT provides:
 • Hook trigger points   • Scripts that run at each hook
 • Agent system          • Learning capture logic
 • Steering files        • Memory management
                         • Identity context
</code></pre>

</div>



<p><strong>Agents</strong> tie it together. The <code>pilot.json</code> configuration defines:</p>

<ul>
<li>Which hooks to run and when</li>
<li>Which resources to load</li>
<li>The system prompt with the algorithm phases</li>
<li>Tool permissions and safety rules</li>
</ul>

<p>Hooks must exit 0. A crashed hook breaks the session.</p>

<p>It's all shell scripts. No TypeScript, no build tools, no dependencies.</p>

<p>Why bash? Three reasons:</p>

<ul>
<li>No runtime startup cost</li>
<li>Works on macOS and Linux out of the box</li>
<li>Plain text scripts - open any file and see exactly what it does</li>
</ul>




<h2>
  
  
  What Changed
</h2>

<p>Persistent learning compounds.</p>

<p>Early on, the benefit is small: a few captured solutions, some context loaded at startup. But each session adds to the knowledge base. Each verified fix becomes available for next time.</p>

<p>After a few weeks, the difference becomes noticeable. Not because any single feature is transformative, but because the system remembers what you've already figured out.</p>




<h2>
  
  
  The Tradeoffs
</h2>

<p><strong>What works well:</strong></p>

<ul>
<li>Learning capture is automatic</li>
<li>Past solutions surface when relevant</li>
<li>The algorithm keeps work structured</li>
<li>Bash is fast and debuggable</li>
</ul>

<p><strong>What's limited:</strong></p>

<ul>
<li>Each project has isolated memory (no cross-project patterns yet)</li>
<li>Learning detection is keyword-based, not ML</li>
<li>Semantic search requires enabling Kiro's <code>/knowledge</code> feature</li>
</ul>

<p><strong>What's missing:</strong></p>

<ul>
<li>Team collaboration (your learnings stay yours)</li>
<li>Visual dashboard</li>
<li>Cross-project pattern sharing</li>
</ul>

<p>This is an MVP. Try it and see if it fits your workflow.</p>




<h2>
  
  
  Try It
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/requix/pilot
<span class="nb">cd </span>pilot
./install.sh
</code></pre>

</div>



<p>For semantic search over learnings (optional but recommended):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kiro-cli settings chat.enableKnowledge <span class="nb">true</span>
</code></pre>

</div>



<p>Then: <code>kiro-cli --agent pilot</code></p>

<p>The code is simple. The impact compounds over time.</p>




<p><em>Questions? <a href="https://github.com/requix/pilot/issues" rel="noopener noreferrer">Open an issue</a>. The system improves through use.</em></p>

