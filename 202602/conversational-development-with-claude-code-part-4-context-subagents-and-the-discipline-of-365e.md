---
Title: Conversational Development With Claude Code — Part 4: Context, Subagents, and the Discipline of Thinking
Description: 
Author: Cristian Sifuentes
Date: 2026-02-02T21:39:03.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcs6ccoqqdpnciyd37xsc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcs6ccoqqdpnciyd37xsc.png" alt="Conversational Development With Claude Code — Part 4" width="800" height="533"></a></p>

<h2>
  
  
  Conversational Development With Claude Code — Part 4
</h2>

<p><strong>TL;DR</strong> — Claude Code is not powered by perfect prompts, but by well‑designed context. To work with intention and precision, you must understand five pillars: the context window, subagents, the Model Context Protocol (MCP), the CLI, and context engineering. Together, they explain how Claude thinks, remembers, and collaborates — and how you can build reliable systems without micromanaging an LLM.</p>




<h2>
  
  
  Why Context Beats Prompts
</h2>

<p>Most failures with AI tooling do not come from weak models, but from poorly designed context. Developers chase the mythical “perfect prompt” when the real leverage lies elsewhere: in what the system can <em>see</em>, <em>remember</em>, and <em>prioritize</em>.</p>

<p>Claude Code is designed for engineers who think in systems, not incantations. It rewards structure, references, and intent — not verbosity.</p>




<h2>
  
  
  The Context Window: Claude’s Working Memory
</h2>

<p>The context window is Claude Code’s working memory. It contains everything the model can reason about at once: source code, conversations, command outputs, and intermediate results.</p>

<p>By default, Claude Code operates with <strong>~200,000 tokens</strong>, with an optional <strong>1M‑token window</strong> for larger projects. That scale is enough to analyze medium‑to‑large codebases without artificial fragmentation.</p>

<p>Key properties of the context window:</p>

<ul>
<li>It holds code, conversation, and execution results together</li>
<li>It prioritizes relevance over chronology</li>
<li>It automatically evicts low‑value information when space is constrained</li>
</ul>

<p>This is not long‑term memory. It is active cognition.</p>

<p>When the window fills, Claude does not panic — it compresses. Your job is to ensure what remains is what matters.</p>




<h2>
  
  
  Memory Hygiene: References Over Copy‑Paste
</h2>

<p>The fastest way to poison context is to paste massive blocks of code.</p>

<p>Instead, Claude Code is designed around <strong>references</strong>:</p>

<ul>
<li>Reference files using <code>@path/to/file</code>
</li>
<li>Let Claude load only what it needs</li>
<li>Re‑anchor decisions explicitly when starting new sessions</li>
</ul>

<p>If Claude begins to “forget,” the fix is not repetition — it is <strong>reset and refocus</strong>.</p>

<p>Open a new session. Re‑reference the critical files. Restate architectural constraints. Continue.</p>

<p>Clean context beats persistent noise.</p>




<h2>
  
  
  Subagents: A Modular Team, Not a Monolith
</h2>

<p>Subagents are where Claude Code stops feeling like a tool and starts behaving like a team.</p>

<p>Each subagent operates with:</p>

<ul>
<li>Its own isolated context</li>
<li>Specialized instructions</li>
<li>Dedicated tools</li>
</ul>

<p>Think of them as focused collaborators:</p>

<ul>
<li>
<strong>Architecture agent</strong> — analyzes system structure and constraints</li>
<li>
<strong>Backend agent</strong> — designs and implements APIs</li>
<li>
<strong>Frontend agent</strong> — builds UI components and state flows</li>
<li>
<strong>QA agent</strong> — validates behavior, tests, and edge cases</li>
</ul>

<p>The main session becomes a coordinator, not a bottleneck.</p>

<p>This mirrors real engineering teams: specialization without fragmentation.</p>




<h2>
  
  
  When to Invoke Each Subagent
</h2>

<p>Use subagents deliberately:</p>

<ul>
<li>Backend agent → endpoints, schemas, integrations</li>
<li>Frontend agent → components, UX flows, state management</li>
<li>QA agent → tests, regressions, validation</li>
<li>Architecture agent → refactors, boundaries, long‑term decisions</li>
</ul>

<p>If your main conversation feels crowded, you waited too long.</p>




<h2>
  
  
  MCP and the CLI: The Nervous System
</h2>

<p>Claude Code lives in the terminal. MCP (Model Context Protocol) extends its reach.</p>

<p>Together, they form a tight loop:</p>

<ul>
<li>MCP connects Claude to real tools</li>
<li>The CLI executes commands and returns truth</li>
<li>Claude interprets, not guesses</li>
</ul>

<p>With MCP, Claude can:</p>

<ul>
<li>Query databases</li>
<li>Run automated tests (e.g. Playwright)</li>
<li>Inspect GitHub issues and pull requests</li>
<li>Interact with tools like Figma or Zapier</li>
</ul>

<p>Once configured, these integrations feel native — not bolted on.</p>




<h2>
  
  
  Using the CLI Without Breaking Flow
</h2>

<p>The CLI is not a side channel; it is the execution layer.</p>

<p>Best practices:</p>

<ul>
<li>Prefix commands with <code>!</code> to execute</li>
<li>Let Claude read and interpret outputs</li>
<li>
<p>Keep roles clear:</p>

<ul>
<li>Terminal → execution</li>
<li>Editor → writing</li>
<li>Browser → validation</li>
</ul>


</li>

</ul>

<p>Claude reasons <em>after</em> commands run. That distinction matters.</p>




<h2>
  
  
  Context Engineering: Designing Shared Thought
</h2>

<p>Context engineering is not prompt engineering.</p>

<p>It is the discipline of shaping a shared thinking space.</p>

<p>In practice:</p>

<ul>
<li>Define purpose before action</li>
<li>State constraints explicitly</li>
<li>Reference files instead of pasting code</li>
<li>Specify quality bars and anti‑patterns</li>
<li>Remove irrelevant information aggressively</li>
</ul>

<p>Avoid vague instructions. Ambiguity creates noise.</p>

<p>Precision creates leverage.</p>




<h2>
  
  
  Final Thoughts
</h2>

<p>Claude Code does not replace judgment — it amplifies it.</p>

<p>When you treat context as an architectural artifact, not a side effect, the system becomes predictable, composable, and trustworthy.</p>

<p>You stop chasing prompts.<br>
You start designing cognition.</p>

<p>That is the real upgrade.</p>

<p>— Written by <strong>Cristian Sifuentes</strong><br>
Senior Software Engineer · AI‑Driven Architecture</p>

