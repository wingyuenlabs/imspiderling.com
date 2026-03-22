---
Title: The First Karpathy Loop for Production Coding Agents
Description: 
Author: Dean Sharon
Date: 2026-03-22T21:47:28.000Z
Robots: noindex,nofollow
Template: index
---
<p>Karpathy showed what happens when you let an AI agent run 700 experiments overnight. The model proposes hypotheses, runs them, scores results, keeps what works, throws away what doesn't. Repeat.</p>

<p>The part nobody talks about: how do you know which experiments actually mattered?</p>

<p>I've been building with AI coding agents for months. Claude Code, Codex, Gemini CLI. The pattern is always the same: you give an agent a task, it runs, it produces output. Sometimes the output is good. Sometimes it's not. You squint at logs, compare diffs, make a judgment call. Move on.</p>

<p>That loop works fine for single tasks. It breaks completely when you want the agent to iterate on its own work.</p>

<h2>
  
  
  The Problem
</h2>

<p>Say you want an agent to optimize a function. Or fix a flaky test. Or refactor a module until it passes a quality gate.</p>

<p>Without loops, you're doing this manually. Run the agent. Check the output. Run it again with different instructions. Check again. Copy paste the good parts. This is not what "autonomous" means.</p>

<p>Karpathy's autoresearch proved the loop works for research. Run, score, keep, discard, iterate. The scoring function is the key. Without a scoring function, you're just running the same thing over and over hoping something changes.</p>

<h2>
  
  
  The Solution: Backbeat Loops
</h2>

<p>Backbeat v0.7.0 shipped loops. Two strategies.</p>

<p><strong>Retry</strong>: run a task until a shell command returns exit code 0.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>beat loop <span class="s2">"fix the failing test in auth.test.ts"</span> <span class="nt">--until</span> <span class="s2">"npm test"</span>
</code></pre>

</div>



<p>The agent runs. npm test fails. The agent runs again with fresh context. npm test passes. Done.</p>

<p><strong>Optimize</strong>: score each iteration with an eval script. Keep the best.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>beat loop <span class="s2">"reduce bundle size of the dashboard module"</span> <span class="se">\</span>
  <span class="nt">--eval</span> <span class="s2">"node scripts/measure-bundle.js"</span> <span class="se">\</span>
  <span class="nt">--direction</span> minimize
</code></pre>

</div>



<p>Each iteration gets scored. Backbeat tracks the best result. After 10 iterations (configurable), you get the version that scored lowest. No squinting at experiment logs.</p>

<h2>
  
  
  How It Works
</h2>

<p>Each loop iteration runs in a clean agent context by default. The agent doesn't carry baggage from previous failures. Fresh start, same goal, same scoring function.</p>

<p>For more complex workflows, by the next release you will be able to loop entire pipelines:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>beat loop <span class="nt">--pipeline</span> <span class="se">\</span>
  <span class="nt">--step</span> <span class="s2">"refactor the payment module"</span> <span class="se">\</span>
  <span class="nt">--step</span> <span class="s2">"run the integration tests"</span> <span class="se">\</span>
  <span class="nt">--step</span> <span class="s2">"measure test coverage"</span> <span class="se">\</span>
  <span class="nt">--until</span> <span class="s2">"node scripts/check-coverage.js --min 90"</span>
</code></pre>

</div>



<p>All three steps run per iteration. The exit condition evaluates after the full pipeline completes.</p>

<p>Safety controls keep things sane:</p>

<ul>
<li>Max iterations (default 10, 0 for unlimited if you're feeling brave)</li>
<li>Max consecutive failures before stopping (default 3)</li>
<li>Cooldown between iterations in milliseconds</li>
</ul>

<h2>
  
  
  This Is the Karpathy Loop for Production
</h2>

<p>Autoresearch runs experiments in cycles. Propose, train, evaluate, keep or discard. Backbeat does the same thing but for production coding tasks instead of research.</p>

<p>The scoring function is what makes it work. Without one, the agent just retries blindly. With one, it optimizes. <code>npm test</code> is a scoring function. Bundle size measurement is a scoring function. Test coverage is a scoring function. Anything that returns a number or an exit code works.</p>

<p>First production implementation of this pattern for coding agents. Claude Code, Codex, Gemini CLI, any agent that speaks MCP.</p>

<h2>
  
  
  Getting Started
</h2>

<p>Add to your project's <code>.mcp.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"backbeat"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npx"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"-y"</span><span class="p">,</span><span class="w"> </span><span class="s2">"backbeat"</span><span class="p">,</span><span class="w"> </span><span class="s2">"mcp"</span><span class="p">,</span><span class="w"> </span><span class="s2">"start"</span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Or use the CLI directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> backbeat
beat loop <span class="s2">"your task"</span> <span class="nt">--until</span> <span class="s2">"your exit condition"</span>
</code></pre>

</div>



<p>As always, open source, MIT. <a href="https://github.com/dean0x/backbeat" rel="noopener noreferrer">github.com/dean0x/backbeat</a></p>

<p>Particularly interested in how people are evaluating their agent outputs. What does your eval function look like?</p>

