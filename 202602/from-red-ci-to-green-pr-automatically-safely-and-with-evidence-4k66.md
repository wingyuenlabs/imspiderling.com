---
Title: From Red CI to Green PR — Automatically, Safely, and with Evidence
Description: 
Author: manoj mallick
Date: 2026-02-06T21:22:38.000Z
Robots: noindex,nofollow
Template: index
---


<p><em>This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge</a>.</em></p>




<h2>
  
  
  What I Built
</h2>

<p>I built <strong>copilot-ci-doctor</strong>, a CLI tool that diagnoses and fixes <strong>GitHub Actions CI failures</strong> using <strong>GitHub Copilot CLI as its core reasoning engine</strong>.</p>

<p>Instead of manually digging through noisy logs and guessing fixes, the tool turns a failed CI run into a structured, evidence-based workflow:</p>

<blockquote>
<p><strong>failure → evidence → reasoning → safe fix → green CI → Pull Request</strong></p>
</blockquote>

<p>Given a failed workflow, copilot-ci-doctor:</p>

<ul>
<li>Collects a tagged <strong>Evidence Bundle</strong> (repo metadata, failed jobs, logs, workflow YAML)</li>
<li>Uses <strong>GitHub Copilot CLI</strong> to reason about the failure</li>
<li>Explains <em>why</em> the CI failed in plain English</li>
<li>Generates <strong>minimal, safe patch diffs</strong> with confidence scores</li>
<li>Iteratively applies fixes until CI passes</li>
<li>Automatically opens a <strong>Pull Request</strong> against <code>main</code>
</li>
</ul>

<p>This is <strong>not</strong> log summarization or autocomplete.<br>
Copilot is used as a <strong>reasoning engine</strong> that must justify its conclusions using evidence.</p>


<h2>
  
  
  Demo
</h2>

<p><strong>40-second end-to-end demo (recommended viewing):</strong></p>

<p>👉 <a href="https://github.com/manojmallick/copilot-ci-doctor#-40-second-demo-end-to-end" rel="noopener noreferrer">https://github.com/manojmallick/copilot-ci-doctor#-40-second-demo-end-to-end</a></p>

<blockquote>
<p>One command → failing CI → Copilot reasoning → safe fixes → green CI → PR<br>
</p>


</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx copilot-ci-doctor demo
</code></pre>

</div>



<p>What the demo shows:</p>

<ol>
<li>A demo repository is created with a deliberately broken GitHub Actions workflow</li>
<li>CI fails ❌</li>
<li>copilot-ci-doctor enters an automated loop:</li>
</ol>

<ul>
<li>analyzes the failure</li>
<li>explains the root cause</li>
<li>proposes a minimal patch</li>
<li>applies and pushes the fix</li>
<li>waits for CI to re-run

<ol>
<li>The process repeats (multiple iterations if needed)</li>
<li>CI turns green ✅</li>
<li>A Pull Request is automatically opened with the fix</li>
</ol>
</li>
</ul>

<p>The demo handles <strong>real GitHub latency</strong> and shows the full lifecycle, including:</p>

<ul>
<li>multiple CI failures</li>
<li>diff previews</li>
<li>iteration scoreboard</li>
<li>final PR link</li>
</ul>

<p>Source code and demo assets:<br>
<a href="https://github.com/manojmallick/copilot-ci-doctor" rel="noopener noreferrer">https://github.com/manojmallick/copilot-ci-doctor</a></p>

<p>npm package:<br>
<a href="https://www.npmjs.com/package/copilot-ci-doctor" rel="noopener noreferrer">https://www.npmjs.com/package/copilot-ci-doctor</a></p>




<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<p>This project fundamentally changed how I think about GitHub Copilot.</p>

<p>Instead of using Copilot to <em>write code</em>, I used <strong>GitHub Copilot CLI to reason about systems</strong>.</p>

<p>Copilot CLI is used to:</p>

<ul>
<li>analyze CI evidence and form <strong>ranked hypotheses</strong>
</li>
<li>explain failures in plain English (including <em>why CI fails but local passes</em>)</li>
<li>generate <strong>minimal unified diffs</strong>, not full rewrites</li>
<li>attach confidence scores and risk levels to each fix</li>
</ul>

<p>To make this reliable:</p>

<ul>
<li>Every Copilot response must follow a <strong>strict JSON contract</strong>
</li>
<li>Every conclusion must reference evidence IDs (E1, E2, …)</li>
<li>Patch diffs are validated and normalized before being applied</li>
<li>A <strong>single-call mode</strong> combines analysis + explanation + patch to reduce token usage by ~60%</li>
</ul>

<p>The result is a workflow where Copilot behaves less like an assistant and more like a <strong>careful, explainable CI engineer</strong>.</p>

<p>This challenge pushed me to think beyond autocomplete and explore how Copilot CLI can safely automate complex, real-world developer workflows.</p>

