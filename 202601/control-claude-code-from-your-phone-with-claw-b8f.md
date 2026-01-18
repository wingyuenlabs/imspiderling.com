---
Title: Control Claude Code from Your Phone with Claw
Description: 
Author: Raullen Chai
Date: 2026-01-18T22:03:15.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Problem
</h2>

<p>You're deep in a Claude Code session. It's working through a complex task.</p>

<p>But you need to step away - grab coffee, take a call, pick up kids.</p>

<p>What do you do? Leave it running and hope nothing goes wrong?</p>

<p>## The Solution: Claw</p>

<p>I built <strong>Claw</strong> (CLaude AnyWhere) - a zero-dependency Python tool that<br>
  lets<br>
  you monitor and control Claude Code from any device with a browser.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyv0ur0janrsw4vm9ek5x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyv0ur0janrsw4vm9ek5x.png" alt="Claw Screenshot" width="800" height="786"></a></p>

<p>## Features</p>

<ul>
<li>👀 <strong>Live terminal view</strong> - see what Claude is doing in real-time</li>
<li>⚡ <strong>Quick actions</strong> - tap <code>yes</code>, <code>no</code>, <code>continue</code>, or <code>Ctrl+C</code>
</li>
<li>📱 <strong>Mobile-first</strong> - designed for phones with pull-to-refresh</li>
<li>🌐 <strong>Access anywhere</strong> - <code>--share</code> flag creates instant public URL</li>
</ul>

<p>## Quick Start</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
bash
  # Install
  pip install claw-cli

  # Run with remote access
  claw --share

  That's it. Open the URL on your phone. You're in control.

  How It Works

  Claw is a lightweight HTTP server that:
  1. Captures tmux pane content in real-time
  2. Sends keystrokes via tmux send-keys
  3. Serves a mobile-optimized dashboard

  No dependencies beyond Python stdlib. Works on macOS, Linux, and Windows
  (WSL).

  Try It Out

  GitHub: https://github.com/raullenchai/claw
  PyPI: https://pypi.org/project/claw-cli/

  Contributions welcome! Check out our good first issue labels.

  ---
  Built for developers who got tired of walking back to their desks 🦞
</code></pre>

</div>

