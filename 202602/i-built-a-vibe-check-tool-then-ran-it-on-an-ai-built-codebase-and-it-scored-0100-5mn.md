---
Title: I Built a Vibe-Check Tool — Then Ran It on an AI-Built Codebase and It Scored 0/100
Description: 
Author: Lakshmi Sravya Vedantham
Date: 2026-02-24T21:50:04.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Setup
</h2>

<p>A few weeks ago I built <a href="https://github.com/LakshmiSravyaVedantham/vibe-check" rel="noopener noreferrer">vibe-check</a> — a CLI that scores how much of your codebase was written by AI, file by file, from 0 (human) to 100 (vibe-coded).</p>

<p>It works by detecting patterns that AI models reliably leave behind: over-commenting, generic naming conventions, hallucinated imports, repetitive structure, placeholder code.</p>

<p>I was happy with it. Ran it on a few projects, got plausible scores. Published a blog post. Got some stars on GitHub.</p>

<p>Then I ran it on a large full-stack web application — React frontend, Express middleware, FastAPI ML backend, multiple machine-learning models for data classification and real-time analytics.</p>

<p>It returned: <strong>0/100. MOSTLY HUMAN.</strong></p>




<h2>
  
  
  The Numbers
</h2>

<p>Here's what vibe-check found across the full repo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Scan path: /path/to/project
Files analyzed: 244   Skipped: 11   Errors: 0

╭───────────────────────────── Repository Summary ─────────────────────────────╮
│   Repo Vibe Score              0/100 — MOSTLY HUMAN                          │
│   Average Score                2                                             │
│   Highest Score                13                                            │
│   Lowest Score                 0                                             │
│   High Risk Files (&gt;=60)       0                                             │
│   Medium Risk Files (40-59)    0                                             │
╰──────────────────────────────────────────────────────────────────────────────╯
</code></pre>

</div>



<p>For contrast, here's commit-prophet — a CLI tool built <em>entirely</em> by an AI agent in a single session, zero human-authored lines:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Files analyzed: 9

╭───────────────────────────── Repository Summary ─────────────────────────────╮
│   Repo Vibe Score              2/100 — MOSTLY HUMAN                          │
│   Average Score                6.4                                           │
│   Highest Score                14                                            │
│   High Risk Files (&gt;=60)       0                                             │
╰──────────────────────────────────────────────────────────────────────────────╯
</code></pre>

</div>



<p><strong>A tool I know was 100% AI-generated scored 2/100.</strong></p>

<p>Something is wrong. Let me break down exactly why.</p>




<h2>
  
  
  Blind Spot #1: Polyglot Codebases
</h2>

<p>vibe-check analyzes Python files. That's it.</p>

<p>The project has:</p>

<ul>
<li>~10,000 lines of Python (ML backend)</li>
<li>~7,000 lines of TypeScript</li>
<li>~14,000 lines of JavaScript</li>
</ul>

<p>That means <strong>roughly 70% of the codebase is completely invisible to vibe-check.</strong> The React frontend — 17 pages, 49 UI components, custom hooks, TanStack Query integration — scored 0 on every single file. Not because it's human-written. Because vibe-check never looked at it.</p>

<p>Here's what a typical frontend file looks like — <code>App.tsx</code>, the routing layer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Switch</span><span class="p">,</span> <span class="nx">Route</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">wouter</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">QueryClientProvider</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tanstack/react-query</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">Dashboard</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@/pages/dashboard</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">Analytics</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@/pages/analytics</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">Settings</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@/pages/settings</span><span class="dl">"</span><span class="p">;</span>
<span class="c1">// ... 20 more page imports</span>
</code></pre>

</div>



<p>That import block — 20+ named page components, perfectly organized, consistent naming convention — is a textbook AI signature. A human would have added them one by one over weeks, with inconsistent capitalization and a few typos along the way. The tool gave it 0 because TypeScript isn't Python.</p>

<p><strong>The fix</strong>: vibe-check needs language-agnostic detectors — or at minimum, separate analyzers for JS/TS, Go, Rust.</p>




<h2>
  
  
  Blind Spot #2: Domain-Specific Vocabulary Defeats Generic Detectors
</h2>

<p>vibe-check's naming detector looks for "generic AI naming" — words like <code>helper</code>, <code>manager</code>, <code>handler</code>, <code>process_data</code>, <code>get_result</code>. The assumption is that AI tends to reach for bland, placeholder-sounding names when it doesn't have strong domain context.</p>

<p>But look at actual variable names in the ML backend:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">confidence_weighted_score</span> <span class="o">=</span> <span class="nf">weighted_avg</span><span class="p">(</span><span class="n">model_outputs</span><span class="p">,</span> <span class="n">confidence_weights</span><span class="p">)</span>
<span class="n">normalized_feature_vector</span> <span class="o">=</span> <span class="nf">standardize</span><span class="p">(</span><span class="n">raw_features</span><span class="p">,</span> <span class="n">per_channel</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="n">inter_class_variance</span> <span class="o">=</span> <span class="n">between_class</span> <span class="o">/</span> <span class="n">within_class</span>
<span class="n">calibrated_threshold</span> <span class="o">=</span> <span class="n">baseline_mean</span> <span class="o">+</span> <span class="p">(</span><span class="mf">2.5</span> <span class="o">*</span> <span class="n">baseline_std</span><span class="p">)</span>
<span class="n">rolling_accuracy</span> <span class="o">=</span> <span class="nf">ema</span><span class="p">(</span><span class="n">correct_predictions</span><span class="p">,</span> <span class="n">window</span><span class="o">=</span><span class="mi">50</span><span class="p">)</span>
</code></pre>

</div>



<p>These are highly specific. <code>calibrated_threshold</code>, <code>inter_class_variance</code>, <code>confidence_weighted_score</code> — none of these match any "generic" pattern. They look exactly like code a domain expert would write.</p>

<p>Except they weren't. They emerged from an AI that had deeply internalized the technical literature — the right terminology, the right mathematical relationships, the right variable names for the problem space. The vocabulary is domain-correct, so the detector reads it as human.</p>

<p><strong>The insight</strong>: AI doesn't always produce generic code. When given sufficient domain context — research papers, technical terminology, specific dataset names — it produces vocabulary that is <em>more</em> specific and precise than what a junior developer would write, not less.</p>




<h2>
  
  
  Blind Spot #3: The Patterns That Actually Reveal AI Are Subtle
</h2>

<p>vibe-check looks for obvious surface signals. But the real AI fingerprints in a mature codebase are structural, not lexical:</p>

<p><strong>Perfect docstring coverage.</strong> Every function in the ML backend has a docstring. Every single one. With Args, Returns, and Raises sections. Human engineers docstring the complex stuff and skip the obvious stuff. Uniform docstring coverage is a strong signal — but vibe-check doesn't check coverage <em>uniformity</em>, just whether comments exist.</p>

<p><strong>Suspiciously comprehensive error handling.</strong> Every API endpoint handles the exact error cases you'd list in a prompt: <code>ValueError</code>, <code>HTTPException</code> with correct status codes, proper JSON error bodies. Real code usually has at least one <code>except Exception as e: pass</code> somewhere. Zero of them here.</p>

<p><strong>Exact adherence to one style guide.</strong> No <code>camelCase</code> leaked into Python. No <code>snake_case</code> leaked into TypeScript. No inconsistency anywhere across 30,000 lines. Human codebases always have some inconsistency — a variable named <code>userID</code> next to <code>user_name</code>, an import sorted slightly wrong. Absolute consistency at scale is itself a signal.</p>

<p><strong>The problem</strong>: these patterns require understanding the <em>distribution</em> of human inconsistency, not just checking for known-bad patterns. That's a much harder problem.</p>




<h2>
  
  
  What a Better Detector Would Look Like
</h2>

<p><strong>1. Cross-language support.</strong> TypeScript and JavaScript are now first-class targets. AST-level analysis, not regex. Parse the import graph, check for the "20 imports added in one commit" pattern.</p>

<p><strong>2. Consistency scoring.</strong> Measure variance in style, naming, comment density across files. High consistency = higher vibe score. Calculate the coefficient of variation for docstring length, comment ratio, function naming patterns.</p>

<p><strong>3. Vocabulary specificity index.</strong> Build a corpus of domain-specific terminology by crawling GitHub repos in specific domains (finance, genomics, logistics). A file with 80% domain-specific terms and 0% generic terms is <em>more</em> suspicious, not less — because humans mix domain and generic terms. Experts still write <code>tmp</code>, <code>data</code>, <code>result</code>.</p>

<p><strong>4. Commit-level analysis.</strong> The biggest tell isn't in the code — it's in the git history. Commits that touch 50 files at once, add 2,000 lines of tested, formatted, documented code in a single push, with zero fix-up commits afterward. Human development has a different tempo: small commits, then fix commits, then refactor commits.</p>

<p><strong>5. Test-to-source ratio uniformity.</strong> AI-generated code tends to have tests that perfectly mirror the source structure, with identical coverage across all modules. Human code has coverage deserts — the boring utility functions have 95% coverage, the complex business logic has 40%.</p>




<h2>
  
  
  The Meta-Irony
</h2>

<p>I built vibe-check using Claude to help write the detectors and tests.</p>

<p>I then ran vibe-check on a project built with Claude and got 0/100.</p>

<p>The tool is measuring the wrong thing. It's measuring <em>style markers of careless AI usage</em> — the AI that doesn't know the domain, doesn't have context, produces generic boilerplate. It gives a clean bill of health to AI that <em>does</em> understand the domain and writes code that looks expert.</p>

<p>Which is, of course, the harder problem to solve. And the more important one.</p>




<h2>
  
  
  What This Means for AI Detection Tools Generally
</h2>

<p>The broader lesson: <strong>AI detection works on distribution shift, and the distribution is shifting.</strong></p>

<p>Early AI-generated code (GPT-3 era) had obvious tells: hallucinated imports, identical function names, <code># Step 1</code>, <code># Step 2</code> comments, <code>TODO: implement this</code>. Those are easy to detect because they're anomalous relative to human code.</p>

<p>Modern AI-generated code, given good context, looks like the best code in the codebase. Perfect naming, comprehensive tests, thorough error handling. The distribution shift is now in the <em>positive</em> direction — the AI code looks better than average human code, not worse.</p>

<p>Detectors trained on "AI code looks sloppy" will miss all of it.</p>




<h2>
  
  
  The Honest Score
</h2>

<p>If I had to estimate the real vibe percentage for this project:</p>

<ul>
<li>React TypeScript frontend: <strong>~60-70%</strong> (scaffold, routing, component structure mostly AI-generated; business logic more human)</li>
<li>FastAPI ML backend: <strong>~40-50%</strong> (API endpoints and boilerplate AI-generated; core algorithms more human-guided)</li>
<li>Data processing pipeline: <strong>~30%</strong> (core logic is human-researched, AI-implemented)</li>
<li>Model training scripts: <strong>~20%</strong> (training design is human, AI transcribed)</li>
</ul>

<p>Overall: probably <strong>45-55%</strong> AI-assisted. vibe-check said <strong>0</strong>.</p>

<p>That's a 45-55 point miss.</p>




<h2>
  
  
  Where to Go From Here
</h2>

<p>I'm planning to rebuild vibe-check's detector suite around these findings:</p>

<ul>
<li>Language-agnostic AST analysis (Python + TypeScript + JavaScript)</li>
<li>Style consistency measurement (variance, not just presence)</li>
<li>Git history analysis (commit size, tempo, fix-up rate)</li>
<li>Test coverage distribution analysis</li>
</ul>

<p>If this is interesting to you, follow along: <a href="https://github.com/LakshmiSravyaVedantham/vibe-check" rel="noopener noreferrer">github.com/LakshmiSravyaVedantham/vibe-check</a></p>

<p>And if you've built an AI detection tool that actually handles these cases — I'd genuinely love to know how.</p>




<p><em>The uncomfortable truth: the better AI gets at writing domain-specific code, the more it looks like expertise rather than generation. Detection tools need to evolve faster than the models they're detecting.</em></p>

