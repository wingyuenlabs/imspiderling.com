---
Title: Build a Sales Lead Qualification Tool with Technology Detection
Description: 
Author: DAPDEV
Date: 2026-02-27T21:19:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you sell software, the technology your prospects use matters more than almost any other qualifying signal. A company running Shopify is a categorically different buyer than one running a custom Rails app. A team using HubSpot is further down the sales maturity curve than one using a spreadsheet.</p>

<p>This is called <strong>technographic data</strong> — knowing what technology stack a prospect uses — and it's how companies like Clearbit and ZoomInfo justify their price tags. The insight is simple: technology choices predict buying behavior, budget range, and fit for your product better than company size or industry alone.</p>

<p>This post shows you how to build a basic version of this yourself, using the <a href="https://rapidapi.com/dapdev-dapdev-default/api/technology-detection-api" rel="noopener noreferrer">Technology Detection API</a> and about 100 lines of Python.</p>




<h2>
  
  
  The Use Case: Qualifying Leads by Tech Stack
</h2>

<p>Let's make this concrete. Say you've built a Shopify plugin — maybe it handles subscription billing, or adds a custom reviews widget, or integrates with a specific 3PL. Your ideal customer is any store running Shopify.</p>

<p>The problem is that most lead lists don't come pre-tagged with "runs Shopify." You get a CSV of domains from a scrape, a trade show contact list, a LinkedIn export — and you have to figure out which ones are actually Shopify stores.</p>

<p>Manually checking 500 URLs is an afternoon of tedious work. Writing your own detector means maintaining fingerprint patterns as Shopify updates its platform. An API call per URL solves the problem cleanly.</p>

<p>Here's the basic pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">techdetect</span> <span class="kn">import</span> <span class="n">TechDetectClient</span>

<span class="n">client</span> <span class="o">=</span> <span class="nc">TechDetectClient</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="sh">"</span><span class="s">your_rapidapi_key</span><span class="sh">"</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">is_shopify</span><span class="p">(</span><span class="n">url</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">bool</span><span class="p">:</span>
    <span class="n">result</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="nf">detect</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">any</span><span class="p">(</span>
        <span class="n">t</span><span class="p">.</span><span class="n">name</span> <span class="o">==</span> <span class="sh">"</span><span class="s">Shopify</span><span class="sh">"</span> <span class="ow">and</span> <span class="n">t</span><span class="p">.</span><span class="n">confidence</span> <span class="o">&gt;=</span> <span class="mi">80</span>
        <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">result</span><span class="p">.</span><span class="n">technologies</span>
    <span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="nf">is_shopify</span><span class="p">(</span><span class="sh">"</span><span class="s">https://allbirds.com</span><span class="sh">"</span><span class="p">))</span>   <span class="c1"># True
</span><span class="nf">print</span><span class="p">(</span><span class="nf">is_shopify</span><span class="p">(</span><span class="sh">"</span><span class="s">https://techcrunch.com</span><span class="sh">"</span><span class="p">))</span> <span class="c1"># False
</span></code></pre>

</div>



<p>That's it. Now let's scale it up.</p>




<h2>
  
  
  Bulk Scanning a Lead List
</h2>

<p>Assume you have a CSV of prospect domains — call it <code>leads.csv</code> — with a <code>domain</code> column. Here's a script that scans all of them, tags each with their detected tech stack, and writes results to a new CSV.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">csv</span>
<span class="kn">import</span> <span class="n">time</span>
<span class="kn">from</span> <span class="n">techdetect</span> <span class="kn">import</span> <span class="n">TechDetectClient</span>

<span class="n">client</span> <span class="o">=</span> <span class="nc">TechDetectClient</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="sh">"</span><span class="s">your_rapidapi_key</span><span class="sh">"</span><span class="p">)</span>

<span class="n">TARGET_TECH</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Shopify</span><span class="sh">"</span>
<span class="n">CONFIDENCE_THRESHOLD</span> <span class="o">=</span> <span class="mi">80</span>

<span class="k">def</span> <span class="nf">detect_with_retry</span><span class="p">(</span><span class="n">url</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">retries</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">2</span><span class="p">):</span>
    <span class="k">for</span> <span class="n">attempt</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">retries</span> <span class="o">+</span> <span class="mi">1</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">client</span><span class="p">.</span><span class="nf">detect</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">attempt</span> <span class="o">&lt;</span> <span class="n">retries</span><span class="p">:</span>
                <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">2</span> <span class="o">**</span> <span class="n">attempt</span><span class="p">)</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="k">raise</span> <span class="n">e</span>

<span class="n">input_file</span> <span class="o">=</span> <span class="sh">"</span><span class="s">leads.csv</span><span class="sh">"</span>
<span class="n">output_file</span> <span class="o">=</span> <span class="sh">"</span><span class="s">leads_tagged.csv</span><span class="sh">"</span>

<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">input_file</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">""</span><span class="p">)</span> <span class="k">as</span> <span class="n">infile</span><span class="p">,</span> <span class="nf">open</span><span class="p">(</span><span class="n">output_file</span><span class="p">,</span> <span class="sh">"</span><span class="s">w</span><span class="sh">"</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">""</span><span class="p">)</span> <span class="k">as</span> <span class="n">outfile</span><span class="p">:</span>
    <span class="n">reader</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nc">DictReader</span><span class="p">(</span><span class="n">infile</span><span class="p">)</span>
    <span class="n">fieldnames</span> <span class="o">=</span> <span class="n">reader</span><span class="p">.</span><span class="n">fieldnames</span> <span class="o">+</span> <span class="p">[</span><span class="sh">"</span><span class="s">is_target</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">detected_technologies</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">]</span>
    <span class="n">writer</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nc">DictWriter</span><span class="p">(</span><span class="n">outfile</span><span class="p">,</span> <span class="n">fieldnames</span><span class="o">=</span><span class="n">fieldnames</span><span class="p">)</span>
    <span class="n">writer</span><span class="p">.</span><span class="nf">writeheader</span><span class="p">()</span>

    <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">reader</span><span class="p">:</span>
        <span class="n">domain</span> <span class="o">=</span> <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">domain</span><span class="sh">"</span><span class="p">].</span><span class="nf">strip</span><span class="p">()</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">domain</span> <span class="k">if</span> <span class="n">domain</span><span class="p">.</span><span class="nf">startswith</span><span class="p">(</span><span class="sh">"</span><span class="s">http</span><span class="sh">"</span><span class="p">)</span> <span class="k">else</span> <span class="sa">f</span><span class="sh">"</span><span class="s">https://</span><span class="si">{</span><span class="n">domain</span><span class="si">}</span><span class="sh">"</span>

        <span class="k">try</span><span class="p">:</span>
            <span class="n">result</span> <span class="o">=</span> <span class="nf">detect_with_retry</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
            <span class="n">target_tech</span> <span class="o">=</span> <span class="nf">next</span><span class="p">(</span>
                <span class="p">(</span><span class="n">t</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">result</span><span class="p">.</span><span class="n">technologies</span> <span class="k">if</span> <span class="n">t</span><span class="p">.</span><span class="n">name</span> <span class="o">==</span> <span class="n">TARGET_TECH</span><span class="p">),</span>
                <span class="bp">None</span>
            <span class="p">)</span>
            <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">is_target</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="sh">"</span><span class="s">yes</span><span class="sh">"</span> <span class="nf">if </span><span class="p">(</span><span class="n">target_tech</span> <span class="ow">and</span> <span class="n">target_tech</span><span class="p">.</span><span class="n">confidence</span> <span class="o">&gt;=</span> <span class="n">CONFIDENCE_THRESHOLD</span><span class="p">)</span> <span class="k">else</span> <span class="sh">"</span><span class="s">no</span><span class="sh">"</span>
            <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">detected_technologies</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="sh">"</span><span class="s">, </span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">t</span><span class="p">.</span><span class="n">name</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">result</span><span class="p">.</span><span class="n">technologies</span><span class="p">)</span>
            <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">target_tech</span><span class="p">.</span><span class="n">confidence</span> <span class="k">if</span> <span class="n">target_tech</span> <span class="k">else</span> <span class="mi">0</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">is_target</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="sh">"</span><span class="s">error</span><span class="sh">"</span>
            <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">detected_technologies</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span>
            <span class="n">row</span><span class="p">[</span><span class="sh">"</span><span class="s">confidence</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span>

        <span class="n">writer</span><span class="p">.</span><span class="nf">writerow</span><span class="p">(</span><span class="n">row</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">domain</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="n">row</span><span class="p">[</span><span class="sh">'</span><span class="s">is_target</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>Run this against a 500-row lead list overnight, and you wake up to a pre-qualified CSV you can import directly into your CRM.</p>




<h2>
  
  
  Building a Lead Scoring System
</h2>

<p>Binary filtering (Shopify / not Shopify) is just the start. The more interesting approach is <strong>lead scoring</strong> — ranking leads by how well their tech stack matches your ideal customer profile.</p>

<p>The intuition: a Shopify store using Stripe, Klaviyo, and Google Analytics is more sophisticated — and likely higher-revenue — than one using only basic Shopify with no third-party tooling.</p>

<p>Here's a simple scoring model:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">techdetect</span> <span class="kn">import</span> <span class="n">TechDetectClient</span>

<span class="n">client</span> <span class="o">=</span> <span class="nc">TechDetectClient</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="sh">"</span><span class="s">your_rapidapi_key</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Define scoring rules: (technology_name, points, reason)
</span><span class="n">SCORE_RULES</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Shopify</span><span class="sh">"</span><span class="p">,</span>          <span class="mi">30</span><span class="p">,</span> <span class="sh">"</span><span class="s">core platform match</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Shopify Plus</span><span class="sh">"</span><span class="p">,</span>     <span class="mi">25</span><span class="p">,</span> <span class="sh">"</span><span class="s">high-revenue indicator</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Stripe</span><span class="sh">"</span><span class="p">,</span>           <span class="mi">10</span><span class="p">,</span> <span class="sh">"</span><span class="s">payment sophistication</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Klaviyo</span><span class="sh">"</span><span class="p">,</span>          <span class="mi">10</span><span class="p">,</span> <span class="sh">"</span><span class="s">email marketing investment</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Google Analytics</span><span class="sh">"</span><span class="p">,</span> <span class="mi">5</span><span class="p">,</span>  <span class="sh">"</span><span class="s">tracking maturity</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Hotjar</span><span class="sh">"</span><span class="p">,</span>           <span class="mi">8</span><span class="p">,</span>  <span class="sh">"</span><span class="s">CRO investment</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Yotpo</span><span class="sh">"</span><span class="p">,</span>            <span class="mi">8</span><span class="p">,</span>  <span class="sh">"</span><span class="s">reviews platform — growing brand</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Recharge</span><span class="sh">"</span><span class="p">,</span>         <span class="mi">15</span><span class="p">,</span> <span class="sh">"</span><span class="s">subscription billing — recurring revenue</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Gorgias</span><span class="sh">"</span><span class="p">,</span>          <span class="mi">10</span><span class="p">,</span> <span class="sh">"</span><span class="s">customer support investment</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">(</span><span class="sh">"</span><span class="s">Loop Returns</span><span class="sh">"</span><span class="p">,</span>     <span class="mi">10</span><span class="p">,</span> <span class="sh">"</span><span class="s">returns management — scale indicator</span><span class="sh">"</span><span class="p">),</span>
<span class="p">]</span>

<span class="k">def</span> <span class="nf">score_lead</span><span class="p">(</span><span class="n">url</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
    <span class="n">result</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="nf">detect</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
    <span class="n">tech_names</span> <span class="o">=</span> <span class="p">{</span><span class="n">t</span><span class="p">.</span><span class="n">name</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">result</span><span class="p">.</span><span class="n">technologies</span><span class="p">}</span>

    <span class="n">score</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="n">matched_signals</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">for</span> <span class="n">tech_name</span><span class="p">,</span> <span class="n">points</span><span class="p">,</span> <span class="n">reason</span> <span class="ow">in</span> <span class="n">SCORE_RULES</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">tech_name</span> <span class="ow">in</span> <span class="n">tech_names</span><span class="p">:</span>
            <span class="n">score</span> <span class="o">+=</span> <span class="n">points</span>
            <span class="n">matched_signals</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">+</span><span class="si">{</span><span class="n">points</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">tech_name</span><span class="si">}</span><span class="s"> (</span><span class="si">{</span><span class="n">reason</span><span class="si">}</span><span class="s">)</span><span class="sh">"</span><span class="p">)</span>

    <span class="n">tier</span> <span class="o">=</span> <span class="p">(</span>
        <span class="sh">"</span><span class="s">hot</span><span class="sh">"</span>  <span class="k">if</span> <span class="n">score</span> <span class="o">&gt;=</span> <span class="mi">60</span> <span class="k">else</span>
        <span class="sh">"</span><span class="s">warm</span><span class="sh">"</span> <span class="k">if</span> <span class="n">score</span> <span class="o">&gt;=</span> <span class="mi">35</span> <span class="k">else</span>
        <span class="sh">"</span><span class="s">cold</span><span class="sh">"</span> <span class="k">if</span> <span class="n">score</span> <span class="o">&gt;=</span> <span class="mi">15</span> <span class="k">else</span>
        <span class="sh">"</span><span class="s">disqualified</span><span class="sh">"</span>
    <span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">url</span><span class="sh">"</span><span class="p">:</span> <span class="n">url</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">:</span> <span class="n">score</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">tier</span><span class="sh">"</span><span class="p">:</span> <span class="n">tier</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">signals</span><span class="sh">"</span><span class="p">:</span> <span class="n">matched_signals</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">all_technologies</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="n">t</span><span class="p">.</span><span class="n">name</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">result</span><span class="p">.</span><span class="n">technologies</span><span class="p">],</span>
    <span class="p">}</span>

<span class="c1"># Example
</span><span class="n">result</span> <span class="o">=</span> <span class="nf">score_lead</span><span class="p">(</span><span class="sh">"</span><span class="s">https://somestore.com</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Score: </span><span class="si">{</span><span class="n">result</span><span class="p">[</span><span class="sh">'</span><span class="s">score</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> — </span><span class="si">{</span><span class="n">result</span><span class="p">[</span><span class="sh">'</span><span class="s">tier</span><span class="sh">'</span><span class="p">].</span><span class="nf">upper</span><span class="p">()</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="k">for</span> <span class="n">signal</span> <span class="ow">in</span> <span class="n">result</span><span class="p">[</span><span class="sh">"</span><span class="s">signals</span><span class="sh">"</span><span class="p">]:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  </span><span class="si">{</span><span class="n">signal</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>A store scoring 60+ (Shopify Plus + Recharge + Klaviyo + Gorgias) is a high-intent, high-budget lead. One scoring 15 (bare Shopify, nothing else) might be too early-stage to convert.</p>




<h2>
  
  
  The Cost Math
</h2>

<p>This kind of enrichment used to require either a Clearbit subscription ($X,XXX/month) or a ZoomInfo seat (similar ballpark). For a developer at a small SaaS company, those price points are hard to justify before you've validated the channel.</p>

<p>Here's what this approach costs with the Technology Detection API:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Volume</th>
<th>Plan</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Up to 100 URLs/month</td>
<td>Free tier</td>
<td>$0</td>
</tr>
<tr>
<td>Up to 2,000 URLs/month</td>
<td>Pro</td>
<td>$9/month</td>
</tr>
<tr>
<td>Up to 10,000 URLs/month</td>
<td>Ultra</td>
<td>~$29/month</td>
</tr>
</tbody>
</table></div>

<p>Scanning 2,000 leads per month for $9 is a reasonable budget for almost any growth experiment. If you close even one deal from a batch of enriched leads, the ROI is obvious.</p>




<h2>
  
  
  Next Steps
</h2>

<p>The script above is a starting point. A few directions worth extending:</p>

<ul>
<li>
<strong>Add CRM integration</strong> — POST scored leads directly to a HubSpot or Pipedrive custom property instead of writing a CSV</li>
<li>
<strong>Schedule it as a cron job</strong> — Run the scan weekly against new leads as they enter your pipeline</li>
<li>
<strong>Expand the scoring model</strong> — Add negative signals (e.g., "uses WooCommerce" scores negative if you only support Shopify)</li>
<li>
<strong>Filter by geography</strong> — Combine with a WHOIS or IP geolocation lookup to target specific markets</li>
</ul>

<p>The Python client and full source: <a href="https://github.com/dapdevsoftware/techdetect-python" rel="noopener noreferrer">github.com/dapdevsoftware/techdetect-python</a></p>

<p>Get an API key (free, no credit card required): <a href="https://rapidapi.com/dapdev-dapdev-default/api/technology-detection-api" rel="noopener noreferrer">Technology Detection API on RapidAPI</a></p>

<p>If you build something with this — especially if you adapt the scoring model for a specific niche — I'd be curious to hear what signals turned out to be most predictive.</p>

