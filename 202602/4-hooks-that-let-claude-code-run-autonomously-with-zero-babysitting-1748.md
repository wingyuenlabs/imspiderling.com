---
Title: 4 Hooks That Let Claude Code Run Autonomously (With Zero Babysitting)
Description: 
Author: Yurukusa
Date: 2026-02-14T21:36:54.000Z
Robots: noindex,nofollow
Template: index
---
<p>I built a 15,000-line game with Claude Code while sleeping. Here are the 4 hooks that made it possible.</p>

<h2>
  
  
  The Problem
</h2>

<p>Claude Code is powerful, but it stops every few minutes to ask "should I continue?" or "which approach?" In unattended sessions, that means your AI sits idle until you come back.</p>

<h2>
  
  
  The Solution
</h2>

<p><a href="https://github.com/yurukusa/claude-code-ops-starter" rel="noopener noreferrer">Claude Code Ops Starter</a> — 4 production-tested bash hooks that enforce autonomous decision-making.</p>

<h2>
  
  
  What's Inside
</h2>

<h3>
  
  
  1. Context Monitor (<code>context-monitor.sh</code>)
</h3>

<p>Counts tool calls as a proxy for context window usage. Warns at 3 thresholds:</p>

<ul>
<li>
<strong>Soft (80)</strong>: "Consider deferring large tasks"</li>
<li>
<strong>Hard (120)</strong>: "Wrap up and hand off"</li>
<li>
<strong>Critical (150)</strong>: Auto-generates checkpoint file for session handoff
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Configurable thresholds</span>
<span class="nv">SOFT_WARNING</span><span class="o">=</span>80
<span class="nv">HARD_WARNING</span><span class="o">=</span>120
<span class="nv">CRITICAL</span><span class="o">=</span>150
</code></pre>

</div>



<h3>
  
  
  2. No-Ask-Human (<code>no-ask-human.sh</code>)
</h3>

<p>Blocks all <code>AskUserQuestion</code> tool calls. Instead of stopping to ask, the AI:</p>

<ol>
<li>Decides on its own</li>
<li>Logs uncertainty to <code>~/pending_for_human.md</code>
</li>
<li>Moves to the next task</li>
</ol>

<p>Override with <code>CC_ALLOW_QUESTIONS=1</code> when you want questions back.</p>

<h3>
  
  
  3. Syntax Check (<code>syntax-check.sh</code>)
</h3>

<p>Auto-runs verification after every file edit:</p>

<ul>
<li>Python: <code>python -m py_compile</code>
</li>
<li>Shell: <code>bash -n</code>
</li>
<li>JSON: <code>jq empty</code>
</li>
</ul>

<p>Catches errors immediately instead of 50 tool calls later. Gracefully skips if <code>jq</code> or <code>python</code> aren't installed.</p>

<h3>
  
  
  4. Decision Warn (<code>decision-warn.sh</code>)
</h3>

<p>Flags <code>rm -rf</code>, <code>git reset --hard</code>, <code>DROP TABLE</code> before execution. Warns but doesn't block — uncomment one line for hard blocking.</p>

<h2>
  
  
  Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/yurukusa/claude-code-ops-starter.git
<span class="nb">cd </span>claude-code-ops-starter
bash install.sh
</code></pre>

</div>



<p>No dependencies besides bash. <code>jq</code> and <code>python</code> are optional (hooks skip gracefully without them).</p>

<h2>
  
  
  How It Works
</h2>

<p>Claude Code has a <a href="https://docs.anthropic.com/en/docs/claude-code/hooks" rel="noopener noreferrer">hooks system</a> that runs shell scripts at specific lifecycle points:</p>

<ul>
<li>
<strong>PreToolUse</strong>: Runs before a tool call (can block it with exit 1)</li>
<li>
<strong>PostToolUse</strong>: Runs after a tool call (for monitoring and validation)</li>
</ul>

<p>The starter kit uses both:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Hook</th>
<th>Trigger</th>
<th>Effect</th>
</tr>
</thead>
<tbody>
<tr>
<td>no-ask-human.sh</td>
<td>PreToolUse (AskUserQuestion)</td>
<td>Blocks with exit 1</td>
</tr>
<tr>
<td>decision-warn.sh</td>
<td>PreToolUse (Bash)</td>
<td>Warns (exit 0)</td>
</tr>
<tr>
<td>syntax-check.sh</td>
<td>PostToolUse (Edit/Write)</td>
<td>Validates (exit 0)</td>
</tr>
<tr>
<td>context-monitor.sh</td>
<td>PostToolUse (all)</td>
<td>Monitors (exit 0)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Background
</h2>

<p>These hooks come from 200+ hours of real autonomous Claude Code operation. Every hook solved a real problem:</p>

<ul>
<li>The context monitor was born from sessions that ran out of context without warning</li>
<li>No-ask-human from overnight sessions where the AI sat idle for hours waiting for a human answer</li>
<li>Syntax check from discovering errors 50 tool calls after they were introduced</li>
<li>Decision warn from a <code>git reset --hard</code> that wiped an afternoon of work</li>
</ul>

<p>The full setup (multi-agent orchestration, stall detection, watchdog, task queue, 20+ hooks) is available as the <a href="https://yurukusa.gumroad.com/l/cc-codex-ops-kit?utm_source=devto&amp;utm_medium=article&amp;utm_campaign=ops-kit-launch" rel="noopener noreferrer">CC-Codex Ops Kit</a>.</p>




<p><a href="https://github.com/yurukusa/claude-code-ops-starter" rel="noopener noreferrer">GitHub</a> | <a href="https://x.com/yurukusa_dev" rel="noopener noreferrer">@yurukusa_dev on X</a></p>

