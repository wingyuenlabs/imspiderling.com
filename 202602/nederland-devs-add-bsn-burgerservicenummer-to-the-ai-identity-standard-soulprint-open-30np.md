---
Title: 🇳🇱 Nederland devs: Add BSN (Burgerservicenummer) to the AI identity standard — Soulprint open source (30 min PR)
Description: 
Author: Manuel Felipe Arias Pineda
Date: 2026-02-24T21:45:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every day, AI agents make decisions on our behalf — buying, sending emails, signing documents — and nobody verifies there's a real human behind them.</p>

<p><strong>Soulprint</strong> solves this with Zero-Knowledge Proofs: 100% on-device, open source (MIT), free to run. <a href="https://soulprint.digital" rel="noopener noreferrer">soulprint.digital</a></p>

<p><strong>🇳🇱 Nederland's BSN (Burgerservicenummer) is not in Soulprint yet. You can add it in ~30 minutes with one PR.</strong></p>




<h2>
  
  
  What Soulprint does
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx soulprint verify-me       <span class="c"># scan ID + face match — all local</span>
<span class="c"># → SPT token (score 0-100)</span>

<span class="c"># AI agent includes token in every call</span>
<span class="c"># X-Soulprint: eyJ... (score: 84)</span>

<span class="c"># API verifies in 3 lines:</span>
import <span class="o">{</span> requireSoulprint <span class="o">}</span> from <span class="s2">"soulprint-mcp"</span><span class="p">;</span>
server.tool<span class="o">(</span><span class="s2">"premium"</span>, requireSoulprint<span class="o">({</span> minScore: 80 <span class="o">})</span>, handler<span class="o">)</span><span class="p">;</span>
</code></pre>

</div>



<p>ZK proof: Circom 2.1.8 · Groth16 · 844 constraints · 564ms prove · 25ms verify.</p>




<h2>
  
  
  🇳🇱 Nederland: BSN (Burgerservicenummer)
</h2>

<p>BSN: 9 digits with mod-11 check (11-proef: weighted sum must be divisible by 11).</p>




<h2>
  
  
  The code — fill in the blanks and open a PR
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// packages/verify-local/src/document/countries/NL.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CountryVerifier</span><span class="p">,</span> <span class="nx">DocumentResult</span><span class="p">,</span> <span class="nx">NumberValidation</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../verifier.interface</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">NL</span><span class="p">:</span> <span class="nx">CountryVerifier</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">countryCode</span><span class="p">:</span>   <span class="dl">"</span><span class="s2">NL</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">countryName</span><span class="p">:</span>   <span class="dl">"</span><span class="s2">Nederland</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">documentTypes</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">bsn</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">paspoort</span><span class="dl">"</span><span class="p">],</span>

  <span class="nf">parse</span><span class="p">(</span><span class="na">ocrText</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">DocumentResult</span> <span class="p">{</span>
    <span class="c1">// BSN (Burgerservicenummer) format: 9 digits mod-11</span>
    <span class="kd">const</span> <span class="nx">doc_number</span> <span class="o">=</span> <span class="nx">ocrText</span><span class="p">.</span><span class="nf">match</span><span class="p">(</span><span class="sr">/</span><span class="se">(\d{9})</span><span class="sr">/</span><span class="p">)?.[</span><span class="mi">1</span><span class="p">]</span> <span class="o">??</span> <span class="dl">""</span><span class="p">;</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">valid</span><span class="p">:</span> <span class="o">!!</span><span class="nx">doc_number</span><span class="p">,</span> <span class="nx">doc_number</span><span class="p">,</span> <span class="na">country</span><span class="p">:</span> <span class="dl">"</span><span class="s2">NL</span><span class="dl">"</span> <span class="p">};</span>
  <span class="p">},</span>

  <span class="nf">validate</span><span class="p">(</span><span class="na">docNumber</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">NumberValidation</span> <span class="p">{</span>
    <span class="c1">// 11-proef mod-11 algorithm</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">valid</span><span class="p">:</span> <span class="nf">validateBSN</span><span class="p">(</span><span class="nx">docNumber</span><span class="p">)</span> <span class="p">};</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">NL</span><span class="p">;</span>
</code></pre>

</div>



<p>Then add one line in <code>registry.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">NL</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./countries/NL</span><span class="dl">"</span><span class="p">;</span>
<span class="c1">// add to registry map: "NL": NL,</span>
</code></pre>

</div>



<p>Open a PR → your country joins the global AI identity standard. 🌍</p>




<h2>
  
  
  Why contribute?
</h2>

<ul>
<li>
<strong>Nederland joins the AI age</strong> — local developers can verify their AI agents</li>
<li>
<strong>Permanent git credit</strong> — you're in the history forever</li>
<li>
<strong>Decentralized identity</strong> — no Big Tech as gatekeeper</li>
<li>
<strong>Fast</strong> — 30 min partial, 2-3h full with MRZ</li>
</ul>

<h2>
  
  
  Resources
</h2>

<ul>
<li>🌀 <a href="https://soulprint.digital" rel="noopener noreferrer">https://soulprint.digital</a>
</li>
<li>💻 <a href="https://github.com/manuelariasfz/soulprint" rel="noopener noreferrer">GitHub — fork here</a>
</li>
<li>📖 <a href="https://github.com/manuelariasfz/soulprint/blob/main/CONTRIBUTING.md" rel="noopener noreferrer">Contributing guide</a>
</li>
</ul>

<p><em>One PR. One country. The future of AI identity is open source.</em></p>

