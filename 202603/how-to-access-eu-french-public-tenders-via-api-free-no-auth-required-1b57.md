---
Title: How to access EU & French public tenders via API (free, no auth required)
Description: 
Author: TNDrApi
Date: 2026-03-14T22:10:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every year, 50,000+ public procurement notices are published in France and across the EU. Most developers trying to build tender monitoring tools waste days scraping fragile government websites that break without warning.</p>

<p>There's a better way. And it's completely free.</p>

<h2>
  
  
  The official sources nobody talks about
</h2>

<p>Both <strong>BOAMP</strong> (France's official public procurement bulletin) and <strong>TED</strong> (Tenders Electronic Daily - the EU's official source) have free, documented APIs that require zero authentication. No API key, no registration, no rate limit headaches.</p>

<ul>
<li>BOAMP: powered by Opendatasoft, returns clean JSON</li>
<li>TED API v3: official EU endpoint covering all 27 member states</li>
</ul>

<p>Together they represent ~50,000 new notices per year for France alone.</p>

<h2>
  
  
  The problem with using them directly
</h2>

<p>The raw APIs return inconsistent formats, buried budget fields, opaque deadline structures, and no sector labeling. You end up writing 200 lines of parsing code before you can do anything useful.</p>

<h2>
  
  
  What I built
</h2>

<p>I built <strong>Tender Intelligence API</strong>. A unified wrapper around both sources that gives you structured, clean data per tender:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Field</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>Title</td>
<td>"IT infrastructure maintenance"</td>
</tr>
<tr>
<td>Buyer</td>
<td>City of Lyon</td>
</tr>
<tr>
<td>Budget</td>
<td>120,000 €</td>
</tr>
<tr>
<td>Deadline</td>
<td>Oct 15 (44 days left)</td>
</tr>
<tr>
<td>Sector</td>
<td>IT / Software</td>
</tr>
<tr>
<td>Source</td>
<td>BOAMP / TED</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Quick example in Python
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>

<span class="n">url</span> <span class="o">=</span> <span class="sh">"</span><span class="s">https://tender-intelligence.p.rapidapi.com/search</span><span class="sh">"</span>
<span class="n">headers</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">X-RapidAPI-Key</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">YOUR_KEY</span><span class="sh">"</span><span class="p">}</span>
<span class="n">params</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">q</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">informatique</span><span class="sh">"</span><span class="p">}</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">headers</span><span class="o">=</span><span class="n">headers</span><span class="p">,</span> <span class="n">params</span><span class="o">=</span><span class="n">params</span><span class="p">)</span>
<span class="n">tenders</span> <span class="o">=</span> <span class="n">response</span><span class="p">.</span><span class="nf">json</span><span class="p">()</span>

<span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">tenders</span><span class="p">[</span><span class="sh">"</span><span class="s">results</span><span class="sh">"</span><span class="p">]:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">t</span><span class="p">[</span><span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> — </span><span class="si">{</span><span class="n">t</span><span class="p">[</span><span class="sh">'</span><span class="s">budget</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> — </span><span class="si">{</span><span class="n">t</span><span class="p">[</span><span class="sh">'</span><span class="s">days_left</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s"> days left</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  Available endpoints
</h2>

<ul>
<li>
<code>GET /search?q=...</code> — keyword search across BOAMP + TED in parallel</li>
<li>
<code>GET /sectors</code> — list of 15 labeled sectors</li>
<li>
<code>GET /sectors/{id}</code> — all tenders in a sector (IT, Health, Construction...)</li>
<li>
<code>GET /notices/{source}/{id}</code> — full detail of a single notice</li>
<li>
<code>GET /health</code> — upstream source status</li>
</ul>

<h2>
  
  
  Free tier available
</h2>

<p>A free plan with 100 requests/month is available. No credit card required. Paid plans start at $29/month for unlimited access.</p>

<p>👉 <a href="https://rapidapi.com/TNDrApi/api/tender-intelligence" rel="noopener noreferrer">Try Tender Intelligence API on RapidAPI</a></p>

<p>The GitHub repo is also public if you want to contribute or self-host.</p>

