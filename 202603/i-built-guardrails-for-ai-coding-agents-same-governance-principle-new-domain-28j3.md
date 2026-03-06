---
Title: I built guardrails for AI coding agents — same governance principle, new domain
Description: 
Author: dimitri
Date: 2026-03-06T21:55:21.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  I built guardrails for AI coding agents — same governance principle, new domain
</h2>

<p>Over 25 years of working on complex IT programmes I arrived at a principle I now believe is universal: effective intelligent action — whether by a person, a team, or an AI agent — requires five things. Clear direction. Awareness of context — what happened before, what was decided, what failed. Awareness of resource constraints. Awareness of what your actions will affect downstream. And people who are genuinely engaged and capable of acting. Remove any one and the system degrades.</p>

<p>I did not derive this from AI theory. I derived it from watching transitions succeed and fail. At Shell I built a governance framework for IT transitions — quality gates, assurance areas, structured handovers. Shell adopted it as the global standard. It has been used for over 1,000 transitions worldwide.</p>

<p>When I started building with agentic coding tools I recognised the same failure modes. So I built a framework for that too.</p>

<h2>
  
  
  The problem is structural
</h2>

<p>AI coding agents — Claude Code, Cursor, Copilot, Aider — are capable tools. What they lack is governance. Without it, the same failure modes appear that I have seen in every ungoverned programme:</p>

<p><strong>No traceability.</strong> Files change with no record of why. No task, no decision trail, no audit history. Three weeks later you are reading a diff with no way to reconstruct the reasoning behind it.</p>

<p><strong>No memory.</strong> Every session starts from zero. The agent does not know what it did yesterday, what decisions were made, what failed. You re-explain context repeatedly. Or worse — the agent contradicts a decision from the previous session because it has no record of it.</p>

<p><strong>No risk awareness.</strong> The agent may ask before a force push, but it has no model for understanding why that action is risky, what it affects, or who should approve it. There is no structured authority model — no distinction between what the agent may decide and what requires human approval.</p>

<p><strong>No learning loop.</strong> Failures are not recorded. The same mistake recurs across sessions because there is no mechanism to capture what went wrong and surface it next time.</p>

<p>These are not tool-specific problems. They are governance problems. The same ones I spent two decades solving in enterprise IT.</p>

<h2>
  
  
  What I built
</h2>

<p>The <a href="https://github.com/DimitriGeelen/agentic-engineering-framework" rel="noopener noreferrer">Agentic Engineering Framework</a> applies structural governance to AI coding agents — not guidelines or best practices, but mechanical enforcement.</p>

<p>The core principle: <strong>nothing gets done without a task.</strong> This is enforced as a gate, not a convention. With Claude Code, the framework intercepts every file modification and blocks it unless an active task exists.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Agent attempts to edit a file
    │
    ▼
┌─────────────────────┐
│  Task gate (Tier 1)  │──── No active task? → BLOCKED
└─────────────────────┘
    │ ✓ Task exists
    ▼
┌─────────────────────┐
│  Budget gate         │──── Context &gt; 75%? → BLOCKED (auto-handover)
└─────────────────────┘
    │ ✓ Budget OK
    ▼
    Edit proceeds           Every commit traces to a task
</code></pre>

</div>



<p>This maps directly to those five requirements:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Requirement</th>
<th>Framework mechanism</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Clear direction</strong></td>
<td>Task-first enforcement. Every action has a task with acceptance criteria and verification commands.</td>
</tr>
<tr>
<td><strong>Awareness of context</strong></td>
<td>Context Fabric (the framework's memory subsystem) — three layers of persistent memory (working, project, episodic). The agent recalls prior decisions, learned patterns, and failure resolutions across sessions.</td>
</tr>
<tr>
<td><strong>Awareness of context window</strong></td>
<td>Context budget management tracks resource consumption and triggers automatic handover before the agent loses coherence.</td>
</tr>
<tr>
<td><strong>Awareness of impact</strong></td>
<td>Component Fabric — a live structural map of the codebase. Before changing a file, the agent queries what depends on it and assesses downstream impact.</td>
</tr>
<tr>
<td><strong>Engaged, capable actors</strong></td>
<td>Tiered authority model. The agent has initiative but not authority. Destructive actions require human approval.</td>
</tr>
</tbody>
</table></div>

<p>Tasks flow through a visible lifecycle — Captured, In Progress, Issues, Completed — tracked on a Kanban board that surfaces what needs attention:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6p0gq9vnf7hr1eueorl0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6p0gq9vnf7hr1eueorl0.png" alt="Task Board" width="800" height="2006"></a><br>
<em>Tasks are not hidden in text files. They are visible, trackable, and auditable.</em></p>
<h2>
  
  
  How it works in practice
</h2>

<p>Here is what this looks like in a terminal.</p>

<p><strong>Before governance:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Agent operates without constraints</span>
git add <span class="nb">.</span> <span class="o">&amp;&amp;</span> git commit <span class="nt">-m</span> <span class="s2">"updates"</span>
git push <span class="nt">--force</span> origin main
</code></pre>

</div>



<p>No task reference. No traceability. Destructive command executed without approval.</p>

<p><strong>After governance:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Work starts with a task</span>
fw work-on <span class="s2">"Add JWT validation"</span> <span class="nt">--type</span> build

<span class="c"># Every commit references the task</span>
fw git commit <span class="nt">-m</span> <span class="s2">"T-042: Add JWT validation middleware"</span>

<span class="c"># Destructive commands are intercepted</span>
<span class="nv">$ </span>git push <span class="nt">--force</span>
══════════════════════════════════════════════════════════
  TIER 0 BLOCK — Destructive Command Detected
══════════════════════════════════════════════════════════
  Risk: FORCE PUSH overwrites remote <span class="nb">history
  </span>To proceed: fw tier0 approve <span class="o">(</span>requires human approval<span class="o">)</span>
══════════════════════════════════════════════════════════

<span class="c"># Session ends with context preserved for the next</span>
fw handover <span class="nt">--commit</span>
</code></pre>

</div>



<p>That Tier 0 block is not a warning. It is a gate. Which leads to the question: who has authority over what?</p>

<h2>
  
  
  The authority model
</h2>

<p>In transition management, the single most common failure mode is unclear accountability. Who decides? Who approves? Who can override?</p>

<p>The framework codifies this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Human     → SOVEREIGNTY  → Can override anything, is accountable
Framework → AUTHORITY    → Enforces rules, checks gates, logs everything
Agent     → INITIATIVE   → Can propose, request, suggest — never decides
</code></pre>

</div>



<p>The agent may choose which task to work on. It may choose an implementation approach. It may not bypass a structural gate, complete a human-owned task, or execute a destructive command without approval. Initiative is not authority. This distinction prevents the most dangerous failure mode in agentic systems: the agent making consequential decisions that no one reviewed.</p>

<p>The tiered approval model enforces this mechanically:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tier</th>
<th>Scope</th>
<th>Approval</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>0</strong></td>
<td>Destructive commands (<code>--force</code>, <code>rm -rf</code>, <code>DROP TABLE</code>)</td>
<td>Human must approve</td>
</tr>
<tr>
<td><strong>1</strong></td>
<td>All file modifications</td>
<td>Active task required</td>
</tr>
<tr>
<td><strong>2</strong></td>
<td>Situational exceptions</td>
<td>Single-use, logged</td>
</tr>
<tr>
<td><strong>3</strong></td>
<td>Read-only operations</td>
<td>Pre-approved</td>
</tr>
</tbody>
</table></div>

<p>You do not prevent action. You ensure the right checks occur at the right points.</p>

<p>The gates handle enforcement. But what happens to the knowledge the agent builds up during a session?</p>

<h2>
  
  
  Context Fabric — memory across sessions
</h2>

<p>The most expensive failure in agent-assisted development is not a bug. It is lost context. An agent works for an hour, the session ends, and the next session starts from zero. Decisions are re-made. Mistakes are repeated. The reasoning trail disappears.</p>

<p>The Context Fabric solves this with three layers of persistent memory:</p>

<ul>
<li>
<strong>Working memory</strong> — current session state, active focus, pending actions</li>
<li>
<strong>Project memory</strong> — patterns, decisions, and learnings that persist across all sessions. When the agent encounters a failure it has seen before, the resolution is already there</li>
<li>
<strong>Episodic memory</strong> — condensed histories of every completed task, auto-generated at completion. What was done, what was decided, what was learned</li>
</ul>

<p>Semantic search across all three layers means the agent can recall relevant context by meaning:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>fw recall <span class="s2">"authentication timeout pattern"</span>
<span class="c"># → Returns: L-037 (from T-118), FP-003 (from T-089), episodic T-042</span>
</code></pre>

</div>



<p>Without this, every session is a cold start. With it, the framework accumulates institutional knowledge — the same way it does in a well-run organisation.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwutt944qve3hntm89gvg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwutt944qve3hntm89gvg.png" alt="Watchtower Dashboard" width="800" height="2952"></a><br>
<em>The Watchtower dashboard surfaces tasks awaiting human verification, work direction, and system health in one view.</em></p>
<h2>
  
  
  Component Fabric — structural awareness
</h2>

<p>Memory tells the agent what happened before. But it also needs to know what it is about to affect. In a programme, this is stakeholder impact analysis. In a codebase, it is dependency tracking.</p>

<p>The Component Fabric is a live topology map of every significant file in the project. 126 components across 12 subsystems, with 175 dependency edges tracked. Each component has a YAML card recording what it does, what it depends on, and what depends on it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># What depends on this file?</span>
<span class="nv">$ </span>fw fabric deps agents/git/git.sh
  → 6 dependents: commit.sh, hooks.sh, ...

<span class="c"># What will this commit break downstream?</span>
<span class="nv">$ </span>fw fabric blast-radius HEAD
  → 3 files changed, 12 downstream components potentially affected

<span class="c"># Detect unregistered files (structural drift)</span>
<span class="nv">$ </span>fw fabric drift
  → 2 unregistered files, 0 orphaned cards
</code></pre>

</div>



<p>The difference is between modifying a file without knowing its dependents and modifying it with a verified understanding of downstream impact.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fps5yg8yzbk2yb6rmuahw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fps5yg8yzbk2yb6rmuahw.png" alt="Component Fabric — dependency graph" width="800" height="562"></a><br>
<em>Interactive dependency graph — filter by subsystem, switch layouts, click nodes to inspect relationships.</em></p>
<h2>
  
  
  The healing loop
</h2>

<p>Context and structural awareness handle the forward path. But what about failures?</p>

<p>When a task encounters issues, the framework classifies the failure, searches for similar patterns, and suggests recovery:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>fw healing diagnose T-015            <span class="c"># Classify and suggest</span>
fw healing resolve T-015 <span class="nt">--mitigation</span> <span class="s2">"Added retry logic"</span>  <span class="c"># Record as pattern</span>
</code></pre>

</div>



<p>The escalation ladder is deliberate: <strong>A</strong> — do not repeat the same failure. <strong>B</strong> — improve technique. <strong>C</strong> — improve tooling. <strong>D</strong> — change ways of working. Over 312 completed tasks, these patterns accumulate. Resolutions from prior failures are surfaced when similar issues recur.</p>

<h2>
  
  
  Continuous audit
</h2>

<p>The healing loop handles individual failures. To catch systemic drift, the framework audits itself. 90+ compliance checks run automatically — every 30 minutes, on every push, and on demand:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>fw audit

<span class="o">===</span> SUMMARY <span class="o">===</span>
Pass: 94
Warn: 5
Fail: 2
</code></pre>

</div>



<p>This is the equivalent of assurance reporting. Not retrospective. Continuous. Drift is detected before it becomes a problem.</p>

<h2>
  
  
  Evidence
</h2>

<p>I used the framework to build the framework. 312 tasks completed. 96% commit traceability across the full task history. Every architectural decision recorded with rationale and rejected alternatives.</p>

<p>A typical commit log:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>27e8ed1 T-332: Research awesome list targets — 5 lists with ready-to-submit entries
d8cd81e T-326: Complete README rewrite — all 17 agent ACs + 5 screenshots verified
2138d17 T-329: Draft launch article — I built guardrails for Claude Code
25ba46e T-328: Add NOTICE file for Apache 2.0 attribution preservation
c6287d4 T-328: Add Apache 2.0 license (Geelen &amp; Company) and update README
</code></pre>

</div>



<p>Every commit traces to a task. Every task has acceptance criteria that were verified before completion. Every decision is recorded with rationale. The framework is its own proof of concept.</p>

<p>The framework is built with and tested against Claude Code — that is where the full structural enforcement lives, via hooks that intercept every file modification, every destructive command, every context threshold. But the design is provider-neutral. Cursor gets <code>.cursorrules</code> generation and CLI governance. Copilot, Aider, Devin — any agent that can follow a system prompt and run shell commands gets the same <code>fw</code> CLI. One governance interface, regardless of which agent is executing.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feq8k4g2wxhvocdeowaof.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feq8k4g2wxhvocdeowaof.png" alt="Task detail" width="800" height="2020"></a><br>
<em>A task is a rich artifact — acceptance criteria, verification commands, decisions, and episodic summary. Not a one-line ticket.</em></p>

<h2>
  
  
  Where it stands
</h2>

<p>I use this daily for real work. 312 tasks completed. The governance model holds. The context continuity works. The healing loop genuinely improves over time. I would not go back to ungoverned agent development.</p>

<p>That said, the framework is alpha. It is under active development. There are bugs. There are rough edges. I have taken steps to make it usable for others — install script, Homebrew tap, documentation, GitHub Action — but it has not been tested by a wide audience yet.</p>

<p>If that sounds interesting, try it. If you find bugs, report them. If you see improvements, contribute. This is not a finished product — it is a working framework heading in the right direction.</p>

<h2>
  
  
  Try it
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install</span>
curl <span class="nt">-fsSL</span> https://raw.githubusercontent.com/DimitriGeelen/agentic-engineering-framework/main/install.sh | bash

<span class="c"># Or via Homebrew</span>
brew tap DimitriGeelen/agentic-fw <span class="o">&amp;&amp;</span> brew <span class="nb">install </span>fw

<span class="c"># Initialize in your project</span>
<span class="nb">cd </span>your-project <span class="o">&amp;&amp;</span> fw init

<span class="c"># Start your first governed task</span>
fw work-on <span class="s2">"Set up project structure"</span> <span class="nt">--type</span> build
</code></pre>

</div>



<p>Open source under Apache 2.0: <a href="https://github.com/DimitriGeelen/agentic-engineering-framework" rel="noopener noreferrer">github.com/DimitriGeelen/agentic-engineering-framework</a></p>

<h2>
  
  
  The principle holds
</h2>

<p>Effective intelligent action requires clear direction, context awareness, awareness of constraints and impact, and capable engaged actors. This was true for Shell's global transitions. It is true for AI coding agents. The domain changed. The principle did not.</p>




<p><em>I am interested in how others are approaching governance for AI coding agents. If you have experience — or questions — I would welcome the conversation on <a href="https://github.com/DimitriGeelen/agentic-engineering-framework/discussions" rel="noopener noreferrer">GitHub Discussions</a>.</em></p>

