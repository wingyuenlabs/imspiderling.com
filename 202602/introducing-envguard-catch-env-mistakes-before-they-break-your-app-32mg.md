---
Title: Introducing EnvGuard: Catch .env Mistakes Before They Break Your App
Description: 
Author: Adeyemi
Date: 2026-02-23T21:50:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>EnvGuard is an open-source <code>.env</code> validator that catches missing keys, type mismatches, stale variables, and potential secret leaks before they break your app or CI pipeline.</p>

<p>If you work with <code>.env</code> files, this is the guardrail that prevents avoidable config bugs.</p>

<h2>
  
  
  TL;DR
</h2>

<ul>
<li>Validate <code>.env</code> against <code>.env.example</code>
</li>
<li>Validate values with <code>.env.schema</code> types</li>
<li>Detect likely hardcoded secrets</li>
<li>Find likely unused env variables</li>
<li>Run in watch mode for instant feedback while coding</li>
<li>Enforce stricter checks in CI</li>
</ul>

<h2>
  
  
  Why Teams Need a <code>.env</code> Validator
</h2>

<p>Most configuration failures are not hard problems. They are visibility problems.</p>

<p>You pull a branch and the app fails because one variable is missing.<br>
You fix that and hit a runtime bug because a boolean is <code>"yes"</code> instead of <code>true</code>.<br>
You deploy and discover stale env keys nobody remembers adding.</p>

<p>These issues are easy to fix once identified, but expensive when discovered late.</p>

<p>EnvGuard shifts that feedback earlier.</p>
<h2>
  
  
  What EnvGuard Checks
</h2>
<h3>
  
  
  Missing required keys
</h3>

<p>Compares <code>.env</code> against <code>.env.example</code> and reports missing keys.</p>
<h3>
  
  
  Extra/stale keys
</h3>

<p>Warns on env keys that exist in <code>.env</code> but not in <code>.env.example</code>.</p>
<h3>
  
  
  Type validation
</h3>

<p>With <code>.env.schema</code>, validates types like:</p>

<ul>
<li><code>string</code></li>
<li><code>int</code></li>
<li><code>float</code></li>
<li><code>bool</code></li>
<li><code>url</code></li>
<li><code>email</code></li>
<li><code>json</code></li>
</ul>
<h3>
  
  
  Secret detection
</h3>

<p>Flags suspicious high-entropy values and known token patterns.</p>
<h3>
  
  
  Unused variable detection
</h3>

<p>Scans for env keys that appear unused in the codebase.</p>
<h2>
  
  
  <code>.env.example</code> vs <code>.env.schema</code>
</h2>

<p>Use both, but for different contracts:</p>

<ul>
<li>
<code>.env.example</code> defines which keys should exist (key contract)</li>
<li>
<code>.env.schema</code> defines what each key should look like (type contract)</li>
</ul>

<p>If you only pick one, start with <code>.env.example</code>.<br>
Best coverage comes from using both.</p>
<h2>
  
  
  The Most Practical Feature: <code>watch</code>
</h2>

<p>One-time validation is good.<br>
Continuous validation while coding is better.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>envguard watch
</code></pre>

</div>



<p>Watch mode automatically re-runs validation when env files change.</p>

<p>By default, it watches:</p>

<ul>
<li><code>.env</code></li>
<li><code>.env.example</code></li>
</ul>

<p>Optionally, include schema watching:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>envguard watch <span class="nt">--schema</span> .env.schema
</code></pre>

</div>



<p>This gives immediate feedback after every save and helps prevent late discovery of config breakage.</p>

<h2>
  
  
  Quick Start
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1) Basic key validation</span>
envguard validate

<span class="c"># 2) Add type validation</span>
envguard validate <span class="nt">--schema</span> .env.schema

<span class="c"># 3) CI-friendly strict mode</span>
envguard validate <span class="nt">--strict</span>

<span class="c"># 4) JSON output for tooling</span>
envguard validate <span class="nt">--json</span>

<span class="c"># 5) Continuous checks while coding</span>
envguard watch <span class="nt">--schema</span> .env.schema
</code></pre>

</div>



<h2>
  
  
  Suggested Team Workflow
</h2>

<ol>
<li>Maintain required keys in <code>.env.example</code>.</li>
<li>Add <code>.env.schema</code> for type validation.</li>
<li>Keep <code>envguard watch</code> running during development.</li>
<li>Run <code>envguard validate --strict</code> in CI.</li>
</ol>

<h2>
  
  
  What EnvGuard Is Not
</h2>

<p>EnvGuard is not a secret manager.<br>
It does not replace Vault or cloud secret stores.</p>

<p>It is a focused validation layer for env correctness.</p>
<h2>
  
  
  Closing
</h2>

<p>Configuration bugs are boring and expensive.</p>

<p>EnvGuard helps you catch them early, keep local setups stable, and reduce avoidable CI/deploy failures.</p>

<p>Install:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go <span class="nb">install </span>github.com/atoyegbe/envguard@latest
</code></pre>

</div>



<p>Repo: <a href="https://github.com/atoyegbe/envguard" rel="noopener noreferrer">envguard</a></p>

<p>If you already use another <code>.env</code> checker, what does it catch well and where does it fall short?</p>

