---
Title: I built a CLI that gives any AI instant context about your project
Description: 
Author: Lakshmi Sravya Vedantham
Date: 2026-03-02T21:46:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every time I start a new AI session, I spend the first few minutes explaining the same things:</p>

<ul>
<li>"This is a FastAPI project"</li>
<li>"We use SQLAlchemy for the ORM"</li>
<li>"The main entry point is <code>src/api/main.py</code>"</li>
<li>"Recent work has been on the auth module"</li>
</ul>

<p>It's tedious. And AI tools like Claude Code, ChatGPT, and Gemini start cold every session.</p>

<p>So I built <code>ctx</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>ctx

ctx save myproject          <span class="c"># scan project, save as context pack</span>
ctx inject myproject        <span class="c"># paste into any AI chat instantly</span>
ctx inject myproject <span class="nt">--target</span> claude  <span class="c"># write CLAUDE.md for Claude Code</span>
</code></pre>

</div>



<h2>
  
  
  What it does
</h2>

<p><code>ctx save</code> scans your project and builds a context pack automatically:</p>

<ul>
<li>
<strong>Stack detection</strong> — finds <code>pyproject.toml</code>, <code>package.json</code>, <code>Cargo.toml</code>, <code>go.mod</code>, <code>Gemfile</code>, etc.</li>
<li>
<strong>Structure map</strong> — directory tree of your <code>src/</code>, <code>tests/</code>, <code>api/</code> folders</li>
<li>
<strong>Git log</strong> — last 10 commits so the AI understands what you've been working on</li>
<li>
<strong>README summary</strong> — first few lines as project context</li>
<li>
<strong>Your notes</strong> — add anything extra on top</li>
</ul>

<p>The result is a clean Markdown file that any AI can parse immediately.</p>

<h2>
  
  
  Inject anywhere
</h2>

<p><code>ctx inject myproject</code> puts the context pack where you need it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ctx inject myproject                      <span class="c"># → clipboard (paste into ChatGPT, Gemini, etc.)</span>
ctx inject myproject <span class="nt">--target</span> claude      <span class="c"># → writes CLAUDE.md in current directory</span>
ctx inject myproject <span class="nt">--target</span> chatgpt     <span class="c"># → clipboard, formatted as a system prompt</span>
</code></pre>

</div>



<p>Claude Code reads <code>CLAUDE.md</code> automatically when you open a project. No paste needed.</p>

<p>For ChatGPT, Gemini, or anything else — one paste at the start of the session and you're fully loaded.</p>

<h2>
  
  
  What a pack looks like
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># myproject</span>

<span class="gu">## Stack</span>
<span class="p">-</span> Python
<span class="p">-</span> Detected from: pyproject.toml

<span class="gu">## Structure</span>
src/
  api/
  models/
tests/

<span class="gu">## Recent commits</span>
<span class="p">-</span> feat: add user auth
<span class="p">-</span> fix: resolve migration conflict
<span class="p">-</span> refactor: extract service layer

<span class="gu">## README</span>
MyProject is a FastAPI app for managing...

<span class="gu">## Notes</span>
Main entry: src/api/main.py
Auth lives in src/auth/ — JWT-based, no sessions
</code></pre>

</div>



<h2>
  
  
  Global vs local packs
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ctx save myproject <span class="nt">--scope</span> global   <span class="c"># ~/.ctx/packs/myproject.md (default, any directory)</span>
ctx save myproject <span class="nt">--scope</span> <span class="nb">local</span>    <span class="c"># .ctx/myproject.md (project-specific, git-committable)</span>
</code></pre>

</div>



<p>Local packs take priority. Commit <code>.ctx/</code> to your repo and your whole team gets the same context.</p>

<h2>
  
  
  The full command set
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ctx save myproject          <span class="c"># scan + save</span>
ctx list                    <span class="c"># show all packs</span>
ctx show myproject          <span class="c"># print pack to terminal</span>
ctx inject myproject        <span class="c"># inject (clipboard by default)</span>
ctx edit myproject          <span class="c"># open in $EDITOR</span>
ctx delete myproject        <span class="c"># remove pack</span>
</code></pre>

</div>



<h2>
  
  
  Why this matters
</h2>

<p>"Context engineering" is the new prompt engineering. The models are good. What holds them back is not having enough context about <em>your</em> project — your conventions, your current work, your architecture decisions.</p>

<p><code>ctx</code> is a local, zero-dependency way to fix that. No account. No sync service. Just Markdown files you control.</p>

<h2>
  
  
  Try it
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>ctx

<span class="c"># In any project</span>
ctx save myproject <span class="nt">--notes</span> <span class="s2">"Add anything you want the AI to know"</span>
ctx inject myproject <span class="nt">--target</span> claude
</code></pre>

</div>



<p>Source: <a href="https://github.com/LakshmiSravyaVedantham/ctx" rel="noopener noreferrer">github.com/LakshmiSravyaVedantham/ctx</a></p>




<p><em>What's your biggest friction starting an AI session on an existing codebase? Drop it in the comments.</em></p>

