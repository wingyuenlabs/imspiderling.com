---
Title: DIY GitHub Issue Bot — Just Your LLM API Key
Description: 
Author: Masaaki Hirano
Date: 2025-12-05T22:01:17.000Z
Robots: noindex,nofollow
Template: index
---
<p>You've seen those fancy AI bots that answer GitHub issues. Most of them require subscriptions, SaaS accounts, or complex setups.</p>

<p>What if you could build one yourself with just your LLM API key?</p>

<p>With <a href="https://github.com/perstack-ai/perstack" rel="noopener noreferrer">Perstack</a>, you can. Copy two files, add one secret, done.</p>

<p><strong>See it in action:</strong> <a href="https://github.com/perstack-ai/perstack/issues/55" rel="noopener noreferrer">This issue</a> was answered by the bot reading the actual codebase.</p>

<h2>
  
  
  What You Get
</h2>

<p>An AI bot that:</p>

<ul>
<li>👀 Reacts to show it's processing</li>
<li>🔍 Actually reads your codebase (not just guessing)</li>
<li>💬 Posts answers with activity logs</li>
<li>💰 Costs only what you use (your LLM API)</li>
</ul>

<p>Activity log example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>💭 I need to understand how the runtime state machine works...
📁 Listing: packages/runtime/src
📖 Reading: runtime-state-machine.ts
💭 Found the state transitions...
✅ Done
</code></pre>

</div>



<h2>
  
  
  Setup (5 Minutes)
</h2>

<h3>
  
  
  Step 1: Copy Files
</h3>

<p>Copy from the <a href="https://github.com/perstack-ai/perstack/tree/main/examples/github-issue-bot" rel="noopener noreferrer">example repo</a>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>your-repo/
├── .github/workflows/issue-bot.yml   ← workflow
└── scripts/checkpoint-filter.ts       ← formats output
</code></pre>

</div>



<p>The workflow runs on:</p>

<ul>
<li>New issue opened</li>
<li>Comment containing <code>@perstack-issue-bot</code>
</li>
</ul>

<h3>
  
  
  Step 2: Add Secret
</h3>

<p><strong>Settings → Secrets → Actions → <code>ANTHROPIC_API_KEY</code></strong></p>

<p>That's it. Open an issue and watch it work.</p>

<h2>
  
  
  Customize It
</h2>

<p>Want custom behavior? Define your own Expert in <code>perstack.toml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="py">model</span> <span class="p">=</span> <span class="s">"claude-sonnet-4-5"</span>

<span class="nn">[provider]</span>
<span class="py">providerName</span> <span class="p">=</span> <span class="s">"anthropic"</span>

<span class="nn">[experts."my-issue-bot"]</span>
<span class="py">description</span> <span class="p">=</span> <span class="s">"Custom issue bot for my project"</span>
<span class="py">instruction</span> <span class="p">=</span> <span class="s">"""
You are an issue bot for the Acme project.

## Rules
- Always check docs/ directory first
- Add labels: use `gh issue edit --add-label`
  - "bug" for bug reports
  - "feature" for feature requests  
  - "question" for questions
- If the issue is about authentication, mention @security-team
- Keep answers under 500 words
"""</span>

<span class="nn">[experts."my-issue-bot".skills."@perstack/base"]</span>
<span class="py">type</span> <span class="p">=</span> <span class="s">"mcpStdioSkill"</span>
<span class="py">command</span> <span class="p">=</span> <span class="s">"npx"</span>
<span class="py">packageName</span> <span class="p">=</span> <span class="s">"@perstack/base"</span>
<span class="py">requiredEnv</span> <span class="p">=</span> <span class="p">[</span><span class="s">"GH_TOKEN"</span><span class="p">,</span> <span class="s">"GITHUB_REPO"</span><span class="p">,</span> <span class="s">"ISSUE_NUMBER"</span><span class="p">]</span>
</code></pre>

</div>



<p>Then update the workflow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">run</span><span class="pi">:</span> <span class="s">npx perstack run --config ./path/to/perstack.toml my-issue-bot "Answer issue</span> <span class="c1">#$ISSUE_NUMBER"</span>
</code></pre>

</div>



<p>Now it's yours — add project-specific rules, auto-labeling, team mentions, whatever you need.</p>

<h2>
  
  
  Why DIY?
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th></th>
<th>SaaS Bots</th>
<th>DIY with Perstack</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cost</td>
<td>Monthly subscription</td>
<td>Pay-per-use (your API)</td>
</tr>
<tr>
<td>Data</td>
<td>Sent to third party</td>
<td>Stays in your repo</td>
</tr>
<tr>
<td>Customization</td>
<td>Limited</td>
<td>Full control</td>
</tr>
<tr>
<td>Setup</td>
<td>Account signup</td>
<td>2 files</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Under the Hood
</h2>

<h3>
  
  
  Event Stream
</h3>

<p>Perstack emits JSON events to stdout as the Expert runs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="s2">"callTool"</span><span class="p">,</span><span class="nl">"toolCall"</span><span class="p">:{</span><span class="nl">"toolName"</span><span class="p">:</span><span class="s2">"listDirectory"</span><span class="p">,</span><span class="nl">"args"</span><span class="p">:{</span><span class="nl">"path"</span><span class="p">:</span><span class="s2">"src/"</span><span class="p">}}}</span><span class="w">
</span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="s2">"callTool"</span><span class="p">,</span><span class="nl">"toolCall"</span><span class="p">:{</span><span class="nl">"toolName"</span><span class="p">:</span><span class="s2">"readTextFile"</span><span class="p">,</span><span class="nl">"args"</span><span class="p">:{</span><span class="nl">"path"</span><span class="p">:</span><span class="s2">"src/index.ts"</span><span class="p">}}}</span><span class="w">
</span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="s2">"completeRun"</span><span class="p">,</span><span class="nl">"text"</span><span class="p">:</span><span class="s2">"Here's the answer..."</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Pipe this to a script, and you can build real-time UIs or logging. See the <a href="https://github.com/perstack-ai/perstack/blob/main/examples/github-issue-bot/checkpoint-filter.ts" rel="noopener noreferrer">checkpoint-filter.ts</a> in the example repo.</p>

<h3>
  
  
  Skills &amp; MCP
</h3>

<p>Experts use <strong>Skills</strong> to interact with the world. Skills are <a href="https://modelcontextprotocol.io/" rel="noopener noreferrer">MCP (Model Context Protocol)</a> servers that expose tools.</p>

<p><code>@perstack/base</code> provides:</p>

<ul>
<li>
<code>readTextFile</code>, <code>listDirectory</code> — file operations</li>
<li>
<code>exec</code> — run shell commands (like <code>gh</code>)</li>
<li>
<code>think</code> — structured reasoning</li>
</ul>

<p>The <code>requiredEnv</code> field passes environment variables to the MCP server, enabling <code>gh</code> CLI to authenticate:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="nn">[experts."my-bot".skills."@perstack/base"]</span>
<span class="py">requiredEnv</span> <span class="p">=</span> <span class="p">[</span><span class="s">"GH_TOKEN"</span><span class="p">]</span>  <span class="c"># Passed to the MCP server process</span>
</code></pre>

</div>



<h2>
  
  
  What's Perstack?
</h2>

<p>Think npm for AI agents. Define modular agents, publish to a registry, compose them like packages.</p>

<p>No vendor lock-in. No subscriptions. Just your code and your API key.</p>




<p><strong>Get started:</strong></p>

<ul>
<li><a href="https://github.com/perstack-ai/perstack/tree/main/examples/github-issue-bot" rel="noopener noreferrer">GitHub Issue Bot Example</a></li>
<li><a href="https://github.com/perstack-ai/perstack" rel="noopener noreferrer">Perstack GitHub</a></li>
<li><a href="https://docs.perstack.ai" rel="noopener noreferrer">Documentation</a></li>
</ul>

