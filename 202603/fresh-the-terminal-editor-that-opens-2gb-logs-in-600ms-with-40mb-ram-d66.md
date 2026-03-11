---
Title: Fresh: The Terminal Editor that Opens 2GB Logs in ~600ms with <40MB RAM
Description: 
Author: Tobias weissmann
Date: 2026-03-11T21:59:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve ever cracked open a multi-gigabyte log only to watch your editor choke, this’ll feel like cheating.</p>

<p>Fresh is a new terminal-based editor written in Rust that behaves like a modern GUI editor — command palette, mouse support, LSP, multi-cursor — yet stays tiny and fast enough to open a 2GB ANSI-colored log in about 600ms while sipping ~36MB RAM (author’s benchmark). It’s open-source and solo-built.</p>

<p>Rust Production Cheatsheet: <a href="https://tobiweissmann.gumroad.com/l/pvaqvy" rel="noopener noreferrer">https://tobiweissmann.gumroad.com/l/pvaqvy</a></p>

<p>Why Fresh exists (and why it’s interesting)<br>
Most terminal editors trade power for a steep learning curve or for plugin sprawl. Fresh aims for a different triangle: discoverable UX, modern extensibility, and zero-latency feel — inside the terminal.</p>

<p>Discoverable UX. Menus, a Command Palette (Ctrl+P), familiar keybindings, mouse &amp; scrollbars. If you can use VS Code, you can use Fresh.<br>
Batteries included. Built-in LSP client, multi-cursor, and syntax highlighting out of the box.<br>
Extensible with TypeScript. Write plugins in TS; they run in a sandboxed Deno environment. Today there are examples like Git Log navigation and a Git Blame inspector.<br>
Built for huge files. Fresh was engineered to lazy-load and display even multi-GB files quickly — including rendering ANSI colors properly so your log retains meaning.<br>
The headline number: 2GB in ~600ms<br>
In an early comparison on a 2GB ANSI-heavy log:</p>

<p>Fresh: ~600ms load, ~36MB RAM, ANSI colors shown<br>
Neovim: ~6.5s, ~2GB RAM, ANSI as raw control codes<br>
Emacs: ~10s, ~2GB RAM, ANSI as raw control codes<br>
VS Code: ~20s, OOM-killed on a ~4.3GB-RAM machine<br>
These are author-run, early benchmarks (configs and plugins can change results; e.g., specialized Emacs packages for large files may alter the picture). Still: Fresh’s lazy approach to big files is the point. Try reproducing locally (recipe below).</p>

<p>Quickstart (pick one)<br>
macOS (Homebrew)</p>

<p>brew tap sinelaw/fresh<br>
brew install fresh-editor<br>
Debian/Ubuntu (.deb)</p>
<h2>
  
  
  download from Releases
</h2>

<p>sudo dpkg -i fresh-editor_*.deb<br>
Fedora/RHEL/openSUSE (.rpm)</p>
<h2>
  
  
  download from Releases
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>rpm <span class="nt">-i</span> fresh-editor-<span class="k">*</span>.rpm
Arch <span class="o">(</span>AUR<span class="o">)</span>

yay <span class="nt">-S</span> fresh-editor
Rust <span class="nb">users</span> <span class="o">(</span>crates.io<span class="o">)</span>

cargo <span class="nb">install </span>fresh-editor
Just try it <span class="o">(</span>Node<span class="o">)</span>

npx @fresh-editor/fresh-editor
Releases &amp; instructions are on the project’s README.
</code></pre>

</div>


<p>60-second tour<br>
Ctrl+P: Command Palette for everything.<br>
Mouse + Scrollbars: Select, resize splits, navigate — no modal gymnastics.<br>
Multi-cursor: Hold your breath and edit 30 lines at once.<br>
LSP: Go to definition, references, rename, code actions, diagnostics — works with the language servers you already use.<br>
Extensibility that feels modern<br>
Fresh’s plugin API uses TypeScript running in Deno. That means up-to-date tooling, a rich JS ecosystem, and a sandbox designed for stability. If you’ve ever wanted to script a terminal editor without learning a niche DSL, this is your on-ramp.</p>

<p>Ideas to build next:</p>

<p>“Triage Log” plugin: colorize severity, jump between error frames, extract request IDs.<br>
“Git Time Machine”: blame + inline history slider.<br>
“Code Navigation++”: cross-file symbol search powered by ripgrep + LSP.<br>
Reproduce the big-file demo yourself<br>
Create a synthetic 2GB log with ANSI colors, then time your editor:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># make a 2GB file with colored lines
</span><span class="n">python</span> <span class="o">-</span> <span class="o">&lt;&lt;</span><span class="sh">'</span><span class="s">PY</span><span class="sh">'</span>
<span class="kn">import</span> <span class="n">os</span><span class="p">,</span> <span class="n">sys</span>
<span class="n">line</span> <span class="o">=</span> <span class="sh">"</span><span class="se">\x1b</span><span class="s">[32mINFO</span><span class="se">\x1b</span><span class="s">[0m something happened id=1234</span><span class="se">\n</span><span class="sh">"</span>
<span class="n">target</span> <span class="o">=</span> <span class="mi">2</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span>
<span class="n">written</span> <span class="o">=</span> <span class="mi">0</span>
<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">huge.log</span><span class="sh">"</span><span class="p">,</span><span class="sh">"</span><span class="s">w</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="k">while</span> <span class="n">written</span> <span class="o">&lt;</span> <span class="n">target</span><span class="p">:</span>
        <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">line</span><span class="p">)</span>
        <span class="n">written</span> <span class="o">+=</span> <span class="nf">len</span><span class="p">(</span><span class="n">line</span><span class="p">)</span>
</code></pre>

</div>



<h1>
  
  
  try Fresh
</h1>

<p>/usr/bin/time -v fresh huge.log</p>

<h1>
  
  
  try your usual editor for comparison
</h1>

<p>/usr/bin/time -v nvim huge.log<br>
(Performance will vary by disk, cache warmth, and terminal.)</p>

<p>Where Fresh fits today<br>
Log triage &amp; prod incidents. Open gargantuan logs instantly with colors intact, jump via Command Palette, and keep memory overhead tiny.<br>
SSH &amp; containers. A capable editor that doesn’t need a windowing system; npx makes it trivial to try in throwaway environments.<br>
Roadmap vibes (my take)<br>
Fresh already ships with documentation (User Guide, Plugin Development, Architecture) and regular releases under GPL-2.0. It’s early, but momentum looks real.</p>

<p>The bottom line<br>
Fresh treats the terminal like a first-class editing surface: discoverable UI, modern language smarts, real extensibility, and the kind of big-file performance you usually get from bespoke viewers — not editors. If your week involves tailing logs, wrangling monorepos over SSH, or just craving snappy tools, give it a try.</p>

<p>Optimal setup for massive-log workflows (practical recipe)<br>
If your goal is “open, search, annotate, and keep following a huge changing log” with minimal RAM:</p>

<p>Install Fresh using your platform’s method (above).</p>

<p>Enable auto-revert/auto-reload in Fresh so the file view updates as logs append (feature is listed in the README).</p>

<p>Combine with ripgrep for jumps:</p>

<p>rg -n "ERROR|WARN" huge.log &gt; hits.txt &amp;&amp; fresh hits.txt huge.log<br>
Use the Command Palette to hop between matches.</p>

<p>Add a tiny TS plugin that:</p>

<p>Highlights \b(ERROR|WARN|FATAL)\b<br>
Adds a “Next Error (Alt+E)” command<br>
Extracts fields like request_id= into a side panel<br>
(Plugin scaffolding is documented under “Plugin Development”.)<br>
Keep memory flat. Avoid piping stdin to editors that buffer; work on the file directly so Fresh can lazily map/scan it.</p>

<p>This combo gives you instant open, colored log semantics, jump-to-signal navigation, and low memory.</p>

<p>Quick adoption playbook (for you, the author)<br>
Ship reproducible benchmarks. Add a repo script that creates the synthetic 2GB ANSI log and records /usr/bin/time -v for Fresh and a few editors (with configs disclosed). Publish a “Benchmarks” page and version it with each release.</p>

<p>One-minute demo video + GIFs. Show (1) 2GB open, (2) Command Palette, (3) multi-cursor, (4) LSP. Link from README &amp; website.<br>
Extension gallery. Curate 5–8 “killer” plugins (Log Triage, Blame, TODOs, Color Highlighter, Merge Conflicts) and a template repo for new plugins.<br>
Package everywhere. You already have Homebrew, AUR, .deb, .rpm, crates, and npx — great coverage. Add Scoop/winget for Windows terminals and a static MUSL build for Alpine.</p>

<p>Keymap packs. Offer VS Code / Sublime / Emacs-like keymaps so switching is painless. (Your README already lists keymap support — double down.)<br>
Try Fresh, file an issue, or star the repo to nudge development forward. The project site and README have everything you need to get started.</p>

<p>Note: I would like to express my sincere gratitude to Noam Lewis, the maintainer of the Fresh Git repository, for his valuable work.</p>

<p>GithubLink : <a href="https://sinelaw.github.io/fresh/" rel="noopener noreferrer">https://sinelaw.github.io/fresh/</a></p>

<p>Rust Production Cheatsheet: <a href="https://tobiweissmann.gumroad.com/l/pvaqvy" rel="noopener noreferrer">https://tobiweissmann.gumroad.com/l/pvaqvy</a></p>

