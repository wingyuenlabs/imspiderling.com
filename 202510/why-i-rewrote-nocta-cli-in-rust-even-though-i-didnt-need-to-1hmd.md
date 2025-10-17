---
Title: Why I Rewrote Nocta CLI in Rust (Even Though I Didn't Need To)
Description: 
Author: Marek Jóźwiak
Date: 2025-10-17T21:58:06.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi3yyxtz10mxzt82xpn6r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi3yyxtz10mxzt82xpn6r.png" alt=" " width="800" height="420"></a></p>

<h2>
  
  
  The Backstory
</h2>

<p>When I first built the Nocta UI CLI, it was a simple JavaScript tool — a few Node scripts that let developers initialize projects, install components, and sync their design tokens. It worked great, and honestly, it didn't need to change.</p>

<p>But like many dev experiments, this rewrite started with curiosity, not necessity.</p>

<p>I'd recently come across how packages like <code>@openai/codex</code> ship native Rust binaries inside an npm package — completely transparent to the end user. You install or run it via <code>npx</code>, and under the hood, you're actually executing optimized compiled code. That idea stuck with me.</p>

<p><strong>Could I make Nocta CLI work the same way — still npm-first, but powered by Rust under the hood?</strong></p>

<p>Turns out, yes.</p>

<h2>
  
  
  Introducing the New Nocta CLI
</h2>

<p>The new CLI lives here:</p>

<ul>
<li>👉 <a href="https://www.npmjs.com/package/@nocta-ui/cli" rel="noopener noreferrer">@nocta-ui/cli on npm</a>
</li>
<li>👉 <a href="https://github.com/nocta-ui/nocta-ui-cli" rel="noopener noreferrer">GitHub repo</a>
</li>
</ul>

<p>You can still run it exactly like before:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx @nocta-ui/cli init
npx @nocta-ui/cli add button card accordion
</code></pre>

</div>



<p>But now, when you do, you're running <strong>native Rust binaries</strong> — distributed through npm and executed transparently via a lightweight JavaScript wrapper.</p>

<h2>
  
  
  Why Rewrite Something That Worked Fine?
</h2>

<p>Honestly? <strong>Curiosity.</strong></p>

<p>This rewrite wasn't driven by performance bottlenecks or bugs — it was driven by fascination. I wanted to explore what happens when you combine the ergonomics of npm with the performance and reliability of Rust.</p>

<p>And it turned out to be a surprisingly elegant setup.</p>

<h2>
  
  
  Under the Hood
</h2>

<p>The project is now structured as a polyglot monorepo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.
├── Cargo.lock
├── Cargo.toml
├── crates
│   ├── cli          # Handles commands like init, add, list
│   └── core         # Core logic: config, fs ops, registry, deps
├── js
│   ├── bin/nocta-ui.js   # Tiny Node wrapper around the native binary
│   └── package.json
└── package.json
</code></pre>

</div>



<p>Here's what happens when you run <code>npx @nocta-ui/cli init</code>:</p>

<ol>
<li>
<strong>Node wrapper runs</strong> → detects your platform</li>
<li>
<strong>Loads the right Rust binary</strong> → compiled for macOS, Linux, or Windows</li>
<li>
<strong>Executes native logic</strong> — all the heavy lifting (file ops, config parsing, dependency detection) is done in Rust</li>
</ol>

<p>No crazy runtime setup, no dependencies — just a fast native executable doing its job.</p>

<h2>
  
  
  What's New Under the Hood
</h2>

<p>The CLI still does everything you'd expect:</p>

<ul>
<li>Initializes new projects</li>
<li>Detects frameworks like Next.js, Vite + React, Tanstack Start or React Router 7</li>
<li>Installs Tailwind v4 tokens</li>
<li>Scaffolds components with their dependencies</li>
<li>Integrates seamlessly with your existing package manager</li>
</ul>

<p>But now it feels <strong>instant</strong>. Every command runs with near-zero startup time, thanks to Rust's compiled performance.</p>

<h2>
  
  
  The Real Win: Simplicity in Distribution
</h2>

<p>What made this rewrite truly interesting wasn't the Rust code itself — it was how simple npm makes distributing native binaries once you step outside the "pure JS" mindset.</p>

<p>Each platform gets its own small binary, published alongside the npm package. The JavaScript layer just picks the right one and runs it. No users even notice the difference — except for the speed.</p>

<p>It's a hybrid model that feels… right.</p>

<h2>
  
  
  Lessons Learned
</h2>

<ul>
<li><strong>Rust + npm is a surprisingly powerful combo.</strong></li>
<li>Native performance can live inside your JS workflow without friction.</li>
<li>And most importantly: <strong>rewrites don't always need a reason</strong> — sometimes, they're just how we learn.</li>
</ul>

<h2>
  
  
  The Future
</h2>

<p>This rewrite won't change how developers use Nocta UI — it'll just make their CLI faster, more stable, and easier to maintain.</p>

<p>If you're curious about how native binaries and npm can coexist, check out the repo and peek under the hood:</p>

<p>👉 <a href="https://github.com/nocta-ui/nocta-ui-cli" rel="noopener noreferrer">https://github.com/nocta-ui/nocta-ui-cli</a></p>

<p>And if you're already using Nocta UI, just keep running the same command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx @nocta-ui/cli init
</code></pre>

</div>



<p><strong>It's the same CLI you know — just rebuilt for fun, and made faster by Rust.</strong></p>

