---
Title: Your AI Agent is Bleeding Money (And You're Making It Worse)
Description: 
Author: Franklin Mayoyo
Date: 2025-12-30T21:19:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>Let me guess: you're building a SaaS product with Claude. You've got customers making API calls, burning through tokens like a Tesla on fire. Your AWS bill looks like a phone number. And somewhere, a product manager is asking why your margins are trash.</p>

<p>I'll tell you why: you're shoving encyclopedias into every single API call when you could be carrying a library card.</p>

<h2>
  
  
  The $10K/Month Mistake
</h2>

<p>Here's what I see in every production codebase I audit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Your current nightmare
</span><span class="n">system_prompt</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
You are DataCorp</span><span class="sh">'</span><span class="s">s financial analyst assistant.

BRAND GUIDELINES:
- Use active voice in all communications
- Keep sentences under 20 words
- Reference our Q4 2024 performance metrics
- Always cite from our approved data sources
</span><span class="gp">...</span> <span class="p">[</span><span class="mi">2</span><span class="p">,</span><span class="mi">000</span> <span class="n">more</span> <span class="n">words</span><span class="p">]</span>

<span class="s">FINANCIAL ANALYSIS PROCEDURES:
1. Always validate data sources
2. Calculate rolling 12-month averages
3. Compare to industry benchmarks from FinData Inc.
4. Generate variance reports
</span><span class="gp">...</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span><span class="mi">500</span> <span class="n">more</span> <span class="n">words</span><span class="p">]</span>

<span class="s">DATA SOURCES:
Our approved databases are:
- PostgreSQL cluster at db.datacorp.com
  Schema: [500 words of table definitions]
- MongoDB analytics at analytics.datacorp.com
  Collections: [400 words of collection schemas]
- S3 data lake structure: [600 words]

EXCEL FORMATTING STANDARDS:
- Header row always bold, 12pt Calibri
- Currency formatted to 2 decimals
- Charts use company colors: #1E3A8A, #3B82F6, #60A5FA
</span><span class="gp">...</span> <span class="p">[</span><span class="mi">800</span> <span class="n">more</span> <span class="n">words</span><span class="p">]</span>

<span class="s">REPORT TEMPLATES:
[Monthly report template: 500 words]
[Quarterly analysis template: 700 words]
[Executive summary template: 400 words]

Now please analyze this quarter</span><span class="sh">'</span><span class="s">s revenue data.
</span><span class="sh">"""</span>

<span class="c1"># Token cost per request: ~9,000 tokens
# Requests per month: 50,000
# Cost at Claude Sonnet 4: $27/million input tokens
# Your monthly burn: $12,150
# And that's JUST the system prompt
</span></code></pre>

</div>



<p>Every. Single. API. Call.</p>

<p>You're paying to load the same information 50,000 times a month. That's not engineering—that's setting money on fire while your CFO watches.</p>

<h2>
  
  
  The Skill Solution: Pay Per Use, Not Per Pray
</h2>

<p>Skills work on progressive disclosure. Claude loads them in three levels:</p>

<p><strong>Level 1: Metadata</strong> (~100 tokens, ALWAYS loaded)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">datacorp-financial-analysis</span>
<span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Analyze</span><span class="nv"> </span><span class="s">financial</span><span class="nv"> </span><span class="s">data</span><span class="nv"> </span><span class="s">following</span><span class="nv"> </span><span class="s">DataCorp</span><span class="nv"> </span><span class="s">standards</span><span class="nv"> </span><span class="s">and</span><span class="nv"> </span><span class="s">procedures.</span><span class="nv"> </span><span class="s">Use</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">revenue</span><span class="nv"> </span><span class="s">analysis,</span><span class="nv"> </span><span class="s">variance</span><span class="nv"> </span><span class="s">reports,</span><span class="nv"> </span><span class="s">and</span><span class="nv"> </span><span class="s">executive</span><span class="nv"> </span><span class="s">summaries."</span>
<span class="nn">---</span>
</code></pre>

</div>



<p>That's it. 100 tokens. Claude now knows this Skill exists.</p>

<p><strong>Level 2: Instructions</strong> (Under 5k tokens, loaded WHEN TRIGGERED)</p>

<p>Only loaded when someone actually asks for financial analysis:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># DataCorp Financial Analysis Skill</span>

<span class="gu">## Quick Start</span>
Use our PostgreSQL cluster for source data.
For schema details, see <span class="p">[</span><span class="nv">SCHEMAS.md</span><span class="p">](</span><span class="sx">SCHEMAS.md</span><span class="p">)</span>.
For Excel formatting, see <span class="p">[</span><span class="nv">FORMATTING.md</span><span class="p">](</span><span class="sx">FORMATTING.md</span><span class="p">)</span>.

<span class="gu">## Analysis Procedures</span>
<span class="p">1.</span> Validate data sources
<span class="p">2.</span> Calculate 12-month rolling averages
<span class="p">3.</span> Compare to benchmarks
<span class="p">4.</span> Generate variance reports

For detailed procedures, see <span class="p">[</span><span class="nv">PROCEDURES.md</span><span class="p">](</span><span class="sx">PROCEDURES.md</span><span class="p">)</span>.
</code></pre>

</div>



<p><strong>Level 3: Resources</strong> (Unlimited, loaded AS NEEDED)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>financial-analysis-skill/
├── SKILL.md              # Loaded when Skill triggers
├── SCHEMAS.md            # Only if Claude needs schema details
├── FORMATTING.md         # Only if creating Excel files
├── PROCEDURES.md         # Only if doing complex analysis
├── templates/
│   ├── monthly.md        # Only if monthly report requested
│   ├── quarterly.md      # Only if quarterly report requested
│   └── executive.md      # Only if exec summary requested
└── scripts/
    ├── validate_data.py  # Never enters context, just executes
    └── benchmark.py      # Never enters context, just executes
</code></pre>

</div>



<p>Now watch your token economics transform:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">anthropic</span>

<span class="n">client</span> <span class="o">=</span> <span class="n">anthropic</span><span class="p">.</span><span class="nc">Anthropic</span><span class="p">()</span>

<span class="c1"># One-time setup: Upload your Skill
</span><span class="kn">from</span> <span class="n">anthropic.lib</span> <span class="kn">import</span> <span class="n">files_from_dir</span>

<span class="n">skill</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">skills</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">display_title</span><span class="o">=</span><span class="sh">"</span><span class="s">DataCorp Financial Analysis</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">files</span><span class="o">=</span><span class="nf">files_from_dir</span><span class="p">(</span><span class="sh">"</span><span class="s">/path/to/financial_analysis_skill</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># Now your API calls look like this:
</span><span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">],</span>
    <span class="n">container</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span>
            <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">custom</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="n">skill</span><span class="p">.</span><span class="nb">id</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span>
        <span class="p">}]</span>
    <span class="p">},</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[{</span>
        <span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Analyze Q4 revenue vs Q3</span><span class="sh">"</span>
    <span class="p">}],</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[{</span>
        <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span>
    <span class="p">}]</span>
<span class="p">)</span>

<span class="c1"># Token cost breakdown:
# - Skill metadata: 100 tokens (always)
# - SKILL.md: 2,000 tokens (when financial analysis triggered)
# - SCHEMAS.md: 500 tokens (if needed)
# - PROCEDURES.md: 1,500 tokens (if needed)
# Total: 4,100 tokens vs 9,000 tokens
# Savings: 54% on this request
# But wait, it gets better...
</span></code></pre>

</div>



<h2>
  
  
  The Script Multiplier: Zero-Token Execution
</h2>

<p>Here's where your accountant starts crying tears of joy:</p>

<p><strong>Executable scripts never enter the context window.</strong></p>

<p>When Claude runs a script, only the OUTPUT consumes tokens. The code itself? Zero.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># scripts/validate_revenue_data.py
</span><span class="kn">import</span> <span class="n">pandas</span> <span class="k">as</span> <span class="n">pd</span>
<span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">sys</span>

<span class="k">def</span> <span class="nf">validate_revenue_data</span><span class="p">(</span><span class="n">filepath</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    Comprehensive revenue data validation.
    50 lines of sophisticated logic that you</span><span class="sh">'</span><span class="s">d normally
    need to explain to Claude in excruciating detail.
    </span><span class="sh">"""</span>
    <span class="n">df</span> <span class="o">=</span> <span class="n">pd</span><span class="p">.</span><span class="nf">read_csv</span><span class="p">(</span><span class="n">filepath</span><span class="p">)</span>

    <span class="n">issues</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="c1"># Check for negative revenue (red flag)
</span>    <span class="nf">if </span><span class="p">(</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">revenue</span><span class="sh">'</span><span class="p">]</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">).</span><span class="nf">any</span><span class="p">():</span>
        <span class="n">negative_count</span> <span class="o">=</span> <span class="p">(</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">revenue</span><span class="sh">'</span><span class="p">]</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">).</span><span class="nf">sum</span><span class="p">()</span>
        <span class="n">issues</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">CRITICAL: </span><span class="si">{</span><span class="n">negative_count</span><span class="si">}</span><span class="s"> negative revenue entries</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Validate date continuity
</span>    <span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">pd</span><span class="p">.</span><span class="nf">to_datetime</span><span class="p">(</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">])</span>
    <span class="n">date_gaps</span> <span class="o">=</span> <span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">].</span><span class="nf">diff</span><span class="p">().</span><span class="n">dt</span><span class="p">.</span><span class="n">days</span>
    <span class="nf">if </span><span class="p">(</span><span class="n">date_gaps</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">).</span><span class="nf">any</span><span class="p">():</span>
        <span class="n">issues</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">WARNING: Date gaps detected in data</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Check for outliers (3 sigma rule)
</span>    <span class="n">mean_rev</span> <span class="o">=</span> <span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">revenue</span><span class="sh">'</span><span class="p">].</span><span class="nf">mean</span><span class="p">()</span>
    <span class="n">std_rev</span> <span class="o">=</span> <span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">revenue</span><span class="sh">'</span><span class="p">].</span><span class="nf">std</span><span class="p">()</span>
    <span class="n">outliers</span> <span class="o">=</span> <span class="n">df</span><span class="p">[</span><span class="nf">abs</span><span class="p">(</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">revenue</span><span class="sh">'</span><span class="p">]</span> <span class="o">-</span> <span class="n">mean_rev</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">3</span> <span class="o">*</span> <span class="n">std_rev</span><span class="p">]</span>
    <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">outliers</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="n">issues</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">INFO: </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">outliers</span><span class="p">)</span><span class="si">}</span><span class="s"> statistical outliers detected</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Validate currency consistency
</span>    <span class="k">if</span> <span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">currency</span><span class="sh">'</span><span class="p">].</span><span class="nf">nunique</span><span class="p">()</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">:</span>
        <span class="n">issues</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sh">"</span><span class="s">ERROR: Multiple currencies in dataset</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Check completeness
</span>    <span class="n">missing_data</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">isnull</span><span class="p">().</span><span class="nf">sum</span><span class="p">()</span>
    <span class="k">if</span> <span class="n">missing_data</span><span class="p">.</span><span class="nf">any</span><span class="p">():</span>
        <span class="n">issues</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">WARNING: Missing data in columns: </span><span class="si">{</span><span class="n">missing_data</span><span class="p">[</span><span class="n">missing_data</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">].</span><span class="nf">to_dict</span><span class="p">()</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">'</span><span class="s">valid</span><span class="sh">'</span><span class="p">:</span> <span class="nf">len</span><span class="p">([</span><span class="n">i</span> <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="n">issues</span> <span class="k">if</span> <span class="sh">'</span><span class="s">ERROR</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">i</span> <span class="ow">or</span> <span class="sh">'</span><span class="s">CRITICAL</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">i</span><span class="p">])</span> <span class="o">==</span> <span class="mi">0</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">issues</span><span class="sh">'</span><span class="p">:</span> <span class="n">issues</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">:</span> <span class="nf">len</span><span class="p">(</span><span class="n">df</span><span class="p">),</span>
        <span class="sh">'</span><span class="s">date_range</span><span class="sh">'</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">].</span><span class="nf">min</span><span class="p">()</span><span class="si">}</span><span class="s"> to </span><span class="si">{</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">].</span><span class="nf">max</span><span class="p">()</span><span class="si">}</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">total_revenue</span><span class="sh">'</span><span class="p">:</span> <span class="nf">float</span><span class="p">(</span><span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">revenue</span><span class="sh">'</span><span class="p">].</span><span class="nf">sum</span><span class="p">()),</span>
        <span class="sh">'</span><span class="s">currency</span><span class="sh">'</span><span class="p">:</span> <span class="n">df</span><span class="p">[</span><span class="sh">'</span><span class="s">currency</span><span class="sh">'</span><span class="p">].</span><span class="n">iloc</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>
    <span class="p">}</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">result</span> <span class="o">=</span> <span class="nf">validate_revenue_data</span><span class="p">(</span><span class="n">sys</span><span class="p">.</span><span class="n">argv</span><span class="p">[</span><span class="mi">1</span><span class="p">])</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">result</span><span class="p">,</span> <span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">))</span>
</code></pre>

</div>



<p>Claude runs: <code>python scripts/validate_revenue_data.py q4_revenue.csv</code></p>

<p>Claude gets back:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"valid"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"issues"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"INFO: 2 statistical outliers detected"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"records"</span><span class="p">:</span><span class="w"> </span><span class="mi">1247</span><span class="p">,</span><span class="w">
  </span><span class="nl">"date_range"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2024-10-01 to 2024-12-31"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"total_revenue"</span><span class="p">:</span><span class="w"> </span><span class="mf">1847293.42</span><span class="p">,</span><span class="w">
  </span><span class="nl">"currency"</span><span class="p">:</span><span class="w"> </span><span class="s2">"USD"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Token cost of that 50-line validation script: ZERO</strong></p>

<p>If you made Claude write that logic from scratch every time? 800+ tokens. Per request. Forever.</p>

<h2>
  
  
  Real Numbers From Real Production Systems
</h2>

<p>Let me show you what this looks like at scale.</p>

<h3>
  
  
  Case Study: MarketingCo (B2B SaaS, 200 employees)
</h3>

<p><strong>Before Skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Every request included:
</span><span class="n">system_prompt</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
[Brand guidelines: 2,000 tokens]
[SEO requirements: 1,500 tokens]
[Product catalog: 3,000 tokens]
[Writing templates: 2,500 tokens]
[Competitor positioning: 1,000 tokens]
Total: 10,000 tokens per request
</span><span class="sh">"""</span>

<span class="c1"># Monthly usage:
# - 100,000 API calls
# - 10,000 tokens per call = 1 billion input tokens
# - Claude Sonnet 4: $27/million tokens
# - Monthly cost: $27,000 (just system prompts!)
</span></code></pre>

</div>



<p><strong>After Skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Marketing Skill structure:
</span><span class="n">marketing</span><span class="o">-</span><span class="n">skill</span><span class="o">/</span>
<span class="err">├──</span> <span class="n">SKILL</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="mi">2</span><span class="n">k</span> <span class="n">tokens</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">BRAND</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="n">loaded</span> <span class="k">if</span> <span class="n">brand</span> <span class="n">voice</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">SEO</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="n">loaded</span> <span class="k">if</span> <span class="n">SEO</span> <span class="n">optimization</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">products</span><span class="o">/</span>
<span class="err">│</span>   <span class="err">├──</span> <span class="n">product_a</span><span class="p">.</span><span class="nf">json </span><span class="p">(</span><span class="n">loaded</span> <span class="n">per</span> <span class="n">product</span><span class="p">)</span>
<span class="err">│</span>   <span class="err">├──</span> <span class="n">product_b</span><span class="p">.</span><span class="n">json</span>
<span class="err">│</span>   <span class="err">└──</span> <span class="bp">...</span>
<span class="err">├──</span> <span class="n">templates</span><span class="o">/</span> <span class="p">(</span><span class="n">loaded</span> <span class="n">per</span> <span class="n">content</span> <span class="nb">type</span><span class="p">)</span>
<span class="err">└──</span> <span class="n">scripts</span><span class="o">/</span>
    <span class="err">└──</span> <span class="n">check_seo</span><span class="p">.</span><span class="nf">py </span><span class="p">(</span><span class="n">zero</span> <span class="n">tokens</span><span class="p">,</span> <span class="n">just</span> <span class="n">runs</span><span class="p">)</span>

<span class="c1"># Typical blog post request:
# - Skill metadata: 100 tokens (always)
# - SKILL.md: 2,000 tokens (triggered)
# - SEO.md: 1,500 tokens (needed for blog)
# - templates/blog.md: 600 tokens (needed)
# - One product file: 200 tokens (needed)
# Total: 4,400 tokens per request
</span>
<span class="c1"># New monthly cost:
# - 100,000 API calls
# - 4,400 tokens per call = 440 million input tokens
# - Monthly cost: $11,880
# SAVINGS: $15,120/month = $181,440/year
</span></code></pre>

</div>



<h3>
  
  
  Case Study: FinTechStartup (Series A, Tight Margins)
</h3>

<p><strong>The Setup:</strong></p>

<ul>
<li>Building AI-powered financial advisory tool</li>
<li>10,000 users, averaging 5 queries/day</li>
<li>50,000 API calls/day = 1.5M/month</li>
</ul>

<p><strong>Before Skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># System prompt included:
# - Compliance guidelines: 3,000 tokens
# - Investment strategies: 4,000 tokens
# - Risk assessment procedures: 2,000 tokens
# - Market data schemas: 2,500 tokens
# - Report formatting: 1,500 tokens
# Total: 13,000 tokens per request
</span>
<span class="c1"># Monthly burn:
# 1.5M calls × 13,000 tokens = 19.5 billion tokens
# Cost: $526,500/month on input tokens alone
# Annual run rate: $6.3 MILLION
</span></code></pre>

</div>



<p><strong>After Skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Investment advisory Skill:
</span><span class="n">investment</span><span class="o">-</span><span class="n">skill</span><span class="o">/</span>
<span class="err">├──</span> <span class="n">SKILL</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="mi">3</span><span class="n">k</span> <span class="n">tokens</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">COMPLIANCE</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="n">only</span> <span class="k">if</span> <span class="n">compliance</span> <span class="n">check</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">STRATEGIES</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="n">only</span> <span class="k">if</span> <span class="n">strategy</span> <span class="n">analysis</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">RISK_MODELS</span><span class="p">.</span><span class="nf">md </span><span class="p">(</span><span class="n">only</span> <span class="k">if</span> <span class="n">risk</span> <span class="n">assessment</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">schemas</span><span class="o">/</span>
<span class="err">│</span>   <span class="err">└──</span> <span class="n">market_data</span><span class="p">.</span><span class="nf">sql </span><span class="p">(</span><span class="n">only</span> <span class="k">if</span> <span class="n">schema</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">├──</span> <span class="n">templates</span><span class="o">/</span>
<span class="err">│</span>   <span class="err">└──</span> <span class="n">reports</span><span class="o">/</span> <span class="p">(</span><span class="n">only</span> <span class="k">if</span> <span class="n">report</span> <span class="n">generation</span> <span class="n">needed</span><span class="p">)</span>
<span class="err">└──</span> <span class="n">scripts</span><span class="o">/</span>
    <span class="err">├──</span> <span class="n">risk_calculator</span><span class="p">.</span><span class="nf">py </span><span class="p">(</span><span class="n">zero</span> <span class="n">tokens</span><span class="p">)</span>
    <span class="err">├──</span> <span class="n">compliance_check</span><span class="p">.</span><span class="nf">py </span><span class="p">(</span><span class="n">zero</span> <span class="n">tokens</span><span class="p">)</span>
    <span class="err">└──</span> <span class="n">portfolio_optimizer</span><span class="p">.</span><span class="nf">py </span><span class="p">(</span><span class="n">zero</span> <span class="n">tokens</span><span class="p">)</span>

<span class="c1"># Average request now:
# - Skill metadata: 100 tokens
# - SKILL.md: 3,000 tokens
# - One additional file: 2,000 tokens (average)
# - Script execution: 0 tokens
# Total: 5,100 tokens per request
</span>
<span class="c1"># New monthly cost:
# 1.5M calls × 5,100 tokens = 7.65 billion tokens
# Cost: $206,550/month
# SAVINGS: $319,950/month = $3.8 MILLION/year
</span></code></pre>

</div>



<p>That's not a typo. $3.8 million saved per year.</p>

<h2>
  
  
  The Multi-Skill Power Move
</h2>

<p>Here's where it gets spicy. You can combine up to 8 Skills per request:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># The old way: Everything in one bloated prompt
</span><span class="n">system_prompt</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
[Financial analysis: 5,000 tokens]
[Excel formatting: 2,000 tokens]
[PowerPoint templates: 2,500 tokens]
[Company branding: 1,500 tokens]
[Data validation: 2,000 tokens]
Total: 13,000 tokens every request
</span><span class="sh">"""</span>

<span class="c1"># The Skills way: Load only what's needed
</span><span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">],</span>
    <span class="n">container</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">custom</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">skill_financial</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">},</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">xlsx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">},</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pptx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">}</span>
        <span class="p">]</span>
    <span class="p">},</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[{</span>
        <span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Analyze Q4 revenue and create an executive presentation</span><span class="sh">"</span>
    <span class="p">}],</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span><span class="p">}]</span>
<span class="p">)</span>

<span class="c1"># What actually loads:
# - All Skill metadata: 300 tokens (3 × 100)
# - Financial Skill: 3,000 tokens (triggered)
# - Excel Skill: 2,000 tokens (triggered for data analysis)
# - PowerPoint Skill: 2,500 tokens (triggered for presentation)
# Total: 7,800 tokens
</span>
<span class="c1"># But if user just asks "What was our Q4 revenue?":
# - All Skill metadata: 300 tokens
# - Financial Skill: 3,000 tokens
# - Excel Skill: NOT loaded (not needed)
# - PowerPoint Skill: NOT loaded (not needed)
# Total: 3,300 tokens
# Savings: 58% on this simpler query
</span></code></pre>

</div>



<h2>
  
  
  The Anthropic Skills: Free Optimization
</h2>

<p>Anthropic provides pre-built Skills for common tasks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Use Anthropic's optimized Skills for document generation
</span><span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">],</span>
    <span class="n">container</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">xlsx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">},</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pptx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">},</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">docx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">},</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pdf</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">}</span>
        <span class="p">]</span>
    <span class="p">},</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[{</span>
        <span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Create a quarterly report with data analysis and charts</span><span class="sh">"</span>
    <span class="p">}],</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span><span class="p">}]</span>
<span class="p">)</span>

<span class="c1"># These are maintained by Anthropic
# Zero effort, just better performance
</span></code></pre>

</div>



<h2>
  
  
  Downloading Generated Files: The Right Way
</h2>

<p>When Skills create documents, you need to download them properly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Step 1: Create the document
</span><span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">],</span>
    <span class="n">container</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">xlsx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">}]</span>
    <span class="p">},</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[{</span>
        <span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Create Q4 revenue analysis spreadsheet</span><span class="sh">"</span>
    <span class="p">}],</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span><span class="p">}]</span>
<span class="p">)</span>

<span class="c1"># Step 2: Extract file IDs from response
</span><span class="k">def</span> <span class="nf">extract_file_ids</span><span class="p">(</span><span class="n">response</span><span class="p">):</span>
    <span class="n">file_ids</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">content</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">item</span><span class="p">.</span><span class="nb">type</span> <span class="o">==</span> <span class="sh">'</span><span class="s">bash_code_execution_tool_result</span><span class="sh">'</span><span class="p">:</span>
            <span class="n">content_item</span> <span class="o">=</span> <span class="n">item</span><span class="p">.</span><span class="n">content</span>
            <span class="k">if</span> <span class="n">content_item</span><span class="p">.</span><span class="nb">type</span> <span class="o">==</span> <span class="sh">'</span><span class="s">bash_code_execution_result</span><span class="sh">'</span><span class="p">:</span>
                <span class="k">for</span> <span class="nb">file</span> <span class="ow">in</span> <span class="n">content_item</span><span class="p">.</span><span class="n">content</span><span class="p">:</span>
                    <span class="k">if</span> <span class="nf">hasattr</span><span class="p">(</span><span class="nb">file</span><span class="p">,</span> <span class="sh">'</span><span class="s">file_id</span><span class="sh">'</span><span class="p">):</span>
                        <span class="n">file_ids</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="nb">file</span><span class="p">.</span><span class="n">file_id</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">file_ids</span>

<span class="c1"># Step 3: Download using Files API
</span><span class="k">for</span> <span class="n">file_id</span> <span class="ow">in</span> <span class="nf">extract_file_ids</span><span class="p">(</span><span class="n">response</span><span class="p">):</span>
    <span class="c1"># Get metadata
</span>    <span class="n">file_metadata</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">files</span><span class="p">.</span><span class="nf">retrieve_metadata</span><span class="p">(</span>
        <span class="n">file_id</span><span class="o">=</span><span class="n">file_id</span><span class="p">,</span>
        <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">files-api-2025-04-14</span><span class="sh">"</span><span class="p">]</span>
    <span class="p">)</span>

    <span class="c1"># Download content
</span>    <span class="n">file_content</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">files</span><span class="p">.</span><span class="nf">download</span><span class="p">(</span>
        <span class="n">file_id</span><span class="o">=</span><span class="n">file_id</span><span class="p">,</span>
        <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">files-api-2025-04-14</span><span class="sh">"</span><span class="p">]</span>
    <span class="p">)</span>

    <span class="c1"># Save to disk
</span>    <span class="n">file_content</span><span class="p">.</span><span class="nf">write_to_file</span><span class="p">(</span><span class="n">file_metadata</span><span class="p">.</span><span class="n">filename</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Downloaded: </span><span class="si">{</span><span class="n">file_metadata</span><span class="p">.</span><span class="n">filename</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  Version Control: Production vs Development
</h2>

<p>Smart teams version their Skills:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Production: Pin to specific versions for stability
</span><span class="n">production_config</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span>
        <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">custom</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">skill_01AbCdEfGhIjKlMnOpQrStUv</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">1759178010641129</span><span class="sh">"</span>  <span class="c1"># Specific version timestamp
</span>    <span class="p">}]</span>
<span class="p">}</span>

<span class="c1"># Development: Use latest for rapid iteration
</span><span class="n">development_config</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span>
        <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">custom</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">skill_01AbCdEfGhIjKlMnOpQrStUv</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span>  <span class="c1"># Always get newest
</span>    <span class="p">}]</span>
<span class="p">}</span>

<span class="c1"># Update workflow:
# 1. Develop and test with "latest"
</span><span class="kn">from</span> <span class="n">anthropic.lib</span> <span class="kn">import</span> <span class="n">files_from_dir</span>

<span class="n">new_version</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">skills</span><span class="p">.</span><span class="n">versions</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">skill_id</span><span class="o">=</span><span class="sh">"</span><span class="s">skill_01AbCdEfGhIjKlMnOpQrStUv</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">files</span><span class="o">=</span><span class="nf">files_from_dir</span><span class="p">(</span><span class="sh">"</span><span class="s">/path/to/updated_skill</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># 2. Deploy to production with specific version
# 3. Monitor and rollback if needed
</span></code></pre>

</div>



<h2>
  
  
  The Prompt Caching Gotcha
</h2>

<p>Skills work WITH prompt caching, but there's a trap:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># This caches well:
</span><span class="n">response1</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span>
        <span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">prompt-caching-2024-07-31</span><span class="sh">"</span>
    <span class="p">],</span>
    <span class="n">container</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">xlsx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">}</span>
        <span class="p">]</span>
    <span class="p">},</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Analyze sales data</span><span class="sh">"</span><span class="p">}],</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span><span class="p">}]</span>
<span class="p">)</span>

<span class="c1"># This breaks the cache (different Skills list):
</span><span class="n">response2</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span>
        <span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">prompt-caching-2024-07-31</span><span class="sh">"</span>
    <span class="p">],</span>
    <span class="n">container</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">xlsx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">},</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pptx</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">}</span>  <span class="c1"># Cache miss
</span>        <span class="p">]</span>
    <span class="p">},</span>
    <span class="n">messages</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Create presentation</span><span class="sh">"</span><span class="p">}],</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span><span class="p">}]</span>
<span class="p">)</span>

<span class="c1"># Solution: Keep Skills list consistent for related workflows
# Group your API calls by Skill combinations
</span></code></pre>

</div>



<h2>
  
  
  The Migration Checklist
</h2>

<p>Ready to stop hemorrhaging money? Here's your path:</p>

<h3>
  
  
  Week 1: Audit
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Calculate your current burn
</span><span class="k">def</span> <span class="nf">audit_current_costs</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">
    1. Count your API calls per month
    2. Measure your average system prompt length
    3. Calculate: calls × prompt_tokens × $27/million
    4. Cry into your spreadsheet
    </span><span class="sh">"""</span>
    <span class="n">monthly_calls</span> <span class="o">=</span> <span class="mi">100000</span>  <span class="c1"># Your number
</span>    <span class="n">avg_system_tokens</span> <span class="o">=</span> <span class="mi">8000</span>  <span class="c1"># Measure yours
</span>
    <span class="n">monthly_input_tokens</span> <span class="o">=</span> <span class="n">monthly_calls</span> <span class="o">*</span> <span class="n">avg_system_tokens</span>
    <span class="n">monthly_cost</span> <span class="o">=</span> <span class="p">(</span><span class="n">monthly_input_tokens</span> <span class="o">/</span> <span class="mi">1_000_000</span><span class="p">)</span> <span class="o">*</span> <span class="mi">27</span>

    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Current monthly burn: $</span><span class="si">{</span><span class="n">monthly_cost</span><span class="si">:</span><span class="p">,.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Annual run rate: $</span><span class="si">{</span><span class="n">monthly_cost</span> <span class="o">*</span> <span class="mi">12</span><span class="si">:</span><span class="p">,.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">monthly_cost</span>
</code></pre>

</div>



<h3>
  
  
  Week 2: Extract Your First Skill
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Start with your most-used procedures
# Financial analysis, content generation, data validation, etc.
</span>
<span class="c1"># Create skill directory:
</span><span class="sh">"""</span><span class="s">
my-first-skill/
├── SKILL.md
└── README.md
</span><span class="sh">"""</span>

<span class="c1"># SKILL.md template:
</span><span class="sh">"""</span><span class="s">
---
name: my-company-analytics
description: Company-specific data analysis procedures. Use for financial analysis, revenue reporting, and KPI calculations.
---

# Analytics Skill

## Quick Start
For standard revenue analysis, use our PostgreSQL cluster.
Connection details in company vault.

## Common Tasks
- Revenue analysis: See [REVENUE.md](REVENUE.md)
- KPI dashboards: See [KPI.md](KPI.md)

## Data Sources
All company data sources documented in [SOURCES.md](SOURCES.md)
</span><span class="sh">"""</span>
</code></pre>

</div>



<h3>
  
  
  Week 3: Upload and Test
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">anthropic.lib</span> <span class="kn">import</span> <span class="n">files_from_dir</span>

<span class="c1"># Upload
</span><span class="n">skill</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">skills</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
    <span class="n">display_title</span><span class="o">=</span><span class="sh">"</span><span class="s">Company Analytics</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">files</span><span class="o">=</span><span class="nf">files_from_dir</span><span class="p">(</span><span class="sh">"</span><span class="s">/path/to/my-first-skill</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># Test with real queries
</span><span class="n">test_queries</span> <span class="o">=</span> <span class="p">[</span>
    <span class="sh">"</span><span class="s">Analyze Q4 revenue</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">Generate monthly KPI report</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">Compare revenue vs last quarter</span><span class="sh">"</span>
<span class="p">]</span>

<span class="k">for</span> <span class="n">query</span> <span class="ow">in</span> <span class="n">test_queries</span><span class="p">:</span>
    <span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">beta</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
        <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-sonnet-4-5-20250929</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
        <span class="n">betas</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">code-execution-2025-08-25</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skills-2025-10-02</span><span class="sh">"</span><span class="p">],</span>
        <span class="n">container</span><span class="o">=</span><span class="p">{</span>
            <span class="sh">"</span><span class="s">skills</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">custom</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">skill_id</span><span class="sh">"</span><span class="p">:</span> <span class="n">skill</span><span class="p">.</span><span class="nb">id</span><span class="p">,</span> <span class="sh">"</span><span class="s">version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">latest</span><span class="sh">"</span><span class="p">}]</span>
        <span class="p">},</span>
        <span class="n">messages</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">query</span><span class="p">}],</span>
        <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution_20250825</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">code_execution</span><span class="sh">"</span><span class="p">}]</span>
    <span class="p">)</span>

    <span class="c1"># Validate responses
</span>    <span class="c1"># Measure token usage
</span>    <span class="c1"># Calculate savings
</span></code></pre>

</div>



<h3>
  
  
  Week 4: Measure and Scale
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">measure_savings</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">
    Track token usage before/after
    Calculate cost savings
    Project annual impact
    Send victory email to CFO
    </span><span class="sh">"""</span>
    <span class="n">old_tokens_per_request</span> <span class="o">=</span> <span class="mi">8000</span>
    <span class="n">new_tokens_per_request</span> <span class="o">=</span> <span class="mi">3500</span>  <span class="c1"># Your actual number
</span>
    <span class="n">savings_per_request</span> <span class="o">=</span> <span class="n">old_tokens_per_request</span> <span class="o">-</span> <span class="n">new_tokens_per_request</span>
    <span class="n">monthly_calls</span> <span class="o">=</span> <span class="mi">100000</span>

    <span class="n">monthly_savings</span> <span class="o">=</span> <span class="p">(</span><span class="n">monthly_calls</span> <span class="o">*</span> <span class="n">savings_per_request</span> <span class="o">/</span> <span class="mi">1_000_000</span><span class="p">)</span> <span class="o">*</span> <span class="mi">27</span>
    <span class="n">annual_savings</span> <span class="o">=</span> <span class="n">monthly_savings</span> <span class="o">*</span> <span class="mi">12</span>

    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Monthly savings: $</span><span class="si">{</span><span class="n">monthly_savings</span><span class="si">:</span><span class="p">,.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Annual savings: $</span><span class="si">{</span><span class="n">annual_savings</span><span class="si">:</span><span class="p">,.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Efficiency gain: </span><span class="si">{</span><span class="p">(</span><span class="n">savings_per_request</span><span class="o">/</span><span class="n">old_tokens_per_request</span><span class="p">)</span><span class="o">*</span><span class="mi">100</span><span class="si">:</span><span class="p">.</span><span class="mi">1</span><span class="n">f</span><span class="si">}</span><span class="s">%</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  The Bottom Line
</h2>

<p>Skills are not a "nice-to-have" feature. They're a fundamental architectural shift in how you should build with LLMs.</p>

<p><strong>Stop doing this:</strong></p>

<ul>
<li>Copying 10,000-token system prompts into every API call</li>
<li>Burning $500K+ annually on repeated context</li>
<li>Explaining the same procedures to Claude 100,000 times</li>
</ul>

<p><strong>Start doing this:</strong></p>

<ul>
<li>Load instructions on-demand (54-75% token reduction)</li>
<li>Execute scripts at zero token cost</li>
<li>Compose Skills for complex workflows</li>
<li>Version your domain knowledge like code</li>
</ul>

<p>The math is brutal: a mid-size SaaS company making 1.5M API calls/month with bloated prompts burns $526K monthly. With Skills, that drops to $206K. <strong>That's $3.8M saved per year.</strong></p>

<p>Your investors didn't give you millions to spend it on redundant context loading.</p>

<p>Fix your architecture. Deploy Skills. Save money.</p>




<p><em>P.S. - If you're still copying system prompts after reading this, I can't help you. But your competitors using Skills will eat your lunch while spending 60% less on AI costs.</em></p>

