---
Title: How We Built an Automated MCP Security Scanner (And What We Found)
Description: 
Author: ecap0
Date: 2026-02-21T21:59:28.000Z
Robots: noindex,nofollow
Template: index
---
<p>AI agents are executing code, reading files, and making API calls on your behalf every day. The tools they use — MCP servers — are the new attack surface nobody is talking about. Here's how we built a scanner to audit them automatically.</p>

<h2>
  
  
  The Problem We Set Out to Solve
</h2>

<p>When you install an MCP server, you're giving an AI agent a new capability. That server might read your filesystem, execute shell commands, or call external APIs. But who audited that code before it ran on your machine?</p>

<p>Nobody. Until now.</p>

<p>At <a href="https://agentaudit.dev" rel="noopener noreferrer">AgentAudit</a>, we built an automated multi-agent pipeline that audits MCP servers, npm packages, pip packages, and AgentSkills — and flags security risks before your agent ever touches them.</p>

<h2>
  
  
  The Architecture
</h2>

<p>Our audit pipeline runs three specialized sub-agents in parallel, each with a different security lens:</p>

<h3>
  
  
  Agent 1: Static Analysis
</h3>

<p>Scans the source code for known vulnerability patterns:</p>

<ul>
<li>Unsanitized shell command injection (<code>child_process.exec</code> with user input)</li>
<li>Hardcoded credentials and API keys</li>
<li>Overly broad filesystem access permissions</li>
<li>Insecure deserialization</li>
</ul>

<h3>
  
  
  Agent 2: Capability Graph Analysis
</h3>

<p>This is where we go beyond traditional scanners. We parse the MCP server's <strong>tool schema declarations</strong> — the JSON descriptions of what each tool can do — and cross-reference them against what the code actually does.</p>

<p>A weather MCP server that declares it only reads weather data but internally has access to your filesystem? That's a red flag. We catch that gap.</p>

<h3>
  
  
  Agent 3: Dependency Chain Auditor
</h3>

<p>Recursively scans the dependency tree for:</p>

<ul>
<li>Known CVEs in transitive dependencies</li>
<li>Packages with unusually broad permissions</li>
<li>Supply chain anomalies (e.g., a package that changed maintainers 2 weeks ago)</li>
</ul>

<h2>
  
  
  Multi-Agent Consensus
</h2>

<p>Each agent produces a structured audit report. A consensus layer then:</p>

<ol>
<li>
<strong>Deduplicates</strong> overlapping findings</li>
<li>
<strong>Assigns severity</strong> based on exploitability in an agent context</li>
<li>
<strong>Generates a Trust Score</strong> (0–100) for the package</li>
</ol>

<p>Why multi-agent consensus? Because a single model hallucinates. Three models with different system prompts, cross-checking each other, don't.</p>

<h2>
  
  
  The Results (So Far)
</h2>

<p>After running 211 independent audit reports across 194 packages:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Severity</th>
<th>Count</th>
<th>% of Total</th>
</tr>
</thead>
<tbody>
<tr>
<td>🔴 Critical</td>
<td>5</td>
<td>4.2%</td>
</tr>
<tr>
<td>🟠 High</td>
<td>9</td>
<td>7.6%</td>
</tr>
<tr>
<td>🟡 Medium</td>
<td>63</td>
<td>53.4%</td>
</tr>
<tr>
<td>🟢 Low</td>
<td>41</td>
<td>34.7%</td>
</tr>
</tbody>
</table></div>

<p><strong>Average Trust Score: 98/100.</strong> The MCP ecosystem is mostly safe — but those 14 critical/high findings represent real, exploitable vulnerabilities.</p>

<p>The most common patterns:</p>

<ol>
<li>
<strong>Shell command injection via prompt input</strong> — crafted prompts cause MCP servers to execute arbitrary shell commands</li>
<li>
<strong>Environment variable leakage</strong> — API keys accidentally included in LLM context windows</li>
<li>
<strong>Overly broad filesystem access</strong> — servers requesting full <code>~/</code> access when they need one directory</li>
</ol>

<h2>
  
  
  What Makes MCP Security Different
</h2>

<p>Traditional scanners (Snyk, Socket) are great for known CVEs and supply chain risks. But MCP servers introduce a different threat model:</p>

<ul>
<li>
<strong>The attack vector is the prompt</strong>, not the network</li>
<li>
<strong>The "user" is an AI agent</strong> — it won't notice suspicious behavior</li>
<li>
<strong>The execution context</strong> is your local machine or production server</li>
</ul>

<p>A package can pass every traditional security check and still be exploitable through adversarial prompts. That's the gap we're filling.</p>

<h2>
  
  
  Try It
</h2>

<p>Audit any MCP server, npm package, or pip package at <strong><a href="https://agentaudit.dev" rel="noopener noreferrer">agentaudit.dev</a></strong>.</p>

<p>Full findings: <a href="https://dev.to/ecap0/the-state-of-mcp-server-security-in-2026-118-findings-across-68-packages-4fkd">State of MCP Security 2026</a></p>




<p><em>Every finding is cross-validated by three independent AI agents before it reaches you.</em></p>

