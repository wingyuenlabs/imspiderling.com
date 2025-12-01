---
Title: How I Stopped AI Codebases From Collapsing: Architecture Drift vs. Deterministic Slices
Description: 
Author: vibecodiq
Date: 2025-12-01T21:23:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>AI coding tools are amazing… until they silently break your architecture.</p>

<p>This article shows <strong>exactly</strong> why drift happens — and how <strong>ASA Core v1.0</strong> fixes it with deterministic slice-based generation.</p>




<h2>
  
  
  1. The Real Pain: AI Code Is Not Deterministic
</h2>

<p>Run the same instruction twice and you get different outputs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Version 1
</span><span class="k">class</span> <span class="nc">LoginService</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">execute</span><span class="p">(</span><span class="n">self</span><span class="p">):</span> <span class="bp">...</span>

<span class="c1"># Version 2
</span><span class="k">class</span> <span class="nc">Login</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">):</span> <span class="bp">...</span>
</code></pre>

</div>



<p>Same prompt → different structure → instant drift.</p>




<h2>
  
  
  2. Drift Gets Worse When Specs Change
</h2>

<p>Traditional workflow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>edit spec
↓
Regenerate
↓
💀 Custom code overwritten
</code></pre>

</div>



<p>So engineers stop regenerating.<br>
Schemas drift from implementation.<br>
Tests no longer match the API.</p>


<h2>
  
  
  3. ASA Core v1.0: Deterministic Pipeline
</h2>

<p>ASA uses a strict 3-step flow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>slice.spec.md → slice.contract.json → skeleton code
</code></pre>

</div>



<p>Every step is deterministic — no randomness, no inference.</p>




<h2>
  
  
  4. Before vs After (Real ASA Behavior)
</h2>

<h3>
  
  
  ❌ Before (traditional generation)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Business logic gets overwritten when spec changes
</span><span class="k">def</span> <span class="nf">execute</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">req</span><span class="p">):</span>
    <span class="c1"># custom logic
</span></code></pre>

</div>



<p>Change the spec → overwrite → cry.</p>




<h3>
  
  
  ✅ After (ASA regeneration)
</h3>

<p>You write logic ONLY between markers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># === BEGIN USER CODE ===
</span><span class="k">def</span> <span class="nf">execute</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">request</span><span class="p">:</span> <span class="n">LoginRequest</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">LoginResponse</span><span class="p">:</span>
    <span class="n">user</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">repo</span><span class="p">.</span><span class="nf">get_user_by_email</span><span class="p">(</span><span class="n">request</span><span class="p">.</span><span class="n">email</span><span class="p">)</span>
    <span class="k">return</span> <span class="nc">LoginResponse</span><span class="p">(</span><span class="n">jwt_token</span><span class="o">=</span><span class="sh">"</span><span class="s">123</span><span class="sh">"</span><span class="p">,</span> <span class="n">expires_in</span><span class="o">=</span><span class="mi">3600</span><span class="p">)</span>
<span class="c1"># === END USER CODE ===
</span></code></pre>

</div>



<p>Change the spec:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>asa generate-contract auth/login
asa regenerate-slice auth/login
</code></pre>

</div>



<p>ASA regenerates the file structure and <strong>injects your preserved logic back in</strong>.</p>

<p>This is guaranteed — documented and implemented in Core.</p>




<h2>
  
  
  5. Boundary Violations Stop Instantly
</h2>

<p>ASA lints with AST analysis:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>asa lint auth/login

❌ <span class="o">[</span>LINT FAIL] Boundary violation <span class="k">in </span>repository.py:
   Line 1: Illegal import <span class="s1">'domains.billing.invoice'</span>
   -&gt; Cannot import from other domains.
</code></pre>

</div>



<p>Same-domain imports are fully allowed; cross-domain imports are forbidden.</p>




<h2>
  
  
  6. Quick Reproducible Workflow
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install</span> <span class="nt">-e</span> <span class="s2">".[dev]"</span>

asa create-slice auth/login

<span class="c"># edit slice.spec.md</span>

asa generate-contract auth/login
asa generate-skeleton auth/login

<span class="c"># implement logic between markers</span>

asa lint auth/login
</code></pre>

</div>



<p>And when specs change:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>asa generate-contract auth/login
asa regenerate-slice auth/login
</code></pre>

</div>



<p>Zero drift. Zero overwrite. Fully deterministic.</p>




<h2>
  
  
  7. Why This Matters
</h2>

<p>Clean architecture + deterministic regeneration =<br>
<strong>AI codebases that don’t collapse.</strong></p>

<p>This is the foundation for stable AI-assisted engineering.</p>




<p><strong>Source code &amp; starter kit:</strong><br>
<a href="https://github.com/vibecodiq/asa-starter-kit" rel="noopener noreferrer">https://github.com/vibecodiq/asa-starter-kit</a></p>

