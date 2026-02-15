---
Title: binmate - a binary version manager
Description: 
Author: cameron turner
Date: 2026-02-15T21:33:19.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p><strong><a href="https://github.com/cturner8/copilot-cli-challenge" rel="noopener noreferrer">binmate</a></strong> is a CLI/TUI application for managing binary installations from GitHub releases. It provides an easy way to install, manage, and switch between different versions of command-line tools.</p>

<h3>
  
  
  Why I Built It
</h3>

<p>Depending on which platform you are working with, especially Linux, many of the common public package manager repositories such as apt may be several versions behind the latest available for software you want to install forcing you to install manually. This can be particularly problematic in project development when working with different toolchains which may not be available in runtime specific package managers such as <code>npm</code>, <code>go.pkg.dev</code> or <code>pypi</code>. This then leads to having to manually maintain updates to new versions which could get tedious depending on the volume of packages you need to upgrade.</p>

<p>Additionally, unlike tools such as <a href="https://github.com/nvm-sh/nvm" rel="noopener noreferrer">nvm</a> for Node.js version management, there is often no straightforward way to switch between different installation versions for troubleshooting or rollback purposes.</p>

<h3>
  
  
  Key Features
</h3>

<ul>
<li>
<strong>Interactive TUI</strong>: Browse and manage binaries with a Terminal User Interface</li>
<li>
<strong>CLI Commands</strong>: Automate binary management with command-line interface</li>
<li>
<strong>Version Management</strong>: Install multiple versions and switch between them</li>
<li>
<strong>GitHub Integration</strong>: Automatically fetch releases from GitHub repositories</li>
<li>
<strong>Database Tracking</strong>: SQLite database tracks all installations and versions</li>
<li>
<strong>Checksum Verification</strong>: Ensures integrity of downloaded binaries</li>
</ul>

<h3>
  
  
  Tech Stack
</h3>

<ul>
<li>Go</li>
<li>Bubble Tea</li>
<li>Lip Gloss</li>
<li>Cobra</li>
<li>Viper</li>
<li>SQLite</li>
</ul>

<h2>
  
  
  Demo
</h2>

<p>This short recording demonstrates the following functionality:</p>

<ul>
<li>first check current version of the <code>gh</code> CLI (initially not installed)</li>
<li>launch <code>binmate</code>
</li>
<li>install non-latest version</li>
<li>verify installed version</li>
<li>re-launch <code>binmate</code>
</li>
<li>update to latest version</li>
<li>verify latest version is installed</li>
</ul>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/i0yZUVkNmwQ">
</iframe>
</p>

<p>Screenshots for remaining features are available on <a href="https://github.com/cturner8/copilot-cli-challenge/blob/main/DEMO.md" rel="noopener noreferrer">GitHub</a></p>


<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/cturner8" rel="noopener noreferrer">
        cturner8
      </a> / <a href="https://github.com/cturner8/copilot-cli-challenge" rel="noopener noreferrer">
        copilot-cli-challenge
      </a>
    </h2>
    <h3>
      Submission for the 2026 DEV Copilot CLI challenge
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">DEV GitHub Copilot CLI Challenge</h1>
</div>
<p><a href="https://github.com/cturner8/copilot-cli-challenge/actions/workflows/test.yml" rel="noopener noreferrer"><img src="https://github.com/cturner8/copilot-cli-challenge/actions/workflows/test.yml/badge.svg" alt="Tests"></a>
<a href="https://github.com/cturner8/copilot-cli-challenge/actions/workflows/release.yml" rel="noopener noreferrer"><img src="https://github.com/cturner8/copilot-cli-challenge/actions/workflows/release.yml/badge.svg" alt="Release"></a>
<a href="https://goreportcard.com/report/github.com/cturner8/copilot-cli-challenge" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/335eef2c8e818f39bcca6e3e2b73e65807d30310b09b15fa562eebbde30d4756/68747470733a2f2f676f7265706f7274636172642e636f6d2f62616467652f6769746875622e636f6d2f637475726e6572382f636f70696c6f742d636c692d6368616c6c656e6765" alt="Go Report Card"></a></p>
<p>Submission for the <a href="https://dev.to/devteam/join-the-github-copilot-cli-challenge-win-github-universe-tickets-copilot-pro-subscriptions-and-50af" rel="nofollow">2026 DEV GitHub Copilot CLI challenge</a>.</p>
<div class="markdown-heading">
<h2 class="heading-element">About binmate</h2>
</div>
<p><strong>binmate</strong> is a CLI/TUI application for managing binary installations from GitHub releases. It provides an easy way to install, manage, and switch between different versions of command-line tools.</p>
<p><a rel="noopener noreferrer" href="https://github.com/cturner8/copilot-cli-challenge/./images/cover.png"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Fcturner8%2Fcopilot-cli-challenge%2F.%2Fimages%2Fcover.png" alt="cover image"></a></p>
<div class="markdown-heading">
<h3 class="heading-element">Key Features</h3>
</div>
<ul>
<li>
<strong>Interactive TUI</strong>: Browse and manage binaries with a Terminal User Interface</li>
<li>
<strong>CLI Commands</strong>: Automate binary management with command-line interface</li>
<li>
<strong>Version Management</strong>: Install multiple versions and switch between them</li>
<li>
<strong>GitHub Integration</strong>: Automatically fetch releases from GitHub repositories</li>
<li>
<strong>Database Tracking</strong>: SQLite database tracks all installations and versions</li>
<li>
<strong>Checksum Verification</strong>: Ensures integrity of downloaded binaries</li>
</ul>
<p>See <a href="https://github.com/cturner8/copilot-cli-challenge/./DEMO.md" rel="noopener noreferrer">demo.md</a> for a demo and screenshots.</p>
<div class="markdown-heading">
<h2 class="heading-element">Installation</h2>

</div>
<div class="markdown-heading">
<h3 class="heading-element">Using the Install Script (Recommended)</h3>

</div>
<div class="markdown-heading">
<h4 class="heading-element">Unix (Linux/macOS)</h4>

</div>
<p>Install the latest version using our install script:</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>curl -fsSL https://raw.githubusercontent.com/cturner8/copilot-cli-challenge/main/install.sh <span class="pl-k">|</span> bash</pre>

</div>
<p>By default, the installer now auto-imports the installed <code>binmate</code> binary for self-management using its resolved release URL…</p>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/cturner8/copilot-cli-challenge" rel="noopener noreferrer">View on GitHub</a></div>
</div>


<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<p>Development of the application used the following GitHub Copilot CLI capabilities:</p>

<ul>
<li>
<strong>Agent instructions</strong> for baseline codebase structure and standards, later extended to also heavily utilise <a href="https://github.com/steveyegge/beads" rel="noopener noreferrer">beads</a> for improved task tracking and persistent agent memory.</li>
<li>
<strong>Custom agents</strong> to provide more focused and specialised agents. The following were originally sourced from the <a href="https://github.com/github/awesome-copilot/blob/main/docs/README.agents.md" rel="noopener noreferrer">awesome-copilot</a> repository:

<ul>
<li>Critical Thinking, see <a href="https://github.com/cturner8/copilot-cli-challenge/blob/main/.github/agents/critical-thinking.agent.md" rel="noopener noreferrer">critical-thinking.agent.md</a>
</li>
<li>Go Development Expert, revised from "Go MCP Server Development Expert" to be a generic go agent, see <a href="https://github.com/cturner8/copilot-cli-challenge/blob/main/.github/agents/go-expert.agent.md" rel="noopener noreferrer">go-expert.agent.md</a>
</li>
<li>SQLite Database Administrator, revised from "PostgreSQL Database Administrator" agent, see <a href="https://github.com/cturner8/copilot-cli-challenge/blob/main/.github/agents/sqlite-dba.agent.md" rel="noopener noreferrer">sqlite-dba.agent.md</a>
</li>
</ul>


</li>

<li>

<strong>MCP</strong>

<ul>
<li>
<code>context7</code> for providing up to date documentation for packages and libraries.</li>
<li>The <code>GitHub MCP</code> built into Copilot CLI was ideal for allowing the agent direct access to search public codebases, access to action workflow logs for troubleshooting, allowing follow up tasks and recommendations to be extracted from PR comments.</li>
</ul>


</li>

<li>

<strong>Plan mode</strong> to refine and improve the following implementations:

<ul>
<li>initial application architecture and structure</li>
<li>database schema</li>
<li>TUI functionality based on existing CLI functionality</li>
<li>build and release action workflows</li>
<li>end-to-end testing workflow (still in progress)</li>
</ul>


</li>

<li>

<strong>Delegate to cloud agent</strong> once an implementation plan is finalised, allowing Copilot to continue working on a task freeing me up to focus on something else. It was also useful in allowing development to continue in moments where I had less time available to spend at my desk.</li>

<li>

<strong>Code review</strong> to identify any potential issues or areas for improvement in code changes, particularly helpful given I worked alone on the project.</li>

</ul>

