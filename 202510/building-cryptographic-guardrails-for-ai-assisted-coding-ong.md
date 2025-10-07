---
Title: Building Cryptographic Guardrails for AI-Assisted Coding
Description: 
Author: Bewaker AI
Date: 2025-10-07T21:32:53.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Bewaker: local-first cryptographic guardrails for critical code (VS Code + git hooks)
</h2>

<p>AI coding assistants are great! That is until a tiny diff lands in the wrong place. For things like <strong>infrastructure-as-code</strong>, <strong>authorization paths</strong>, or <strong>payment logic</strong>, even subtle changes can carry real risk.</p>

<p><strong>Bewaker</strong> is an open-source, local-first tool that helps you lock down the sensitive parts of your repo and <strong>prove</strong> they weren’t changed without an explicit unlock. It runs <strong>inside VS Code</strong> and enforces integrity with <strong>git pre-commit/pre-push hooks</strong>. No backend, no telemetry; keys and audit stay on your machine. <strong>Apache-2.0</strong>.</p>

<ul>
<li>Website: <a href="https://www.bewaker.ai" rel="noopener noreferrer">https://www.bewaker.ai</a>
</li>
<li>GitHub: <a href="https://github.com/bewakerai/bewaker" rel="noopener noreferrer">https://github.com/bewakerai/bewaker</a>
</li>
<li>Marketplace: <a href="https://marketplace.visualstudio.com/items?itemName=BewakerAI.Bewaker" rel="noopener noreferrer">https://marketplace.visualstudio.com/items?itemName=BewakerAI.Bewaker</a>
</li>
</ul>




<h2>
  
  
  What it does
</h2>

<ul>
<li>
<strong>Policy-driven protection</strong> — define a simple YAML policy that lists what to protect.
</li>
<li>
<strong>Cryptographic lockfile</strong> — Bewaker writes a Merkle-rooted, <strong>Ed25519-signed</strong> lock that represents the expected state.
</li>
<li>
<strong>Editor + hooks enforcement</strong> — VS Code surfaces guardrails as you edit; <strong>pre-commit</strong>/<strong>pre-push</strong> hooks block unapproved changes.
</li>
<li>
<strong>Tamper-evident audit</strong> — events are chained locally so you have a verifiable trail.</li>
</ul>

<p>Works alongside Copilot/Cursor—goal is <strong>secure AI-assisted dev</strong>, not blocking AI.</p>




<h2>
  
  
  Quickstart (90 seconds)
</h2>

<ol>
<li><p><strong>Install the VS Code extension</strong><br><br>
Marketplace: <a href="https://marketplace.visualstudio.com/items?itemName=BewakerAI.Bewaker" rel="noopener noreferrer">https://marketplace.visualstudio.com/items?itemName=BewakerAI.Bewaker</a></p></li>
<li>
<p><strong>Create a policy</strong><br><br>
In VS Code, open the Command Palette and run <strong>"Bewaker: Recommend Policy"</strong> (or create <code>.guardpolicy.yml</code>).<br>
</p>
<pre class="highlight yaml"><code><span class="c1"># .guardpolicy.yml (minimal example)</span>
<span class="na">protect</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">infra/**"</span>
  <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/auth/**"</span>
  <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">.github/workflows/**"</span>
</code></pre>

</li>
<li><p><strong>Lock the repository</strong><br><br>
Command Palette → <strong>"Bewaker: Lock Repository"</strong><br><br>
This writes <code>.guardlock</code> (Merkle + Ed25519) to represent the expected state.</p></li>
<li><p><strong>Install git hooks</strong><br><br>
Command Palette → <strong>"Bewaker: Install Git Hooks"</strong><br><br>
This adds pre-commit / pre-push verification.</p></li>
<li><p><strong>Try it</strong><br><br>
Edit a protected file and attempt a commit. The hook will block it until you <strong>explicitly unlock</strong> in the extension.</p></li>
</ol>




<h2>
  
  
  How it works
</h2>

<ul>
<li>
<strong>Policy → <code>.guardpolicy.yml</code></strong> tells Bewaker which files/paths are considered sensitive.
</li>
<li>
<strong>Lockfile → <code>.guardlock</code></strong> stores a Merkle root and signature so drift is detectable and the lock itself is tamper-evident.
</li>
<li>
<strong>Hooks → pre-commit / pre-push</strong> recompute hashes for protected paths and compare against the lockfile before a commit/push is allowed.
</li>
<li>
<strong>Audit → local JSONL</strong> keeps an append-only, tamper-evident record of locks/unlocks/verification events.</li>
</ul>

<p>Local-first by design: <strong>no server, no telemetry, keys stay on your machine.</strong></p>




<h2>
  
  
  How this differs from CODEOWNERS
</h2>

<p><code>CODEOWNERS</code> is great for <strong>review</strong> workflows, but it triggers <strong>during PR</strong>, after a commit exists.<br><br>
Bewaker focuses on <strong>pre-commit</strong>, on the developer’s machine. It’s a <strong>guardrail while editing</strong>, not just during review.</p>

<p>Use both: CODEOWNERS for review gates; Bewaker for local cryptographic integrity.</p>




<h2>
  
  
  Threat model &amp; limitations
</h2>

<ul>
<li>
<strong>Local enforcement</strong>: disables “it was the bot” surprises; hooks still run even if the editor is bypassed.
</li>
<li>
<strong>No network</strong>: reduces data exfil/telemetry risks.
</li>
<li>
<strong>Language-agnostic</strong>: path/policy-based across text files.</li>
</ul>

<p>Known gaps / roadmap:</p>

<ul>
<li>Heuristics are intentionally simple today; “adaptive/ML-ready” risk scoring is in progress.
</li>
<li>Org-wide controls (e.g., server-side pre-receive checks) and multi-approver unlocks are part of a future <strong>Team Edition</strong>.</li>
</ul>




<h2>
  
  
  Call for feedback
</h2>

<ul>
<li>Does the <strong>UX</strong> make sense?
</li>
<li>Any obvious <strong>bypasses</strong> or threat-model blind spots?
</li>
<li>What policy shapes would make this easier to adopt in monorepos?</li>
</ul>

<p>If this scratches an itch for your team, I’d love to hear how you’d extend it.</p>

<ul>
<li>GitHub issues: <a href="https://github.com/bewakerai/bewaker/issues" rel="noopener noreferrer">https://github.com/bewakerai/bewaker/issues</a>
</li>
<li>Website: <a href="https://www.bewaker.ai" rel="noopener noreferrer">https://www.bewaker.ai</a>
</li>
</ul>




<h3>
  
  
  Appendix: tiny policy + what happens
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .guardpolicy.yml</span>
<span class="na">protect</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">infra/**"</span>
  <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/auth/**"</span>
  <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">.github/workflows/**"</span>
</code></pre>

</div>



<ul>
<li>Save the file, Lock Repository, Install Git Hooks.</li>
<li>Change infra/terraform.tf and try git commit.</li>
<li>The pre-commit hook verifies the hash against .guardlock and rejects the commit until you unlock in VS Code (events show up in the local audit).</li>
</ul>

