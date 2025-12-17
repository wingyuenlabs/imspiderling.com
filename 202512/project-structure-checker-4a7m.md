---
Title: Project Structure Checker
Description: 
Author: Genix
Date: 2025-12-17T21:20:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey everyone!</p>

<p>I made a small CLI tool called <strong>PSX</strong> that checks and fixes the basic structure of a project so you don't have to repeat the same boring setup every time. I built it because whenever I start a repo I end up creating the same README, LICENSE, .gitignore, changelog and the usual files. PSX detects the project type, tells you what's missing, and can even create the files for you.</p>

<h3>
  
  
  What it does
</h3>

<p>PSX auto-detects project type (Node, Go, etc.) and runs a set of rules to ensure the repo has the essentials. If something’s missing you can run <code>psx fix</code> and it creates sensible defaults. Rules are configurable via a simple YAML and checks run in parallel so it’s fast even on bigger repos.</p>

<h3>
  
  
  Features
</h3>

<p>Automatic detection of project type and basic scaffolding. Auto-fix mode that creates missing files and folders. Support for multiple languages (Node.js, Go, generic projects). Configurable rules via a simple YAML file.</p>

<h2>
  
  
  Installation
</h2>

<h3>
  
  
  Quick install
</h3>

<h4>
  
  
  Linux / macOS
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-sSL</span> https://raw.githubusercontent.com/m-mdy-m/psx/main/scripts/install.sh | bash
</code></pre>

</div>



<h4>
  
  
  Windows
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">Invoke-WebRequest</span><span class="w"> </span><span class="nt">-Uri</span><span class="w"> </span><span class="s2">"https://raw.githubusercontent.com/m-mdy-m/psx/main/scripts/install.ps1"</span><span class="w"> </span><span class="nt">-OutFile</span><span class="w"> </span><span class="nx">install.ps1</span><span class="p">;</span><span class="w"> </span><span class="o">.</span><span class="n">\install.ps1</span><span class="w"> </span><span class="nx">github</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  Download binary
</h4>

<p>Pre-built releases are available on GitHub for Linux (amd64, arm64), macOS (amd64, arm64) and Windows (amd64). Check Releases on the repo.</p>

<h5>
  
  
  Build from source
</h5>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/m-mdy-m/psx
<span class="nb">cd </span>psx
make build
<span class="nb">sudo </span>make <span class="nb">install</span>
</code></pre>

</div>



<h5>
  
  
  Docker
</h5>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker pull bitsgenix/psx:latest
docker run <span class="nt">--rm</span> <span class="nt">-v</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>:/project psx:latest check
</code></pre>

</div>



<h3>
  
  
  Quick start
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>my-project
psx check
</code></pre>

</div>



<p>If PSX finds missing or weak items you can auto-fix:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>psx fix
</code></pre>

</div>



<p>To confirm each change:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>psx fix <span class="nt">--interactive</span>
</code></pre>

</div>



<h2>
  
  
  Example config (psx.yml)
</h2>

<p>Drop this in the repo root. PSX will auto-detect if you leave <code>type</code> empty.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">version</span><span class="pi">:</span> <span class="m">1</span>
<span class="na">project</span><span class="pi">:</span>
  <span class="na">type</span><span class="pi">:</span> <span class="s2">"</span><span class="s">go"</span>
<span class="na">rules</span><span class="pi">:</span>
  <span class="na">readme</span><span class="pi">:</span> <span class="s">error</span>
  <span class="na">license</span><span class="pi">:</span> <span class="s">warning</span>
  <span class="na">gitignore</span><span class="pi">:</span> <span class="s">warning</span>
  <span class="na">changelog</span><span class="pi">:</span> <span class="s">info</span>
</code></pre>

</div>



<h3>
  
  
  How auto-fix works
</h3>

<p><code>psx fix</code> reads your rules, figures out what’s missing, and creates sensible defaults for README, .gitignore, LICENSE, etc. LICENSE uses standard templates; README is minimal (name, short description, install/run hints). <code>--interactive</code> asks before applying each change.</p>

<h3>
  
  
  Development notes
</h3>

<p>PSX is written in Go and needs Go 1.25+ to build. The code is modular so detectors and rules can be extended. Right now the repo doesn’t include tests (that’s intentional for v1), so if you want to help the project a lot: add tests or lint rules — that’s very welcome.</p>

<h3>
  
  
  Why I made it
</h3>

<p>Every new project felt like repeating the same first 10 minutes of setup. I wanted a tiny tool that enforces a sane baseline so I can start coding faster and keep repos consistent.</p>

<h3>
  
  
  Future enhancements (not in v1)
</h3>

<p><strong>Plugin system for custom rules</strong> — let people add/share custom rule sets.<br>
<strong>Multi-project scanning</strong> — scan multiple repositories at once.<br>
<strong>Git pre-commit hook integration</strong> — run checks locally before commits.<br>
<strong>Language support: Python, Rust, Java</strong> — expand detectors and templates.<br>
<strong>Better code structure and quality improvements</strong> — refactor, cleaner modules, more tests.<br>
<strong>Editor/CI integrations</strong> — VS Code extension, CI checks.<br>
<strong>Other usability improvements</strong> — templates, more interactive flows.</p>

<h3>
  
  
  A note about v1
</h3>

<p>This is the first public version. It’s normal for v1 to have bugs or missing features, so please try it and tell me what breaks or what feels wrong. Use it, open issues, and give feedback — that’s how it gets better.</p>

<h3>
  
  
  If you want to help
</h3>

<p>If you like the idea, please star the repo. Contributions that are most useful right now: add tests, write linters, create new language detectors, improve templates, or send small fixes. Open issues or PRs and I’ll review.</p>

<h3>
  
  
  Links &amp; support
</h3>

<p>Repository: <a href="https://github.com/m-mdy-m/psx" rel="noopener noreferrer">https://github.com/m-mdy-m/psx</a><br>
Issues: <a href="https://github.com/m-mdy-m/psx/issues" rel="noopener noreferrer">https://github.com/m-mdy-m/psx/issues</a><br>
Releases: <a href="https://github.com/m-mdy-m/psx/releases" rel="noopener noreferrer">https://github.com/m-mdy-m/psx/releases</a></p>

<p>Thanks for reading. If you try PSX, tell me what files you always add when starting a project — I’ll consider adding them to default templates. Let’s discuss.</p>

