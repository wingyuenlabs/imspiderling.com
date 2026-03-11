---
Title: Your Docs Are Likely Obsolete
Description: 
Author: Renat
Date: 2026-03-11T21:42:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every codebase I've worked on has had the same problem: documentation that was accurate when it was written but slowly drifted out of sync with the code.<br>
Someone adds a new enum variant or config option, the PR gets reviewed, tests pass, it ships - and nobody remembers to update the README. Three months later a new hire follows the docs and nothing works.</p>

<p>Code reviews are supposed to catch this, but in practice "did you update the docs?" is one of the first things to slip through. It's tedious to check, easy to forget, and there's no tooling to enforce it.</p>

<p>So I built <a href="https://github.com/mennanov/blockwatch" rel="noopener noreferrer"><strong>BlockWatch</strong></a> — a linter that creates explicit, machine-enforced links between code and its documentation.</p>
<h2>
  
  
  What it does
</h2>

<p>BlockWatch uses HTML-like tags inside your existing comments to define "blocks" with validation rules. The most useful rule is <code>affects</code>: it creates a dependency between two blocks across files, so changing one will fail the linter until you update the other.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fawgxxruw64y5xa1p5r9s.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fawgxxruw64y5xa1p5r9s.gif" alt=" " width="900" height="350"></a></p>

<p>Here's what that looks like in practice. Say you have a Python config and a README:</p>

<p><strong>config.py:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">SUPPORTED_FORMATS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c1"># &lt;block affects="README.md:formats" keep-sorted&gt;
</span>    <span class="sh">"</span><span class="s">json</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">toml</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">yaml</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># &lt;/block&gt;
</span><span class="p">]</span>
</code></pre>

</div>



<p><strong>README.md:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Supported Formats</span>

<span class="c">&lt;!-- &lt;block name="formats" keep-sorted keep-unique&gt;</span> --&gt;
<span class="p">-</span> JSON
<span class="p">-</span> TOML
<span class="p">-</span> YAML
<span class="c">&lt;!-- &lt;/block&gt;</span> --&gt;
</code></pre>

</div>



<p>If someone adds <code>"xml"</code> to the Python list and opens a PR without touching the README, CI fails with an error pointing to the exact block that needs updating.</p>

<p>The dependency is directional: changes to the code block trigger a check on the docs block, but not the other way around. Fixing a typo in the docs won't cause a failure.</p>

<h2>
  
  
  Other validators
</h2>

<p>Once I had the block-tagging infrastructure, other things fell out naturally.</p>

<h3>
  
  
  Sorted lists
</h3>

<p>If you've ever spent a code review round on "please keep this list alphabetized", <code>keep-sorted</code> automates that:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">allowed_origins</span><span class="pi">:</span>
  <span class="c1"># &lt;block keep-sorted&gt;</span>
  <span class="pi">-</span> <span class="s">api.example.com</span>
  <span class="pi">-</span> <span class="s">app.example.com</span>
  <span class="pi">-</span> <span class="s">docs.example.com</span>
  <span class="c1"># &lt;/block&gt;</span>
</code></pre>

</div>



<p>You can also sort by a regex capture group if the sort key isn't the full line:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">items</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c1"># &lt;block keep-sorted="asc" keep-sorted-pattern="id: (?P&lt;value&gt;\d+)"&gt;
</span>    <span class="sh">"</span><span class="s">id: 1  apple</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">id: 2  banana</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">id: 10 orange</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># &lt;/block&gt;
</span><span class="p">]</span>
</code></pre>

</div>



<h3>
  
  
  Unique entries
</h3>

<p><code>keep-unique</code> prevents duplicates in lists. Like sorting, uniqueness can be scoped to a regex match — useful when you want to enforce unique IDs but the rest of the line may differ.</p>

<h3>
  
  
  Regex validation and line counts
</h3>

<p><code>line-pattern</code> checks that every line matches a pattern. <code>line-count</code> enforces size limits (<code>&lt;=5</code>, <code>&gt;=2</code>, <code>==10</code>, etc.). These are simple but I've found them surprisingly useful for things like slug formats and keeping generated blocks from growing unbounded.</p>

<h2>
  
  
  How it works
</h2>

<p>BlockWatch doesn't use regex to find comments — that approach breaks down quickly across 20+ languages. Instead it uses <a href="https://tree-sitter.github.io/tree-sitter/" rel="noopener noreferrer">Tree-sitter</a> grammars to parse source files and extract comments with actual knowledge of each language's syntax.</p>

<p>Once comments are extracted, a <a href="https://github.com/winnow-rs/winnow" rel="noopener noreferrer">winnow</a>-based parser reads the block definitions. This two-stage design means adding support for a new language is mostly just wiring up its Tree-sitter grammar. Currently supported: Bash, C, C++, C#, CSS, Go (including go.mod/go.sum/go.work), HTML, Java, JavaScript, Kotlin, Makefile, Markdown, PHP, Python, Ruby, Rust, SQL, Swift, TOML, TypeScript, XML, and YAML.</p>

<p>The <code>affects</code> validator builds a dependency graph across files. In diff mode, it checks whether any modified block's dependents were also touched.</p>

<h2>
  
  
  Trying it out
</h2>

<p>Install:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># macOS/Linux</span>
brew <span class="nb">install </span>mennanov/blockwatch/blockwatch

<span class="c"># or via Cargo</span>
cargo <span class="nb">install </span>blockwatch

<span class="c"># or grab a prebuilt binary from GitHub Releases</span>
</code></pre>

</div>



<p>Drop a <code>&lt;block keep-sorted&gt;</code> tag into any comment in your codebase and run <code>blockwatch</code>. No config files needed.</p>

<h3>
  
  
  Diff mode
</h3>

<p>For pre-commit hooks you probably don't want to validate the entire repo every time. You can pipe a git diff to only check blocks you actually touched:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Unstaged changes</span>
git diff <span class="nt">--patch</span> | blockwatch

<span class="c"># Staged changes</span>
git diff <span class="nt">--cached</span> <span class="nt">--patch</span> | blockwatch
</code></pre>

</div>



<p>This also makes adoption incremental — you don't have to fix every existing issue to start using it.</p>

<h3>
  
  
  Pre-commit hook
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .pre-commit-config.yaml</span>
<span class="pi">-</span> <span class="na">repo</span><span class="pi">:</span> <span class="s">local</span>
  <span class="na">hooks</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">blockwatch</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">blockwatch</span>
      <span class="na">entry</span><span class="pi">:</span> <span class="s">bash -c 'git diff --patch --cached --unified=0 | blockwatch'</span>
      <span class="na">language</span><span class="pi">:</span> <span class="s">system</span>
      <span class="na">stages</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">pre-commit</span><span class="pi">]</span>
      <span class="na">pass_filenames</span><span class="pi">:</span> <span class="kc">false</span>
</code></pre>

</div>



<h3>
  
  
  GitHub Actions
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">mennanov/blockwatch-action@v1</span>
</code></pre>

</div>



<h2>
  
  
  Comparison with alternatives
</h2>

<p><strong>Google's <code>keep-sorted</code></strong> handles sorting but nothing else. BlockWatch covers sorting, uniqueness, drift detection, regex validation, and line count limits with a single tag syntax.</p>

<p><strong>Custom CI scripts</strong> can do some of this but tend to be fragile and project-specific. You won't get diff-mode, Tree-sitter parsing, or cross-file dependency tracking without significant effort.</p>

<p><strong>PR review discipline</strong> is the status quo, and it works until it doesn't. The whole point of this tool is to not rely on humans remembering to do something tedious consistently.</p>

<h2>
  
  
  Experimental: AI validation
</h2>

<p>There's also a <code>check-ai</code> validator that sends block content to an LLM for validation using natural language rules:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="c">&lt;!-- &lt;block check-ai="All prices must be under $100"&gt; --&gt;</span>
<span class="nt">&lt;p&gt;</span>Widget A: $50<span class="nt">&lt;/p&gt;</span>
<span class="nt">&lt;p&gt;</span>Widget B: $150<span class="nt">&lt;/p&gt;</span>
<span class="c">&lt;!-- &lt;/block&gt; --&gt;</span>
</code></pre>

</div>



<p>It's optional and off by default. I wouldn't gate CI on it since LLM responses aren't deterministic, but it can be handy for local checks.</p>

<h2>
  
  
  Links
</h2>

<p>The project is MIT-licensed and ships as a single binary with no runtime dependencies.</p>

<ul>
<li>
<strong>GitHub</strong>: <a href="https://github.com/mennanov/blockwatch" rel="noopener noreferrer">github.com/mennanov/blockwatch</a>
</li>
<li>
<strong>GitHub Action</strong>: <a href="https://github.com/mennanov/blockwatch-action" rel="noopener noreferrer">mennanov/blockwatch-action@v1</a>
</li>
</ul>

<p>Feedback and feature requests are welcome. Thanks for reading.</p>

