---
Title: I built an MCP server that forces AI to spec before it codes
Description: 
Author: HendryCode
Date: 2026-02-25T22:05:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>I built an MCP server in Go that gives your AI persistent memory, structured specifications, and an adaptive change pipeline. It's called <strong>Hoofy</strong>. It's open source. And its only reason to exist is to make your AI build what YOU want, not what it hallucinates.</p>

<p>The Problem Nobody Wants to Admit</p>

<p>We're all using AI to code. And we're all suffering the same things:</p>

<ul>
<li>Ask for a feature → get something that looks right but isn't</li>
<li>New session → AI has no idea what you did yesterday</li>
<li>"Fix this bug" → AI rewrites half your codebase</li>
<li>Complex request → invented APIs, mismatched schemas, architecture that contradicts your existing code</li>
</ul>

<p>The METR 2025 study confirmed it with data: experienced developers were 19% slower with AI, despite feeling 20% faster. DORA 2025 found a 7.2% increase in delivery instability for every 25% of AI adoption without structure. McKinsey says only those using structured specifications see real improvements (16-30%).</p>

<p>AI without structure is a junior with blind confidence.</p>

<p>What is Hoofy</p>

<p>Hoofy is an <strong>MCP server</strong> — a single binary, written in Go, zero external dependencies, with embedded SQLite. Connect it to any AI tool that supports MCP (Claude Code, Cursor, VS Code Copilot, OpenCode, Gemini CLI) and it gives your AI three superpowers:</p>

<ol>
<li>Persistent Memory (17 tools)</li>
</ol>

<p>Your AI finally remembers. Architecture decisions, bugs you fixed, patterns you established, technical discoveries — everything is saved in SQLite with full-text search (FTS5) and a <strong>knowledge graph</strong> with typed relations.</p>

<p>These aren't flat notes. You can connect observations<br>
When the AI starts a new session, it loads this context automatically. It already knows where it was.</p>

<ol>
<li>Adaptive Change Pipeline (6 tools)</li>
</ol>

<p>Here's the magic for day-to-day work. When you need to make a change, Hoofy automatically selects the right steps based on <strong>type × size</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Small</th>
<th>Medium</th>
<th>Large</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Fix</strong></td>
<td>4 stages</td>
<td>5 stages</td>
<td>6 stages</td>
</tr>
<tr>
<td><strong>Feature</strong></td>
<td>4 stages</td>
<td>5 stages</td>
<td>7 stages</td>
</tr>
<tr>
<td><strong>Refactor</strong></td>
<td>4 stages</td>
<td>5 stages</td>
<td>5 stages</td>
</tr>
<tr>
<td><strong>Enhancement</strong></td>
<td>4 stages</td>
<td>5 stages</td>
<td>7 stages</td>
</tr>
</tbody>
</table></div>

<p>12 flows, all deterministic. A small bugfix doesn't need the same ceremony as a new authentication system. But ALL of them start with a <strong>context-check</strong> — Hoofy scans your existing specs, completed changes, memory, and convention files to detect conflicts BEFORE writing a single line.</p>

<ol>
<li>Greenfield Project Pipeline (9 tools)</li>
</ol>

<p>For when you're starting from scratch. The full pipeline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Init → Propose → Requirements → Business Rules → Clarity Gate → Design → Tasks → Validate
</code></pre>

</div>



<p>The Clarity Gate is the core innovation. It analyzes your requirements across 8 dimensions (users, functionality, data model, integrations, edge cases, security, scale, scope) and blocks advancement until ambiguities are resolved.</p>

<p>The Business Rules stage extracts declarative rules using BRG taxonomy (Definitions, Facts, Constraints, Derivations) and DDD Ubiquitous Language — before the Clarity Gate evaluates them.</p>

<p>Validate at the end cross-checks all artifacts: every requirement has at least one task, every design component has assigned tasks, nothing goes out of the proposal's scope.</p>

<p>What Hoofy is NOT</p>

<ul>
<li>It doesn't generate code. Hoofy generates specifications. The AI generates code AFTER, using those specs as guardrails.</li>
<li>It doesn't replace the developer. You still make the decisions. Hoofy forces the AI to ASK you before assuming.</li>
<li>It's not exclusive to one tool. It's standard MCP — works with any IDE/tool that supports it.</li>
</ul>




<p>The Research Behind It</p>

<p>Hoofy isn't built on opinions:</p>

<ul>
<li>
<strong><a href="https://ieeexplore.ieee.org/document/720574" rel="noopener noreferrer">IEEE 29148</a></strong> and <strong>IREB</strong>: Requirements engineering standards for structured elicitation and ambiguity detection.</li>
<li>
<strong><a href="https://www.businessrulesgroup.org/brmanifesto.htm" rel="noopener noreferrer">Business Rules Group (BRG)</a></strong>: The Business Rules Manifesto establishes that rules are first-class citizens, not buried in code.</li>
<li>
<strong>EARS (Easy Approach to Requirements Syntax)</strong>: Research-backed templates that eliminate ambiguity in natural language requirements.</li>
<li>
<strong>DDD Ubiquitous Language</strong>: Eric Evans' principle that a shared language eliminates translation errors.</li>
</ul>

<p>A requirements error costs <strong>10-100x more</strong> to fix in production than during specification (IEEE). With AI-generated code, that multiplier is worse.</p>




<p>Quick Start</p>

<p>Install:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># macOS (Homebrew)</span>
brew <span class="nb">install </span>HendryAvila/hoofy/hoofy

<span class="c"># macOS/Linux (script)</span>
curl <span class="nt">-sSL</span> https://raw.githubusercontent.com/HendryAvila/Hoofy/main/install.sh | bash

<span class="c"># Go</span>
go <span class="nb">install </span>github.com/HendryAvila/Hoofy/cmd/hoofy@latest
</code></pre>

</div>



<p>Connect (example with Claude Code):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>claude mcp add <span class="nt">--scope</span> user hoofy hoofy serve
</code></pre>

</div>



<p>Done. Just talk to your AI. Hoofy's built-in instructions tell it when and how to use each system.</p>




<h2>
  
  
  By the Numbers
</h2>

<ul>
<li>
<strong>32 tools</strong> MCP</li>
<li>
<strong>12 flow variants</strong> in the change pipeline</li>
<li>
<strong>8 dimensions</strong> of clarity</li>
<li>
<strong>6 relation types</strong> in the knowledge graph</li>
<li>One binary. Zero external dependencies. MIT license.</li>
</ul>




<p><strong>GitHub</strong>: <a href="https://github.com/HendryAvila/Hoofy" rel="noopener noreferrer">github.com/HendryAvila/Hoofy</a></p>

<p><strong>Stop prompting. Start specifying.</strong></p>




<p><em>I'm a self-taught developer — started as a NOC operator and learned to code on my own. Hoofy is my first open source Go project. Feedback and contributions are welcome!</em></p>

