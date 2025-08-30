---
Title: Speed-run a SvelteKit app with the MOTA starter
Description: 
Author: Rastislav ₡ORE
Date: 2025-08-30T21:41:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Want a fresh, batteries-included SvelteKit app without wading through prompts and boilerplate? The <strong>MOTA</strong> starter wraps the official <code>sv</code> CLI with a one-shot installer that scaffolds your app, merges a template, sets sensible dotfiles, handles licensing, and more.</p>

<h2>
  
  
  TL;DR
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bash <span class="nt">-c</span> <span class="s2">"</span><span class="si">$(</span>curl <span class="nt">-fsSL</span> https://cdn.jsdelivr.net/gh/bchainhub/sveltekit-starter/sv-starter.sh<span class="si">)</span><span class="s2">"</span>
</code></pre>

</div>



<p>Run that, follow the prompts, and you’re coding. The script calls <code>npx sv create</code> behind the scenes, then automates the rest. (<a href="https://github.com/bchainhub/sveltekit-starter" rel="noopener noreferrer">GitHub</a>)</p>

<h2>
  
  
  What you get (automatically)
</h2>

<p>After you run the command, the installer:</p>

<ol>
<li>
<strong>Creates a SvelteKit project</strong> via <code>npx sv create</code> (you can still pass extra <code>sv</code> flags if you want—see below).</li>
<li>
<strong>Detects the project folder</strong> and <code>cd</code>s into it.</li>
<li>
<strong>Installs a curated set of base deps</strong> for this starter.</li>
<li>
<strong>Optional Auth setup</strong> (interactive):

<ul>
<li>
<code>None</code> (default)</li>
<li>
<code>@auth/sveltekit</code> (runs <code>npx auth secret</code>)</li>
<li><code>lucia</code></li>
</ul>
</li>
<li>
<strong>Optional Data layer</strong> picker (interactive): Prisma, Drizzle ORM, Supabase, Neon, MongoDB, Redis, or none—kicks off tiny init steps where relevant.</li>
<li>
<strong>Merges a template repository</strong> (default: the <strong>MOTA</strong> template) so you start with a real app structure instead of a blank skeleton.</li>
<li>
<strong>Initializes git</strong> (if needed) and can make a quiet “scaffold” commit for a clean baseline.</li>
<li>
<strong>Runs <code>npm-check-updates</code> locally</strong> to bump ranges, reinstalls, and removes the tool—so you begin on fresh deps.</li>
<li>
<strong>Augments <code>.gitignore</code></strong> (OS cruft, logs, editor dirs) and can optionally <strong>ignore lockfiles</strong> (default <strong>Yes</strong>).</li>
<li>
<strong>Copies shared assets</strong> from the starter repo when you choose them:

<ul>
<li>
<code>.editorconfig</code> (default <strong>Yes</strong>)</li>
<li>
<code>.github/ISSUE_TEMPLATE</code> (default <strong>No</strong>)</li>
</ul>
</li>
<li>
<strong>Sets a LICENSE</strong> (interactive):

<ul>
<li>
<strong>CORE</strong> (custom, non-SPDX) with the right <code>package.json</code> field, or</li>
<li>Common <strong>SPDX</strong> licenses (MIT, Apache-2.0, GPL-3.0-or-later, AGPL-3.0-or-later, LGPL-3.0-or-later, BSD-2/3, MPL-2.0, Unlicense, CC0-1.0, ISC, EPL-2.0).</li>
</ul>
</li>
<li>
<strong>Optional final local commit</strong> of all changes (never pushes).</li>
</ol>

<p>When it’s done, open the new folder and run your dev server with your package manager of choice (the script already installed deps).</p>

<h2>
  
  
  Why this saves time
</h2>

<ul>
<li>
<strong>One command, all the things</strong> – You don’t have to remember the right order of <code>sv</code> + packages + auth + DB + dotfiles + license. It’s orchestrated for you.</li>
<li>
<strong>Real template, not just a skeleton</strong> – By default it merges the <strong>MOTA</strong> template so you start with a working app structure and assets. Less yak-shaving, more shipping.</li>
<li>
<strong>Team-friendly consistency</strong> – Everyone spins up the same baseline (git commit included), which makes diffs and reviews saner.</li>
<li>
<strong>Up-to-date deps from the start</strong> – The <code>npm-check-updates</code> pass means you aren’t starting behind on versions.</li>
<li>
<strong>Sane housekeeping</strong> – <code>.gitignore</code>, <code>.editorconfig</code>, and optional <code>.github</code> templates come prepped, so you don’t copy/paste from old projects.</li>
</ul>

<h2>
  
  
  Options you might want
</h2>

<ul>
<li>
<strong>Use a different template</strong>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  bash <span class="nt">-c</span> <span class="s2">"</span><span class="si">$(</span>curl <span class="nt">-fsSL</span> https://cdn.jsdelivr.net/gh/bchainhub/sveltekit-starter/sv-starter.sh<span class="si">)</span><span class="s2">"</span> <span class="se">\</span>
    <span class="nt">--</span> <span class="nt">--template</span> https://github.com/your-org/your-sveltekit-template.git
</code></pre>

</div>



<p>The default template is the <strong>MOTA</strong> repo. Swap it for your own when needed. (<a href="https://github.com/bchainhub/sveltekit-starter" rel="noopener noreferrer">GitHub</a>)</p>

<ul>
<li>
<strong>Forward flags to <code>sv create</code></strong>
Anything after <code>--</code> goes straight to <code>sv</code>. For example:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  bash <span class="nt">-c</span> <span class="s2">"</span><span class="si">$(</span>curl <span class="nt">-fsSL</span> https://cdn.jsdelivr.net/gh/bchainhub/sveltekit-starter/sv-starter.sh<span class="si">)</span><span class="s2">"</span> <span class="se">\</span>
    <span class="nt">--</span> <span class="nt">--name</span> my-app
</code></pre>

</div>



<h2>
  
  
  Requirements &amp; platform notes
</h2>

<ul>
<li>
<strong>Node 18+</strong> (20+ recommended), <strong>git</strong>, <strong>curl</strong>, and at least one of: <strong>pnpm</strong>, <strong>bun</strong>, <strong>yarn</strong>, or <strong>npm</strong>. The script auto-detects what you have.</li>
<li>Tested on <strong>macOS 14/13</strong> and <strong>Ubuntu 22.04+</strong>.</li>
</ul>

<h2>
  
  
  Security tip
</h2>

<p>Running remote scripts is super convenient—and something you should always eyeball first. You can review it before executing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://cdn.jsdelivr.net/gh/bchainhub/sveltekit-starter/sv-starter.sh | less
</code></pre>

</div>



<p>Then run it once you’re comfortable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bash <span class="nt">-c</span> <span class="s2">"</span><span class="si">$(</span>curl <span class="nt">-fsSL</span> https://cdn.jsdelivr.net/gh/bchainhub/sveltekit-starter/sv-starter.sh<span class="si">)</span><span class="s2">"</span>
</code></pre>

</div>



<p>The repo’s README documents this flow and the safety notes. (<a href="https://github.com/bchainhub/sveltekit-starter" rel="noopener noreferrer">GitHub</a>)</p>

<h2>
  
  
  Handy links
</h2>

<ul>
<li>
<strong>Installer / README</strong> (full step-by-step &amp; flags): (<a href="https://github.com/bchainhub/sveltekit-starter" rel="noopener noreferrer">GitHub</a>)</li>
<li>
<strong>MOTA template repo</strong> (the default template the script merges): (<a href="https://github.com/bchainhub/sveltekit-mota" rel="noopener noreferrer">GitHub</a>)</li>
</ul>

<h3>
  
  
  Wrap-up
</h3>

<p>If you find yourself starting SvelteKit projects often-or onboarding teammates-the <strong>MOTA</strong> starter condenses 15-30 minutes of setup into a guided minute. One command, clear prompts, production-minded defaults. Copy the TL;DR and go build something.</p>

