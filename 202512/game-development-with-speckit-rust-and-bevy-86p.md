---
Title: Game development with SpecKit, Rust and Bevy
Description: 
Author: Christian Ledermann
Date: 2025-12-02T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  brkrs — a fun, playable brick-breaker game &amp; learning playground
</h1>

<p><strong>brkrs</strong> is a real, playable Breakout/Arkanoid-style game written in <a href="https://rust-lang.org/" rel="noopener noreferrer"><strong>Rust</strong> 🦀</a> using the <a href="https://bevy.org/" rel="noopener noreferrer"><strong>Bevy</strong></a> engine.<br>
It’s also a <strong>hands-on learning project</strong>, letting you explore:</p>

<ul>
<li>
<strong>Spec-first development</strong> with GitHub <strong>speckit</strong>
</li>
<li>Incremental feature development through issues &amp; PRs</li>
<li>AI-assisted and agentic coding experiments</li>
</ul>

<p>Every feature starts as a spec, flows through an issue or PR, and ends as working Rust code. You can <strong>play the game, explore the code, and learn modern Rust/Bevy workflows all at the same time</strong>.</p>

<blockquote>
<p>Linus Torvalds said: <strong>“Talk is cheap. Show me the code.”</strong></p>
</blockquote>

<p>brkrs lets you play, tinker, and see the specs come alive in a real game.</p>


<h2>
  
  
  The Story Behind brkrs
</h2>

<p>I always wanted to <strong>rewrite my old <a href="https://github.com/cleder/yaac" rel="noopener noreferrer">Arkanoid/Breakout-style game, YaAC 🐧</a></strong>, in a modern game framework.</p>

<p>I began by <strong>manually implementing the core gameplay foundations</strong>: reading documentation, following examples, and building a basic proof-of-concept with the essential mechanics (ball, paddle, bricks).</p>

<p>It quickly became clear that doing everything manually would involve <strong>a steep learning curve and a lot of time</strong>.</p>

<p>brkrs was born as a solution: a way to <strong>learn modern Rust game development</strong>, apply <strong>spec-first workflows</strong>, and experiment with <strong>AI-assisted coding</strong>, all while still having fun playing a real game.</p>


<h2>
  
  
  Try it now
</h2>

<p>You can play a web version on <a href="https://cleder.github.io/brkrs/" rel="noopener noreferrer">GitHub Pages</a></p>
<h2>
  
  
  Key Features
</h2>

<p><code>brkrs</code> is a Breakout/Arkanoid style game implemented in Rust with the Bevy engine. It extends the classic formula with richer physics, paddle rotation, and per-level configuration.</p>

<ul>
<li>Classic Breakout-style gameplay: paddle, ball, bricks, and levels</li>
<li>Levels are human-readable and easy to modify</li>
<li>Spec-first workflow: every feature begins as a spec and ends as working Rust code</li>
<li>Small, incremental PRs demonstrate the development workflow and learning path</li>
<li>Crate-ready and cross-platform (desktop + WebAssembly builds)</li>
<li>A <strong>fun, approachable way to learn Rust, Bevy, and modern coding practices</strong>
</li>
</ul>


<h2>
  
  
  Quickstart (play &amp; learn)
</h2>

<p>Prerequisites: Rust + Cargo + Git<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/cleder/brkrs.git
<span class="nb">cd </span>brkrs
cargo run <span class="nt">--release</span>
</code></pre>

</div>



<p>Controls: move paddle with mouse, scroll wheel to rotate (if enabled), ESC to pause.</p>

<p>Play, tweak, and learn — modify levels, bricks, or mechanics to see specs turn into features.</p>




<h2>
  
  
  Core Systems
</h2>

<ol>
<li>
<strong>Physics (Rapier3D)</strong> – 3D physics constrained to a flat play plane.</li>
<li>
<strong>Game State</strong> – (planned) menu, playing, paused, game over, transitions.</li>
<li>
<strong>Level Loader</strong> – RON file parsing, entity spawning, per-level gravity.</li>
<li>
<strong>Brick System</strong> – Extensible brick behaviors via components &amp; events.</li>
<li>
<strong>Pause System</strong> – ESC to pause, click to resume, with window mode switching (native).</li>
</ol>

<h2>
  
  
  Learning Path &amp; Contribution
</h2>

<p>This project is intended to be <strong>fun and educational</strong>. Suggested learning steps:</p>

<ol>
<li>
<strong>Read a spec</strong> in the repo or wiki</li>
<li>
<strong>Pick a small issue</strong> to implement</li>
<li>
<strong>Submit a PR</strong> that fulfills the spec</li>
<li>
<strong>Experiment</strong> with AI-assisted features or gameplay tweaks</li>
</ol>

<h2>
  
  
  Documentation
</h2>

<p>Full documentation is available at <strong><a href="https://brkrs.readthedocs.io/" rel="noopener noreferrer">brkrs.readthedocs.io</a></strong>:</p>

<ul>
<li>
<a href="https://brkrs.readthedocs.io/en/latest/quickstart.html" rel="noopener noreferrer">Quickstart Guide</a> — Get running in 10 minutes</li>
<li>
<a href="https://brkrs.readthedocs.io/en/latest/developer-guide.html" rel="noopener noreferrer">Developer Guide</a> — Set up a development environment</li>
<li>
<a href="https://brkrs.readthedocs.io/en/latest/api-reference.html" rel="noopener noreferrer">API Reference</a> — Rust API documentation</li>
</ul>




<h2>
  
  
  Why You’ll Enjoy It
</h2>

<ul>
<li>Play a real game while learning coding practices</li>
<li>Watch specs transform into working features</li>
<li>Experiment safely with Rust, Bevy, and AI-assisted workflows</li>
<li>Learn by doing in a <strong>hands-on, playful way</strong>
</li>
</ul>

