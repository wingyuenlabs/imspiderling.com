---
Title: Introducing FlameIQ — Deterministic Performance Regression Detection for Python
Description: 
Author: angufibo lincoln
Date: 2026-03-07T22:06:07.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Problem
</h2>

<p>Performance regressions are invisible in code review.</p>

<p>A careless refactor that recompiles a regex on every function call. A new dependency that adds 40ms to your p95 latency. A database query that wasn't indexed. None of these show up in a diff. They accumulate silently across hundreds of commits — a 3ms latency increase here, a 2% throughput drop there — until they become expensive production incidents.</p>

<p>Type checkers enforce correctness automatically. Linters enforce style automatically. <strong>Nothing enforces performance — until now.</strong></p>




<h2>
  
  
  Introducing FlameIQ
</h2>

<p>Today we are releasing <strong>FlameIQ v1.0.0</strong> — an open-source, deterministic, CI-native performance regression engine for Python.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>flameiq-core
</code></pre>

</div>



<p>FlameIQ compares your current benchmark results against a stored baseline and <strong>fails your CI pipeline</strong> if any metric exceeds its configured threshold — the same way a type checker fails your build on a type error.</p>




<h2>
  
  
  Quick Start
</h2>

<p><strong>Step 1 — Initialise</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>my-project
flameiq init
</code></pre>

</div>



<p><strong>Step 2 — Run your benchmarks and produce a metrics file</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"schema_version"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
  </span><span class="nl">"metadata"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"commit"</span><span class="p">:</span><span class="w"> </span><span class="s2">"abc123"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"branch"</span><span class="p">:</span><span class="w"> </span><span class="s2">"main"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"environment"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ci"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"metrics"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"latency"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"mean"</span><span class="p">:</span><span class="w"> </span><span class="mf">120.5</span><span class="p">,</span><span class="w">
      </span><span class="nl">"p95"</span><span class="p">:</span><span class="w"> </span><span class="mf">180.0</span><span class="p">,</span><span class="w">
      </span><span class="nl">"p99"</span><span class="p">:</span><span class="w"> </span><span class="mf">240.0</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"throughput"</span><span class="p">:</span><span class="w"> </span><span class="mf">950.2</span><span class="p">,</span><span class="w">
    </span><span class="nl">"memory_mb"</span><span class="p">:</span><span class="w"> </span><span class="mf">512.0</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Step 3 — Set a baseline</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flameiq baseline <span class="nb">set</span> <span class="nt">--metrics</span> benchmark.json
</code></pre>

</div>



<p><strong>Step 4 — Compare on every PR</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flameiq compare <span class="nt">--metrics</span> current.json <span class="nt">--fail-on-regression</span>
</code></pre>

</div>



<p><strong>Output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  Metric           Baseline    Current      Change   Threshold  Status
  ────────────────────────────────────────────────────────────────────
  latency.p95       2.45 ms     4.51 ms     +84.08%    ±10.0%  REGRESSION
  throughput        412.30      231.50      -43.84%    ±10.0%  REGRESSION

  ✗ REGRESSION — 2 metric(s) exceeded threshold.
</code></pre>

</div>



<p>Exit code <code>1</code>. Pipeline fails. Regression caught before merge.</p>




<h2>
  
  
  A Real Example: Catching a Regex Regression
</h2>

<p>Here is the kind of bug FlameIQ is designed to catch. A developer refactors a text processing function and accidentally recompiles the regex on every call:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># FAST — original implementation
</span><span class="k">def</span> <span class="nf">clean</span><span class="p">(</span><span class="n">text</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="n">text</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">[^\w\s]</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">,</span> <span class="n">text</span><span class="p">)</span>   <span class="c1"># Python caches compiled regex
</span>    <span class="n">text</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">\s+</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s"> </span><span class="sh">"</span><span class="p">,</span> <span class="n">text</span><span class="p">).</span><span class="nf">strip</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">text</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span>

<span class="c1"># SLOW — regressed implementation
</span><span class="k">def</span> <span class="nf">clean</span><span class="p">(</span><span class="n">text</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="n">punct_re</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">compile</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">[^\w\s]</span><span class="sh">"</span><span class="p">)</span>     <span class="c1"># recompiled on every call!
</span>    <span class="n">space_re</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">compile</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">\s+</span><span class="sh">"</span><span class="p">)</span>          <span class="c1"># recompiled on every call!
</span>    <span class="n">text</span> <span class="o">=</span> <span class="n">punct_re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sh">""</span><span class="p">,</span> <span class="n">text</span><span class="p">)</span>
    <span class="n">text</span> <span class="o">=</span> <span class="n">space_re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sh">"</span><span class="s"> </span><span class="sh">"</span><span class="p">,</span> <span class="n">text</span><span class="p">).</span><span class="nf">strip</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">text</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span>
</code></pre>

</div>



<p>This is invisible in code review. The logic is identical. The diff looks clean.<br>
FlameIQ catches it with an 84% p95 latency increase — well above the 10% threshold.</p>


<h2>
  
  
  GitHub Actions Integration
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install FlameIQ</span>
  <span class="na">run</span><span class="pi">:</span> <span class="s">pip install flameiq-core</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Restore baseline cache</span>
  <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/cache@v4</span>
  <span class="na">with</span><span class="pi">:</span>
    <span class="na">path</span><span class="pi">:</span> <span class="s">.flameiq/</span>
    <span class="na">key</span><span class="pi">:</span> <span class="s">flameiq-${{ github.base_ref }}</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run benchmarks</span>
  <span class="na">run</span><span class="pi">:</span> <span class="s">python run_benchmarks.py &gt; metrics.json</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Check for regressions</span>
  <span class="na">run</span><span class="pi">:</span> <span class="s">flameiq compare --metrics metrics.json --fail-on-regression</span>
</code></pre>

</div>



<h2>
  
  
  Key Design Decisions
</h2>

<p><strong>Deterministic by design</strong><br>
Given identical inputs, FlameIQ always produces identical outputs. No randomness, no network calls, no <code>datetime.now()</code>. Safe for any CI environment including air-gapped infrastructure.</p>

<p><strong>No vendor dependency</strong><br>
Baselines are local JSON files. No SaaS account. No API keys. No telemetry. Your performance data stays on your infrastructure.</p>

<p><strong>Direction-aware thresholds</strong><br>
FlameIQ knows that latency increases are regressions and throughput decreases are regressions. Thresholds are sign-aware per metric type — no manual configuration required for known metrics.</p>

<p><strong>Statistical mode</strong><br>
For noisy benchmark environments, FlameIQ can apply the <strong>Mann-Whitney U test</strong> alongside threshold comparison. A regression is only declared if both the threshold is exceeded <em>and</em> the result is statistically significant.</p>

<p><strong>Versioned schema</strong><br>
The metrics schema is versioned (currently v1) with a formal specification. The threshold algorithm and statistical methodology are both fully documented in <code>/specs</code>.</p>


<h2>
  
  
  HTML Reports
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flameiq report <span class="nt">--metrics</span> current.json <span class="nt">--output</span> report.html
</code></pre>

</div>


<p>Generates a self-contained HTML report with a full metric diff table, regression highlights, and trend analysis. No external assets — works offline.</p>


<h2>
  
  
  Configuration
</h2>

<p><code>flameiq.yaml</code> (created by <code>flameiq init</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">thresholds</span><span class="pi">:</span>
  <span class="na">latency.p95</span><span class="pi">:</span>   <span class="s">10%</span>    <span class="c1"># Allow up to 10% latency increase</span>
  <span class="na">latency.p99</span><span class="pi">:</span>   <span class="s">15%</span>
  <span class="na">throughput</span><span class="pi">:</span>    <span class="s">-5%</span>    <span class="c1"># Allow up to 5% throughput decrease</span>
  <span class="na">memory_mb</span><span class="pi">:</span>      <span class="s">8%</span>

<span class="na">baseline</span><span class="pi">:</span>
  <span class="na">strategy</span><span class="pi">:</span> <span class="s">rolling_median</span>
  <span class="na">rolling_window</span><span class="pi">:</span> <span class="m">5</span>

<span class="na">statistics</span><span class="pi">:</span>
  <span class="na">enabled</span><span class="pi">:</span> <span class="kc">false</span>
  <span class="na">confidence</span><span class="pi">:</span> <span class="m">0.95</span>

<span class="na">provider</span><span class="pi">:</span> <span class="s">json</span>
</code></pre>

</div>






<h2>
  
  
  Try the Demo
</h2>

<p>We built a demo project — <strong>flameiq-demo</strong> — that walks through the full regression detection workflow using a real Python library:</p>

<p>👉 <a href="https://github.com/flameiq/demo-flameiq" rel="noopener noreferrer">https://github.com/flameiq/demo-flameiq</a></p>




<h2>
  
  
  Links
</h2>

<ul>
<li>
<strong>PyPI:</strong> <a href="https://pypi.org/project/flameiq-core/" rel="noopener noreferrer">https://pypi.org/project/flameiq-core/</a>
</li>
<li>
<strong>Documentation:</strong> <a href="https://flameiq-core.readthedocs.io" rel="noopener noreferrer">https://flameiq-core.readthedocs.io</a>
</li>
<li>
<strong>Source:</strong> <a href="https://github.com/flameiq/flameiq-core" rel="noopener noreferrer">https://github.com/flameiq/flameiq-core</a>
</li>
<li>
<strong>Demo project:</strong> <a href="https://github.com/flameiq/demo-flameiq" rel="noopener noreferrer">https://github.com/flameiq/demo-flameiq</a>
</li>
</ul>




<p>Feedback, issues, and contributions are welcome. If you have caught a regression with FlameIQ or have a use case we haven't considered, open an issue or start a discussion on GitHub.</p>




<p><strong>Tags:</strong> <code>python</code> <code>opensource</code> <code>devtools</code> <code>ci</code> <code>performance</code></p>

