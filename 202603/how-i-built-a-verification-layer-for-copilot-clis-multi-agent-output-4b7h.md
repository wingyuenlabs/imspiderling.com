---
Title: How I Built a Verification Layer for Copilot CLI's Multi-Agent Output
Description: 
Author: Brad Kinnard
Date: 2026-03-12T22:01:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>GitHub shipped <code>/fleet</code> for parallel subagent dispatch in Copilot CLI earlier this year. It works. The problem is what happens after: you get code from multiple agents and no structured way to know if any of it actually works.</p>

<p>I built <strong>Copilot Swarm Orchestrator</strong> to fill that gap. It wraps <code>copilot -p</code> as isolated subprocesses, each on its own git branch, and verifies every agent's output against its session transcript before anything merges.</p>

<h2>
  
  
  What verification looks like
</h2>

<p>The orchestrator captures each agent's <code>/share</code> transcript and parses it for concrete evidence: commit SHAs, test runner output, build markers, file changes. Every claim the agent makes gets cross-referenced against that evidence. If the agent says "all tests pass" but the transcript shows a test failure, the step fails verification.</p>

<p>Failed steps don't just get retried blindly. The Repair Agent classifies the failure (build error, test failure, missing artifact, dependency issue, timeout) and applies a strategy specific to that failure type. Context accumulates across retries so the agent doesn't repeat the same mistake.</p>

<h2>
  
  
  Quality gates
</h2>

<p>After merge, six automated gates scan the generated code:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Gate</th>
<th>What it catches</th>
</tr>
</thead>
<tbody>
<tr>
<td>Scaffold leftovers</td>
<td>TODO placeholders, Lorem ipsum</td>
</tr>
<tr>
<td>Duplicate detection</td>
<td>Repeated code blocks</td>
</tr>
<tr>
<td>Hardcoded config</td>
<td>Magic strings and values</td>
</tr>
<tr>
<td>README drift</td>
<td>Claims that don't match actual code</td>
</tr>
<tr>
<td>Test isolation</td>
<td>Cross-test dependencies and shared state</td>
</tr>
<tr>
<td>Runtime correctness</td>
<td>Execution-time failures</td>
</tr>
</tbody>
</table></div>

<p>Gates that fail can auto-inject follow-up remediation steps. The orchestrator spawns a targeted Copilot session to fix what the gate flagged.</p>

<h2>
  
  
  Cost tracking
</h2>

<p>Every <code>copilot -p</code> invocation burns a premium request. Model multipliers compound this: o3 costs 20x per invocation, o4-mini costs 5x. An 8-step plan with retries on a 20x model can consume a month's Pro allowance in one run.</p>

<p>The cost estimator predicts consumption before execution starts, using model multipliers and historical failure rates from a persistent knowledge base. You can preview the estimate and exit, or set a hard budget that aborts if the estimate exceeds it. After execution, per-step attribution shows exactly where requests went.</p>

<p>
  Example: cost estimate output for dashboard-showcase demo
  <br>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Plan Analysis:
  Steps: 8 (4 parallel waves)
  Model: o4-mini (5x multiplier)
  Estimated requests: 40-65 (accounting for ~30% retry rate)
  Budget impact: ~13-22% of monthly Pro allocation

Per-step breakdown:
  Step 1 (scaffold):     5 requests (low retry probability)
  Step 2 (API routes):   8 requests (moderate complexity)
  Step 3 (chart logic):  10 requests (high retry probability)
  ...
</code></pre>

</div>




</p>

<h2>
  
  
  v3.2: the speed release
</h2>

<p>The latest release replaced the wave-barrier scheduler with greedy scheduling. Steps launch the moment their dependencies resolve instead of waiting for an entire wave to finish.</p>

<p>Other changes in this release:</p>

<ul>
<li>
<strong>Prompt compression</strong> extracts shared boilerplate into <code>.copilot-instructions.md</code> (which Copilot CLI reads natively), cutting ~60% of repeated tokens per step</li>
<li>
<strong>Octopus merge</strong> for parallel branch completion: one merge commit instead of N</li>
<li>
<strong>Event-driven dependency resolution</strong> via <code>EventEmitter</code> instead of file polling</li>
</ul>

<p>The <code>dashboard-showcase</code> demo (4 agents building a React + Chart.js + Express app with 27 tests) dropped from 8m 48s to 7m 56s.</p>

<h2>
  
  
  By the numbers
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>TypeScript source files</td>
<td>71</td>
</tr>
<tr>
<td>Lines of code</td>
<td>17,903</td>
</tr>
<tr>
<td>Tests passing</td>
<td>649 (Mocha + Node.js assert)</td>
</tr>
<tr>
<td>Built-in demo scenarios</td>
<td>6 (1-min smoke test to 40-min SaaS MVP)</td>
</tr>
<tr>
<td>Contributors</td>
<td>1</td>
</tr>
<tr>
<td>License</td>
<td>ISC</td>
</tr>
</tbody>
</table></div>

<p><a href="https://github.com/moonrunnerkc/copilot-swarm-orchestrator" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">View on GitHub</a>
</p>

<p>The quickest way to see it work:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm start demo-fast
</code></pre>

</div>



<p>Runs two parallel agents in about a minute.</p>




