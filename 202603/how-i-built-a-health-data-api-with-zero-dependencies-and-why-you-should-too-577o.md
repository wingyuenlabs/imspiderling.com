---
Title: How I Built a Health Data API With Zero Dependencies (And Why You Should Too)
Description: 
Author: Botánica Andina
Date: 2026-03-28T21:50:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every health app I've seen follows the same pattern: user enters data → data goes to server → server queries database → response comes back. But what if your health tool didn't need a server at all?</p>

<p>I built an <a href="https://botanicaandina.com/herramientas/interacciones/" rel="noopener noreferrer">herb-drug interaction checker</a> that runs entirely in the browser. 592 interactions, autocomplete search, severity ratings — all client-side. Zero API calls. Zero backend. Zero privacy concerns.</p>

<p>Here's what I learned building it, and why this architecture makes sense for health tools specifically.</p>

<h2>
  
  
  Why Client-Side for Health Data?
</h2>

<p>Health data is sensitive. When someone searches "metformin + St. John's Wort," they're telling you they take metformin (diabetes) and are considering St. John's Wort (depression). That's two diagnoses revealed in one query.</p>

<p>With a server-side API, you're collecting that data whether you want to or not. Server logs, analytics, database queries — all contain sensitive health information subject to regulations you probably don't want to deal with.</p>

<p><strong>Client-side means the query never leaves the browser.</strong> No HIPAA concerns. No GDPR health data processing. No breach notification obligations. The user's health data stays on their device, period.</p>

<h2>
  
  
  The Architecture
</h2>

<p>The entire database lives in a single JavaScript constant:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">INTERACTIONS</span> <span class="o">=</span> <span class="p">[</span>
  <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">int-001</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">substance_a</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">warfarin</span><span class="dl">"</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Warfarin</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">drug</span><span class="dl">"</span> <span class="p">},</span>
    <span class="na">substance_b</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">ginkgo</span><span class="dl">"</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Ginkgo biloba</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">herb</span><span class="dl">"</span> <span class="p">},</span>
    <span class="na">severity</span><span class="p">:</span> <span class="dl">"</span><span class="s2">high</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">effect</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Increased bleeding risk</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">mechanism</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Ginkgo inhibits platelet-activating factor (PAF)</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">evidence</span><span class="p">:</span> <span class="dl">"</span><span class="s2">clinical</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">sources</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">Bent et al. 2005</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">WHO Monographs</span><span class="dl">"</span><span class="p">]</span>
  <span class="p">},</span>
  <span class="c1">// ... 591 more</span>
<span class="p">];</span>
</code></pre>

</div>



<p>When the user types a substance name, the autocomplete searches this array with a fuzzy matcher:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">fuzzyMatch</span><span class="p">(</span><span class="nx">query</span><span class="p">,</span> <span class="nx">items</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">q</span> <span class="o">=</span> <span class="nx">query</span><span class="p">.</span><span class="nf">toLowerCase</span><span class="p">().</span><span class="nf">normalize</span><span class="p">(</span><span class="dl">"</span><span class="s2">NFD</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">[\u</span><span class="sr">0300-</span><span class="se">\u</span><span class="sr">036f</span><span class="se">]</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span> <span class="c1">// strip accents</span>

  <span class="k">return</span> <span class="nx">items</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">item</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">name</span> <span class="o">=</span> <span class="nx">item</span><span class="p">.</span><span class="nx">name</span><span class="p">.</span><span class="nf">toLowerCase</span><span class="p">().</span><span class="nf">normalize</span><span class="p">(</span><span class="dl">"</span><span class="s2">NFD</span><span class="dl">"</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">[\u</span><span class="sr">0300-</span><span class="se">\u</span><span class="sr">036f</span><span class="se">]</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>

      <span class="c1">// Exact prefix match scores highest</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">name</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="nx">q</span><span class="p">))</span> <span class="k">return</span> <span class="p">{</span> <span class="nx">item</span><span class="p">,</span> <span class="na">score</span><span class="p">:</span> <span class="mi">100</span> <span class="p">};</span>

      <span class="c1">// Contains match</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">name</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">q</span><span class="p">))</span> <span class="k">return</span> <span class="p">{</span> <span class="nx">item</span><span class="p">,</span> <span class="na">score</span><span class="p">:</span> <span class="mi">80</span> <span class="p">};</span>

      <span class="c1">// Word-start match (e.g., "john" matches "St. John's Wort")</span>
      <span class="kd">const</span> <span class="nx">words</span> <span class="o">=</span> <span class="nx">name</span><span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="sr">/</span><span class="se">[\s</span><span class="sr">.-</span><span class="se">]</span><span class="sr">+/</span><span class="p">);</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">words</span><span class="p">.</span><span class="nf">some</span><span class="p">(</span><span class="nx">w</span> <span class="o">=&gt;</span> <span class="nx">w</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="nx">q</span><span class="p">)))</span> <span class="k">return</span> <span class="p">{</span> <span class="nx">item</span><span class="p">,</span> <span class="na">score</span><span class="p">:</span> <span class="mi">60</span> <span class="p">};</span>

      <span class="k">return</span> <span class="p">{</span> <span class="nx">item</span><span class="p">,</span> <span class="na">score</span><span class="p">:</span> <span class="mi">0</span> <span class="p">};</span>
    <span class="p">})</span>
    <span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">score</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">sort</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">score</span> <span class="o">-</span> <span class="nx">a</span><span class="p">.</span><span class="nx">score</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">8</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">item</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The accent normalization is crucial for a Spanish-language tool. Users type "maca" or "máca" interchangeably — both need to match.</p>

<h2>
  
  
  Performance: 592 Interactions in &lt;1ms
</h2>

<p>You might think searching 592 records on every keystroke would be slow. It's not. JavaScript engines are ridiculously fast at iterating small arrays.</p>

<p>I benchmarked it:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Operation</th>
<th>Time</th>
</tr>
</thead>
<tbody>
<tr>
<td>Full array scan (592 items)</td>
<td>0.08ms</td>
</tr>
<tr>
<td>Fuzzy match + sort + slice</td>
<td>0.12ms</td>
</tr>
<tr>
<td>DOM update (8 suggestions)</td>
<td>0.3ms</td>
</tr>
<tr>
<td><strong>Total per keystroke</strong></td>
<td><strong>~0.5ms</strong></td>
</tr>
</tbody>
</table></div>

<p>For context, a 200ms API round-trip would be 400x slower. And that's on a good connection. On a 3G connection in rural Peru (where many of our users are), the API approach would take 800ms+.</p>

<p>The threshold where you need server-side search is roughly 50,000+ records with complex queries. For a curated medical database? Client-side wins every time.</p>

<h2>
  
  
  The Data Pipeline
</h2>

<p>The hardest part wasn't the code — it was the data. Building a reliable herb-drug interaction database from scratch required:</p>

<ol>
<li>
<strong>PubMed search</strong> — queried 250+ herb names × "drug interaction" → 1,200 papers</li>
<li>
<strong>WHO Monographs</strong> — 92 monographs on medicinal plants, each with an interactions section</li>
<li>
<strong>Natural Medicines Comprehensive Database</strong> — gold standard for evidence ratings</li>
<li>
<strong>Manual review</strong> — every interaction cross-referenced with at least 2 sources</li>
</ol>

<p>Each interaction gets an evidence level:</p>

<ul>
<li>
<strong>Clinical</strong> — randomized controlled trials or systematic reviews</li>
<li>
<strong>Pharmacological</strong> — mechanism understood, case reports exist</li>
<li>
<strong>Theoretical</strong> — based on known pharmacology, no human data</li>
<li>
<strong>Traditional</strong> — based on traditional medicine practice only</li>
</ul>

<p>This matters. Telling someone "Ginkgo + Warfarin = bleeding risk" backed by clinical trials is very different from "Cat's Claw might theoretically affect immunosuppressants."</p>

<h2>
  
  
  Lessons for Developers Building Health Tools
</h2>

<h3>
  
  
  1. Evidence levels are your liability shield
</h3>

<p>If you present health information without evidence context, you're implicitly telling users it's all equally reliable. It's not. Label everything.</p>

<h3>
  
  
  2. Disclaimers aren't just legal boilerplate
</h3>

<p>Our disclaimer explicitly states: "This tool is for educational purposes. Always consult your healthcare provider." But more importantly, we show it <em>before</em> results appear, not buried in a footer. Users should know the limitations before they act on the information.</p>

<h3>
  
  
  3. Offline-first is a feature, not a limitation
</h3>

<p>Health information is most needed in places with bad internet: rural clinics, pharmacies in developing countries, field research. A tool that works offline is genuinely more useful than one that requires connectivity.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Service worker for offline support (optional but valuable)</span>
<span class="nb">self</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">install</span><span class="dl">'</span><span class="p">,</span> <span class="nx">event</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">event</span><span class="p">.</span><span class="nf">waitUntil</span><span class="p">(</span>
    <span class="nx">caches</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="dl">'</span><span class="s1">health-tools-v1</span><span class="dl">'</span><span class="p">).</span><span class="nf">then</span><span class="p">(</span><span class="nx">cache</span> <span class="o">=&gt;</span>
      <span class="nx">cache</span><span class="p">.</span><span class="nf">addAll</span><span class="p">([</span><span class="dl">'</span><span class="s1">/interacciones/</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">/interacciones/index.html</span><span class="dl">'</span><span class="p">])</span>
    <span class="p">)</span>
  <span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  4. Embed-ready tools get organic backlinks
</h3>

<p>We added an embed feature that generates an iframe snippet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;iframe</span>
  <span class="na">src=</span><span class="s">"https://botanicaandina.com/herramientas/interacciones/?embed=true"</span>
  <span class="na">width=</span><span class="s">"100%"</span> <span class="na">height=</span><span class="s">"600"</span> <span class="na">frameborder=</span><span class="s">"0"</span>
  <span class="na">title=</span><span class="s">"Herb-Drug Interaction Checker"</span><span class="nt">&gt;</span>
<span class="nt">&lt;/iframe&gt;</span>
</code></pre>

</div>



<p>Health blogs and university sites can embed the tool directly. Each embed is a backlink. This is how open-source health tools grow organically.</p>

<h3>
  
  
  5. Internationalization from day one
</h3>

<p>Our tool supports Spanish and English. The data model stores both:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">effect</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Increased bleeding risk</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">effect_es</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Mayor riesgo de sangrado</span><span class="dl">"</span><span class="p">,</span>
<span class="p">}</span>
</code></pre>

</div>



<p>For LATAM health tools, Spanish-first with English fallback serves 95% of users. But don't auto-translate medical terms — "anticoagulante" and "blood thinner" aren't perfectly equivalent in clinical context.</p>

<h2>
  
  
  What I'd Do Differently
</h2>

<p><strong>More structured data from the start.</strong> I initially stored interactions as flat text. Switching to structured objects (substance IDs, severity enums, evidence enums) made the tool 10x more powerful but required a painful migration.</p>

<p><strong>Test with real pharmacists earlier.</strong> A pharmacist found 3 severity misclassifications in the first week. Domain experts catch things that unit tests never will.</p>

<p><strong>Schema.org markup.</strong> Adding WebApplication + FAQPage structured data increased our visibility in Google rich results. Do this from the start, not as an afterthought.</p>

<h2>
  
  
  The Stack
</h2>

<ul>
<li>
<strong>HTML/CSS/JS</strong> — single file, no build tools</li>
<li>
<strong>No frameworks</strong> — vanilla JS is fast enough for this scale</li>
<li>
<strong>No backend</strong> — all data embedded in the page</li>
<li>
<strong>Hosting</strong> — GitHub Pages (free, fast, global CDN)</li>
<li>
<strong>Total size</strong> — 127KB gzipped (entire app + 592 interactions)</li>
</ul>

<p>The entire tool is one HTML file. You can save it to your desktop and it works.</p>

<h2>
  
  
  Try It
</h2>

<p>The <a href="https://botanicaandina.com/herramientas/interacciones/" rel="noopener noreferrer">Herb-Drug Interaction Checker</a> is live and free. Search any supplement or herb name to see known interactions with medications.</p>

<p>If you're building health tools, consider whether your data really needs a backend. For curated, reference-style databases (drug interactions, dosing calculators, symptom checkers), client-side architecture gives you better performance, zero privacy concerns, and offline capability — at the cost of a larger initial page load.</p>

<p>The tradeoff is worth it.</p>




<p><em>I'm building open-source health tools for the Latin American market at <a href="https://botanicaandina.com" rel="noopener noreferrer">Botanica Andina</a>. If you're interested in health data, NLP for medical Spanish, or client-side tool architecture, let's connect.</em></p>

