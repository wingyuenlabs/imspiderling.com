---
Title: Building a Privacy-First Financial Analysis PWA with Blazor WebAssembly + On-Device AI (Gemini Nano)
Description: 
Author: Alex
Date: 2025-11-18T21:14:38.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Free, offline-capable, multi-language financial statement analysis for Ukrainian businesses - powered 100% in the browser (no servers, no API keys).</p>
</blockquote>

<p>Live Demo: <a href="https://whitewaw.github.io/Assessment-of-Ukrainian-financial-statements/" rel="noopener noreferrer">https://whitewaw.github.io/Assessment-of-Ukrainian-financial-statements/</a><br><br>
GitHub Repo: <a href="https://github.com/whitewAw/Assessment-of-Ukrainian-financial-statements" rel="noopener noreferrer">https://github.com/whitewAw/Assessment-of-Ukrainian-financial-statements</a></p>


<h2>
  
  
  Why I Built This
</h2>

<p>Small and medium businesses in Ukraine (and anywhere) often need quick insights from their balance sheet and income statement without sending data to third-party services or paying SaaS fees. I wanted:</p>

<ul>
<li>Zero back-end infrastructure</li>
<li>Professional-grade ratio analysis and multi-year comparison</li>
<li>Installable PWA with offline mode</li>
<li>Strict privacy: data never leaves the browser</li>
<li>AI assistance without OpenAI keys or billing surprises</li>
</ul>

<p>Blazor WebAssembly + Chrome's built-in Gemini Nano made this possible.</p>


<h2>
  
  
  Core Highlights
</h2>

<p>Feature -&gt; Benefit</p>

<ul>
<li>Blazor WebAssembly (.NET 10) -&gt; Full C# front-end, no server required </li>
<li>AOT Compilation -&gt; Faster startup + optimized runtime</li>
<li>On-Device AI (Gemini Nano) -&gt; Private financial chat &amp; contextual insights</li>
<li>16 Analysis Tables -&gt; Comprehensive coverage of capital, liquidity, solvency, efficiency </li>
<li>7 Interactive Charts -&gt; Visual composition &amp; trend exploration </li>
<li>Multi-Language (6) -&gt; Ukrainian, English, German, Spanish, French, Russian</li>
<li>PWA + Offline -&gt; Works after first load; installable</li>
<li>Local Storage + JSON Import/Export -&gt; Data portability without backend</li>
<li>Privacy-First -&gt; No telemetry, no tracking, no API keys</li>
</ul>


<h2>
  
  
  AI Assistant: 100% Client-Side
</h2>

<p>Chrome 127+ exposes a Prompt API enabling local inference via Gemini Nano. No keys, no cloud calls.</p>

<p>Example capability prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">capabilities</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">LanguageModel</span><span class="p">.</span><span class="nf">availability</span><span class="p">();</span>
<span class="dl">"</span><span class="s2">available</span><span class="dl">"</span>
</code></pre>

</div>



<p>Sample financial question internally passed to the AI service:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">prompt</span> <span class="p">=</span> <span class="err">$</span><span class="s">$"""
</span><span class="n">Using</span> <span class="n">the</span> <span class="n">company</span><span class="err">'</span><span class="n">s</span> <span class="n">working</span> <span class="nf">capital</span> <span class="p">(</span> <span class="p">{</span><span class="n">model</span><span class="p">.</span><span class="n">WorkingCapitalCurrent</span><span class="p">}</span> <span class="p">),</span> <span class="n">current</span> <span class="nf">ratio</span> <span class="p">(</span> <span class="p">{</span><span class="n">model</span><span class="p">.</span><span class="n">CurrentRatio</span><span class="p">}</span> <span class="p">),</span> <span class="k">and</span> <span class="n">debt</span><span class="p">-</span><span class="n">to</span><span class="p">-</span><span class="nf">equity</span> <span class="p">(</span> <span class="p">{</span><span class="n">model</span><span class="p">.</span><span class="n">DebtToEquity</span><span class="p">}</span> <span class="p">),</span> <span class="n">provide</span><span class="p">:</span>
<span class="m">1</span><span class="p">.</span> <span class="n">A</span> <span class="n">concise</span> <span class="n">stability</span> <span class="n">assessment</span>
<span class="m">2</span><span class="p">.</span> <span class="n">Top</span> <span class="m">3</span> <span class="nf">risks</span> <span class="p">(</span><span class="k">if</span> <span class="n">any</span><span class="p">)</span>
<span class="m">3</span><span class="p">.</span> <span class="n">Recommended</span> <span class="n">next</span> <span class="n">action</span>
<span class="n">Answer</span> <span class="k">in</span> <span class="n">Ukrainian</span> <span class="k">if</span> <span class="n">source</span> <span class="n">data</span> <span class="n">locale</span> <span class="k">is</span> <span class="err">'</span><span class="n">uk</span><span class="err">'</span><span class="p">.</span>
<span class="s">"""$$;
</span></code></pre>

</div>



<p>The assistant streams responses for better UX and can explain formulas contextually.</p>




<h2>
  
  
  Architecture Overview
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AFS/ (Blazor WASM App)
  App.razor          → Root + Router
  Pages/             → UI pages (AI Assistant, Tables, Charts)
  wwwroot/index.html → SEO meta + PWA shell
AFS.ComponentLibrary/
  Components/        → Reusable UI (Tables, Charts, Chat)
AFS.Core/
  Models/            → Financial &amp; ratio models
  Services/          → Calculation + AI integration + Storage
  Interfaces/        → Abstractions (IAIFinancialAdvisor, etc.)
</code></pre>

</div>



<p>Key separation: <code>AFS.Core</code> handles pure calculations (testable logic), while UI stays thin.</p>




<h2>
  
  
  Financial Analysis Coverage
</h2>

<p>Includes:</p>

<ul>
<li>Liquidity: Current, Quick, Cash, Working Capital</li>
<li>Solvency: Debt-to-Equity, Interest Coverage, Leverage</li>
<li>Profitability: ROA, ROE, Margins</li>
<li>Efficiency: Turnover metrics (assets, receivables, inventory, payables)</li>
<li>Stability: Autonomy coefficient, classification levels</li>
<li>Fixed &amp; Intangible asset efficiency</li>
</ul>

<p>Each table is generated from strongly typed models instead of ad-hoc dictionaries, enabling safer refactors.</p>




<h2>
  
  
  Performance Choices
</h2>

<ul>
<li>AOT publish shrinks cold start and improves runtime math-heavy loops</li>
<li>IL trimming + Brotli compression reduces payload</li>
<li>Critical CSS inlined in <code>index.html</code> for faster LCP</li>
<li>Service Worker caches static assets → instant repeat visits</li>
</ul>

<p>Publishing command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Release build with AOT</span>
 dotnet publish AFS/AFS.csproj <span class="nt">-c</span> Release <span class="nt">-o</span> build
</code></pre>

</div>






<h2>
  
  
  Privacy &amp; Data Handling
</h2>

<p>No calls are made to external APIs for analysis. Data stays:</p>

<ul>
<li>In memory (runtime calculations)</li>
<li>In browser Local Storage (optional save)</li>
<li>Optional JSON export:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Year"</span><span class="p">:</span><span class="w"> </span><span class="mi">2024</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Form1"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"AssetsTotal"</span><span class="p">:</span><span class="w"> </span><span class="mi">1234567</span><span class="w"> </span><span class="p">},</span><span class="w">
  </span><span class="nl">"Form2"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"NetProfit"</span><span class="p">:</span><span class="w"> </span><span class="mi">45678</span><span class="w"> </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>You control export/import - nothing is uploaded.</p>




<h2>
  
  
  Getting Started (Local)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Clone</span>
 git clone https://github.com/whitewAw/Assessment-of-Ukrainian-financial-statements.git
 <span class="nb">cd </span>Assessment-of-Ukrainian-financial-statements

<span class="c"># Restore</span>
 dotnet restore

<span class="c"># Ensure WASM tooling</span>
 dotnet workload <span class="nb">install </span>wasm-tools

<span class="c"># Run</span>
 dotnet run <span class="nt">--project</span> AFS
<span class="c"># Visit https://localhost:7157</span>
</code></pre>

</div>



<p>Enable AI (Chrome):</p>

<ol>
<li>Update to Chrome 127+</li>
<li>
<code>chrome://flags/#prompt-api-for-gemini-nano</code> → Enabled</li>
<li>First run downloads Gemini Nano (~1.7GB one-time)</li>
</ol>




<h2>
  
  
  Example: Ratio Calculation Service (Simplified)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">LiquidityRatioService</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">decimal</span> <span class="nf">CurrentRatio</span><span class="p">(</span><span class="kt">decimal</span> <span class="n">currentAssets</span><span class="p">,</span> <span class="kt">decimal</span> <span class="n">currentLiabilities</span><span class="p">)</span>
        <span class="p">=&gt;</span> <span class="n">currentLiabilities</span> <span class="p">==</span> <span class="m">0</span> <span class="p">?</span> <span class="m">0</span> <span class="p">:</span> <span class="n">currentAssets</span> <span class="p">/</span> <span class="n">currentLiabilities</span><span class="p">;</span>

    <span class="k">public</span> <span class="kt">decimal</span> <span class="nf">QuickRatio</span><span class="p">(</span><span class="kt">decimal</span> <span class="n">liquidAssets</span><span class="p">,</span> <span class="kt">decimal</span> <span class="n">currentLiabilities</span><span class="p">)</span>
        <span class="p">=&gt;</span> <span class="n">currentLiabilities</span> <span class="p">==</span> <span class="m">0</span> <span class="p">?</span> <span class="m">0</span> <span class="p">:</span> <span class="n">liquidAssets</span> <span class="p">/</span> <span class="n">currentLiabilities</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Integrated into a composite financial model so derived ratios are accessible to the AI advisor.</p>




<h2>
  
  
  Internationalization (i18n)
</h2>

<p>Resx-based resources under <code>AFS.ComponentLibrary/Resources/</code>:</p>

<ul>
<li><code>Resource.uk.resx</code></li>
<li><code>Resource.en.resx</code></li>
<li>etc.</li>
</ul>

<p>Culture selector updates displayed language instantly - no reload.</p>




<h2>
  
  
  SEO &amp; GitHub Pages Quirk
</h2>

<p>Because this is a Blazor SPA hosted on GitHub Pages, Google Search Console may show a "False 404" during inspection of dynamic routes. A <code>404.html</code> fallback mirrors <code>index.html</code> so all routes hydrate properly. Canonical + hreflang tags are manually set in <code>index.html</code>.</p>




<h2>
  
  
  Roadmap (Selected)
</h2>

<ul>
<li>PDF report generation (AI summary + graphs)</li>
<li>Multi-company comparison panel</li>
<li>Forecasting &amp; anomaly detection (local AI augment)</li>
<li>Optional cloud sync layer (opt-in)</li>
</ul>

<p>Contributions welcome - especially localization or advanced financial models.</p>




<h2>
  
  
  Lessons Learned
</h2>

<ul>
<li>On-device AI drastically lowers friction (no keys, instant trust)</li>
<li>Blazor AOT is now fast enough for real-time ratio computation</li>
<li>Strong typing across financial models prevents subtle column mapping errors</li>
<li>PWA + offline + privacy is a compelling trio for sensitive domains (finance, health, legal)</li>
</ul>




<h2>
  
  
  How You Can Help
</h2>

<ul>
<li>⭐ Star the repo if you find it useful</li>
<li>🐛 Open issues for incorrect ratio definitions or edge cases</li>
<li>🌍 Add new language translations</li>
<li>📊 Contribute advanced metrics or sector benchmarks</li>
<li>🤖 Improve AI prompts for financial diagnostics</li>
</ul>




<h2>
  
  
  Final Thoughts
</h2>

<p>This project shows what’s now possible entirely in the browser: serious domain analysis + AI guidance without surrendering data or paying API bills. If you’re building privacy-first analytical tools, combining Blazor WebAssembly with on-device AI is absolutely worth exploring.</p>

<p>Explore the demo, inspect the code, and feel free to fork.</p>

<p>👉 Demo: <a href="https://whitewaw.github.io/Assessment-of-Ukrainian-financial-statements/" rel="noopener noreferrer">https://whitewaw.github.io/Assessment-of-Ukrainian-financial-statements/</a><br><br>
👉 Code: <a href="https://github.com/whitewAw/Assessment-of-Ukrainian-financial-statements" rel="noopener noreferrer">https://github.com/whitewAw/Assessment-of-Ukrainian-financial-statements</a></p>




<h3>
  
  
  Questions?
</h3>

<p>Drop them in the comments or open a GitHub Discussion.</p>

<p>Made with ❤️ + .NET + WebAssembly + On-Device AI.</p>

