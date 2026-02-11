---
Title: Your Coding Agent Doesn't Need a Bigger Context Window. It Needs Coworkers.
Description: 
Author: 정재혁
Date: 2026-02-11T21:00:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been building with Claude Code for months. It's genuinely impressive — until your codebase gets big enough that the agent starts drowning in its own context.</p>

<p>The 1M token context window sounds huge. But feed it a real project — a few hundred files, import chains six layers deep, config scattered across yaml and env files — and you start hitting walls. Responses slow down. Quality degrades. Costs climb. The model spends half its tokens reading code it doesn't need for your question.</p>

<p>The instinct is to wait for bigger context windows. But I think that's the wrong fix. A smarter CTO doesn't mean the CTO should personally review every line of code.</p>

<p>So I built AgentHub: a lightweight Python library that splits your codebase into specialized agents, routes queries to the right one, and lets them collaborate when a question spans multiple domains.</p>

<p>Here's how it works, layer by layer.</p>

<h2>
  
  
  The Problem in Practice
</h2>

<p>Say you're working on a Django project. You ask your coding agent: "How does the payment flow work?" The agent now has to process your auth middleware, your URL routing, your serializers, your payment service, your Stripe webhooks, your database models, your test fixtures — all of it loaded into one context window, most of it irrelevant.</p>

<p>The real answer lives in maybe 4-5 files across 2-3 modules. But the agent doesn't know that upfront. So it ingests everything, burns through tokens, and gives you a diluted answer because it's trying to hold too many things in its head at once.</p>

<p>This is a routing problem, not a capacity problem.</p>

<h2>
  
  
  Layer 1: Auto-Generated Domain Agents
</h2>

<p>The first insight is that your codebase already tells you where the natural boundaries are. Folders map to domains. Files map to responsibilities. Import graphs map to dependencies. Why not use that structure to create agents automatically?</p>

<p>AgentHub's <code>build</code> command scans your repo and generates what I call <strong>Tier B agents</strong> — one per logical domain, each preloaded with only the files it owns.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjdi0f5vk0tq51dp6w39o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjdi0f5vk0tq51dp6w39o.png" alt=" " width="800" height="444"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">agenthub</span> <span class="kn">import</span> <span class="n">AgentHub</span>
<span class="kn">from</span> <span class="n">agenthub.auto</span> <span class="kn">import</span> <span class="n">discover_all_agents</span>

<span class="n">hub</span><span class="p">,</span> <span class="n">summary</span> <span class="o">=</span> <span class="nf">discover_all_agents</span><span class="p">(</span><span class="sh">"</span><span class="s">./my-project</span><span class="sh">"</span><span class="p">)</span>
<span class="n">response</span> <span class="o">=</span> <span class="n">hub</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="sh">"</span><span class="s">How does user authentication work?</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>Under the hood, the build step does three things. First, it walks your file tree and maps the structure — which folders exist, how big they are, what languages they use. Second, it builds an import graph by parsing <code>import</code> statements across your codebase, so it knows that <code>api/views.py</code> depends on <code>services/auth.py</code> which depends on <code>models/user.py</code>. Third, it uses those two signals to create focused agents, each with a context window loaded only with the files in its domain.</p>

<p>When a query comes in, a router scores it against each agent's keywords and domain, and dispatches to the best match. The <code>src/api/</code> agent answers API questions. The <code>src/models/</code> agent answers schema questions. No wasted tokens.</p>

<p>There's also a <strong>Tier A</strong> layer — hand-written agents for business logic that can't be inferred from code structure. A pricing agent that knows your margin rules. An analytics agent that knows your KPI definitions. These are optional but powerful when you need domain expertise that lives in people's heads, not in source files.</p>

<h2>
  
  
  Layer 2: DAG Teams for Cross-Cutting Questions
</h2>

<p>Single-agent routing works great for scoped questions ("What does the <code>UserSerializer</code> class do?"). It breaks down for cross-cutting ones ("How does the checkout flow work end to end?"), where the answer lives across your API layer, your service layer, your models, and your payment integration.</p>

<p>For these, AgentHub spins up a <strong>DAG team</strong>. The process works like this: a complexity classifier detects that the query spans multiple domains. A decomposer identifies which agents are relevant. The import graph provides the dependency edges between them. Independent agents execute in parallel, dependent ones run in sequence, and a synthesizer merges their results into a coherent answer.</p>

<p>The key decision here was making the DAG dynamic rather than hardcoded. It's built per-query from the import graph, so the collaboration structure reflects how your code actually connects, not how you think it connects.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxzkbyofodbrp8h3ub0p7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxzkbyofodbrp8h3ub0p7.png" alt=" " width="800" height="414"></a></p>

<h2>
  
  
  Layer 3: Parallel Sessions
</h2>

<p>This is where it gets ambitious. When a developer says "Add a save button to the toolbar and also build a time series chart component," those are two independent tasks touching completely different files. Today, coding agents handle them sequentially. But why?</p>

<p>AgentHub's parallel sessions layer decomposes multi-part requests into tasks, analyzes file overlap using the import graph, and when tasks are independent, spins up separate Claude Code sessions on separate git branches. Each session is scoped — it only sees the files relevant to its task. When all sessions complete, a merge coordinator brings the branches together, running tests to catch semantic conflicts that git can't detect.</p>

<p>I think of this as the <strong>company model</strong>. When a startup is two people, the CTO (your coding agent) knows everything. As the company grows, you hire specialists (Tier B agents). When departments get big, you organize teams (DAG teams). And when there's a company-wide initiative, teams work simultaneously on their own tracks, syncing at boundaries.</p>

<p>The merge step is where it gets tricky. Textual conflicts are easy — git handles those. Semantic conflicts are harder: two branches that merge cleanly but break each other's assumptions. For those, AgentHub runs the test suite post-merge. If tests fail, a resolution agent examines the failure, traces it back to the conflicting changes, and either fixes it or escalates to the developer.</p>

<h2>
  
  
  Where It Stands
</h2>

<p>AgentHub is at v0.1.0. The core auto-agent generation, DAG teams, and parallel session infrastructure are all implemented. It ships as a Python CLI and integrates with Claude Code via MCP, so the agents show up as tools right in your terminal.</p>

<p>The honest status: auto-agent generation works well — point it at a repo and you get useful agents. DAG teams work but need tuning on the decomposer prompts. The routing layer — deciding which agent gets a query — still needs some manual tweaking to perform reliably across different codebases. And parallel sessions are the most experimental; the git orchestration is solid, but semantic conflict resolution is still rough around the edges.</p>

<p>I'm not open-sourcing this yet. There's enough rough edges that I want to get the routing and agent quality to a place I'm happy with before putting it out there. But I'm sharing the architecture because I think the core ideas — using import graphs for agent boundaries, DAG-based collaboration, the company growth model — are useful regardless of implementation.</p>

<p>If you're building with coding agents and hitting context limits, I'd love to hear how you're solving it.</p>




<p><em>Built by John. The entire AgentHub codebase was developed collaboratively with Claude — which felt appropriately recursive.</em></p>

