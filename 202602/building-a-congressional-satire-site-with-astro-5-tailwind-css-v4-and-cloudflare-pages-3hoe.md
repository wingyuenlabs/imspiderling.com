---
Title: Building a Congressional Satire Site with Astro 5, Tailwind CSS v4, and Cloudflare Pages
Description: 
Author: Absurdity Index
Date: 2026-02-11T21:44:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>Did you know Congress once spent federal research money studying the effects of methamphetamine on zebrafish? Or that pizza was officially classified as a vegetable for school lunch purposes? These aren't jokes — they're real legislation.</p>

<p>I built <a href="https://absurdityindex.org" rel="noopener noreferrer">Absurdity Index</a> to score real federal bills on a 1-10 absurdity scale and pair them with satirical "Not Bills" — fictional legislation so reasonable that no actual Congress would ever pass it. Here's how the tech stack came together.</p>

<h2>
  
  
  The Stack
</h2>

<ul>
<li>
<strong>Astro 5</strong> — Static site generation with content collections</li>
<li>
<strong>Tailwind CSS v4</strong> — Styling with a custom government-parody theme</li>
<li>
<strong>MDX</strong> — Content authoring (60+ bills with rich frontmatter)</li>
<li>
<strong>Zod</strong> — Schema validation for bill data at build time</li>
<li>
<strong>Cloudflare Pages</strong> — Hosting with near-instant global delivery</li>
<li>
<strong>Cloudflare Workers</strong> — Dynamic JSON API endpoints</li>
<li>
<strong>Pagefind</strong> — Client-side full-text search (zero server cost)</li>
</ul>

<h2>
  
  
  Content Collections: The Heart of the Site
</h2>

<p>Every bill — real or satirical — lives as an MDX file in <code>src/data/bills/</code>. Astro 5's content collections with the <code>glob</code> loader made this incredibly clean:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/content.config.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">defineCollection</span><span class="p">,</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">astro:content</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">glob</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">astro/loaders</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">bills</span> <span class="o">=</span> <span class="nf">defineCollection</span><span class="p">({</span>
  <span class="na">loader</span><span class="p">:</span> <span class="nf">glob</span><span class="p">({</span> <span class="na">pattern</span><span class="p">:</span> <span class="dl">'</span><span class="s1">**/*.mdx</span><span class="dl">'</span><span class="p">,</span> <span class="na">base</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./src/data/bills</span><span class="dl">'</span> <span class="p">}),</span>
  <span class="na">schema</span><span class="p">:</span> <span class="nx">baseBillSchema</span><span class="p">.</span><span class="nf">extend</span><span class="p">({</span>
    <span class="c1">// Real bill fields</span>
    <span class="na">sponsorParty</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">optional</span><span class="p">(),</span>
    <span class="na">sponsorState</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">optional</span><span class="p">(),</span>
    <span class="na">absurdityIndex</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">max</span><span class="p">(</span><span class="mi">10</span><span class="p">).</span><span class="nf">optional</span><span class="p">(),</span>
    <span class="na">congressDotGovUrl</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">url</span><span class="p">().</span><span class="nf">optional</span><span class="p">(),</span>
    <span class="c1">// ... more fields</span>
  <span class="p">})</span>
  <span class="p">.</span><span class="nf">refine</span><span class="p">((</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">billType</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">real</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return </span><span class="p">(</span>
        <span class="nx">data</span><span class="p">.</span><span class="nx">sponsorParty</span> <span class="o">!==</span> <span class="kc">undefined</span> <span class="o">&amp;&amp;</span>
        <span class="nx">data</span><span class="p">.</span><span class="nx">sponsorState</span> <span class="o">!==</span> <span class="kc">undefined</span> <span class="o">&amp;&amp;</span>
        <span class="nx">data</span><span class="p">.</span><span class="nx">congressNumber</span> <span class="o">!==</span> <span class="kc">undefined</span>
      <span class="p">);</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
  <span class="p">},</span> <span class="p">{</span>
    <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Real bills require sponsorParty, sponsorState, and congressNumber</span><span class="dl">'</span>
  <span class="p">}),</span>
<span class="p">});</span>
</code></pre>

</div>



<p>The <code>.refine()</code> at the end is key — it enforces that real bills have congressional metadata while satirical bills don't need it. If someone adds a real bill without a sponsor party, the build fails with a clear error. This catches mistakes before they reach production.</p>

<h3>
  
  
  What a Bill Looks Like
</h3>

<p>Here's a simplified version of a real bill's frontmatter:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="na">title</span><span class="pi">:</span> <span class="s2">"</span><span class="s">FairTax</span><span class="nv"> </span><span class="s">Act</span><span class="nv"> </span><span class="s">of</span><span class="nv"> </span><span class="s">2025"</span>
<span class="na">subtitle</span><span class="pi">:</span> <span class="s2">"</span><span class="s">14</span><span class="nv"> </span><span class="s">Congresses,</span><span class="nv"> </span><span class="s">Zero</span><span class="nv"> </span><span class="s">Floor</span><span class="nv"> </span><span class="s">Votes,</span><span class="nv"> </span><span class="s">One</span><span class="nv"> </span><span class="s">Dream"</span>
<span class="na">billNumber</span><span class="pi">:</span> <span class="s2">"</span><span class="s">H.R.</span><span class="nv"> </span><span class="s">25"</span>
<span class="na">billType</span><span class="pi">:</span> <span class="s2">"</span><span class="s">real"</span>
<span class="na">category</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Budget"</span>
<span class="na">tags</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">taxes"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">irs"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">fair</span><span class="nv"> </span><span class="s">tax"</span><span class="pi">]</span>

<span class="na">sponsor</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Rep.</span><span class="nv"> </span><span class="s">Buddy</span><span class="nv"> </span><span class="s">Carter"</span>
<span class="na">sponsorParty</span><span class="pi">:</span> <span class="s2">"</span><span class="s">R"</span>
<span class="na">sponsorState</span><span class="pi">:</span> <span class="s2">"</span><span class="s">GA"</span>

<span class="na">committee</span><span class="pi">:</span> <span class="s2">"</span><span class="s">House</span><span class="nv"> </span><span class="s">Committee</span><span class="nv"> </span><span class="s">on</span><span class="nv"> </span><span class="s">Ways</span><span class="nv"> </span><span class="s">and</span><span class="nv"> </span><span class="s">Means"</span>
<span class="na">status</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Referred</span><span class="nv"> </span><span class="s">to</span><span class="nv"> </span><span class="s">Committee"</span>
<span class="na">dateIntroduced</span><span class="pi">:</span> <span class="s">2025-01-03</span>

<span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Abolishes</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">IRS</span><span class="nv"> </span><span class="s">entirely</span><span class="nv"> </span><span class="s">and</span><span class="nv"> </span><span class="s">replaces</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">entire</span>
<span class="s">federal</span><span class="nv"> </span><span class="s">tax</span><span class="nv"> </span><span class="s">system</span><span class="nv"> </span><span class="s">with</span><span class="nv"> </span><span class="s">a</span><span class="nv"> </span><span class="s">single</span><span class="nv"> </span><span class="s">23%</span><span class="nv"> </span><span class="s">national</span><span class="nv"> </span><span class="s">sales</span><span class="nv"> </span><span class="s">tax."</span>

<span class="na">absurdityIndex</span><span class="pi">:</span> <span class="m">8</span>
<span class="na">congressNumber</span><span class="pi">:</span> <span class="m">119</span>
<span class="na">congressDotGovUrl</span><span class="pi">:</span> <span class="s2">"</span><span class="s">https://www.congress.gov/bill/119th-congress/house-bill/25"</span>
<span class="nn">---</span>

<span class="c1">## The Gist</span>

<span class="s">The FairTax has been introduced in **every single Congress** since the</span>
<span class="s">106th (1999). That's 14 consecutive sessions...</span>
</code></pre>

</div>



<p>The MDX body below the frontmatter contains the editorial commentary — rendered as rich HTML with links, emphasis, and embedded components. Astro 5's <code>render()</code> function handles this beautifully:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">render</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">astro:content</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="p">{</span> <span class="nx">Content</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">render</span><span class="p">(</span><span class="nx">entry</span><span class="p">);</span>
</code></pre>

</div>



<p>One gotcha I hit early: in Astro 5, you use <code>render(entry)</code> imported from <code>astro:content</code>, not <code>entry.render()</code>. The old method from Astro 4 doesn't work anymore.</p>

<h2>
  
  
  Bill Evolution Tracking
</h2>

<p>One of the more complex schemas tracks how bills change as they move through Congress. Each bill can have evolution stages:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">billEvolutionStageSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
  <span class="na">stage</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">enum</span><span class="p">([</span>
    <span class="dl">'</span><span class="s1">introduced</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">origin-committee</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">origin-passed</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">receiving-committee</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">receiving-amended</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">conference-requested</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">signed</span><span class="dl">'</span><span class="p">,</span>
    <span class="dl">'</span><span class="s1">became-law</span><span class="dl">'</span><span class="p">,</span>
    <span class="c1">// ... 25+ stages total</span>
  <span class="p">]),</span>
  <span class="na">date</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nx">coerce</span><span class="p">.</span><span class="nf">date</span><span class="p">(),</span>
  <span class="na">paraphrasedText</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span>
  <span class="na">cumulativePork</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
  <span class="na">porkAddedThisStage</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
  <span class="na">keyChanges</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">array</span><span class="p">(</span><span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()),</span>
  <span class="na">vote</span><span class="p">:</span> <span class="nx">stageVoteSchema</span><span class="p">.</span><span class="nf">optional</span><span class="p">(),</span>
<span class="p">});</span>
</code></pre>

</div>



<p>The stage names are chamber-agnostic — "origin" means whichever chamber introduced the bill, "receiving" means the other one. This avoids duplicating stages for House-originated vs. Senate-originated bills.</p>

<h2>
  
  
  Tailwind CSS v4: The Theme System
</h2>

<p>Tailwind v4 changed how you define custom themes. Instead of a <code>tailwind.config.js</code>, everything lives in your CSS file with the <code>@theme</code> directive:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="k">@import</span> <span class="s2">'tailwindcss'</span><span class="p">;</span>

<span class="k">@theme</span> <span class="p">{</span>
  <span class="c">/* Government Parody Colors */</span>
  <span class="nt">--color-navy-900</span><span class="o">:</span> <span class="err">#0</span><span class="nt">a1628</span><span class="o">;</span>
  <span class="nt">--color-navy-800</span><span class="o">:</span> <span class="err">#121</span><span class="nt">f36</span><span class="o">;</span>
  <span class="nt">--color-gold-500</span><span class="o">:</span> <span class="nf">#c5a572</span><span class="o">;</span>
  <span class="nt">--color-gold-300</span><span class="o">:</span> <span class="nf">#e8d5b0</span><span class="o">;</span>
  <span class="nt">--color-cream-100</span><span class="o">:</span> <span class="nf">#faf7f0</span><span class="o">;</span>
  <span class="nt">--color-parchment</span><span class="o">:</span> <span class="nf">#f5f0e1</span><span class="o">;</span>

  <span class="c">/* Typography */</span>
  <span class="nt">--font-serif</span><span class="o">:</span> <span class="s2">'Libre Caslon Text'</span><span class="o">,</span> <span class="s2">'Georgia'</span><span class="o">,</span> <span class="nt">serif</span><span class="o">;</span>
  <span class="nt">--font-sans</span><span class="o">:</span> <span class="s2">'Inter'</span><span class="o">,</span> <span class="s2">'system-ui'</span><span class="o">,</span> <span class="nt">sans-serif</span><span class="o">;</span>
  <span class="nt">--font-mono</span><span class="o">:</span> <span class="s2">'JetBrains Mono'</span><span class="o">,</span> <span class="s2">'Fira Code'</span><span class="o">,</span> <span class="nt">monospace</span><span class="o">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Then you use the custom colors throughout your templates with the standard Tailwind utility classes — <code>bg-parchment</code>, <code>text-navy-900</code>, <code>border-gold-300</code>, etc. The government-parody aesthetic (navy, gold, cream, parchment) gives the site a feel of official gravitas that contrasts with the satirical content.</p>

<p>With Astro 5, you wire Tailwind v4 through the Vite plugin:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// astro.config.mjs</span>
<span class="k">import</span> <span class="nx">tailwindcss</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@tailwindcss/vite</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">vite</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">plugins</span><span class="p">:</span> <span class="p">[</span><span class="nf">tailwindcss</span><span class="p">()],</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p>No <code>@astrojs/tailwind</code> integration needed — the Vite plugin is the v4 way.</p>

<h2>
  
  
  Cloudflare Pages + Workers: Free Hosting with an API
</h2>

<p>The site is statically generated and deployed to Cloudflare Pages. But I also needed a JSON API for the embed widget and programmatic access. Cloudflare Pages Functions (backed by Workers) handle this perfectly.</p>

<p>API endpoints live in <code>functions/api/</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>functions/
  _middleware.js      # CORS headers, security
  api/
    today.json.js     # Dynamic "today" endpoint
    today-generate-satire.json.js
</code></pre>

</div>



<p>The static JSON APIs (<code>/api/bills.json</code>, <code>/api/stats.json</code>, etc.) are generated at build time by Astro and served as static files. The Workers-based endpoints handle anything that needs server-side logic.</p>

<p>The deployment pipeline runs on self-hosted Kubernetes using Argo Workflows, polling the Git repository every 60 seconds. On every push to <code>main</code>, the pipeline runs validation, builds the site, indexes search, and deploys to Cloudflare Pages.</p>

<h2>
  
  
  Pagefind: Zero-Cost Client-Side Search
</h2>

<p>One of my favorite parts of the stack is <a href="https://pagefind.app/" rel="noopener noreferrer">Pagefind</a>. After Astro builds the static site, Pagefind indexes it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>astro build <span class="o">&amp;&amp;</span> npx pagefind <span class="nt">--site</span> dist
</code></pre>

</div>



<p>Pagefind generates a tiny WASM search engine and a compressed index. Search happens entirely in the browser — no server, no API calls, no Algolia bill. For a 60+ bill site, the index is small enough that it loads near-instantly.</p>

<h2>
  
  
  The Public API
</h2>

<p>Every bill is available as JSON. The endpoints:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Endpoint</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/api/bills.json</code></td>
<td>All bills with full metadata</td>
</tr>
<tr>
<td><code>/api/real-bills.json</code></td>
<td>Real legislation with absurdity scores</td>
</tr>
<tr>
<td><code>/api/not-bills.json</code></td>
<td>Satirical bills only</td>
</tr>
<tr>
<td><code>/api/stats.json</code></td>
<td>Aggregate statistics</td>
</tr>
<tr>
<td><code>/api/bills/{id}.json</code></td>
<td>Individual bill by ID</td>
</tr>
</tbody>
</table></div>

<p>There's also an embed widget — a single <code>&lt;script&gt;</code> tag that renders an interactive bill card on any website:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;script
  </span><span class="na">src=</span><span class="s">"https://absurdityindex.org/embed.js"</span>
  <span class="na">data-bill=</span><span class="s">"real-hr-25"</span>
<span class="nt">&gt;&lt;/script&gt;</span>
</code></pre>

</div>



<h2>
  
  
  Build-Time Validation
</h2>

<p>One thing I'm proud of is the validation pipeline. Before anything deploys, multiple checks run:</p>

<ol>
<li>
<strong>Zod schema validation</strong> — Every bill's frontmatter is validated against the schema</li>
<li>
<strong>Bill-type refinements</strong> — Real bills must have congressional metadata</li>
<li>
<strong>Icon checks</strong> — No Unicode emoji allowed (we use Lucide icons)</li>
<li>
<strong>innerHTML checks</strong> — Security scan for XSS vectors</li>
<li>
<strong>Secret scanning</strong> — No API keys or credentials in the codebase</li>
<li>
<strong>ESLint + Prettier</strong> — Code quality and formatting</li>
<li>
<strong>TypeScript checking</strong> — Full type safety</li>
</ol>

<p>If any check fails, the build stops. This is especially important when you have 60+ content files with complex frontmatter — one typo in a date format or a missing required field would otherwise silently produce a broken page.</p>

<h2>
  
  
  What I'd Do Differently
</h2>

<ol>
<li><p><strong>Start with Zod refinements earlier.</strong> I added the bill-type validation refinement after discovering that several bills had been committed with missing fields. Build-time schema validation is worth setting up on day one.</p></li>
<li><p><strong>Use <code>z.coerce.date()</code> from the start.</strong> YAML dates can be tricky — <code>2025-01-01</code> in YAML is a date, but <code>2025-01-01T12:00:00</code> is a string. Using <code>z.coerce.date()</code> in the Zod schema handles both formats gracefully.</p></li>
<li><p><strong>Plan the content collection schema for extensibility.</strong> The bill schema grew organically to support bill evolution tracking, pork barrel itemization, amendment tracking, and committee details. A more deliberate upfront design would have saved some refactoring.</p></li>
</ol>

<h2>
  
  
  Try It Out
</h2>

<ul>
<li>
<strong>Site:</strong> <a href="https://absurdityindex.org" rel="noopener noreferrer">absurdityindex.org</a>
</li>
<li>
<strong>API:</strong> <a href="https://absurdityindex.org/api/bills.json" rel="noopener noreferrer">absurdityindex.org/api/bills.json</a>
</li>
<li>
<strong>Scoring methodology:</strong> <a href="https://absurdityindex.org/how-we-score/" rel="noopener noreferrer">absurdityindex.org/how-we-score</a>
</li>
<li>
<strong>Quiz — Real or Satire?:</strong> <a href="https://absurdityindex.org/quiz/" rel="noopener noreferrer">absurdityindex.org/quiz</a>
</li>
<li>
<strong>Embed widget docs:</strong> <a href="https://absurdityindex.org/embed/" rel="noopener noreferrer">absurdityindex.org/embed</a>
</li>
</ul>

<p>The project is open source under the MIT License. If you're building civic tech with Astro, I'd love to hear about it.</p>

