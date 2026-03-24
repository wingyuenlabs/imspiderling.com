---
Title: I Built a Script That Finds Hidden APIs on Any Website (Here's the Code)
Description: 
Author: Alex Spinov 
Date: 2026-03-24T21:49:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most websites have hidden JSON APIs that return cleaner data than their HTML pages. I built a simple Node.js script that discovers them automatically.</p>

<h2>
  
  
  The Problem
</h2>

<p>Traditional web scraping is fragile:</p>

<ul>
<li>CSS selectors break when sites redesign</li>
<li>HTML parsing is slow and error-prone</li>
<li>Anti-bot systems block headless browsers</li>
</ul>

<p>But many sites expose <strong>internal JSON APIs</strong> that are faster, more stable, and return structured data.</p>

<h2>
  
  
  The Discovery Script
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">https</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">https</span><span class="dl">"</span><span class="p">);</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">findAPIs</span><span class="p">(</span><span class="nx">domain</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">commonPaths</span> <span class="o">=</span> <span class="p">[</span>
    <span class="dl">"</span><span class="s2">/api/v1</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/api/v2</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/api/graphql</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/_next/data</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/wp-json/wp/v2/posts</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/feed.json</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/sitemap.xml</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/.well-known/openid-configuration</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/robots.txt</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">/manifest.json</span><span class="dl">"</span>
  <span class="p">];</span>

  <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">path</span> <span class="k">of</span> <span class="nx">commonPaths</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">try</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`https://</span><span class="p">${</span><span class="nx">domain</span><span class="p">}${</span><span class="nx">path</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">contentType</span> <span class="o">=</span> <span class="nx">res</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">content-type</span><span class="dl">"</span><span class="p">)</span> <span class="o">||</span> <span class="dl">""</span><span class="p">;</span>
        <span class="nx">results</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="nx">path</span><span class="p">,</span>
          <span class="na">status</span><span class="p">:</span> <span class="nx">res</span><span class="p">.</span><span class="nx">status</span><span class="p">,</span>
          <span class="na">type</span><span class="p">:</span> <span class="nx">contentType</span><span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="dl">"</span><span class="s2">;</span><span class="dl">"</span><span class="p">)[</span><span class="mi">0</span><span class="p">],</span>
          <span class="na">size</span><span class="p">:</span> <span class="nx">res</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">content-length</span><span class="dl">"</span><span class="p">)</span> <span class="o">||</span> <span class="dl">"</span><span class="s2">unknown</span><span class="dl">"</span>
        <span class="p">});</span>
      <span class="p">}</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// Skip failed requests</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">results</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="nf">findAPIs</span><span class="p">(</span><span class="dl">"</span><span class="s2">dev.to</span><span class="dl">"</span><span class="p">).</span><span class="nf">then</span><span class="p">(</span><span class="nx">apis</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Found </span><span class="p">${</span><span class="nx">apis</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> endpoints:\n`</span><span class="p">);</span>
  <span class="nx">apis</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">api</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  </span><span class="p">${</span><span class="nx">api</span><span class="p">.</span><span class="nx">path</span><span class="p">}</span><span class="s2"> → </span><span class="p">${</span><span class="nx">api</span><span class="p">.</span><span class="nx">type</span><span class="p">}</span><span class="s2"> (</span><span class="p">${</span><span class="nx">api</span><span class="p">.</span><span class="nx">size</span><span class="p">}</span><span class="s2"> bytes)`</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Real Examples I Found
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Website</th>
<th>Hidden API</th>
<th>What It Returns</th>
</tr>
</thead>
<tbody>
<tr>
<td>dev.to</td>
<td><code>/api/articles</code></td>
<td>Full article data as JSON</td>
</tr>
<tr>
<td>npm</td>
<td><code>/registry.npmjs.org/{pkg}</code></td>
<td>Complete package metadata</td>
</tr>
<tr>
<td>PyPI</td>
<td><code>/pypi/{pkg}/json</code></td>
<td>Package info, versions, downloads</td>
</tr>
<tr>
<td>Wikipedia</td>
<td><code>/api/rest_v1/page/summary/{title}</code></td>
<td>Clean article summaries</td>
</tr>
<tr>
<td>GitHub</td>
<td><code>/api/v3/repos/{owner}/{repo}</code></td>
<td>Repository data</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Why This Matters
</h2>

<p>For one of my projects, switching from HTML scraping to the hidden JSON API:</p>

<ul>
<li>
<strong>Reduced code</strong> from 120 lines to 15 lines</li>
<li>
<strong>Increased speed</strong> by 10x (no browser needed)</li>
<li>
<strong>Eliminated breakage</strong> — JSON APIs rarely change structure</li>
</ul>

<h2>
  
  
  Pro Tips
</h2>

<ol>
<li>
<strong>Check Network tab</strong> in DevTools — filter by XHR/Fetch to see what APIs the frontend calls</li>
<li>
<strong>Look for <code>/api/</code>, <code>/_next/data/</code>, <code>/wp-json/</code></strong> — these are the most common patterns</li>
<li>
<strong>Check <code>robots.txt</code></strong> — it often reveals API paths the site doesn't want crawled (but are technically public)</li>
<li>
<strong>Try adding <code>.json</code></strong> to any URL — many frameworks return JSON when you do this</li>
</ol>

<h2>
  
  
  Want More?
</h2>

<p>I maintain a <a href="https://github.com/spinov001-art/awesome-web-scraping-2026" rel="noopener noreferrer">curated list of 60+ free APIs and scraping tools</a> that never break. Star it if you find it useful!</p>




<p><em>Have you found any interesting hidden APIs? Share in the comments!</em></p>

