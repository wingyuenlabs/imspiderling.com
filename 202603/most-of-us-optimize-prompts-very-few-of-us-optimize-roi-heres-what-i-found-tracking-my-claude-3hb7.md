---
Title: Most of us optimize prompts. Very few of us optimize ROI. Here's what I found tracking my Claude Code spend.
Description: 
Author: Akshat Sahu
Date: 2026-03-01T21:35:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’re vibe coding with Claude Code, measure what ships to production.</p>

<p>Run this:<br>
<br>
 </p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx claude-roi
</code></pre>

</div>


<p>See what made it to git vs what just burned tokens.<br>
Cost per commit. Orphaned sessions. Line survival. And many more insights.</p>

<p>Most of us are optimizing prompts. Very few of us are optimizing ROI.</p>

<p>All local. Open source. <br>
GitHub: </p>
<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/Akshat2634" rel="noopener noreferrer">
        Akshat2634
      </a> / <a href="https://github.com/Akshat2634/Codelens-AI" rel="noopener noreferrer">
        Codelens-AI
      </a>
    </h2>
    <h3>
      Agent Productivity-to-Cost Correlatorcodelens-ai
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">Codelens AI</h1>
</div>
<p><strong>Agent Productivity-to-Cost Correlator</strong> — Is your AI coding agent actually shipping code?</p>
<p>Codelens AI ties Claude Code token usage to actual git output. It reads your local Claude Code session files, correlates them with git commits by timestamp, and serves a dashboard answering: <em>"Am I getting ROI from my AI coding agent?"</em></p>
<ul>
<li>One command, zero config</li>
<li>All data stays local</li>
<li>Works with any git repo where you've used Claude Code</li>
</ul>
<div class="markdown-heading">
<h2 class="heading-element">Installation</h2>
</div>
<div class="markdown-heading">
<h3 class="heading-element">Option 1: Run directly (no install)</h3>
</div>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>npx claude-roi</pre>

</div>
<div class="markdown-heading">
<h3 class="heading-element">Option 2: Install globally</h3>

</div>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre><span class="pl-c"><span class="pl-c">#</span> npm</span>
npm install -g claude-roi

<span class="pl-c"><span class="pl-c">#</span> pnpm</span>
pnpm add -g claude-roi

<span class="pl-c"><span class="pl-c">#</span> yarn</span>
yarn global add claude-roi</pre>

</div>
<p>Then run anywhere:</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>claude-roi</pre>

</div>
<div class="markdown-heading">
<h3 class="heading-element">Option 3: Clone and run from source</h3>

</div>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>git clone https://github.com/Akshat2634/Codelens-AI.git
<span class="pl-c1">cd</span> Codelens-AI

<span class="pl-c"><span class="pl-c">#</span> Install dependencies (pick one)</span>
npm install
<span class="pl-c"><span class="pl-c">#</span> or</span>
pnpm install
<span class="pl-c"><span class="pl-c">#</span> or</span>
yarn install

<span class="pl-c"><span class="pl-c">#</span> Run it</span>
node src/index.js</pre>

</div>
<div class="markdown-heading">
<h2 class="heading-element">Prerequisites</h2>

</div>
<ul>
<li>
<strong>Node.js &gt;= 18</strong> — <a href="https://nodejs.org/" rel="nofollow noopener noreferrer">Download</a>
</li>
<li>
<strong>Git</strong> — installed and configured…</li>
</ul>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/Akshat2634/Codelens-AI" rel="noopener noreferrer">View on GitHub</a></div>
</div>


<p>Open source — PRs, feature requests, and ⭐ welcome.</p>

<p>Track your AI ROI Now!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Femoy4v07fm87mpst8w7v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Femoy4v07fm87mpst8w7v.png" alt=" " width="800" height="614"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbj76cif2hcj3b2rxsy18.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbj76cif2hcj3b2rxsy18.png" alt=" " width="800" height="614"></a></p>

