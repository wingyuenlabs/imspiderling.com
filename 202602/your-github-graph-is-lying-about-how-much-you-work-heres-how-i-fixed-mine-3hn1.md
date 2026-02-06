---
Title: Your GitHub Graph is Lying About How Much You Work. Here's How I Fixed Mine.
Description: 
Author: Yuvraj Angad Singh
Date: 2026-02-06T21:49:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>I work 50+ hours a week writing code. My GitHub profile says I'm barely active.</p>

<p>Sound familiar? If you work on private repos — enterprise GitHub, Bitbucket, GitLab, self-hosted — none of that shows on your contribution graph. Recruiters see empty squares. Your profile looks dead.</p>

<p>I got tired of it and built <strong>contrib-mirror</strong> — a CLI that mirrors your real commit timestamps to a public repo. No code exposed. No fake commits. Just your actual work activity, finally visible.</p>

<h2>
  
  
  Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>brew tap yuvrajangadsingh/contrib-mirror
brew <span class="nb">install </span>contrib-mirror
</code></pre>

</div>


<p>Or use the one-liner:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://raw.githubusercontent.com/yuvrajangadsingh/private-work-contributions-mirror/main/install.sh | bash
</code></pre>

</div>


<p>The setup wizard auto-detects your repos, emails, and org:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>contrib-mirror <span class="nt">--setup</span>   <span class="c"># configure once</span>
contrib-mirror           <span class="c"># sync anytime</span>
</code></pre>

</div>

<h2>
  
  
  How It Works
</h2>

<ol>
<li>Discovers repos in your work directory</li>
<li>Creates bare caches (no source code touched)</li>
<li>Extracts commit timestamps matching your email</li>
<li>Optionally pulls PR/review/issue timestamps via GitHub API</li>
<li>Creates empty commits with matching dates in a public mirror repo</li>
<li>Pushes to GitHub</li>
</ol>

<p><strong>Zero code is ever exposed.</strong> The mirror contains only empty commits with timestamps.</p>
<h2>
  
  
  It Also Tracks PRs and Reviews
</h2>

<p>GitHub counts more than commits — PRs opened, reviews submitted, and issues created all count toward your graph. With <code>gh</code> CLI authenticated, contrib-mirror picks those up too.</p>
<h2>
  
  
  Set It and Forget It
</h2>

<p>Add a cron job or launchd agent and it syncs daily at midnight. Your graph stays green without you thinking about it.</p>

<p>The setup wizard handles scheduling too — just pick launchd or cron when prompted.</p>




<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/yuvrajangadsingh" rel="noopener noreferrer">
        yuvrajangadsingh
      </a> / <a href="https://github.com/yuvrajangadsingh/private-work-contributions-mirror" rel="noopener noreferrer">
        private-work-contributions-mirror
      </a>
    </h2>
    <h3>
      Mirror commit timestamps from private work repos to your GitHub contribution graph—without exposing any code
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">Private Work Contributions Mirror</h1>
</div>

<p>Mirror commit timestamps from private work repos to your GitHub contribution graph—without exposing any code.</p>

<div class="snippet-clipboard-content notranslate position-relative overflow-auto"><pre class="notranslate"><code>Before                              After
┌────────────────────────┐          ┌────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░ │          │ ░█░██░█░░█░██░█░░█░██░ │
│ ░░░░░░░░░░░░░░░░░░░░░░ │    →     │ ░█░██░█░░█░██░█░░█░██░ │
│ ░░░░░░░░░░░░░░░░░░░░░░ │          │ ░█░██░█░░█░██░█░░█░██░ │
└────────────────────────┘          └────────────────────────┘
  Private work invisible              Real activity visible
</code></pre></div>

<div class="markdown-heading">
<h2 class="heading-element">Install</h2>
</div>

<p><strong>One-liner (recommended):</strong></p>

<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>curl -fsSL https://raw.githubusercontent.com/yuvrajangadsingh/private-work-contributions-mirror/main/install.sh <span class="pl-k">|</span> bash</pre>

</div>

<p><strong>Homebrew:</strong></p>

<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>brew tap yuvrajangadsingh/contrib-mirror
brew install contrib-mirror</pre>

</div>
<p><strong>Manual:</strong></p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>git clone https://github.com/yuvrajangadsingh/private-work-contributions-mirror.git
<span class="pl-c1">cd</span> private-work-contributions-mirror
bash setup.sh</pre>

</div>
<p>The installer runs an interactive setup wizard that auto-detects your repos, emails, and GitHub org.</p>
<div class="markdown-heading">
<h2 class="heading-element">Usage</h2>
</div>

<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>contrib-mirror            <span class="pl-c"><span class="pl-c">#</span> run sync</span>
contrib-mirror --setup    <span class="pl-c"><span class="pl-c">#</span> reconfigure</span>
contrib-mirror --help     <span class="pl-c"><span class="pl-c">#</span> show help</span>
contrib-mirror --version  <span class="pl-c"><span class="pl-c">#</span> show version</span></pre>

</div>

<div class="markdown-heading">
<h2 class="heading-element">Prerequisites</h2>

</div>

<p><div class="table-wrapper-paragraph"><table>
<br>
<thead>
<br>
<tr>
<br>
<th>Requirement</th>
<br>
<th>Why</th>
<br>
<th>Check</th>
<br>
</tr>
<br>
</thead>
<br>
<tbody>
<br>
<tr>
<br>
<td><strong>Git</strong></td>
<br>
<td>Clone repos, create commits</td>
<br>
<td><code>git --version</code></td>
<br>
</tr>
<br>
<tr>
<br>
<td><strong>Bash</strong></td>
<br>
<td>Run the script</td>
<br>
<td><code>bash --version</code></td>
<br>
</tr>
<br>
<tr>
<br>
<td><strong>Access to private repos</strong></td>
<br>
<td>Fetch commits (SSH or HTTPS)</td>
<br>
<td>
<br>
<code>git remote -v</code> in any work repo</td>
<br>
</tr>
<br>
<tr>
<br>
<td>
<br>
<strong>GitHub CLI</strong> (optional)</td>
<br>
<td>Track PRs, reviews, issues</td>
<br>
<td><a href="https://cli.github.com/" rel="noopener noreferrer">cli.github.com</a></td>
<br>
</tr>
<br>
</tbody>
<br>
</table></div></p>

<p>Works with…</p>
</div>


</div>
<br>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/yuvrajangadsingh/private-work-contributions-mirror" rel="noopener noreferrer">View on GitHub</a></div>
<br>
</div>
<br>


<p>Give it a star if this is useful to you.</p>

