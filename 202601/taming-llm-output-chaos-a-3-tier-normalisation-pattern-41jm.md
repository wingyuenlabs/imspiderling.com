---
Title: Taming LLM Output Chaos: A 3-Tier Normalisation Pattern
Description: 
Author: Armel BOBDA
Date: 2026-01-25T21:12:01.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzqhqfvrc081ifh2wv8k2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzqhqfvrc081ifh2wv8k2.png" alt="3-TIER LLM OUTPUT NORMALIZATION" width="800" height="436"></a></p>

<p>You ask the LLM for "DRAINS" relationships.</p>

<p>You get:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>drains, depletes, exhausts, causes_fatigue,
emotionally_draining, negatively_impacts,
energy_draining, leads_to_exhaustion,
saps_vitality, wears_out, causes_drain...
</code></pre>

</div>



<p>Eleven variations. Per concept. Per run.</p>

<p>If your application depends on consistent output from an LLM, you have a problem. I learned this the hard way while building <a href="https://github.com/armelhbobdad/sentinel" rel="noopener noreferrer">Sentinel</a>, a CLI tool that uses <a href="https://github.com/topoteretes/cognee" rel="noopener noreferrer">Cognee</a> to detect energy conflicts in personal schedules.</p>

<p>This article shares the pattern I developed to normalise chaotic LLM output into predictable, application-ready data.</p>




<h2>
  
  
  The Problem: LLMs Don't Follow Instructions
</h2>

<p>I was building a knowledge graph where activities could <code>DRAINS</code> energy or <code>REQUIRES</code> focus. Simple enough. I wrote a custom extraction prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>**REQUIRED RELATIONSHIP TYPES** (use ONLY these exact names):
- DRAINS: Activity depletes energy/focus/motivation
- REQUIRES: Activity needs energy/focus/resources
</code></pre>

</div>



<p>The LLM nodded along and generated... whatever it felt like.</p>

<p>My collision detection algorithm expected <code>DRAINS</code>. Cognee's LLM returned <code>is_emotionally_draining</code>. My BFS traversal found nothing. Tests passed (mocks are liars). Production was broken.</p>

<p><strong>The harsh truth:</strong> Prompting alone gets you ~70% consistency. For the remaining 30%, you need a normalisation layer.</p>




<h2>
  
  
  Why Prompting Isn't Enough
</h2>

<p>I tried harder. I added examples. I used few-shot prompting. I YELLED IN CAPS.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>**CRITICAL**: Use ONLY these relationship types:
- DRAINS (not "depletes", not "exhausts", not "causes_fatigue")
</code></pre>

</div>



<p>Result: The LLM now generated <code>DRAINS</code> 85% of the time. But also <code>drains_energy</code>, <code>energy_draining</code>, and my personal favourite: <code>negatively_impacts_emotional_state</code>.</p>

<p>The LLM understands semantics, not syntax. It knows these concepts are equivalent. It doesn't care about your string matching.</p>

<p><strong>My detection rate:</strong> 15-20% of actual collisions found.</p>




<h2>
  
  
  The Solution: 3-Tier Normalisation
</h2>

<p>The pattern I landed on processes LLM output through three increasingly fuzzy matching tiers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────┐
│                    LLM Output                           │
│              "totally_exhausting_day"                   │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  TIER 1: Exact Match (O(1))                             │
│  ───────────────────────────                            │
│  Dictionary lookup. Instant.                            │
│  "drains" → DRAINS ✓                                    │
│  "totally_exhausting_day" → miss, try next tier         │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  TIER 2: Keyword Match (O(n))                           │
│  ────────────────────────────                           │
│  Check if input contains known stems.                   │
│  "totally_exhausting_day" contains "exhaust" → DRAINS ✓ │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  TIER 3: Fuzzy Match (O(n×m))                           │
│  ─────────────────────────────                          │
│  RapidFuzz similarity scoring against candidates.       │
│  "reduces vitality" ~= "reduces energy" (82%) → DRAINS  │
└─────────────────────────────────────────────────────────┘
</code></pre>

</div>



<p><strong>Key insight:</strong> Most lookups hit Tier 1. Only novel variations fall through to expensive fuzzy matching. Performance stays fast; coverage stays complete.</p>




<h2>
  
  
  Implementation
</h2>

<h3>
  
  
  Tier 1: Exact Match Dictionary
</h3>

<p>The fast path. Cover every variation you've ever seen:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">RELATION_TYPE_MAP</span><span class="p">:</span> <span class="nb">dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">str</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="c1"># DRAINS mappings
</span>    <span class="sh">"</span><span class="s">drains</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">depletes</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">exhausts</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">tires</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">fatigues</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">causes_fatigue</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">drains_energy</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">is_emotionally_draining</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">emotionally_draining</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">causes_exhaustion</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">negatively_impacts</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">leads_to_exhaustion</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># ... 70+ more entries across all types
</span>
    <span class="c1"># REQUIRES mappings
</span>    <span class="sh">"</span><span class="s">requires</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">needs</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">demands</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">depends_on</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">requires_focus</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">needs_energy</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># ...
</span><span class="p">}</span>
</code></pre>

</div>



<p><strong>Lookup:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">map_relation_type</span><span class="p">(</span><span class="n">raw_type</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span> <span class="o">|</span> <span class="bp">None</span><span class="p">:</span>
    <span class="n">normalized</span> <span class="o">=</span> <span class="n">raw_type</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span>

    <span class="c1"># Tier 1: Exact match (O(1))
</span>    <span class="k">if</span> <span class="n">normalized</span> <span class="ow">in</span> <span class="n">RELATION_TYPE_MAP</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">RELATION_TYPE_MAP</span><span class="p">[</span><span class="n">normalized</span><span class="p">]</span>

    <span class="c1"># Fall through to Tier 2...
</span></code></pre>

</div>



<p><strong>When to grow this dictionary:</strong> Every time you see a new variation in logs, add it. This dictionary is your institutional knowledge of LLM quirks.</p>




<h3>
  
  
  Tier 2: Keyword Matching
</h3>

<p>When an exact match fails, check for semantic keywords. Use word stems to catch variations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">SEMANTIC_KEYWORDS</span><span class="p">:</span> <span class="nb">dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">]]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="sh">"</span><span class="s">drain</span><span class="sh">"</span><span class="p">,</span>     <span class="c1"># matches: draining, drained, drains
</span>        <span class="sh">"</span><span class="s">exhaust</span><span class="sh">"</span><span class="p">,</span>   <span class="c1"># matches: exhausting, exhausted, exhaustion
</span>        <span class="sh">"</span><span class="s">deplet</span><span class="sh">"</span><span class="p">,</span>    <span class="c1"># matches: depleting, depleted, depletion
</span>        <span class="sh">"</span><span class="s">fatigue</span><span class="sh">"</span><span class="p">,</span>   <span class="c1"># matches: fatiguing, fatigued
</span>        <span class="sh">"</span><span class="s">tire</span><span class="sh">"</span><span class="p">,</span>      <span class="c1"># matches: tiring, tired
</span>        <span class="sh">"</span><span class="s">sap</span><span class="sh">"</span><span class="p">,</span>       <span class="c1"># matches: sapping, sapped
</span>        <span class="sh">"</span><span class="s">stress</span><span class="sh">"</span><span class="p">,</span>    <span class="c1"># matches: stressing, stressed, stressful
</span>    <span class="p">],</span>
    <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="sh">"</span><span class="s">require</span><span class="sh">"</span><span class="p">,</span>   <span class="c1"># matches: requiring, required, requirement
</span>        <span class="sh">"</span><span class="s">need</span><span class="sh">"</span><span class="p">,</span>      <span class="c1"># matches: needing, needed, needs
</span>        <span class="sh">"</span><span class="s">demand</span><span class="sh">"</span><span class="p">,</span>    <span class="c1"># matches: demanding, demanded
</span>        <span class="sh">"</span><span class="s">depend</span><span class="sh">"</span><span class="p">,</span>    <span class="c1"># matches: depending, dependent, dependency
</span>    <span class="p">],</span>
    <span class="sh">"</span><span class="s">CONFLICTS_WITH</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="sh">"</span><span class="s">conflict</span><span class="sh">"</span><span class="p">,</span>  <span class="c1"># matches: conflicting, conflicted
</span>        <span class="sh">"</span><span class="s">clash</span><span class="sh">"</span><span class="p">,</span>     <span class="c1"># matches: clashing, clashed
</span>        <span class="sh">"</span><span class="s">interfer</span><span class="sh">"</span><span class="p">,</span>  <span class="c1"># matches: interfering, interference
</span>        <span class="sh">"</span><span class="s">hinder</span><span class="sh">"</span><span class="p">,</span>    <span class="c1"># matches: hindering, hindered
</span>    <span class="p">],</span>
<span class="p">}</span>

<span class="k">def</span> <span class="nf">_keyword_match_relation</span><span class="p">(</span><span class="n">relation_type</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span> <span class="o">|</span> <span class="bp">None</span><span class="p">:</span>
    <span class="n">normalized</span> <span class="o">=</span> <span class="n">relation_type</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span>

    <span class="k">for</span> <span class="n">canonical_type</span><span class="p">,</span> <span class="n">keywords</span> <span class="ow">in</span> <span class="n">SEMANTIC_KEYWORDS</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
        <span class="k">for</span> <span class="n">keyword</span> <span class="ow">in</span> <span class="n">keywords</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">keyword</span> <span class="ow">in</span> <span class="n">normalized</span><span class="p">:</span>
                <span class="k">return</span> <span class="n">canonical_type</span>

    <span class="k">return</span> <span class="bp">None</span>
</code></pre>

</div>



<p><strong>Why stems work:</strong> The LLM might say <code>energy_depleting</code> or <code>causes_depletion</code> or <code>depleted_state</code>. The stem <code>deplet</code> catches all three.</p>




<h3>
  
  
  Tier 3: Fuzzy Matching with RapidFuzz
</h3>

<p>For truly novel outputs, use semantic similarity scoring:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">rapidfuzz</span> <span class="kn">import</span> <span class="n">fuzz</span><span class="p">,</span> <span class="n">process</span>

<span class="n">FUZZY_CANDIDATES</span><span class="p">:</span> <span class="nb">dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">]]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="sh">"</span><span class="s">drains</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">drains energy</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">emotionally draining</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">causes exhaustion</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">leads to fatigue</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">reduces energy</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">saps energy</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">wears out</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="sh">"</span><span class="s">requires</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">needs</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">demands</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">depends on</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">prerequisite for</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">essential for</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="c1"># ...
</span><span class="p">}</span>

<span class="n">DEFAULT_FUZZY_THRESHOLD</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">50</span>  <span class="c1"># Tune based on your domain
</span>
<span class="k">def</span> <span class="nf">_fuzzy_match_relation</span><span class="p">(</span>
    <span class="n">relation_type</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">threshold</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="n">DEFAULT_FUZZY_THRESHOLD</span>
<span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span> <span class="o">|</span> <span class="bp">None</span><span class="p">:</span>
    <span class="c1"># Normalize: lowercase and spaces instead of underscores
</span>    <span class="n">normalized</span> <span class="o">=</span> <span class="n">relation_type</span><span class="p">.</span><span class="nf">lower</span><span class="p">().</span><span class="nf">replace</span><span class="p">(</span><span class="sh">"</span><span class="s">_</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s"> </span><span class="sh">"</span><span class="p">)</span>

    <span class="n">best_match</span><span class="p">:</span> <span class="nb">str</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span>
    <span class="n">best_score</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">0.0</span>

    <span class="k">for</span> <span class="n">canonical_type</span><span class="p">,</span> <span class="n">candidates</span> <span class="ow">in</span> <span class="n">FUZZY_CANDIDATES</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
        <span class="n">result</span> <span class="o">=</span> <span class="n">process</span><span class="p">.</span><span class="nf">extractOne</span><span class="p">(</span>
            <span class="n">normalized</span><span class="p">,</span>
            <span class="n">candidates</span><span class="p">,</span>
            <span class="n">scorer</span><span class="o">=</span><span class="n">fuzz</span><span class="p">.</span><span class="n">WRatio</span>
        <span class="p">)</span>
        <span class="k">if</span> <span class="n">result</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
            <span class="n">match_str</span><span class="p">,</span> <span class="n">score</span><span class="p">,</span> <span class="n">_</span> <span class="o">=</span> <span class="n">result</span>
            <span class="k">if</span> <span class="n">score</span> <span class="o">&gt;</span> <span class="n">best_score</span> <span class="ow">and</span> <span class="n">score</span> <span class="o">&gt;=</span> <span class="n">threshold</span><span class="p">:</span>
                <span class="n">best_score</span> <span class="o">=</span> <span class="n">score</span>
                <span class="n">best_match</span> <span class="o">=</span> <span class="n">canonical_type</span>

    <span class="k">return</span> <span class="n">best_match</span>
</code></pre>

</div>



<p><strong>Why RapidFuzz?</strong> It's fast (written in C++), handles partial matches well, and <code>WRatio</code> balances token order flexibility with precision.</p>




<h3>
  
  
  Putting It Together
</h3>

<p>The main mapping function chains all three tiers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">_map_cognee_relation_to_edge</span><span class="p">(</span><span class="n">relation</span><span class="p">:</span> <span class="nb">dict</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Edge</span> <span class="o">|</span> <span class="bp">None</span><span class="p">:</span>
    <span class="n">relation_type</span> <span class="o">=</span> <span class="n">relation</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">).</span><span class="nf">lower</span><span class="p">()</span>

    <span class="c1"># Tier 1: Exact match (fastest)
</span>    <span class="n">edge_type</span> <span class="o">=</span> <span class="n">RELATION_TYPE_MAP</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">relation_type</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">edge_type</span><span class="p">:</span>
        <span class="k">return</span> <span class="nc">Edge</span><span class="p">(</span>
            <span class="n">relationship</span><span class="o">=</span><span class="n">edge_type</span><span class="p">,</span>
            <span class="n">metadata</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">match_tier</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">exact</span><span class="sh">"</span><span class="p">},</span>
            <span class="c1"># ...
</span>        <span class="p">)</span>

    <span class="c1"># Tier 2: Keyword match
</span>    <span class="n">edge_type</span> <span class="o">=</span> <span class="nf">_keyword_match_relation</span><span class="p">(</span><span class="n">relation_type</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">edge_type</span><span class="p">:</span>
        <span class="k">return</span> <span class="nc">Edge</span><span class="p">(</span>
            <span class="n">relationship</span><span class="o">=</span><span class="n">edge_type</span><span class="p">,</span>
            <span class="n">metadata</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">match_tier</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">keyword</span><span class="sh">"</span><span class="p">},</span>
            <span class="c1"># ...
</span>        <span class="p">)</span>

    <span class="c1"># Tier 3: Fuzzy match (slowest, most flexible)
</span>    <span class="n">edge_type</span> <span class="o">=</span> <span class="nf">_fuzzy_match_relation</span><span class="p">(</span><span class="n">relation_type</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">edge_type</span><span class="p">:</span>
        <span class="k">return</span> <span class="nc">Edge</span><span class="p">(</span>
            <span class="n">relationship</span><span class="o">=</span><span class="n">edge_type</span><span class="p">,</span>
            <span class="n">metadata</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">match_tier</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">fuzzy</span><span class="sh">"</span><span class="p">},</span>
            <span class="c1"># ...
</span>        <span class="p">)</span>

    <span class="c1"># No match - log and filter out
</span>    <span class="n">logger</span><span class="p">.</span><span class="nf">warning</span><span class="p">(</span><span class="sh">"</span><span class="s">Unknown relation type: %s</span><span class="sh">"</span><span class="p">,</span> <span class="n">relation_type</span><span class="p">)</span>
    <span class="k">return</span> <span class="bp">None</span>
</code></pre>

</div>



<p><strong>Notice the metadata.</strong> Tracking which tier matched helps you:</p>

<ol>
<li>Debug why something matched (or didn't)</li>
<li>Know when to add new exact mappings (if Tier 3 hits the same input repeatedly)</li>
<li>Monitor your coverage over time</li>
</ol>

<blockquote>
<p><em>Code simplified for clarity. The <a href="https://github.com/armelhbobdad/sentinel/blob/main/src/sentinel/core/engine.py#L642" rel="noopener noreferrer">actual implementation</a> handles full Edge construction with source/target IDs and confidence scores.</em></p>
</blockquote>




<h2>
  
  
  Testing the Pattern
</h2>

<p>Parametrized tests cover all tiers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pytest</span>

<span class="nd">@pytest.mark.parametrize</span><span class="p">(</span><span class="sh">"</span><span class="s">input_type,expected_type,expected_tier</span><span class="sh">"</span><span class="p">,</span> <span class="p">[</span>
    <span class="c1"># Tier 1: Exact matches
</span>    <span class="p">(</span><span class="sh">"</span><span class="s">drains</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">exact</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">requires</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">exact</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">emotionally_draining</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">exact</span><span class="sh">"</span><span class="p">),</span>

    <span class="c1"># Tier 2: Keyword matches
</span>    <span class="p">(</span><span class="sh">"</span><span class="s">energy_depleting</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">keyword</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">absolutely_exhausting</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">keyword</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">strongly_dependent</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">keyword</span><span class="sh">"</span><span class="p">),</span>

    <span class="c1"># Tier 3: Fuzzy matches (no exact or keyword match)
</span>    <span class="p">(</span><span class="sh">"</span><span class="s">reduces vitality</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">fuzzy</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">critical for success</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">REQUIRES</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">fuzzy</span><span class="sh">"</span><span class="p">),</span>
<span class="p">])</span>
<span class="k">def</span> <span class="nf">test_3_tier_mapping</span><span class="p">(</span><span class="n">input_type</span><span class="p">,</span> <span class="n">expected_type</span><span class="p">,</span> <span class="n">expected_tier</span><span class="p">):</span>
    <span class="n">result</span> <span class="o">=</span> <span class="nf">_map_cognee_relation_to_edge</span><span class="p">({</span>
        <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="n">input_type</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">source_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">a</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">target_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">b</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">})</span>

    <span class="k">assert</span> <span class="n">result</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span>
    <span class="k">assert</span> <span class="n">result</span><span class="p">.</span><span class="n">relationship</span> <span class="o">==</span> <span class="n">expected_type</span>
    <span class="k">assert</span> <span class="n">result</span><span class="p">.</span><span class="n">metadata</span><span class="p">[</span><span class="sh">"</span><span class="s">match_tier</span><span class="sh">"</span><span class="p">]</span> <span class="o">==</span> <span class="n">expected_tier</span>
</code></pre>

</div>



<p><strong>Critical:</strong> Add real LLM outputs to this test as you discover them. Your test suite becomes documentation of LLM behaviour.</p>




<h2>
  
  
  Results
</h2>

<p>Before the 3-tier pattern:</p>

<ul>
<li>
<strong>Collision detection rate:</strong> 15-20%</li>
<li>
<strong>Unmatched relation types:</strong> 11+ per run</li>
</ul>

<p>After:</p>

<ul>
<li>
<strong>Collision detection rate:</strong> 100%</li>
<li>
<strong>Unmatched relation types:</strong> 0</li>
<li>
<strong>Performance impact:</strong> Negligible (most lookups hit Tier 1's O(1) dictionary)</li>
</ul>

<p>The journey from broken to working required:</p>

<ol>
<li>Custom extraction prompt (got edge types right)</li>
<li>3-tier normalisation (got relation mapping right)</li>
<li>Semantic node consolidation (got node matching right)</li>
</ol>

<p>Each layer catches what the previous layer misses.</p>

<blockquote>
<p><strong>Note:</strong> Semantic node consolidation is a separate module that merges equivalent nodes (e.g., "emotional_exhaustion" and "low_energy") using similar RapidFuzz techniques. The 3-tier pattern handles <em>relation types</em>; consolidation handles <em>node identity</em>. Both are needed for reliable graph traversal.</p>
</blockquote>




<h2>
  
  
  When to Use This Pattern
</h2>

<p><strong>Use 3-tier normalisation when:</strong></p>

<ul>
<li>Your application depends on categorical LLM output</li>
<li>You're building knowledge graphs, classification systems, or structured extraction</li>
<li>Prompt engineering alone isn't achieving consistency</li>
<li>You need to handle novel variations gracefully</li>
</ul>

<p><strong>Consider alternatives when:</strong></p>

<ul>
<li>Output is free-form text (summarization, chat)</li>
<li>You control the training/fine-tuning</li>
<li>Exact matching is acceptable (you can reject unknowns)</li>
</ul>




<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li><p><strong>Prompting is necessary but not sufficient.</strong> Even great prompts get ~70-85% consistency.</p></li>
<li><p><strong>Build the fast path first.</strong> Exact match dictionary handles most cases in O(1).</p></li>
<li><p><strong>Use word stems for keyword matching.</strong> Catches inflections without regex complexity.</p></li>
<li><p><strong>Fuzzy matching is your safety net.</strong> RapidFuzz with tunable thresholds handles novel outputs.</p></li>
<li><p><strong>Track which tier matched.</strong> Metadata enables debugging and coverage monitoring.</p></li>
<li><p><strong>Grow your dictionaries from production.</strong> Every new variation you see should become an exact match.</p></li>
</ol>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<a href="https://github.com/armelhbobdad/sentinel" rel="noopener noreferrer">Sentinel on GitHub</a> - The project where this pattern was developed</li>
<li>
<a href="https://github.com/topoteretes/cognee" rel="noopener noreferrer">Cognee</a> - Knowledge graph library for LLM applications</li>
<li>
<a href="https://github.com/rapidfuzz/RapidFuzz" rel="noopener noreferrer">RapidFuzz</a> - Fast fuzzy string matching</li>
</ul>

<h2>
  
  
  Related Articles
</h2>

<ul>
<li>
<a href="https://medium.com/@armelhbobdad/why-your-calendar-app-misses-the-real-conflicts-39f980bb1791" rel="noopener noreferrer">Why Your Calendar App Misses the Real Conflicts</a> - The "why" behind Sentinel (general audience)</li>
<li>
<a href="https://dev.to/armelhbobdad/building-a-cli-tool-with-cognee-lessons-from-5-epics-8mi">Building a CLI Tool with Cognee: Lessons from 5 Epics</a> - Cognee-specific integration lessons</li>
</ul>




<p><em>Built while participating in the <a href="https://discord.com/channels/1120795297094832337/1317073613446185074" rel="noopener noreferrer">Cognee Mini Challenge 2026 - January Edition</a>. The full implementation is in <a href="https://github.com/armelhbobdad/sentinel/blob/main/src/sentinel/core/engine.py" rel="noopener noreferrer"><code>src/sentinel/core/engine.py</code></a>.</em></p>

