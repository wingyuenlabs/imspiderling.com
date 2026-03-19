---
Title: Let AI Control Your Real Browser — Not a Throwaway One
Description: 
Author: Steve Zhang
Date: 2026-03-19T21:57:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most browser MCP servers launch a separate browser instance. Clean slate, no cookies, no auth. Want to automate something behind a login? Re-authenticate every time. Want to test your actual session state? Too bad.</p>

<p><strong>What if the AI could just use the browser you already have open?</strong></p>

<p>That's what <a href="https://chromewebstore.google.com/detail/dramaturg/ppbkmncnmjkfppilnmplpokdfagobipa" rel="noopener noreferrer">Dramaturg</a> + <code>@playwright-repl/mcp</code> does. The AI controls your real Chrome tabs — your cookies, your sessions, your localStorage — all intact.</p>

<h2>
  
  
  How it works
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Claude (or any MCP client)
  ↕ MCP (stdio)
playwright-repl MCP server
  ↕ WebSocket
Dramaturg Chrome extension
  ↕ CDP / chrome.debugger
Your real Chrome tabs
</code></pre>

</div>



<p>The key difference: <strong>Playwright runs inside the browser</strong>, not outside it. The extension uses <a href="https://github.com/nicolo-ribaudo/playwright-crx" rel="noopener noreferrer">playwright-crx</a> to run the full Playwright API directly in Chrome's service worker. No Node.js relay, no separate browser process.</p>

<p>This means:</p>

<ul>
<li>
<strong>No re-authentication</strong> — Gmail, Notion, your internal tools — already logged in</li>
<li>
<strong><code>expect()</code> assertions</strong> — something no other browser MCP server supports</li>
<li>
<strong>You see everything</strong> — the AI works in your actual tabs, in real time</li>
</ul>

<h2>
  
  
  Just two tools
</h2>

<p>While Playwright's official MCP server exposes ~70 tools, <code>@playwright-repl/mcp</code> exposes just <strong>two</strong>:</p>

<ul>
<li>
<strong><code>run_command</code></strong> — execute a single command (keyword or JavaScript)</li>
<li>
<strong><code>run_script</code></strong> — run a multi-line script</li>
</ul>

<p>The AI figures out what to do with them. Here's what a command looks like:</p>

<h3>
  
  
  Keyword syntax (<code>.pw</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>snapshot                    → accessibility tree
goto https://example.com    → navigate
click "Sign in"             → click by text
fill "Email" "user@test.com"  → fill a form field
verify-text "Welcome"       → assert text is visible
</code></pre>

</div>



<h3>
  
  
  Playwright API (JavaScript)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">getByRole</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Sign in</span><span class="dl">'</span> <span class="p">}).</span><span class="nf">click</span><span class="p">();</span>
<span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">page</span><span class="p">.</span><span class="nf">getByText</span><span class="p">(</span><span class="dl">'</span><span class="s1">Welcome</span><span class="dl">'</span><span class="p">)).</span><span class="nf">toBeVisible</span><span class="p">();</span>
</code></pre>

</div>



<p>Both work through the same <code>run_command</code> tool. The extension auto-detects the format.</p>

<h2>
  
  
  AI Agents that actually verify their work
</h2>

<p>The MCP package includes four agents that go beyond "generate and hope":</p>

<p><strong>Planner</strong> — Give it a URL and a goal. It takes snapshots, maps out the page structure, and produces a step-by-step workflow plan with the exact labels and text it found.</p>

<p><strong>Generator</strong> — Takes a plan and turns it into a working script. The key word is <em>working</em> — it runs every command in the real browser, assembles the script, executes the full thing via <code>run_script</code>, and iterates until it passes.</p>

<p><strong>Healer</strong> — Give it a broken script. Wrong selectors, changed page structure, timing issues. It runs the script, diagnoses failures with snapshots, fixes them, and re-runs until all lines pass.</p>

<p><strong>Converter</strong> — Translates between <code>.pw</code> keyword syntax and JavaScript Playwright API. Produces idiomatic output — proper locator strategies, extracted variables, chained methods.</p>

<p>Every agent <strong>must run the script before outputting it</strong>. No "here's what should work" — it either passes or the agent keeps fixing.</p>

<h2>
  
  
  Setup
</h2>

<h3>
  
  
  1. Install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> @playwright-repl/mcp
</code></pre>

</div>



<p>Install <a href="https://chromewebstore.google.com/detail/dramaturg/ppbkmncnmjkfppilnmplpokdfagobipa" rel="noopener noreferrer">Dramaturg from the Chrome Web Store</a>.</p>

<h3>
  
  
  2. Configure Claude Desktop
</h3>

<p>Add to <code>claude_desktop_config.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"playwright-repl"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright-repl-mcp"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Or for Claude Code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>claude mcp add playwright-repl playwright-repl-mcp
</code></pre>

</div>



<h3>
  
  
  3. Install the AI agents (optional)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> .claude/agents
<span class="nb">cp </span>node_modules/@playwright-repl/mcp/agents/<span class="k">*</span>.agent.md .claude/agents/
</code></pre>

</div>



<h3>
  
  
  4. That's it
</h3>

<p>The extension connects automatically. No side panel needed. Just make sure Chrome is running.</p>

<h2>
  
  
  Example prompts
</h2>

<blockquote>
<p>Go to <a href="https://demo.playwright.dev/todomvc" rel="noopener noreferrer">https://demo.playwright.dev/todomvc</a>, add three todos, complete one, and verify the Active filter only shows the remaining two.</p>

<p>@playwright-repl-planner explore <a href="https://demo.playwright.dev/todomvc" rel="noopener noreferrer">https://demo.playwright.dev/todomvc</a> and plan a test for adding and completing todos</p>

<p>@playwright-repl-generator create a .pw script that adds 3 todos, completes one, and verifies the Active filter</p>

<p>@playwright-repl-healer this script is failing on line 5 — the selector changed after a site update: [paste script]</p>

<p>@playwright-repl-converter convert this .pw script to JavaScript: [paste script]</p>
</blockquote>

<h2>
  
  
  Human + AI, side by side
</h2>

<p>The extension isn't just a bridge for the AI — it's a full REPL you can use alongside it:</p>

<ul>
<li>
<strong>Watch commands execute</strong> in real time as the AI works</li>
<li>
<strong>Intervene manually</strong> — type <code>snapshot</code> to check state, or navigate to a different page</li>
<li>
<strong>Record your own interactions</strong> — click Record, interact with the page, get <code>.pw</code> commands or JS code</li>
<li>
<strong>Switch tabs</strong> — the toolbar dropdown lets you redirect both the AI and your commands to any open tab</li>
</ul>

<h2>
  
  
  vs. other browser MCP servers
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th></th>
<th>playwright-repl</th>
<th>Playwright MCP</th>
<th>Playwriter</th>
</tr>
</thead>
<tbody>
<tr>
<td>MCP tools</td>
<td><strong>2</strong></td>
<td>~70</td>
<td>1</td>
</tr>
<tr>
<td>Your real session</td>
<td>Yes</td>
<td>No</td>
<td>Yes</td>
</tr>
<tr>
<td>Playwright inside browser</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>
<code>expect()</code> assertions</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Full Playwright API</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Human REPL alongside AI</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Try it
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> snapshot
<span class="o">&gt;</span> click <span class="s2">"Sign in"</span>
<span class="o">&gt;</span> fill <span class="s2">"Email"</span> <span class="s2">"user@test.com"</span>
<span class="o">&gt;</span> verify-text <span class="s2">"Dashboard"</span>
</code></pre>

</div>



<p>Your browser. Your sessions. AI that verifies its own work.</p>




<ul>
<li><a href="https://chromewebstore.google.com/detail/dramaturg/ppbkmncnmjkfppilnmplpokdfagobipa" rel="noopener noreferrer">Chrome Web Store — Dramaturg</a></li>
<li><a href="https://github.com/stevez/playwright-repl" rel="noopener noreferrer">GitHub — playwright-repl</a></li>
<li><a href="https://www.npmjs.com/package/@playwright-repl/mcp" rel="noopener noreferrer">npm — @playwright-repl/mcp</a></li>
</ul>

