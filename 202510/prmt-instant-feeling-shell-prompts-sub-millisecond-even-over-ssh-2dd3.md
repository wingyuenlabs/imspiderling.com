---
Title: prmt: instant-feeling shell prompts (sub-millisecond, even over SSH)
Description: 
Author: Ivan Zakharchanka
Date: 2025-10-29T22:06:45.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>TL;DR</strong>: Rust-powered shell prompt with sub-millisecond rendering that stays fast over SSH.</p>


<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/3axap4eHko" rel="noopener noreferrer">
        3axap4eHko
      </a> / <a href="https://github.com/3axap4eHko/prmt" rel="noopener noreferrer">
        prmt
      </a>
    </h2>
    <h3>
      Ultra-fast, customizable shell prompt generator
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">prmt 🚀</h1>
</div>
<blockquote>
<p>Ultra-fast, customizable shell prompt that won't slow you down</p>
</blockquote>
<p><a href="https://crates.io/crates/prmt" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/158ca8a922fb36be87a368d289f9d92a4b9a8e44a8d1d0c036944222fdec8a52/68747470733a2f2f696d672e736869656c64732e696f2f6372617465732f762f70726d742e737667" alt="Crates.io"></a>
<a href="https://opensource.org/licenses/MIT" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/6cd0120cc4c5ac11d28b2c60f76033b52db98dac641de3b2644bb054b449d60c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f4c6963656e73652d4d49542d79656c6c6f772e737667" alt="License: MIT"></a>
<a href="https://www.rust-lang.org" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/ed707fc0dcfc871765a860d716f0816f298779cd107aa9c3399b081b762e93cc/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f727573742d323032342d6f72616e67652e737667" alt="Rust"></a></p>
<p><a rel="noopener noreferrer" href="https://github.com/3axap4eHko/prmt/./terminal.png"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2F3axap4eHko%2Fprmt%2F.%2Fterminal.png" alt="Terminal"></a></p>
<blockquote>
<p>Rendered with <code>"{path:#89dceb}{rust:#f38ba8:f: 🦀}{git:#f9e2af:f: }\n{ok:#a6e3a1}{fail:#f38ba8} "</code></p>
</blockquote>
<div class="markdown-heading">
<h2 class="heading-element">Features</h2>
</div>
<ul>
<li>
<strong>⚡ Blazing Fast</strong>: Sub-millisecond rendering for typical prompts (~2ms end-to-end)</li>
<li>
<strong>🎨 Highly Customizable</strong>: Full control over colors, formats, and what information to show</li>
<li>
<strong>🚀 Context Aware</strong>: Automatically detects git repos, project files, shows only what's relevant</li>
<li>
<strong>📦 Zero Dependencies</strong>: Single binary, no runtime dependencies required</li>
<li>
<strong>🦀 Memory Efficient</strong>: Zero-copy parsing with SIMD optimizations</li>
<li>
<strong>✨ Smart Rendering</strong>: Only shows information when relevant to your current directory</li>
</ul>
<div class="markdown-heading">
<h2 class="heading-element">Why prmt?</h2>
</div>
<p><strong>Faster than alternatives</strong> – Typical prompts render in ~2ms. Starship averages 10-50ms, oh-my-posh 20-100ms.</p>
<p><strong>Zero configuration needed</strong> – Works out of the box with sensible defaults. Customize only what you want.</p>
<p><strong>Predictable performance</strong> – No async operations, no network calls, no surprises. Your prompt is always instant.</p>
<p><strong>Single binary</strong> – Just install and go. No configuration files required unless…</p>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/3axap4eHko/prmt" rel="noopener noreferrer">View on GitHub</a></div>
</div>





<h2>
  
  
  The problem
</h2>

<p><strong>Starship</strong>: Beautiful but slow (~10-50ms renders). Async I/O adds unpredictable latency over SSH.</p>

<p><strong>oh-my-posh</strong>: Feature-rich but heavy. 20MB binary, 20-100ms renders.</p>

<p><strong>Pure Bash/Zsh</strong>: Fast but unmaintainable config spaghetti.</p>




<h2>
  
  
  Why prmt?
</h2>

<ul>
<li>
<strong>Sub-millisecond renders</strong> — ~10-50 µs for path + git</li>
<li>
<strong>Predictable</strong> — no async I/O, no surprises over SSH</li>
<li>
<strong>Zero config</strong> — works out of the box</li>
<li>
<strong>Drop-in replacement</strong> — migrate from Starship in 5 minutes</li>
</ul>




<h2>
  
  
  Setup
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cargo <span class="nb">install </span>prmt
</code></pre>

</div>



<h3>
  
  
  Bash / Zsh
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Bash: ~/.bashrc</span>
<span class="nv">PS1</span><span class="o">=</span><span class="s1">'$(prmt --code $? "{path:cyan:s} {git:purple:s:on :} {ok:green}{fail:red} ")'</span>

<span class="c"># Zsh: ~/.zshrc (add this line first)</span>
setopt PROMPT_SUBST
<span class="nv">PROMPT</span><span class="o">=</span><span class="s1">'$(prmt --code $? "{path:cyan:s} {git:purple:s:on :} {ok:green}{fail:red} ")'</span>
</code></pre>

</div>



<h3>
  
  
  Fish
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># ~/.config/fish/config.fish
function fish_prompt
  prmt --code $status '{path:cyan:s} {git:purple:s:on :} {ok:green}{fail:red} '
end
</code></pre>

</div>



<blockquote>
<p>PowerShell is also supported — see the <a href="https://github.com/3axap4eHko/prmt#quick-start" rel="noopener noreferrer">README</a>.</p>
</blockquote>




<h2>
  
  
  Performance breakdown
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Prompt</th>
<th>Modules</th>
<th>Time</th>
<th>What's slow?</th>
</tr>
</thead>
<tbody>
<tr>
<td>Minimal</td>
<td><code>{path}</code></td>
<td>~10 µs</td>
<td>Nothing</td>
</tr>
<tr>
<td>Typical</td>
<td><code>{path} {git} {ok}{fail}</code></td>
<td>~1-2 ms</td>
<td>Git status check</td>
</tr>
<tr>
<td>Full</td>
<td><code>{path} {git} {rust} {node}</code></td>
<td>~25-30 ms</td>
<td><strong>Running <code>rustc --version</code> etc</strong></td>
</tr>
<tr>
<td>Fast mode</td>
<td><code>--no-version</code></td>
<td>&lt; 5 ms</td>
<td>Skips all version calls</td>
</tr>
</tbody>
</table></div>

<p><strong>Key insight</strong>: The renderer is microseconds. Version detection (running <code>rustc --version</code>, <code>node --version</code>) dominates total time. Use <code>--no-version</code> if you don't need versions.</p>

<h3>
  
  
  Reproduce locally
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Internal benchmark</span>
prmt <span class="nt">--bench</span> <span class="s1">'{path} {git} {ok}{fail}'</span>

<span class="c"># End-to-end with hyperfine</span>
hyperfine <span class="nt">--warmup</span> 5 <span class="s1">'prmt --no-version "{path} {git} {ok}{fail}"'</span>
</code></pre>

</div>






<h2>
  
  
  Migrating from Starship
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Starship</th>
<th>prmt</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>directory</code></td>
<td>
<code>{path:r}</code> (relative) or <code>{path:s}</code> (short)</td>
</tr>
<tr>
<td><code>git_branch</code></td>
<td>
<code>{git:s}</code> (branch only)</td>
</tr>
<tr>
<td><code>git_status</code></td>
<td>
<code>{git}</code> (branch + status)</td>
</tr>
<tr>
<td><code>character</code></td>
<td>
<code>{ok}{fail}</code> with <code>--code $?</code>
</td>
</tr>
<tr>
<td><code>nodejs</code></td>
<td>
<code>{node:s}</code> (short version)</td>
</tr>
<tr>
<td><code>rust</code></td>
<td>
<code>{rust:s}</code> (short version)</td>
</tr>
</tbody>
</table></div>

<p><strong>Plain prompt</strong> (path + git + indicator):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>prmt <span class="nt">--code</span> <span class="nv">$?</span> <span class="s2">"{path:cyan:s} {git:purple:s:on :} {ok:green}{fail:red} "</span>
</code></pre>

</div>



<p><strong>With versions</strong> (add Node/Rust):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>prmt <span class="nt">--code</span> <span class="nv">$?</span> <span class="s2">"{path:cyan:s} {node:green:s: ⬢}{rust:red:s: 🦀}{git:purple} {ok}{fail} "</span>
</code></pre>

</div>



<p>Output: <code>projects ⬢20.5 🦀1.90 main ❯</code></p>




<h2>
  
  
  Why it's fast
</h2>

<ul>
<li>
<strong>Zero-copy parsing</strong> — minimal allocations</li>
<li>
<strong>Cached module detection</strong> — only checks relevant directories</li>
<li>
<strong>No async runtime</strong> — no Tokio overhead, predictable latency</li>
</ul>

<p>See the <a href="https://github.com/3axap4eHko/prmt#performance" rel="noopener noreferrer">README</a> for benchmarks and implementation details.</p>




<h2>
  
  
  Format cheatsheet
</h2>

<p>Syntax: <code>{module:style:type:prefix:postfix}</code></p>

<p><strong>Modules</strong>: <code>path</code>, <code>git</code>, <code>rust</code>, <code>node</code>, <code>python</code>, <code>go</code>, <code>time</code>, <code>ok</code>, <code>fail</code></p>

<p><strong>Styles</strong>: <code>red</code>, <code>cyan</code>, <code>#89dceb</code> (hex), <code>bold</code>, <code>dim</code>, <code>italic</code></p>

<p><strong>Path types</strong>: <code>r</code> (relative), <code>a</code> (absolute), <code>s</code> (short — last dir only)</p>

<p><strong>Git types</strong>: <code>f</code> (branch+status), <code>s</code> (branch only)</p>

<p><strong>Version types</strong>: <code>m</code> (major), <code>s</code> (short), <code>f</code> (full)</p>

<p><strong>Examples</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">{</span>path:cyan:s<span class="o">}</span>              <span class="c"># Short path in cyan</span>
<span class="o">{</span>git:purple::on :<span class="o">}</span>         <span class="c"># "on main" in purple</span>
<span class="o">{</span>rust:red:s: 🦀<span class="o">}</span>           <span class="c"># "🦀1.90" in red</span>
<span class="o">{</span>ok:green:✓<span class="o">}{</span>fail:red:✗<span class="o">}</span>  <span class="c"># Custom success/fail symbols</span>
</code></pre>

</div>



<p>Full reference in the <a href="https://github.com/3axap4eHko/prmt#format-specification" rel="noopener noreferrer">README</a>.</p>




<h2>
  
  
  Try it
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cargo <span class="nb">install </span>prmt
</code></pre>

</div>



<p>Like it? ⭐ the <a href="https://github.com/3axap4eHko/prmt" rel="noopener noreferrer">repo</a> and share your config!</p>




<p><strong>Ivan Zakharchanka</strong> · GitHub <a href="https://github.com/3axap4eHko" rel="noopener noreferrer">@3axap4eHko</a></p>

