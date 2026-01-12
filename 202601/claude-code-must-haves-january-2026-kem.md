---
Title: Claude Code Must-Haves - January 2026
Description: 
Author: Sven Pöche
Date: 2026-01-12T21:33:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>October 2025, I was facing a change. My Anthropic API costs had grown too high—partly due to my RooCode Agile Software Development Team Framework (a multi-agent system I'd built for AI-assisted development). My boss suggested an alternative: Claude Code with a team subscription. Within days, I had access to an ecosystem that was just beginning to unfold.</p>

<p>The first weeks were overwhelming. Claude Code was a moving target: the plugin system launched in October while I was already using it productively. Skills, MCPs, Hooks—everywhere new possibilities, but also decisions. What do you really need? What's nice-to-have? And how do you prevent the context window from filling up with unnecessary tools?</p>

<p>After three months of intensive exploration—from October 2025 to today, January 2026—I've gained clarity. Not through perfect planning, but through iteration. By trying, discarding, reconfiguring. By discovering game-changers like the Superpowers plugin in late November, which fundamentally changed how I work.</p>

<p>This article isn't a complete feature list or a beginner's tutorial. It's a curated collection of my must-haves—tested in real projects, optimized for context efficiency, proven in daily use. As of January 2026, knowing that Claude Code will continue to evolve. But this foundation works. For me. And maybe it's a good starting point for your own workflow.</p>




<blockquote>
<p><strong>🚨 Critical Security Update - Claude Code 2.1.0</strong></p>

<p>While finalizing this article, <strong>Claude Code 2.1.0</strong> was released (January 2026) with a <strong>critical security fix</strong>: Versions &lt; 2.1.0 exposed OAuth tokens, API keys, and passwords in debug logs.</p>

<p><strong>Action Required:</strong></p>

<ul>
<li>⚠️ <strong>Update immediately to version 2.1.0+</strong>: <code>npm update -g @anthropic-ai/claude-code</code>
</li>
<li>🔒 Review and delete existing debug logs</li>
<li>🔑 Rotate exposed credentials (API keys, tokens)</li>
</ul>

<p>Version 2.1.0 also brings game-changing features: <strong>Automatic Skill Hot-Reload</strong> (skills instantly available without restart), <strong>Skill Context Forking</strong> (isolated sub-agent contexts), <strong>Hooks in Skill Frontmatter</strong>, <strong>Bash Wildcard Permissions</strong> (<code>Bash(npm *)</code>), and <strong>Language Configuration</strong>. These features are mentioned at relevant sections throughout the article.</p>
</blockquote>




<h2>
  
  
  Quick Setup: The Absolute Essentials
</h2>

<p>Before we dive into details, here's what you should start with.</p>

<p><strong>The 2 absolute must-haves for EVERYONE:</strong></p>

<p>✅ <strong>Language Server for your main language</strong> — Code intelligence isn't optional. Without real-time type information, Claude works in the dark. With a language server, Claude sees type errors immediately after edits, understands semantic relationships, and can use "Go to Definition" and "Find References." This is the single biggest productivity gain.</p>

<p>✅ <strong>Superpowers Plugin</strong> — Transforms Claude from helper to senior developer. 20+ production-proven skills for structured workflows: Test-Driven Development, Systematic Debugging, Structured Planning. Since my discovery in late November, I use it daily. Game-changer.</p>

<p><strong>Also recommended:</strong></p>

<p>✅ <strong>Context7 MCP</strong> — Up-to-date documentation without token overhead. Real-time docs, version-specific, from official sources. Essential for every new library. (API key optional but recommended)</p>

<p>✅ <strong>Sequential Thinking MCP</strong> — Structured problem-solving for complex tasks. LLM-based methodical thinking, like a senior architect who thinks first, then codes. Best used together with Context7.</p>

<p><strong>Installation (copy-paste ready):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Language Server (examples for common languages)</span>
<span class="c"># Note: Requires claude-code-lsps marketplace</span>
/plugin marketplace add boostvolt/claude-code-lsps
/plugin <span class="nb">install </span>pyright@claude-code-lsps        <span class="c"># Python</span>
/plugin <span class="nb">install </span>vtsls@claude-code-lsps          <span class="c"># TypeScript/JavaScript</span>
/plugin <span class="nb">install </span>rust-analyzer@claude-code-lsps  <span class="c"># Rust</span>
/plugin <span class="nb">install </span>gopls@claude-code-lsps          <span class="c"># Go</span>

<span class="c"># Superpowers Plugin</span>
/plugin marketplace add obra/superpowers-marketplace
/plugin <span class="nb">install </span>superpowers

<span class="c"># Context7 MCP (API key optional but recommended for higher rate limits)</span>
claude mcp add context7 <span class="nt">--</span> npx <span class="nt">-y</span> @upstash/context7-mcp@latest

<span class="c"># Sequential Thinking MCP</span>
claude mcp add @modelcontextprotocol/server-sequential-thinking
</code></pre>

</div>



<p>That's the foundation. Everything else comes as needed. Less is more—especially for context management.</p>




<h2>
  
  
  Plugins: The 4 Must-Haves
</h2>

<p>The plugin system launched in October 2025. After three months of intensive use: These 4 plugins are essential.</p>

<h3>
  
  
  1. Language Server for Your Main Language
</h3>

<p><strong>Available:</strong> pyright (Python), vtsls (TypeScript/JavaScript), rust-analyzer (Rust), gopls (Go)</p>

<p>Language Server Protocol (LSP) gives Claude real-time type information and semantic code understanding. This means: "Go to Definition", "Find References", type errors visible immediately after edits. Without LSP, Claude works with text files. With LSP, Claude understands the codebase as a structured system.</p>

<p>My experience: <strong>The single biggest productivity gain.</strong> Claude sees type errors immediately after edits, before tests run. Refactorings become safer because Claude finds all usages of a function. The difference is so large that I don't want to work without LSP anymore.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation (add marketplace first)</span>
/plugin marketplace add boostvolt/claude-code-lsps

<span class="c"># Then install plugin (example: Python)</span>
/plugin <span class="nb">install </span>pyright@claude-code-lsps

<span class="c"># Or TypeScript/JavaScript (vtsls, not typescript-lsp)</span>
/plugin <span class="nb">install </span>vtsls@claude-code-lsps
</code></pre>

</div>



<h3>
  
  
  2. Superpowers
</h3>

<p><strong>What it is:</strong> 20+ production-proven skills for structured workflows—Test-Driven Development, Systematic Debugging, Root Cause Tracing, Structured Planning. Born from real practice, tested and iterated by the community.</p>

<p>Superpowers transforms Claude from reactive helper to proactive senior developer. Instead of just responding to requests, Claude follows structured workflows: Test first, then implement. Find root cause first, then fix. Brainstorm first, then code.</p>

<p>Why essential: The skills encode best practices that would otherwise be lost. Red-Green-Refactor not as theory, but as enforced workflow. Debugging not as trial-and-error, but as a four-phase framework. The result: higher code quality, less rework, better architecture decisions.</p>

<p>My usage: <strong>Daily since discovery in late November.</strong> Indispensable. The wrapper pattern (commands as entry points, skills as implementation) influenced my own skill development. More on this in the "Skills &amp; Commands" section.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation</span>
/plugin marketplace add obra/superpowers-marketplace
/plugin <span class="nb">install </span>superpowers
</code></pre>

</div>



<h3>
  
  
  3. commit-commands
</h3>

<p><strong>What it is:</strong> Git workflow automation. Intelligent commit messages based on actual changes, automated pull requests with context-aware descriptions.</p>

<p><strong>Features:</strong> Analyzes <code>git diff</code> and <code>git log</code> for commit message suggestions, creates PRs with summary and test plan, follows repository-specific commit conventions.</p>

<p><strong>Value:</strong> Eliminates manual Git workflows. No more hand-written commit messages that sound generic after 20 minutes of coding. No PRs where you forget what actually changed. Claude sees all changes and summarizes them structurally.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation</span>
/plugin <span class="nb">install </span>commit-commands
</code></pre>

</div>



<h3>
  
  
  4. pr-review-toolkit
</h3>

<p><strong>What it is:</strong> Multi-agent code reviews with confidence scoring. Multiple specialized review agents check different aspects: tests, error handling, type design, code quality.</p>

<p><strong>Features:</strong> Each agent gives feedback with confidence score (0-100%). Filters false positives through multi-agent consensus. Creates structured review reports with categorized findings.</p>

<p><strong>Value:</strong> Massively increases code quality before every merge. Not as a replacement for human review, but as a first filter. Finds edge cases that would be overlooked in manual review. Especially valuable for solo developers without team review.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation</span>
/plugin <span class="nb">install </span>pr-review-toolkit
</code></pre>

</div>






<h2>
  
  
  MCP Servers: The Essentials
</h2>

<p>MCP servers extend Claude Code with external integrations. But: <strong>Less is more.</strong> Too many MCPs consume context—sometimes over 40% of the 200k context window just for MCP tools at startup. The art is choosing the right MCPs.</p>

<h3>
  
  
  1. Context7 - Up-to-date Documentation (MUST-HAVE)
</h3>

<p><strong>Function:</strong> Real-time, version-specific documentation directly in prompts. No outdated docs, no manual searching, no token overhead from copy-paste.</p>

<p><strong>Features:</strong></p>

<ul>
<li>Official sources (GitHub, official docs)</li>
<li>Version-specific content (e.g., Next.js 14.3.0-canary.87)</li>
<li>Millisecond response time</li>
<li>Resolve Library ID + Query Docs workflow</li>
</ul>

<p><strong>Why essential:</strong> Claude's knowledge cutoff is January 2025. Frameworks change monthly. Context7 solves this: current docs without you having to manually search and copy them into prompts. This saves tokens and prevents hallucinations based on outdated information.</p>

<p>My usage: <strong>Essential for every new library.</strong> Especially with framework updates (React 19, Next.js 15), Context7 is the difference between working code and hours of debugging due to deprecated APIs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation</span>
claude mcp add context7 <span class="nt">--</span> npx <span class="nt">-y</span> @upstash/context7-mcp@latest

<span class="c"># Optional: API key for higher rate limits</span>
<span class="c"># Register at context7.com/dashboard and add key to config</span>
</code></pre>

</div>



<p><strong>Usage in prompts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>I'm using Next.js 15.1.0. Show me how the new Server Actions work.
</code></pre>

</div>



<p>Context7 automatically resolves <code>/vercel/next.js/v15.1.0</code> and delivers current docs.</p>

<h3>
  
  
  2. Sequential Thinking - Structured Problem-Solving
</h3>

<p><strong>Function:</strong> LLM-based structured thinking for complex problems. Methodical working through with explicit reasoning chains, revision of approaches when needed.</p>

<p><strong>Features:</strong></p>

<ul>
<li>Extended reasoning chains (not just first idea, but alternatives)</li>
<li>Explicit thinking steps before code generation</li>
<li>Revision mechanism when approach doesn't work</li>
<li>Integration with other MCPs (e.g., Context7)</li>
</ul>

<p><strong>Why useful:</strong> "Like having a senior architect who thinks before coding." Especially valuable for architectural decisions, complex refactorings, or when multiple solution paths are possible. Sequential Thinking makes reasoning explicit and traceable.</p>

<p><strong>Combination:</strong> Best used together with Context7. Sequential Thinking for structured thinking, Context7 for current framework information.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation</span>
claude mcp add @modelcontextprotocol/server-sequential-thinking
</code></pre>

</div>



<h3>
  
  
  3. Memory Bank / Serena - For Large Codebases
</h3>

<p><strong>When needed:</strong> Only for codebases &gt;1000 files or highly complex architectures.</p>

<p><strong>Memory Bank:</strong> Cross-session memory—persistent across sessions. Stores architecture decisions, frequently used patterns, project context.</p>

<p><strong>Serena:</strong> Semantic code retrieval and editing tools. Understands code semantically (not just text match), finds relevant code locations based on intent.</p>

<p><strong>Trade-off:</strong> Setup effort (indexing, configuration) vs. token savings. For smaller projects, the overhead outweighs the benefit.</p>

<p>My recommendation: <strong>Only when truly needed.</strong> If you notice you're regularly fighting context overflow and the codebase &gt;1000 files, the investment is worth it. Before that: use native tools (see next section).</p>

<p><strong>Installation:</strong> See official documentation—setup is individual per project.</p>

<h3>
  
  
  4. Native Tools First - The 80% Solution
</h3>

<p><strong>Important insight:</strong> For most projects (&lt;1000 files), native tools are sufficient.</p>

<p><strong>Native tools in Claude Code:</strong></p>

<ul>
<li>
<strong>Glob</strong> (File Pattern Matching): <code>**/*.js</code>, <code>src/**/*.ts</code>—finds files lightning-fast</li>
<li>
<strong>Grep</strong> (ripgrep-powered): Full-text search with regex support, extremely fast</li>
<li>
<strong>Explore Subagent</strong> (Haiku-powered): Fast reconnaissance for codebase structure</li>
</ul>

<p><strong>Advantage:</strong> No setup, no context overhead, immediately available. Glob and Grep are so fast that indexing is often unnecessary.</p>

<p><strong>When MCP:</strong> Only at &gt;1000 files, complex architecture, or when semantic queries are needed (not just text match, but intent-based).</p>

<p><strong>Philosophy:</strong> "Search, Don't Index." Modern search tools are so fast that pre-indexing is often overkill. Only when search becomes the bottleneck should you introduce MCPs.</p>




<h2>
  
  
  Skills &amp; Commands: Progressive Disclosure
</h2>

<p>Skills have been one of the most powerful features in Claude Code since October 2025. They allow defining structured workflows that Claude must follow. I use the <strong>wrapper pattern</strong>—learned from Superpowers.</p>

<h3>
  
  
  The Wrapper Pattern
</h3>

<p><strong>Concept:</strong> Commands as thin entry points (minimal context) + skills as full implementation (loaded on-demand, no startup cost).</p>

<p><strong>Concrete:</strong> A command is 6 lines of YAML, ~200 bytes. Only points to the skill. The skill contains the full implementation (1,000+ lines). The skill is only loaded when the command is called.</p>

<p><strong>Example from my practice:</strong></p>

<p><em>Command:</em> <code>update</code> (6 lines, 189 bytes)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">update</span>
<span class="na">description</span><span class="pi">:</span> <span class="s">Analyze conversation/codebase and intelligently distribute insights</span>
<span class="na">command</span><span class="pi">:</span> <span class="pi">|</span>
  <span class="s">Using the 'mayflow:updating-documentation' skill, analyze...</span>
</code></pre>

</div>



<p><em>Skill:</em> <code>updating-documentation</code> (1,326 lines, full implementation)</p>

<ul>
<li>Analyzes conversation + codebase</li>
<li>Distributes insights across CLAUDE.md, README.md, docs/</li>
<li>Handles hierarchies and update strategies</li>
<li>Full logic, only loaded when used</li>
</ul>

<p><strong>Context math:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Without Wrapper</th>
<th>With Wrapper</th>
<th>Savings</th>
</tr>
</thead>
<tbody>
<tr>
<td>At Startup</td>
<td>+2,087 tokens</td>
<td>+93 tokens</td>
<td>-1,994 tokens (-64%)</td>
</tr>
<tr>
<td>At Growth</td>
<td>Scales linearly</td>
<td>Constant</td>
<td>Grows with each skill</td>
</tr>
<tr>
<td>Break-Even</td>
<td>Immediate</td>
<td>2-3 weeks daily use</td>
<td>—</td>
</tr>
</tbody>
</table></div>

<p><strong>Why I use it:</strong></p>

<p>Context efficiency at scale. I can add unlimited skills without burdening startup context. Clear separation between entry point (what does the command do?) and implementation (how is it done?). Progressive disclosure: users see simple command list first, details only when needed.</p>

<p><strong>Inspiration:</strong> Learned from Superpowers—so far the only plugin I know with this approach. The 20+ skills of Superpowers wouldn't be practical without the wrapper pattern (would consume tens of thousands of tokens at startup).</p>

<h3>
  
  
  Developing Your Own Skills
</h3>

<p><strong>When useful:</strong> Recurring workflows, team-specific processes, complex multi-step tasks.</p>

<p><strong>Storage locations:</strong></p>

<ul>
<li>
<strong>Personal Skills:</strong> <code>~/.claude/skills/</code>—available across all projects</li>
<li>
<strong>Project Skills:</strong> <code>.claude/skills/</code>—team-shared via Git, project-specific</li>
<li>
<strong>Plugin Skills:</strong> Automatically available when plugin installed (like Superpowers)</li>
</ul>

<p><strong>New in v2.1.0: Automatic Hot-Reload</strong> 🔥</p>

<p>Game-changer for skill development: Skills become <strong>instantly available</strong> without session restart. Edit skill → switch to Claude Code → skill runs. This eliminates the biggest friction in iterative skill testing.</p>

<p>Before: Edit → Restart Session → Test → Repeat (5+ minutes per iteration)<br>
Now: Edit → Test → Repeat (&lt;30 seconds per iteration)</p>

<p>This makes exploratory skill development practical. Test small changes, iterate quickly, get immediate feedback.</p>

<p><strong>Workflow:</strong></p>

<ol>
<li>First develop for your own use (Personal Skills)</li>
<li>Test and iterate in project</li>
<li>If team needs it: Move to Project Skills (<code>.claude/skills/</code>)</li>
<li>For higher complexity or broader use: Package as plugin</li>
</ol>

<p><strong>Plugin for team sharing:</strong> When skills become complex (multiple files, dependencies), or when you want to share them across multiple teams/projects, plugin format makes sense. But: start simple. Personal Skill → Project Skill → Plugin is the natural development path.</p>

<p><strong>Iteration important:</strong> No skill version is perfect on first try. First write, then use, then iterate. After 2-3 iterations, skills are typically much better than the first draft.</p>

<p><strong>Advanced: Context Forking (v2.1.0)</strong> 🚀</p>

<p>For token-intensive skills: Let them run in <strong>isolated sub-agent contexts</strong>. This is relevant for large codebase analyses (&gt;1000 files), research tasks with many external resources, or parallel workflows.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">heavy-analysis</span>
<span class="na">context</span><span class="pi">:</span> <span class="s">fork</span>          <span class="c1"># Run in separate sub-agent</span>
<span class="na">agent</span><span class="pi">:</span> <span class="s">explore</span>         <span class="c1"># Optional: Use specific agent type</span>
<span class="nn">---</span>
</code></pre>

</div>



<p><strong>Benefits:</strong> Main context stays clean, parallel skill execution possible, token isolation for heavy operations. This is the context management equivalent to the wrapper pattern: costs only on use, not at startup.</p>

<h3>
  
  
  Best Practices
</h3>

<ul>
<li>
<strong>Skill-first for recurring tasks:</strong> If you do something more than 3 times, a skill makes sense</li>
<li>
<strong>Use progressive disclosure:</strong> Split large skills into separate files (main skill + helper files)</li>
<li>
<strong>Test with representative queries:</strong> Use real use cases for testing, not constructed examples</li>
<li>
<strong>Reusability:</strong> Skills are usable by commands, agents, hooks—write once, use multiple times</li>
</ul>




<h2>
  
  
  Hooks: Event-Based Automation
</h2>

<p>Hooks automate workflows event-based. 9 lifecycle events available (as of January 2026): SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, and more. My focus: <strong>Auto-formatting and security.</strong></p>

<h3>
  
  
  PostToolUse - Auto-Formatting
</h3>

<p><strong>When:</strong> Automatically format after every Edit/Write.</p>

<p><strong>File-type specific:</strong> Python (ruff), TypeScript (prettier), Go (goimports), Kotlin (ktlint), Markdown (prettier). Each language has its own formatters, hooks can orchestrate them all.</p>

<p><strong>My recommendation: ryanlewis/claude-format-hook</strong></p>

<ul>
<li>Battle-tested, multi-language support</li>
<li>Supports JS/TS (Biome with fallback to Prettier), Python (ruff), Go (goimports), Kotlin (ktlint), Markdown (prettier)</li>
<li>
<strong>Graceful degradation:</strong> If formatter is missing, hook doesn't fail (via <code>|| true</code>). Never interrupts Claude Code.</li>
</ul>

<p><strong>Installation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Clone repository</span>
git clone https://github.com/ryanlewis/claude-format-hook.git ~/.claude/hooks/format

<span class="c"># Make executable</span>
<span class="nb">chmod</span> +x ~/.claude/hooks/format/format_hook.sh

<span class="c"># Add to Claude Code settings.json (see below)</span>
</code></pre>

</div>



<p><strong>Config example (settings.json):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"trigger"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PostToolUse"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"commandMatch"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Edit|Write|MultiEdit"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"~/.claude/hooks/format/format_hook.sh"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>How it works:</strong> After every Edit/Write, Claude Code calls the hook. The hook detects file type, selects appropriate formatter, formats file. If formatter is missing or fails: <code>|| true</code> prevents error. Claude Code continues working.</p>

<h3>
  
  
  PreToolUse - Security
</h3>

<p><strong>When:</strong> Before tool execution—can block or modify.</p>

<p><strong>Use cases:</strong></p>

<ul>
<li>Blocks dangerous commands (<code>rm -rf</code>, <code>dd</code>, <code>mkfs</code>)</li>
<li>Protects sensitive files (<code>.env</code>, <code>package-lock.json</code>, <code>.git/</code>)</li>
<li>Exit code 2 = tool gets blocked, Claude receives error message</li>
</ul>

<p><strong>New since v2.0.10: Input Modification</strong></p>

<p>Instead of blocking the tool, hooks can transparently correct input. Example: Claude wants to commit <code>.env</code> → hook removes <code>.env</code> from staging → command continues without <code>.env</code>.</p>

<p><strong>Example: .env and .git/ protection</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># PreToolUse Hook: Block .env and .git/ modifications</span>

<span class="nv">TOOL_NAME</span><span class="o">=</span><span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span>
<span class="nv">INPUT_JSON</span><span class="o">=</span><span class="s2">"</span><span class="nv">$2</span><span class="s2">"</span>

<span class="c"># Check if Edit/Write targets .env or .git/</span>
<span class="k">if </span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$INPUT_JSON</span><span class="s2">"</span> | jq <span class="nt">-e</span> <span class="s1">'.file_path | test("\\.env$|\\.git/")'</span> <span class="o">&gt;</span> /dev/null<span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"BLOCKED: Cannot modify .env or .git/ files"</span> <span class="o">&gt;</span>&amp;2
  <span class="nb">exit </span>2  <span class="c"># Exit code 2 = Block tool</span>
<span class="k">fi

</span><span class="nb">exit </span>0  <span class="c"># Allow tool</span>
</code></pre>

</div>



<p><strong>Input modification (new):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># PreToolUse Hook: Remove .env from git add</span>

<span class="nv">TOOL_NAME</span><span class="o">=</span><span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span>
<span class="nv">INPUT_JSON</span><span class="o">=</span><span class="s2">"</span><span class="nv">$2</span><span class="s2">"</span>

<span class="c"># If Bash(git add) includes .env, modify input</span>
<span class="k">if</span> <span class="o">[[</span> <span class="s2">"</span><span class="nv">$TOOL_NAME</span><span class="s2">"</span> <span class="o">==</span> <span class="s2">"Bash"</span> <span class="o">]]</span><span class="p">;</span> <span class="k">then
  </span><span class="nv">COMMAND</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$INPUT_JSON</span><span class="s2">"</span> | jq <span class="nt">-r</span> <span class="s1">'.command'</span><span class="si">)</span>
  <span class="k">if</span> <span class="o">[[</span> <span class="s2">"</span><span class="nv">$COMMAND</span><span class="s2">"</span> <span class="o">=</span>~ <span class="s2">"git add"</span> <span class="o">]]</span> <span class="o">&amp;&amp;</span> <span class="o">[[</span> <span class="s2">"</span><span class="nv">$COMMAND</span><span class="s2">"</span> <span class="o">=</span>~ <span class="s2">".env"</span> <span class="o">]]</span><span class="p">;</span> <span class="k">then</span>
    <span class="c"># Remove .env from command</span>
    <span class="nv">MODIFIED</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$COMMAND</span><span class="s2">"</span> | <span class="nb">sed</span> <span class="s1">'s/\.env//g'</span><span class="si">)</span>
    <span class="c"># Return modified input</span>
    jq <span class="nt">-n</span> <span class="nt">--arg</span> cmd <span class="s2">"</span><span class="nv">$MODIFIED</span><span class="s2">"</span> <span class="s1">'{command: $cmd, updatedInput: true}'</span>
    <span class="nb">exit </span>0
  <span class="k">fi
fi

</span><span class="nb">exit </span>0  <span class="c"># No modification</span>
</code></pre>

</div>



<h3>
  
  
  Best Practices 2026
</h3>

<p><strong>"Block-at-Submit" instead of "Block-at-Write":</strong></p>

<p>Old strategy: Block at every Write → many interruptions, Claude gets stopped often.</p>

<p>New strategy: Let agent finish plan → validate final result. Instead of blocking every edit, let Claude complete the entire plan. At the end (UserPromptSubmit hook or before commit), validate.</p>

<p><strong>Recommendation from creator (Ryan Lewis):</strong> "Run hooks at the end of work, not during work." Fewer interrupts, smoother workflows, better UX.</p>

<p><strong>Input modification over blocking:</strong></p>

<p>Use <code>updatedInput</code> in PreToolUse hooks. Correct inputs instead of blocking. Makes corrections invisible to Claude → no error messages, no interruptions.</p>

<p>Example: Claude wants to edit <code>package-lock.json</code> → hook replaces with <code>/dev/null</code> → no edit, no error.</p>

<p><strong>New in v2.1.0:</strong> <code>updatedInput</code> now also works with <code>ask</code> return (hook can modify input AND request user confirmation). Plus: <code>once: true</code> option for hooks—runs only once per event (performance optimization for expensive operations like linting).</p>

<p><strong>Graceful degradation:</strong></p>

<p>Hooks should never interrupt Claude Code. <code>|| true</code> for optional formatting. Build in error handling (if formatter is missing, don't fail).</p>

<p><strong>Scope precisely:</strong></p>

<p>Match specific commands (<code>Edit|Write|MultiEdit</code>), not broad tool categories. Reduces false positives.</p>

<p><strong>My experience:</strong> Fewer hooks = more stable, fewer surprises. I only use auto-formatting (PostToolUse) and <code>.env</code> protection (PreToolUse). More would increase context and complexity without proportional benefit.</p>




<h2>
  
  
  Best Practices: Learnings from 3 Months
</h2>

<p>After three months of intensive Claude Code use, these practices have proven themselves.</p>

<h3>
  
  
  Context Management
</h3>

<p><strong>Problem:</strong> MCP tools can consume 40%+ context.</p>

<p>Example from my tests: 81,986 tokens just for MCP tools at startup (41% of the 200k context window!). That's almost half the available context—before even a single line of code was loaded.</p>

<p><strong>Solution: Less is more</strong></p>

<ul>
<li>
<strong>Start with 2-3 MCPs, not 10+.</strong> I use Context7 + Sequential Thinking. That's it. Everything else: activate on-demand.</li>
<li>
<strong>Use <code>/context</code> command for usage breakdown.</strong> Shows: System prompt, System tools, MCP tools, Memory files, Messages, Free space. This lets you see what's eating context.</li>
<li>
<strong>My experience:</strong> 4 plugins + 2 MCPs = sweet spot.

<ul>
<li>Language Server + Superpowers + commit-commands + pr-review-toolkit (Plugins)</li>
<li>Context7 + Sequential Thinking (MCPs)</li>
<li>Everything else: activate on-demand, don't load permanently</li>
</ul>


</li>

</ul>

<p><strong>Native tools first:</strong></p>

<p>For &lt;1000 files, Glob/Grep are sufficient. Explore Subagent for quick reconnaissance. Only with real need: MCP indexing (Memory Bank/Serena).</p>

<p><strong>Selective activation:</strong></p>

<p>Not everything at once. McPick tool for toggle management (activate/deactivate MCPs temporarily). Different MCPs per project: frontend project needs different tools than backend microservices.</p>

<h3>
  
  
  Scaling &amp; Organization
</h3>

<p><strong>CLAUDE.md hierarchy:</strong></p>

<p>Claude Code loads three CLAUDE.md files in this order:</p>

<ol>
<li>
<strong>User-global:</strong> <code>~/.claude/CLAUDE.md</code> (personal preferences, IDE settings, private API keys)</li>
<li>
<strong>Project root:</strong> <code>CLAUDE.md</code> (Git-committed, team-shared, project context)</li>
<li>
<strong>Local:</strong> <code>CLAUDE.local.md</code> (Git-ignored, machine-specific, local paths)</li>
</ol>

<p><strong>Tip: Language Configuration (v2.1.0)</strong> — For international teams or non-English documentation, you can now configure Claude's response language:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">In</span><span class="w"> </span><span class="err">.claude/settings.json</span><span class="w"> </span><span class="err">or</span><span class="w"> </span><span class="err">~/.claude/settings.json</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"language"</span><span class="p">:</span><span class="w"> </span><span class="s2">"german"</span><span class="w">  </span><span class="err">//</span><span class="w"> </span><span class="err">or</span><span class="w"> </span><span class="s2">"japanese"</span><span class="p">,</span><span class="w"> </span><span class="s2">"spanish"</span><span class="p">,</span><span class="w"> </span><span class="s2">"french"</span><span class="p">,</span><span class="w"> </span><span class="err">etc.</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This is useful for localized commit messages, documentation in native language, or educational projects. Combinable with CLAUDE.md for hybrid setups (e.g., code comments in English, explanations in German).</p>

<p><strong>What belongs in CLAUDE.md (project root)?</strong></p>

<ul>
<li>Common bash commands (Build, Test, Deploy)</li>
<li>Code style guidelines (Prettier config, linting rules)</li>
<li>Key files/architecture (Where is what? Which files are central?)</li>
<li>Testing instructions (How tests run, what they check)</li>
<li>Important commands (Project-specific scripts)</li>
</ul>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># CLAUDE.md</span>

<span class="gu">## Build &amp; Test</span>
<span class="p">-</span> <span class="sb">`npm run build`</span> - TypeScript compilation
<span class="p">-</span> <span class="sb">`npm test`</span> - Jest tests (must pass before commit)

<span class="gu">## Architecture</span>
<span class="p">-</span> <span class="sb">`src/core/`</span> - Business logic (pure functions)
<span class="p">-</span> <span class="sb">`src/api/`</span> - Express routes (RESTful)
<span class="p">-</span> <span class="sb">`src/db/`</span> - Database layer (Prisma)

<span class="gu">## Code Style</span>
<span class="p">-</span> Prettier enforced via pre-commit hook
<span class="p">-</span> ESLint for static analysis
<span class="p">-</span> No <span class="sb">`any`</span> types in TypeScript (use <span class="sb">`unknown`</span> instead)
</code></pre>

</div>



<p><strong>Plugin strategy:</strong></p>

<ul>
<li>
<strong>Start minimal:</strong> Language Server + Superpowers</li>
<li>
<strong>Expand as needed:</strong> commit-commands, pr-review-toolkit</li>
<li><strong>Not "because available", but "because needed"</strong></li>
</ul>

<p>Marketplace has dozens of plugins. The temptation is great to try everything. But every plugin costs context. The question isn't "Is this cool?", but "Does this solve a real problem I currently have?".</p>

<p><strong>Permissions (v2.1.0: Wildcard support):</strong></p>

<p>Since v2.1.0, Bash permissions support wildcards: <code>Bash(npm *)</code> allows all npm commands instead of dozens of individual rules. This massively simplifies permission management:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"permissions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"allow"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="s2">"Bash(npm *)"</span><span class="p">,</span><span class="w">      </span><span class="err">//</span><span class="w"> </span><span class="err">All</span><span class="w"> </span><span class="err">npm</span><span class="w"> </span><span class="err">commands</span><span class="w">
      </span><span class="s2">"Bash(git *)"</span><span class="p">,</span><span class="w">      </span><span class="err">//</span><span class="w"> </span><span class="err">All</span><span class="w"> </span><span class="err">git</span><span class="w"> </span><span class="err">commands</span><span class="w">
      </span><span class="s2">"Bash(pytest *)"</span><span class="w">    </span><span class="err">//</span><span class="w"> </span><span class="err">All</span><span class="w"> </span><span class="err">pytest</span><span class="w"> </span><span class="err">commands</span><span class="w">
    </span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Skills vs. commands:</strong></p>

<ul>
<li>
<strong>Commands:</strong> User convenience (thin wrappers), entry points</li>
<li>
<strong>Skills:</strong> Implementation (full logic), loaded on-demand</li>
<li>
<strong>Progressive disclosure:</strong> Unlimited skills without context penalty (via wrapper pattern)</li>
</ul>

<p><strong>My recommendation:</strong></p>

<ol>
<li>
<strong>Practice first:</strong> Develop tools for your own needs. Use Claude Code to work with Claude Code (dogfooding).</li>
<li>
<strong>Then document:</strong> What works goes into CLAUDE.md. What doesn't work gets removed.</li>
<li>
<strong>Then share:</strong> Create plugin when team needs it. Only when it works for you does sharing make sense.</li>
</ol>

<h3>
  
  
  Iteration &amp; Learning
</h3>

<p><strong>Iteration is normal:</strong></p>

<p>First version of a skill or hook is good. After 2-3 iterations, it's typically much better. Don't expect perfection on day one. Instead: build quickly, test quickly, iterate quickly.</p>

<p><strong>Moving target:</strong></p>

<p>Claude Code evolves quickly. Plugin system came in October (while I was already using it). Hooks got input modification in v2.0.10. What's best practice today might be obsolete tomorrow.</p>

<p><strong>Stay flexible instead of clinging to perfect plan.</strong> Use new features as soon as they come. Adapt old workflows when better alternatives are available.</p>

<p><strong>Community learning:</strong></p>

<p>Learn from other plugins. Superpowers was my game-changer—not just because of the skills, but because of the wrapper pattern, which I then adapted for my own skills.</p>

<p>Explore marketplace. Adapt best practices, don't blindly copy. What works for others doesn't have to work for you (and vice versa).</p>

<p><strong>Feedback loops:</strong></p>

<p>Claude tools give feedback: Screenshots (visual validation), tests (functional validation), linting (code quality). Use them.</p>

<p>Use Claude itself for plugin development. Dogfooding: use what you build yourself. If you wouldn't use it, why should anyone else?</p>




<h2>
  
  
  Conclusion: From Overwhelm to Clarity
</h2>

<p>From overwhelm by options to clear must-haves—that was my journey over the last three months. October 2025, I had access to Claude Code. Early January 2026, I have a setup that works.</p>

<p>The path there wasn't linear. The plugin system came in October, in the middle of my usage. And while I was writing the final versions of this article, Claude Code 2.1.0 was released—with features like Skill Hot-Reload and Context Forking that address exactly the challenges I'd experienced in recent months. Instead of problems, these were opportunities: use new possibilities, adapt workflows. Flexibility was more important than complete upfront planning.</p>

<p><strong>The most important insight: Less is more.</strong></p>

<p>Context efficiency beats feature abundance. 4 well-chosen plugins &gt; 20 "interesting" plugins. Use native tools instead of immediately adding MCPs. Not "because available", but "because needed".</p>

<h3>
  
  
  The Core Recommendations
</h3>

<p><strong>Start minimal:</strong></p>

<ul>
<li>Language Server for your main language (Plugin)</li>
<li>Superpowers for structured workflows (Plugin)</li>
<li>Context7 for up-to-date documentation (MCP)</li>
</ul>

<p><strong>Expand as needed:</strong></p>

<ul>
<li>commit-commands for Git automation (Plugin)</li>
<li>pr-review-toolkit for code quality (Plugin)</li>
<li>Sequential Thinking for problem-solving (MCP)</li>
<li>Memory Bank/Serena only for &gt;1000 files (MCP)</li>
</ul>

<p><strong>Work context-aware:</strong></p>

<ul>
<li>Don't activate everything that exists</li>
<li>Check <code>/context</code> command regularly</li>
<li>Selective activation per project</li>
</ul>

<p><strong>Practice first:</strong></p>

<ul>
<li>Develop your own workflows</li>
<li>Then document (CLAUDE.md)</li>
<li>Then scale (Skills, Plugins)</li>
<li>Iteration over perfection</li>
</ul>

<h3>
  
  
  Outlook
</h3>

<p>Claude Code continues to evolve. New features come continuously. These recommendations are as of January 2026—not a dogmatic guide, but a snapshot.</p>

<p>The community ecosystem is growing. Marketplaces, plugins, skills. New possibilities arrive daily. Exciting developments to expect.</p>

<p>My usage and learnings continue. Not a finished process. Keep experimenting, keep learning, keep iterating.</p>

<p><strong>This setup works for me. Take it as a starting point for your own workflow.</strong></p>

<p>Not as a blueprint, but as inspiration. Not as a complete feature list, but as a curated collection. Not as a perfect solution, but as a starting point for your own exploration.</p>

<p>October 2025, I faced a fresh start. Early January 2026, I have clarity. In three months: from "What do I need?" to "This works." The next chapter begins now.</p>




<h3>
  
  
  Your Turn
</h3>

<p><strong>Which must-haves did I miss?</strong> Are you using different plugins or MCPs that transformed your workflow? I'd love to hear what's working for you in the comments below!</p>

<p>If you found this helpful:</p>

<ul>
<li>⭐ Star the <a href="https://github.com/obra/superpowers-marketplace" rel="noopener noreferrer">Superpowers plugin</a> and <a href="https://github.com/ryanlewis/claude-format-hook" rel="noopener noreferrer">claude-format-hook</a> repositories</li>
<li>💬 Drop your own must-haves in the comments</li>
</ul>

<p>Thanks for reading! May your context windows stay clean and your workflows efficient. 🚀</p>




<p><em>This article was originally published in German on the <a href="https://blog.mayflower.de/25215-claude-code-must-haves-2026.html" rel="noopener noreferrer">Mayflower Blog</a>.</em></p>

