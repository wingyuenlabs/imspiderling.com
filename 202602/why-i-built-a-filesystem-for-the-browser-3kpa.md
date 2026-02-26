---
Title: Why I Built a Filesystem for the Browser
Description: 
Author: Alessandro Pireno
Date: 2026-02-26T22:04:52.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Why I Built a Filesystem for the Browser
</h1>

<p>Browser automation for AI agents has an impedance mismatch problem. We feed agents high-entropy noise — raw HTML, pixel screenshots, brittle CSS selectors — and expect them to produce low-entropy, structured actions. The result is fragile, expensive, and full of silent failure modes.</p>

<p>DOMShell fixes this by exposing the browser's Accessibility Tree as a virtual filesystem. Agents navigate pages with <code>ls</code>, <code>cd</code>, <code>grep</code>, <code>click</code> — the same commands they already know from every Unix system in their training data. No new API to learn. No screenshots to parse. No selectors to break.</p>

<h2>
  
  
  The Impedance Mismatch
</h2>

<p>Three approaches dominate browser automation today. All three are engineering mismatches.</p>

<p><strong>Screenshots + vision models.</strong> The agent takes a screenshot, sends it to a vision model, gets back pixel coordinates, clicks, takes another screenshot. This burns vision tokens on every action, adds a full round-trip per interaction, and fails silently when coordinates shift. A button at (450, 320) moves to (450, 380) because a cookie banner loaded. The agent clicks empty space and doesn't know it. You debug ghost interactions.</p>

<p><strong>CSS selectors / XPath.</strong> Trading pixel fragility for structural fragility. <code>#main &gt; div:nth-child(3) &gt; ul &gt; li:first-child &gt; a</code> breaks when a wrapper div gets added. Even semantic selectors like <code>[data-testid="submit"]</code> depend on the site's developers having added test IDs. Most haven't. And the agent still needs to reason over raw HTML to write the query — thousands of tokens of noise for a single interaction.</p>

<p><strong>Coordinate-based clicks.</strong> Resolution-dependent. Viewport-dependent. Zoom-dependent. Responsive-layout-dependent. Every variable the agent can't control becomes a failure mode.</p>

<p>The common problem: all three approaches force the agent to work with a representation that wasn't designed for programmatic navigation.</p>

<h2>
  
  
  The Accessibility Tree Is the Right Abstraction
</h2>

<p>Browsers already solved "navigate this page without looking at it." The Accessibility Tree — the internal representation that screen readers consume — is deterministic, semantic, and compact. Every button knows it's a button. Every link carries its href. Every input has a label and type. No invisible wrapper divs, no CSS noise, no layout-dependent coordinates.</p>

<p>The AX tree is the low-entropy, structured signal that agents need. The question was how to expose it.</p>

<h2>
  
  
  The Filesystem Metaphor
</h2>

<p>The AX tree has a natural hierarchy. Container elements (navigation, main content, sidebars, forms) map to directories. Interactive elements (buttons, links, inputs) map to files. The whole structure maps cleanly to a filesystem — and every LLM already knows how to operate a filesystem.</p>

<p><code>ls</code> to see what's on a page. <code>cd</code> to scope into a section. <code>cat</code> to inspect an element. <code>grep</code> to search. <code>find</code> to discover by type. <code>click</code> to interact. <code>text</code> to bulk-extract. These commands are in every model's training data. Zero-shot usability.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>dom@shell:$ cd %here%
✓ Entered tab 386872589
  Title: Wikipedia
  URL:   https://www.wikipedia.org/

dom@shell:$ ls
[d] main/
[d] contentinfo/

dom@shell:$ cd main
dom@shell:$ tree 2
main/
├── [d] top_languages/
│   ├── [x] english_7141000_articles_link
│   ├── [x] deutsch_3099000_artikel_link
│   ├── [x] français_2740000_articles_link
│   └── ...
├── [d] search/
│   └── [x] search_input
└── [x] read_wikipedia_in_your_language_btn
</code></pre>

</div>



<p>The page is a directory tree. <code>submit search_input "Artificial intelligence"</code> navigates. The tree auto-refreshes. You're now looking at the article's filesystem.</p>

<p>No screenshots. No coordinates. No selectors.</p>

<h2>
  
  
  Aggressive Flattening
</h2>

<p>The raw AX tree is noisy. Hundreds of wrapper nodes — <code>role=generic</code>, <code>role=none</code>, unnamed divs — exist for CSS layout, not semantics. Without filtering, you get listings of <code>generic_1</code>, <code>generic_2</code>, <code>generic_3</code> with no indication of what anything is.</p>

<p>DOMShell's VFS mapper (<code>vfs_mapper.ts</code>) recursively flattens through non-semantic nodes, promoting their children up. If a <code>role=generic</code> node has one child, the child replaces it. The result is a clean tree where every visible element has a name derived from its accessible name and role: <code>submit_btn</code>, <code>contact_us_link</code>, <code>email_input</code>. Duplicates get disambiguated with <code>_2</code>, <code>_3</code>.</p>

<p>This is the core design decision. Minimizing node bloat maximizes agent signal-to-noise. Every flattened wrapper node is a token the model doesn't waste reasoning about.</p>

<h2>
  
  
  Architecture
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwovbjp172u5p9ngke3xu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwovbjp172u5p9ngke3xu.png" alt=" " width="800" height="373"></a></p>

<p>Three components, cleanly separated.</p>

<p><strong>Chrome Extension (the kernel).</strong> A background service worker runs the shell: command parsing, AX tree traversal via CDP, filesystem mapping, DOM change detection. The side panel is a thin terminal (React + Xterm.js) — no logic, just I/O. The service worker reads the AX tree through <code>chrome.debugger</code> (Chrome DevTools Protocol 1.3), including cross-iframe discovery via <code>Page.getFrameTree</code>.</p>

<p><strong>MCP Server (the bridge).</strong> A standalone Node.js HTTP server on localhost:3001 that any MCP-compatible client connects to — Claude Desktop, Claude Code, Cursor, Windsurf, Gemini CLI. Translates MCP tool calls into shell commands, pipes them to the extension over WebSocket (localhost:9876), streams results back. Multiple clients can connect simultaneously.</p>

<p><strong>Security tiers.</strong> Read-only by default — agents can browse but not act. Write commands (<code>click</code>, <code>type</code>, <code>scroll</code>, <code>js</code>) require <code>--allow-write</code>. Cookie access (<code>whoami</code>) requires <code>--allow-sensitive</code>. Domain allowlists restrict which sites agents can operate on. Every command is audit-logged with timestamps. Auth tokens gate the WebSocket bridge.</p>

<p>The separation is deliberate. You can use DOMShell interactively without the MCP server. You can let an agent browse your tabs without it being able to click "Delete Account."</p>

<h2>
  
  
  Benchmark: 50% Fewer Tool Calls
</h2>

<p>I ran 8 trials across 4 tasks using Claude Opus 4.6 with both DOMShell and Anthropic's built-in browser automation (Claude in Chrome). The metric was tool call count — directly proportional to latency and API cost.</p>

<p>DOMShell averaged <strong>4.3 calls per task</strong> vs Claude in Chrome's <strong>8.6</strong> — a <strong>50% reduction</strong>.</p>

<p>The biggest win was content extraction: DOMShell completed it in 2 calls (navigate + extract) where CiC needed 5-6. The filesystem metaphor lets the agent scope to the right section (<code>cd main/article</code>) and bulk-extract (<code>text</code>) in a single call, rather than navigating through read_page results iteratively.</p>

<p>Where CiC holds an edge is raw JavaScript execution — <code>javascript_exec</code> can batch multiple DOM operations into a single call. DOMShell's counter is the <code>for</code> + <code>script</code> + <code>each</code> pipeline, which collapses multi-page workflows into 1-2 calls by iterating over command output and replaying saved scripts across URLs.</p>

<p>The 50% reduction translates directly to cost and latency. For agents running in production — where every tool call is an API round-trip — halving the call count is a meaningful operational improvement.</p>

<h2>
  
  
  What's Next
</h2>

<p>DOMShell is open source (MIT) and free. On the roadmap: a headless mode — a self-contained Chromium process that agents launch directly for CI pipelines and server-side automation where no visible browser is needed.</p>

<p>The <code>for</code> + <code>script</code> + <code>each</code> pipeline is where the compound efficiency gains live. Save a command sequence as a script, replay it across N URLs in a single call. O(2N) tool calls become O(2). For any agent doing research, extraction, or monitoring across multiple pages, that's a step change.</p>

<p>The browser is your filesystem. <code>ls</code> it.</p>




<p>GitHub: <a href="https://github.com/apireno/DOMShell" rel="noopener noreferrer">github.com/apireno/DOMShell</a><br>
Project Page: <a href="https://pireno.com/domshell" rel="noopener noreferrer">DOMShell</a><br>
Built by <a href="https://pireno.com" rel="noopener noreferrer">Pireno</a></p>

