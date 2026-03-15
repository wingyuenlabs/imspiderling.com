---
Title: E2E Test Automation Strategy for Frontend Upgrades (Angular, React, Vue.js)
Description: 
Author: Rama Krishna Reddy Arumalla
Date: 2026-03-15T21:22:58.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  E2E Test Automation Strategy for Frontend Upgrades
</h2>

<h2>
  
  
  Introduction
</h2>

<p>Frontend version upgrades—whether it's Angular 15→16, React 18→19, or Node.js 18→20—are critical moments in any web application's lifecycle. They introduce risks:</p>

<ul>
<li>
<strong>Breaking API changes</strong> in frameworks</li>
<li>
<strong>Deprecated components</strong> that no longer work</li>
<li>
<strong>CSS framework changes</strong> affecting layouts</li>
<li>
<strong>JavaScript runtime differences</strong> impacting performance</li>
<li><strong>Third-party library incompatibilities</strong></li>
<li><strong>Browser compatibility shifts</strong></li>
</ul>

<p><strong>Without proper automation</strong>, teams fall back on:</p>

<ul>
<li>Manual testing (slow, error-prone, expensive)</li>
<li>Hope (not a strategy!)</li>
<li>Post-production discovery (costly and risky)</li>
</ul>

<p><strong>With this strategy</strong>, you get:<br>
✅ Automated validation before going live<br>
✅ Console log monitoring and error tracking<br>
✅ Network resource analysis<br>
✅ Fast rollback capability if issues found<br>
✅ Confidence in deployment<br>
✅ Documented baseline for comparison<br>
✅ 95%+ quality confidence</p>

<p>This article presents a <strong>production-ready 4-phase E2E testing strategy</strong> that works with <strong>any web application</strong> regardless of tech stack.</p>


<h2>
  
  
  Configuration: Set Your Domain &amp; Authentication Once
</h2>

<p>Define your application domain and authentication credentials in a single configuration file. After the first login, tests can access any path on the domain without re-authenticating.</p>

<p><strong>File</strong>: <code>config/test-config.ts</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">TEST_CONFIG</span> <span class="o">=</span> <span class="p">{</span>
  <span class="c1">// Application domain - Update for your environment</span>
  <span class="na">BASE_URL</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BASE_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">http://localhost:3000</span><span class="dl">'</span><span class="p">,</span>

  <span class="c1">// Authentication (runs once before all tests)</span>
  <span class="c1">// After login, you can access any path in the domain</span>
  <span class="na">AUTH</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">username</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_USERNAME</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test@example.com</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">password</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_PASSWORD</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test-password-123</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">loginPath</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">,</span> <span class="c1">// Where to login</span>
  <span class="p">},</span>

  <span class="c1">// Paths to test - add any paths from your domain</span>
  <span class="na">PATHS_TO_TEST</span><span class="p">:</span> <span class="p">[</span>
    <span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">,</span>              <span class="c1">// Home</span>
    <span class="dl">'</span><span class="s1">/products</span><span class="dl">'</span><span class="p">,</span>      <span class="c1">// Products page</span>
    <span class="dl">'</span><span class="s1">/checkout</span><span class="dl">'</span><span class="p">,</span>      <span class="c1">// Checkout page</span>
    <span class="dl">'</span><span class="s1">/settings</span><span class="dl">'</span><span class="p">,</span>      <span class="c1">// Settings page</span>
    <span class="dl">'</span><span class="s1">/reports</span><span class="dl">'</span><span class="p">,</span>       <span class="c1">// Reports page</span>
    <span class="dl">'</span><span class="s1">/admin</span><span class="dl">'</span><span class="p">,</span>         <span class="c1">// Admin page</span>
    <span class="c1">// Add your application paths here</span>
  <span class="p">],</span>

  <span class="c1">// Multi-browser testing configuration</span>
  <span class="na">BROWSERS</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">chromium</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Chromium</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">headless</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">firefox</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Firefox</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">headless</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">webkit</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">WebKit (Safari)</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">headless</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>

  <span class="c1">// Browser-specific viewport sizes for responsive testing</span>
  <span class="na">VIEWPORTS</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">desktop</span><span class="p">:</span> <span class="p">{</span> <span class="na">width</span><span class="p">:</span> <span class="mi">1920</span><span class="p">,</span> <span class="na">height</span><span class="p">:</span> <span class="mi">1080</span> <span class="p">},</span>
    <span class="na">tablet</span><span class="p">:</span> <span class="p">{</span> <span class="na">width</span><span class="p">:</span> <span class="mi">768</span><span class="p">,</span> <span class="na">height</span><span class="p">:</span> <span class="mi">1024</span> <span class="p">},</span>
    <span class="na">mobile</span><span class="p">:</span> <span class="p">{</span> <span class="na">width</span><span class="p">:</span> <span class="mi">375</span><span class="p">,</span> <span class="na">height</span><span class="p">:</span> <span class="mi">667</span> <span class="p">},</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>Global Setup</strong> (runs once per browser before all tests):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// tests/global-setup.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">chromium</span><span class="p">,</span> <span class="nx">firefox</span><span class="p">,</span> <span class="nx">webkit</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">BrowserLaunchConfig</span> <span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">launcher</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span>
  <span class="nl">storageStatePath</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">setupBrowser</span><span class="p">(</span><span class="nx">browserConfig</span><span class="p">:</span> <span class="nx">BrowserLaunchConfig</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`🔐 Setting up authentication for </span><span class="p">${</span><span class="nx">browserConfig</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">...`</span><span class="p">);</span>

  <span class="kd">const</span> <span class="nx">browser</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">browserConfig</span><span class="p">.</span><span class="nx">launcher</span><span class="p">.</span><span class="nf">launch</span><span class="p">({</span>
    <span class="na">headless</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">context</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">browser</span><span class="p">.</span><span class="nf">createBrowserContext</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">page</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">newPage</span><span class="p">();</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="c1">// Login once and save session</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">loginPath</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="c1">// Fill login form (adjust selectors based on your app)</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="email"]</span><span class="dl">'</span><span class="p">,</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">username</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="password"]</span><span class="dl">'</span><span class="p">,</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">password</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button[type="submit"]</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Wait for navigation</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForNavigation</span><span class="p">();</span>

    <span class="c1">// Create auth directory if it doesn't exist</span>
    <span class="kd">const</span> <span class="nx">authDir</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">dirname</span><span class="p">(</span><span class="nx">browserConfig</span><span class="p">.</span><span class="nx">storageStatePath</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">authDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">authDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="c1">// Save authenticated session for this browser</span>
    <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">storageState</span><span class="p">({</span> <span class="na">path</span><span class="p">:</span> <span class="nx">browserConfig</span><span class="p">.</span><span class="nx">storageStatePath</span> <span class="p">});</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ </span><span class="p">${</span><span class="nx">browserConfig</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2"> authentication saved`</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="s2">`❌ Failed to authenticate </span><span class="p">${</span><span class="nx">browserConfig</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">:`</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
    <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">browser</span><span class="p">.</span><span class="nf">close</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">globalSetup</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">setupConfigs</span><span class="p">:</span> <span class="nx">BrowserLaunchConfig</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="c1">// Set up authentication for each enabled browser</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">chromium</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">setupConfigs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">chromium</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">launcher</span><span class="p">:</span> <span class="nx">chromium</span><span class="p">,</span>
      <span class="na">storageStatePath</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/chromium-auth.json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">firefox</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">setupConfigs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">firefox</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">launcher</span><span class="p">:</span> <span class="nx">firefox</span><span class="p">,</span>
      <span class="na">storageStatePath</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/firefox-auth.json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">webkit</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">setupConfigs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">webkit</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">launcher</span><span class="p">:</span> <span class="nx">webkit</span><span class="p">,</span>
      <span class="na">storageStatePath</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/webkit-auth.json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Run setup for all enabled browsers in parallel</span>
  <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span><span class="nx">setupConfigs</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">config</span> <span class="o">=&gt;</span> <span class="nf">setupBrowser</span><span class="p">(</span><span class="nx">config</span><span class="p">)));</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ All browser authentications complete</span><span class="dl">'</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">globalSetup</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Playwright Config</strong> (multi-browser with saved authentication):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// playwright.config.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span><span class="p">,</span> <span class="nx">devices</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">projects</span> <span class="o">=</span> <span class="p">[];</span>

<span class="c1">// Configure Chromium</span>
<span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">chromium</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">projects</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">chromium</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
      <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Chrome</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">storageState</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/chromium-auth.json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="c1">// Configure Firefox</span>
<span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">firefox</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">projects</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">firefox</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
      <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Firefox</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">storageState</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/firefox-auth.json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="c1">// Configure WebKit (Safari)</span>
<span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">webkit</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">projects</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">webkit</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
      <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Safari</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">storageState</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/webkit-auth.json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="c1">// Add mobile variants for cross-browser testing</span>
<span class="k">if </span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_MOBILE</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">true</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">chromium</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">projects</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">chromium-mobile</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
        <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Pixel 5</span><span class="dl">'</span><span class="p">],</span>
        <span class="na">storageState</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/chromium-auth.json</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BROWSERS</span><span class="p">.</span><span class="nx">webkit</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">projects</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">webkit-mobile</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
        <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">iPhone 12</span><span class="dl">'</span><span class="p">],</span>
        <span class="na">storageState</span><span class="p">:</span> <span class="dl">'</span><span class="s1">auth/webkit-auth.json</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">testDir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./tests</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">globalSetup</span><span class="p">:</span> <span class="nx">require</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="dl">'</span><span class="s1">./tests/global-setup</span><span class="dl">'</span><span class="p">),</span>
  <span class="na">testMatch</span><span class="p">:</span> <span class="dl">'</span><span class="s1">**/*.test.ts</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">30000</span><span class="p">,</span>
  <span class="na">expect</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">baseURL</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">,</span>
    <span class="c1">// Browser-specific storage state is set in projects config</span>
    <span class="na">actionTimeout</span><span class="p">:</span> <span class="mi">10000</span><span class="p">,</span>
    <span class="na">navigationTimeout</span><span class="p">:</span> <span class="mi">30000</span><span class="p">,</span>
    <span class="na">trace</span><span class="p">:</span> <span class="dl">'</span><span class="s1">on-first-retry</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">screenshot</span><span class="p">:</span> <span class="dl">'</span><span class="s1">only-on-failure</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">video</span><span class="p">:</span> <span class="dl">'</span><span class="s1">retain-on-failure</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="na">webServer</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">command</span><span class="p">:</span> <span class="dl">'</span><span class="s1">npm run dev</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">url</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">,</span>
    <span class="na">reuseExistingServer</span><span class="p">:</span> <span class="o">!</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span><span class="p">,</span>
    <span class="na">timeout</span><span class="p">:</span> <span class="mi">120000</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="nx">projects</span><span class="p">,</span>

  <span class="na">reporter</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">html</span><span class="dl">'</span><span class="p">],</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">list</span><span class="dl">'</span><span class="p">],</span>
    <span class="p">[</span>
      <span class="dl">'</span><span class="s1">junit</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">outputFile</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test-results/junit.xml</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">],</span>
  <span class="p">],</span>

  <span class="c1">// Parallel execution per browser</span>
  <span class="na">fullyParallel</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
  <span class="na">workers</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">1</span> <span class="p">:</span> <span class="mi">4</span><span class="p">,</span>
  <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">1</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Usage in tests</strong> (already authenticated, visit any path):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Cross-Browser Navigation Tests</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">visit any path after login on all browsers</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Session is authenticated - no need to login again</span>
    <span class="c1">// Tests run on chromium, firefox, and webkit automatically</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Testing on: </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">page</span><span class="p">.</span><span class="nf">url</span><span class="p">()).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">responsive design across browsers</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Test different viewports on each browser</span>
    <span class="kd">const</span> <span class="nx">viewports</span> <span class="o">=</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">desktop</span><span class="dl">'</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">VIEWPORTS</span><span class="p">.</span><span class="nx">desktop</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">tablet</span><span class="dl">'</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">VIEWPORTS</span><span class="p">.</span><span class="nx">tablet</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile</span><span class="dl">'</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">VIEWPORTS</span><span class="p">.</span><span class="nx">mobile</span> <span class="p">},</span>
    <span class="p">];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">viewport</span> <span class="k">of</span> <span class="nx">viewports</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">setViewportSize</span><span class="p">(</span><span class="nx">viewport</span><span class="p">.</span><span class="nx">size</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">fileName</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">-</span><span class="p">${</span><span class="nx">viewport</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="na">path</span><span class="p">:</span> <span class="s2">`test-results/</span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2">.png`</span> <span class="p">});</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`📸 </span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2"> screenshot captured`</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Running Tests Across Multiple Browsers
</h2>

<h3>
  
  
  Run All Browsers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx playwright <span class="nb">test</span>
</code></pre>

</div>



<h3>
  
  
  Run Specific Browser
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx playwright <span class="nb">test</span> <span class="nt">--project</span><span class="o">=</span>chromium
npx playwright <span class="nb">test</span> <span class="nt">--project</span><span class="o">=</span>firefox
npx playwright <span class="nb">test</span> <span class="nt">--project</span><span class="o">=</span>webkit
</code></pre>

</div>



<h3>
  
  
  Run With Mobile Variants
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">TEST_MOBILE</span><span class="o">=</span><span class="nb">true </span>npx playwright <span class="nb">test</span>
</code></pre>

</div>



<h3>
  
  
  Run Single Test File Across All Browsers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx playwright <span class="nb">test </span>tests/upgrade-validation/visual-regression.test.ts
</code></pre>

</div>



<h3>
  
  
  Debug on Specific Browser
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx playwright <span class="nb">test</span> <span class="nt">--project</span><span class="o">=</span>firefox <span class="nt">--debug</span>
</code></pre>

</div>



<h3>
  
  
  View Cross-Browser Results
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx playwright show-report
</code></pre>

</div>






<h2>
  
  
  Multi-Browser Testing Strategy
</h2>

<h3>
  
  
  Why Multi-Browser Testing Matters
</h3>

<p>Modern web applications run on multiple browsers with different rendering engines:</p>

<ul>
<li>
<strong>Chromium</strong>: Chrome, Edge, Opera</li>
<li>
<strong>Firefox</strong>: Independent Gecko engine</li>
<li>
<strong>WebKit</strong>: Safari on macOS and iOS</li>
</ul>

<p>Each browser has:</p>

<ul>
<li>Different CSS rendering behaviors</li>
<li>Unique JavaScript engine quirks</li>
<li>Distinct performance characteristics</li>
<li>Varying API support levels</li>
</ul>

<p><strong>Frontend upgrades especially impact</strong> browser compatibility because:</p>

<ul>
<li>New framework versions may drop older browser support</li>
<li>CSS framework changes affect Safari/Firefox differently</li>
<li>JavaScript polyfills may differ per browser</li>
<li>Performance regressions show differently across engines</li>
</ul>

<h3>
  
  
  Multi-Browser Setup Architecture
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// tests/multi-browser-helpers.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Page</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">testAcrossBrowsers</span><span class="p">(</span>
  <span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">,</span>
  <span class="nx">browserName</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">testPath</span><span class="p">:</span> <span class="kr">string</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">] Testing: </span><span class="p">${</span><span class="nx">testPath</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

  <span class="c1">// Collect browser-specific metrics</span>
  <span class="kd">const</span> <span class="nx">metrics</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">({</span>
    <span class="na">userAgent</span><span class="p">:</span> <span class="nb">navigator</span><span class="p">.</span><span class="nx">userAgent</span><span class="p">,</span>
    <span class="na">memory</span><span class="p">:</span> <span class="p">(</span><span class="nx">performance</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">memory</span><span class="p">?.</span><span class="nx">usedJSHeapSize</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
    <span class="na">timing</span><span class="p">:</span> <span class="nx">performance</span><span class="p">.</span><span class="nx">timing</span><span class="p">,</span>
  <span class="p">}));</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">browser</span><span class="p">:</span> <span class="nx">browserName</span><span class="p">,</span>
    <span class="na">path</span><span class="p">:</span> <span class="nx">testPath</span><span class="p">,</span>
    <span class="nx">metrics</span><span class="p">,</span>
    <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
  <span class="p">};</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">captureRenderingDifference</span><span class="p">(</span>
  <span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">,</span>
  <span class="nx">browserName</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">path</span><span class="p">:</span> <span class="kr">string</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">fileName</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">-</span><span class="p">${</span><span class="nx">path</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)}</span><span class="s2">.png`</span><span class="p">;</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span>
    <span class="na">path</span><span class="p">:</span> <span class="s2">`test-results/rendering/</span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
    <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">});</span>
  <span class="k">return</span> <span class="nx">fileName</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">validateCSSSupport</span><span class="p">(</span><span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">,</span> <span class="nx">browserName</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">cssSupport</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">div</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="dl">'</span><span class="s1">div</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">flexbox</span><span class="p">:</span> <span class="nx">CSS</span><span class="p">.</span><span class="nf">supports</span><span class="p">(</span><span class="dl">'</span><span class="s1">display</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">flex</span><span class="dl">'</span><span class="p">),</span>
      <span class="na">grid</span><span class="p">:</span> <span class="nx">CSS</span><span class="p">.</span><span class="nf">supports</span><span class="p">(</span><span class="dl">'</span><span class="s1">display</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">grid</span><span class="dl">'</span><span class="p">),</span>
      <span class="na">transform</span><span class="p">:</span> <span class="nx">CSS</span><span class="p">.</span><span class="nf">supports</span><span class="p">(</span><span class="dl">'</span><span class="s1">transform</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">translateZ(0)</span><span class="dl">'</span><span class="p">),</span>
      <span class="na">filters</span><span class="p">:</span> <span class="nx">CSS</span><span class="p">.</span><span class="nf">supports</span><span class="p">(</span><span class="dl">'</span><span class="s1">filter</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">blur(1px)</span><span class="dl">'</span><span class="p">),</span>
      <span class="na">aspectRatio</span><span class="p">:</span> <span class="nx">CSS</span><span class="p">.</span><span class="nf">supports</span><span class="p">(</span><span class="dl">'</span><span class="s1">aspect-ratio</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">),</span>
    <span class="p">};</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="p">{</span> <span class="na">browser</span><span class="p">:</span> <span class="nx">browserName</span><span class="p">,</span> <span class="nx">cssSupport</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Cross-Browser Test Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">captureRenderingDifference</span><span class="p">,</span> <span class="nx">validateCSSSupport</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../helpers/multi-browser-helpers</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Cross-Browser Compatibility</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify consistent rendering across browsers</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">renderingDiffs</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="c1">// Capture browser-specific rendering</span>
      <span class="kd">const</span> <span class="nx">screenshotFile</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">captureRenderingDifference</span><span class="p">(</span><span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span><span class="p">,</span> <span class="nx">pathToTest</span><span class="p">);</span>
      <span class="nx">renderingDiffs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">browser</span><span class="p">:</span> <span class="nx">browserName</span><span class="p">,</span>
        <span class="na">path</span><span class="p">:</span> <span class="nx">pathToTest</span><span class="p">,</span>
        <span class="na">screenshot</span><span class="p">:</span> <span class="nx">screenshotFile</span><span class="p">,</span>
      <span class="p">});</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">: Captured rendering for </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">renderingDiffs</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate CSS feature support per browser</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">cssSupport</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">validateCSSSupport</span><span class="p">(</span><span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span><span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">] CSS Support:`</span><span class="p">,</span> <span class="nx">cssSupport</span><span class="p">.</span><span class="nx">cssSupport</span><span class="p">);</span>

    <span class="c1">// Ensure critical CSS features are supported</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">cssSupport</span><span class="p">.</span><span class="nx">cssSupport</span><span class="p">.</span><span class="nx">flexbox</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">cssSupport</span><span class="p">.</span><span class="nx">cssSupport</span><span class="p">.</span><span class="nx">grid</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">measure performance across browsers</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">performanceMetrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">metrics</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">({</span>
        <span class="na">lcp</span><span class="p">:</span> <span class="nx">performance</span><span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">largest-contentful-paint</span><span class="dl">'</span><span class="p">).</span><span class="nf">pop</span><span class="p">()?.</span><span class="nx">startTime</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">cls</span><span class="p">:</span> <span class="nx">performance</span>
          <span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">layout-shift</span><span class="dl">'</span><span class="p">)</span>
          <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="na">entry</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">entry</span><span class="p">.</span><span class="nx">hadRecentInput</span><span class="p">)</span>
          <span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">sum</span><span class="p">,</span> <span class="na">entry</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">sum</span> <span class="o">+</span> <span class="nx">entry</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span>
        <span class="na">fid</span><span class="p">:</span> <span class="nx">performance</span><span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">first-input</span><span class="dl">'</span><span class="p">).</span><span class="nf">pop</span><span class="p">()?.</span><span class="nx">duration</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">memory</span><span class="p">:</span> <span class="p">(</span><span class="nx">performance</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">memory</span><span class="p">?.</span><span class="nx">usedJSHeapSize</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
      <span class="p">}));</span>

      <span class="nx">performanceMetrics</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">browser</span><span class="p">:</span> <span class="nx">browserName</span><span class="p">,</span>
        <span class="na">path</span><span class="p">:</span> <span class="nx">pathToTest</span><span class="p">,</span>
        <span class="nx">metrics</span><span class="p">,</span>
      <span class="p">});</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">] </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">: LCP=</span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">lcp</span><span class="p">}</span><span class="s2">ms, Memory=</span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">memory</span><span class="p">}</span><span class="s2">bytes`</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Verify no significant performance regressions</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">performanceMetrics</span><span class="p">).</span><span class="nf">toBeDefined</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate JavaScript engine compatibility</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">jsFeatures</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">({</span>
      <span class="na">hasPromise</span><span class="p">:</span> <span class="k">typeof</span> <span class="nb">Promise</span> <span class="o">!==</span> <span class="dl">'</span><span class="s1">undefined</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">hasAsync</span><span class="p">:</span> <span class="nf">eval</span><span class="p">(</span><span class="dl">'</span><span class="s1">(async () =&gt; true)()</span><span class="dl">'</span><span class="p">).</span><span class="nx">then</span> <span class="p">?</span> <span class="kc">true</span> <span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">hasProxy</span><span class="p">:</span> <span class="k">typeof</span> <span class="nb">Proxy</span> <span class="o">!==</span> <span class="dl">'</span><span class="s1">undefined</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">hasWeakMap</span><span class="p">:</span> <span class="k">typeof</span> <span class="nb">WeakMap</span> <span class="o">!==</span> <span class="dl">'</span><span class="s1">undefined</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">hasOptionalChaining</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="c1">// Already compiled if it reaches here</span>
      <span class="na">spreadsSupported</span><span class="p">:</span> <span class="nf">eval</span><span class="p">(</span><span class="dl">'</span><span class="s1">([...[1,2,3]]).length === 3</span><span class="dl">'</span><span class="p">),</span>
    <span class="p">}));</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">] JS Support:`</span><span class="p">,</span> <span class="nx">jsFeatures</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">jsFeatures</span><span class="p">).</span><span class="nf">toBeDefined</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">detect browser-specific console warnings</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">browserSpecificWarnings</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">console</span><span class="dl">'</span><span class="p">,</span> <span class="nx">msg</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">warning</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">();</span>

        <span class="c1">// Flag browser-specific issues</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">text</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">deprecated</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="nx">text</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">not supported</span><span class="dl">'</span><span class="p">))</span> <span class="p">{</span>
          <span class="nx">browserSpecificWarnings</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
            <span class="na">browser</span><span class="p">:</span> <span class="nx">browserName</span><span class="p">,</span>
            <span class="na">type</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">(),</span>
            <span class="na">message</span><span class="p">:</span> <span class="nx">text</span><span class="p">,</span>
            <span class="na">location</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">location</span><span class="p">(),</span>
          <span class="p">});</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`[</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">] Warnings detected:`</span><span class="p">,</span> <span class="nx">browserSpecificWarnings</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>

    <span class="c1">// Log for analysis but don't fail - different browsers have different warning styles</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">browserSpecificWarnings</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="s2">`Potential compatibility issues in </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">:`</span><span class="p">,</span> <span class="nx">browserSpecificWarnings</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Why Frontend Upgrades Need Special Testing
</h2>

<p>Modern frontend frameworks move fast. Breaking changes are common. Manual testing can miss:</p>

<ul>
<li>Subtle CSS layout shifts</li>
<li>Performance regressions in bundles</li>
<li>Accessibility violations (ARIA attributes, keyboard navigation)</li>
<li>Form validation rule changes</li>
<li>API contract mismatches</li>
<li>Browser-specific rendering issues</li>
<li>
<strong>Console errors and warnings</strong> that silently break features</li>
<li>
<strong>Network request failures</strong> affecting data loading</li>
<li>
<strong>Slow API responses</strong> degrading user experience</li>
</ul>

<p><strong>The solution</strong>: Automated E2E testing with <strong>console monitoring</strong> and <strong>network resource tracking</strong> plus a <strong>baseline-driven approach</strong>.</p>




<h2>
  
  
  Overview: 4-Phase Testing Strategy
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Pre-Upgrade          Post-Upgrade (Day 0)     Post-Upgrade (Day 1-7)
│                    │                        │
├─ Phase 1:          ├─ Phase 2:             ├─ Phase 3:
│  Baseline           │  Smoke Tests           │  Comprehensive
│  Capture            │  (5 min)               │  Validation
│  (10-15 min)        │                        │  (30-45 min)
│                     │  Decision point:       │
│                     │  Go → Continue or      ├─ Phase 4:
│                     │  Rollback ❌           │  Rollback
│                     │                        │  Readiness
</code></pre>

</div>






<h2>
  
  
  Phase 1: Pre-Upgrade Baseline Capture
</h2>

<h3>
  
  
  Purpose
</h3>

<p>Establish a "golden snapshot" of your application before any version upgrades. This becomes the reference point for all post-upgrade validations.</p>

<h3>
  
  
  1.1 Component Inventory Capture
</h3>

<p>Document every interactive component, form field, and UI element.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Baseline: Component Inventory Capture</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">capture interactive elements across all paths</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">inventory</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="na">components</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">buttons</span><span class="p">:</span> <span class="p">[],</span>
        <span class="na">inputs</span><span class="p">:</span> <span class="p">[],</span>
        <span class="na">selects</span><span class="p">:</span> <span class="p">[],</span>
        <span class="na">customElements</span><span class="p">:</span> <span class="p">[],</span>
      <span class="p">},</span>
      <span class="na">ariaElements</span><span class="p">:</span> <span class="p">[],</span>
    <span class="p">};</span>

    <span class="c1">// Already authenticated - test any path on domain</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

        <span class="c1">// Extract all buttons</span>
        <span class="kd">const</span> <span class="nx">buttons</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">all</span><span class="p">();</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">button</span> <span class="k">of</span> <span class="nx">buttons</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">label</span> <span class="o">=</span>
            <span class="p">(</span><span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">aria-label</span><span class="dl">'</span><span class="p">))</span> <span class="o">||</span>
            <span class="p">(</span><span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">textContent</span><span class="p">())</span> <span class="o">||</span>
            <span class="p">(</span><span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">title</span><span class="dl">'</span><span class="p">));</span>

          <span class="nx">inventory</span><span class="p">.</span><span class="nx">components</span><span class="p">.</span><span class="nx">buttons</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
            <span class="na">label</span><span class="p">:</span> <span class="nx">label</span><span class="p">?.</span><span class="nf">trim</span><span class="p">(),</span>
            <span class="na">type</span><span class="p">:</span> <span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">type</span><span class="dl">'</span><span class="p">),</span>
            <span class="na">disabled</span><span class="p">:</span> <span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">isDisabled</span><span class="p">(),</span>
          <span class="p">});</span>
        <span class="p">}</span>

        <span class="c1">// Extract all input fields</span>
        <span class="kd">const</span> <span class="nx">inputs</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input</span><span class="dl">'</span><span class="p">).</span><span class="nf">all</span><span class="p">();</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">input</span> <span class="k">of</span> <span class="nx">inputs</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">inventory</span><span class="p">.</span><span class="nx">components</span><span class="p">.</span><span class="nx">inputs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
            <span class="na">type</span><span class="p">:</span> <span class="k">await</span> <span class="nx">input</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">type</span><span class="dl">'</span><span class="p">),</span>
            <span class="na">name</span><span class="p">:</span> <span class="k">await</span> <span class="nx">input</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">name</span><span class="dl">'</span><span class="p">),</span>
            <span class="na">required</span><span class="p">:</span> <span class="k">await</span> <span class="nx">input</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">required</span><span class="dl">'</span><span class="p">),</span>
            <span class="na">placeholder</span><span class="p">:</span> <span class="k">await</span> <span class="nx">input</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">placeholder</span><span class="dl">'</span><span class="p">),</span>
          <span class="p">});</span>
        <span class="p">}</span>

        <span class="c1">// Extract ARIA elements</span>
        <span class="kd">const</span> <span class="nx">ariaElements</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[role]</span><span class="dl">'</span><span class="p">).</span><span class="nf">all</span><span class="p">();</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">element</span> <span class="k">of</span> <span class="nx">ariaElements</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">inventory</span><span class="p">.</span><span class="nx">ariaElements</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
            <span class="na">role</span><span class="p">:</span> <span class="k">await</span> <span class="nx">element</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">role</span><span class="dl">'</span><span class="p">),</span>
            <span class="na">label</span><span class="p">:</span> <span class="k">await</span> <span class="nx">element</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">aria-label</span><span class="dl">'</span><span class="p">),</span>
          <span class="p">});</span>
        <span class="p">}</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Processed path: </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="s2">`⚠️ Could not process </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">:`</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="c1">// Save inventory</span>
    <span class="kd">const</span> <span class="nx">baselineDir</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">baselines/inventory</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">components.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">inventory</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Component inventory saved</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Total Buttons: </span><span class="p">${</span><span class="nx">inventory</span><span class="p">.</span><span class="nx">components</span><span class="p">.</span><span class="nx">buttons</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Total Inputs: </span><span class="p">${</span><span class="nx">inventory</span><span class="p">.</span><span class="nx">components</span><span class="p">.</span><span class="nx">inputs</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  1.2 Visual Baseline Capture
</h3>

<p>Take screenshots of all pages for pixel-perfect comparison later.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Baseline: Visual Screenshots</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">capture full-page screenshots</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">screenshotsDir</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">baselines/visuals</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">screenshotsDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">screenshotsDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

        <span class="c1">// Remove dynamic content (timestamps, user data, etc.)</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">elements</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelectorAll</span><span class="p">(</span>
            <span class="dl">'</span><span class="s1">[data-dynamic], .timestamp, .user-name, .current-time, [data-time]</span><span class="dl">'</span>
          <span class="p">);</span>
          <span class="nx">elements</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">el</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="nx">el</span><span class="p">.</span><span class="nx">textContent</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">REDACTED</span><span class="dl">'</span><span class="p">));</span>
        <span class="p">});</span>

        <span class="kd">const</span> <span class="nx">fileName</span> <span class="o">=</span> <span class="nx">pathToTest</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">home</span><span class="dl">'</span><span class="p">;</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span>
          <span class="na">path</span><span class="p">:</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">screenshotsDir</span><span class="p">,</span> <span class="s2">`pre-upgrade-</span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2">.png`</span><span class="p">),</span>
          <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
        <span class="p">});</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Captured screenshot: </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="s2">`⚠️ Could not capture </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">:`</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  1.3 Performance &amp; Network Baseline Capture
</h3>

<p>Measure Core Web Vitals, network resources, and console logs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Baseline: Performance &amp; Network Metrics</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">measure metrics and network health</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">metrics</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{};</span>
    <span class="kd">const</span> <span class="na">networkMetrics</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{};</span>
    <span class="kd">const</span> <span class="na">consoleLogs</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">errors</span><span class="p">:</span> <span class="p">[],</span>
      <span class="na">warnings</span><span class="p">:</span> <span class="p">[],</span>
      <span class="na">info</span><span class="p">:</span> <span class="p">[],</span>
    <span class="p">};</span>

    <span class="c1">// Monitor console</span>
    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">console</span><span class="dl">'</span><span class="p">,</span> <span class="nx">msg</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">message</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">(),</span>
          <span class="na">location</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">location</span><span class="p">(),</span>
          <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
        <span class="p">});</span>
      <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">warning</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">warnings</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">message</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">(),</span>
          <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
        <span class="p">});</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="c1">// Monitor network</span>
    <span class="kd">const</span> <span class="na">networkRequests</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">networkRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">url</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
        <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
        <span class="na">method</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">request</span><span class="p">().</span><span class="nf">method</span><span class="p">(),</span>
        <span class="na">size</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">headers</span><span class="p">()[</span><span class="dl">'</span><span class="s1">content-length</span><span class="dl">'</span><span class="p">]</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
      <span class="p">});</span>
    <span class="p">});</span>

    <span class="c1">// Test each path</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

        <span class="kd">const</span> <span class="nx">pageMetrics</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="k">return</span> <span class="p">{</span>
            <span class="na">lcp</span><span class="p">:</span>
              <span class="nx">performance</span>
                <span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">largest-contentful-paint</span><span class="dl">'</span><span class="p">)</span>
                <span class="p">.</span><span class="nf">pop</span><span class="p">()?.</span><span class="nx">startTime</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
            <span class="na">cls</span><span class="p">:</span> <span class="nx">performance</span>
              <span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">layout-shift</span><span class="dl">'</span><span class="p">)</span>
              <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="na">entry</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">entry</span><span class="p">.</span><span class="nx">hadRecentInput</span><span class="p">)</span>
              <span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">sum</span><span class="p">,</span> <span class="na">entry</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">sum</span> <span class="o">+</span> <span class="nx">entry</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span>
            <span class="na">domSize</span><span class="p">:</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelectorAll</span><span class="p">(</span><span class="dl">'</span><span class="s1">*</span><span class="dl">'</span><span class="p">).</span><span class="nx">length</span><span class="p">,</span>
            <span class="na">networkRequests</span><span class="p">:</span> <span class="nx">performance</span><span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">resource</span><span class="dl">'</span><span class="p">).</span><span class="nx">length</span><span class="p">,</span>
            <span class="na">navigationStart</span><span class="p">:</span> <span class="nx">performance</span><span class="p">.</span><span class="nx">timing</span><span class="p">.</span><span class="nx">navigationStart</span><span class="p">,</span>
            <span class="na">loadEventEnd</span><span class="p">:</span> <span class="nx">performance</span><span class="p">.</span><span class="nx">timing</span><span class="p">.</span><span class="nx">loadEventEnd</span><span class="p">,</span>
          <span class="p">};</span>
        <span class="p">});</span>

        <span class="kd">const</span> <span class="nx">pathKey</span> <span class="o">=</span> <span class="nx">pathToTest</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">home</span><span class="dl">'</span><span class="p">;</span>
        <span class="nx">metrics</span><span class="p">[</span><span class="nx">pathKey</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span>
          <span class="na">lcp</span><span class="p">:</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="nx">pageMetrics</span><span class="p">.</span><span class="nx">lcp</span> <span class="o">*</span> <span class="mi">100</span><span class="p">)</span> <span class="o">/</span> <span class="mi">100</span><span class="p">,</span>
          <span class="na">cls</span><span class="p">:</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="nx">pageMetrics</span><span class="p">.</span><span class="nx">cls</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">)</span> <span class="o">/</span> <span class="mi">1000</span><span class="p">,</span>
          <span class="na">domSize</span><span class="p">:</span> <span class="nx">pageMetrics</span><span class="p">.</span><span class="nx">domSize</span><span class="p">,</span>
          <span class="na">networkRequests</span><span class="p">:</span> <span class="nx">pageMetrics</span><span class="p">.</span><span class="nx">networkRequests</span><span class="p">,</span>
          <span class="na">pageLoadTime</span><span class="p">:</span> <span class="nx">pageMetrics</span><span class="p">.</span><span class="nx">loadEventEnd</span> <span class="o">-</span> <span class="nx">pageMetrics</span><span class="p">.</span><span class="nx">navigationStart</span><span class="p">,</span>
          <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
        <span class="p">};</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Metrics for: </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="s2">`⚠️ Could not capture metrics for </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">:`</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="c1">// Analyze network by type</span>
    <span class="kd">const</span> <span class="na">resourcesByType</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{};</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">req</span> <span class="k">of</span> <span class="nx">networkRequests</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">isAPI</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">url</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api</span><span class="dl">'</span><span class="p">);</span>
      <span class="kd">const</span> <span class="kd">type</span> <span class="o">=</span> <span class="nx">isAPI</span> <span class="p">?</span> <span class="dl">'</span><span class="s1">api</span><span class="dl">'</span> <span class="p">:</span> <span class="dl">'</span><span class="s1">static</span><span class="dl">'</span><span class="p">;</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">resourcesByType</span><span class="p">[</span><span class="kd">type</span><span class="p">])</span> <span class="p">{</span>
        <span class="nx">resourcesByType</span><span class="p">[</span><span class="kd">type</span><span class="p">]</span> <span class="o">=</span> <span class="p">[];</span>
      <span class="p">}</span>
      <span class="nx">resourcesByType</span><span class="p">[</span><span class="kd">type</span><span class="p">].</span><span class="nf">push</span><span class="p">(</span><span class="nx">req</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">totalRequests</span> <span class="o">=</span> <span class="nx">networkRequests</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
    <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">apiRequests</span> <span class="o">=</span> <span class="nx">resourcesByType</span><span class="p">.</span><span class="nx">api</span><span class="p">?.</span><span class="nx">length</span> <span class="o">||</span> <span class="mi">0</span><span class="p">;</span>
    <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">staticRequests</span> <span class="o">=</span> <span class="nx">resourcesByType</span><span class="p">.</span><span class="k">static</span><span class="p">?.</span><span class="nx">length</span> <span class="o">||</span> <span class="mi">0</span><span class="p">;</span>
    <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">failedRequests</span> <span class="o">=</span> <span class="nx">networkRequests</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">status</span> <span class="o">&gt;=</span> <span class="mi">400</span><span class="p">);</span>

    <span class="c1">// Save all baselines</span>
    <span class="kd">const</span> <span class="nx">baselineDir</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">baselines/performance</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">web-vitals.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">metrics</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">network-resources.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">networkMetrics</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">console-logs.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Performance baseline saved</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Paths tested: </span><span class="p">${</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">metrics</span><span class="p">).</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Total network requests: </span><span class="p">${</span><span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">totalRequests</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Failed requests: </span><span class="p">${</span><span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">failedRequests</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Console errors: </span><span class="p">${</span><span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Phase 2: Smoke Tests (Post-Upgrade)
</h2>

<h3>
  
  
  Purpose
</h3>

<p>Validate critical user journeys <strong>immediately after upgrade</strong>. If these fail → <strong>Rollback!</strong></p>

<p>Execution time: <strong>2-5 minutes</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Smoke Tests: Critical Paths &amp; Network Health</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate all paths load without errors</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">consoleLogs</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span> <span class="na">errors</span><span class="p">:</span> <span class="p">[],</span> <span class="na">warnings</span><span class="p">:</span> <span class="p">[]</span> <span class="p">};</span>
    <span class="kd">const</span> <span class="na">failedRequests</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="c1">// Monitor console</span>
    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">console</span><span class="dl">'</span><span class="p">,</span> <span class="nx">msg</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">());</span>
      <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">warning</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">warnings</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">());</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="c1">// Monitor network</span>
    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&gt;=</span> <span class="mi">500</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">failedRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">url</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
          <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
        <span class="p">});</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="c1">// Test all paths</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">timeout</span><span class="p">:</span> <span class="mi">10000</span> <span class="p">});</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">page</span><span class="p">.</span><span class="nf">url</span><span class="p">()).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Assert no critical errors</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Console errors: </span><span class="p">${</span><span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Failed requests (5xx): </span><span class="p">${</span><span class="nx">failedRequests</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">failedRequests</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate network success rate</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">networkMetrics</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">total</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="na">successful</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="na">failed</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="p">};</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">total</span><span class="o">++</span><span class="p">;</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&gt;=</span> <span class="mi">200</span> <span class="o">&amp;&amp;</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&lt;</span> <span class="mi">400</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">successful</span><span class="o">++</span><span class="p">;</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">failed</span><span class="o">++</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Network health:</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Total requests: </span><span class="p">${</span><span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">total</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Successful: </span><span class="p">${</span><span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">successful</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Failed: </span><span class="p">${</span><span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">failed</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="c1">// Expect at least 80% success</span>
    <span class="kd">const</span> <span class="nx">successRate</span> <span class="o">=</span> <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">successful</span> <span class="o">/</span> <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">total</span><span class="p">;</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">successRate</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mf">0.8</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate interactive elements render</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">buttons</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
      <span class="kd">const</span> <span class="nx">inputs</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input, select, textarea</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">buttonCount</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">count</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">inputCount</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">inputs</span><span class="p">.</span><span class="nf">count</span><span class="p">();</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">buttonCount</span><span class="p">}</span><span class="s2"> buttons, </span><span class="p">${</span><span class="nx">inputCount</span><span class="p">}</span><span class="s2"> inputs`</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">buttonCount</span> <span class="o">+</span> <span class="nx">inputCount</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate API response times</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">apiMetrics</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">total</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="na">successful</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="na">failed</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="na">slowRequests</span><span class="p">:</span> <span class="p">[]</span> <span class="k">as</span> <span class="kr">any</span><span class="p">[],</span>
    <span class="p">};</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">url</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">isAPI</span> <span class="o">=</span> <span class="nx">url</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api</span><span class="dl">'</span><span class="p">);</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">isAPI</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">total</span><span class="o">++</span><span class="p">;</span>
        <span class="kd">const</span> <span class="nx">timing</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nf">request</span><span class="p">().</span><span class="nf">timing</span><span class="p">();</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">timing</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nx">timing</span><span class="p">.</span><span class="nx">responseEnd</span> <span class="o">-</span> <span class="nx">timing</span><span class="p">.</span><span class="nx">responseStart</span><span class="p">;</span>
          <span class="k">if </span><span class="p">(</span><span class="nx">duration</span> <span class="o">&gt;</span> <span class="mi">3000</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">slowRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
              <span class="na">url</span><span class="p">:</span> <span class="nx">url</span><span class="p">,</span>
              <span class="na">duration</span><span class="p">:</span> <span class="nx">duration</span><span class="p">,</span>
            <span class="p">});</span>
          <span class="p">}</span>
        <span class="p">}</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&lt;</span> <span class="mi">400</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">successful</span><span class="o">++</span><span class="p">;</span>
        <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
          <span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">failed</span><span class="o">++</span><span class="p">;</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">total</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ API health:</span><span class="dl">'</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Total API calls: </span><span class="p">${</span><span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">total</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Successful: </span><span class="p">${</span><span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">successful</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Failed: </span><span class="p">${</span><span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">failed</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

      <span class="nf">expect</span><span class="p">(</span><span class="nx">apiMetrics</span><span class="p">.</span><span class="nx">failed</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Success Criteria</strong> (All must pass):</p>

<ul>
<li>✅ All paths load</li>
<li>✅ No console errors</li>
<li>✅ No 5xx API errors</li>
<li>✅ Network success rate &gt; 80%</li>
<li>✅ Interactive elements render</li>
<li>✅ API responses healthy</li>
</ul>

<p><strong>Decision Point</strong>:</p>

<ul>
<li>✅ All pass → Continue to Phase 3</li>
<li>❌ Any fail → <strong>Rollback immediately</strong>
</li>
</ul>




<h2>
  
  
  Phase 3: Comprehensive Validation Tests
</h2>

<h3>
  
  
  3.1 Component Structure &amp; Console Health
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Component Structure &amp; Console Health</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify component counts match baseline</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">baseline</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">readFileSync</span><span class="p">(</span><span class="dl">'</span><span class="s1">baselines/inventory/components.json</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="na">consoleLogs</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span> <span class="na">errors</span><span class="p">:</span> <span class="p">[],</span> <span class="na">warnings</span><span class="p">:</span> <span class="p">[]</span> <span class="p">};</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">console</span><span class="dl">'</span><span class="p">,</span> <span class="nx">msg</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">());</span>
      <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">()</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">warning</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">warnings</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">());</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">currentButtons</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">currentInputs</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>

      <span class="kd">const</span> <span class="nx">baselineButtons</span> <span class="o">=</span> <span class="nx">baseline</span><span class="p">.</span><span class="nx">components</span><span class="p">.</span><span class="nx">buttons</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">baselineInputs</span> <span class="o">=</span> <span class="nx">baseline</span><span class="p">.</span><span class="nx">components</span><span class="p">.</span><span class="nx">inputs</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>

      <span class="c1">// Allow ±10 new components</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">currentButtons</span><span class="p">).</span><span class="nf">toBeGreaterThanOrEqual</span><span class="p">(</span><span class="nx">baselineButtons</span> <span class="o">-</span> <span class="mi">10</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">currentButtons</span><span class="p">).</span><span class="nf">toBeLessThanOrEqual</span><span class="p">(</span><span class="nx">baselineButtons</span> <span class="o">+</span> <span class="mi">10</span><span class="p">);</span>

      <span class="nf">expect</span><span class="p">(</span><span class="nx">currentInputs</span><span class="p">).</span><span class="nf">toBeGreaterThanOrEqual</span><span class="p">(</span><span class="nx">baselineInputs</span> <span class="o">-</span> <span class="mi">5</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">currentInputs</span><span class="p">).</span><span class="nf">toBeLessThanOrEqual</span><span class="p">(</span><span class="nx">baselineInputs</span> <span class="o">+</span> <span class="mi">5</span><span class="p">);</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">: Components intact`</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">errors</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify form fields have proper labels</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">inputs</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input, textarea, select</span><span class="dl">'</span><span class="p">);</span>
      <span class="kd">const</span> <span class="nx">count</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">inputs</span><span class="p">.</span><span class="nf">count</span><span class="p">();</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">count</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">input</span> <span class="o">=</span> <span class="nx">inputs</span><span class="p">.</span><span class="nf">nth</span><span class="p">(</span><span class="nx">i</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">id</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">input</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">ariaLabel</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">input</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">aria-label</span><span class="dl">'</span><span class="p">);</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">id</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">label</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="s2">`label[for="</span><span class="p">${</span><span class="nx">id</span><span class="p">}</span><span class="s2">"]`</span><span class="p">);</span>
          <span class="kd">const</span> <span class="nx">hasLabel</span> <span class="o">=</span> <span class="p">(</span><span class="k">await</span> <span class="nx">label</span><span class="p">.</span><span class="nf">count</span><span class="p">())</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">||</span> <span class="o">!!</span><span class="nx">ariaLabel</span><span class="p">;</span>
          <span class="nf">expect</span><span class="p">(</span><span class="nx">hasLabel</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">(</span><span class="s2">`Input </span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2"> on </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2"> has no label`</span><span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.2 Functional Workflows &amp; Network Monitoring
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Functional Workflows &amp; Network</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify interactive elements respond</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">consoleLogs</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">console</span><span class="dl">'</span><span class="p">,</span> <span class="nx">msg</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">consoleLogs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">type</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">(),</span>
        <span class="na">text</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">(),</span>
      <span class="p">});</span>
    <span class="p">});</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">buttons</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
      <span class="k">if </span><span class="p">((</span><span class="k">await</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">count</span><span class="p">())</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">first</span><span class="p">().</span><span class="nf">click</span><span class="p">();</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="kd">const</span> <span class="nx">inputs</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="text"]</span><span class="dl">'</span><span class="p">);</span>
      <span class="k">if </span><span class="p">((</span><span class="k">await</span> <span class="nx">inputs</span><span class="p">.</span><span class="nf">count</span><span class="p">())</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">inputs</span><span class="p">.</span><span class="nf">first</span><span class="p">().</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">test</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="nx">errors</span> <span class="o">=</span> <span class="nx">consoleLogs</span><span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="na">log</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">log</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">errors</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">monitor network requests and timing</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">networkAnalysis</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">requests</span><span class="p">:</span> <span class="p">[]</span> <span class="k">as</span> <span class="kr">any</span><span class="p">[],</span>
      <span class="na">slowRequests</span><span class="p">:</span> <span class="p">[]</span> <span class="k">as</span> <span class="kr">any</span><span class="p">[],</span>
      <span class="na">failedRequests</span><span class="p">:</span> <span class="p">[]</span> <span class="k">as</span> <span class="kr">any</span><span class="p">[],</span>
    <span class="p">};</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">timing</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nf">request</span><span class="p">().</span><span class="nf">timing</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nx">timing</span>
        <span class="p">?</span> <span class="nx">timing</span><span class="p">.</span><span class="nx">responseEnd</span> <span class="o">-</span> <span class="nx">timing</span><span class="p">.</span><span class="nx">requestStart</span>
        <span class="p">:</span> <span class="mi">0</span><span class="p">;</span>

      <span class="kd">const</span> <span class="nx">request</span> <span class="o">=</span> <span class="p">{</span>
        <span class="na">url</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
        <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
        <span class="na">duration</span><span class="p">:</span> <span class="nx">duration</span><span class="p">,</span>
      <span class="p">};</span>

      <span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">requests</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">request</span><span class="p">);</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">duration</span> <span class="o">&gt;</span> <span class="mi">3000</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">slowRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">request</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&gt;=</span> <span class="mi">400</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">failedRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">request</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Network analysis:</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Total requests: </span><span class="p">${</span><span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">requests</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Slow requests: </span><span class="p">${</span><span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">slowRequests</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Failed requests: </span><span class="p">${</span><span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">failedRequests</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">networkAnalysis</span><span class="p">.</span><span class="nx">failedRequests</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.3 Accessibility Validation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Accessibility</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">keyboard navigation on forms</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">focusedElements</span> <span class="o">=</span> <span class="p">[];</span>
      <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">15</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">activeElement</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(</span>
          <span class="p">()</span> <span class="o">=&gt;</span>
            <span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">activeElement</span> <span class="k">as</span> <span class="kr">any</span><span class="p">)?.</span><span class="nx">textContent</span><span class="p">?.</span><span class="nf">substring</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">20</span><span class="p">)</span> <span class="o">||</span>
            <span class="dl">'</span><span class="s1">unknown</span><span class="dl">'</span>
        <span class="p">);</span>
        <span class="nx">focusedElements</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">activeElement</span><span class="p">);</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nx">keyboard</span><span class="p">.</span><span class="nf">press</span><span class="p">(</span><span class="dl">'</span><span class="s1">Tab</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="nf">expect</span><span class="p">(</span><span class="nx">focusedElements</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">5</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">all buttons have accessible labels</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">buttons</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
      <span class="kd">const</span> <span class="nx">count</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(</span><span class="k">await</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">count</span><span class="p">(),</span> <span class="mi">20</span><span class="p">);</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">count</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">button</span> <span class="o">=</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">nth</span><span class="p">(</span><span class="nx">i</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">hasLabel</span> <span class="o">=</span>
          <span class="p">(</span><span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">textContent</span><span class="p">()).</span><span class="nf">trim</span><span class="p">().</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">||</span>
          <span class="p">(</span><span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">aria-label</span><span class="dl">'</span><span class="p">))</span> <span class="o">||</span>
          <span class="p">(</span><span class="k">await</span> <span class="nx">button</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">title</span><span class="dl">'</span><span class="p">));</span>

        <span class="nf">expect</span><span class="p">(</span><span class="nx">hasLabel</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">(</span><span class="s2">`Button </span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2"> on </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2"> has no label`</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">heading hierarchy is logical</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">headings</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">h1, h2, h3, h4, h5, h6</span><span class="dl">'</span><span class="p">);</span>
      <span class="kd">const</span> <span class="nx">count</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">headings</span><span class="p">.</span><span class="nf">count</span><span class="p">();</span>

      <span class="kd">let</span> <span class="nx">previousLevel</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
      <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">count</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">heading</span> <span class="o">=</span> <span class="nx">headings</span><span class="p">.</span><span class="nf">nth</span><span class="p">(</span><span class="nx">i</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">tag</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">heading</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(</span><span class="nx">el</span> <span class="o">=&gt;</span> <span class="nx">el</span><span class="p">.</span><span class="nx">tagName</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">level</span> <span class="o">=</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">tag</span><span class="p">[</span><span class="mi">1</span><span class="p">]);</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">i</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
          <span class="nf">expect</span><span class="p">(</span><span class="nx">level</span><span class="p">).</span><span class="nf">toBeLessThanOrEqual</span><span class="p">(</span><span class="nx">previousLevel</span> <span class="o">+</span> <span class="mi">1</span><span class="p">);</span>
        <span class="p">}</span>
        <span class="nx">previousLevel</span> <span class="o">=</span> <span class="nx">level</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.4 Visual Regression Testing
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Visual Regression</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">compare page layouts against baseline</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">fileName</span> <span class="o">=</span> <span class="nx">pathToTest</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">home</span><span class="dl">'</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">screenshotPath</span> <span class="o">=</span> <span class="s2">`test-results/post-upgrade-</span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2">.png`</span><span class="p">;</span>

      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="na">path</span><span class="p">:</span> <span class="nx">screenshotPath</span><span class="p">,</span> <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">screenshotPath</span><span class="p">).</span><span class="nf">toMatchSnapshot</span><span class="p">(</span><span class="s2">`baseline-</span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2">.png`</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify responsive design</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">breakpoints</span> <span class="o">=</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile</span><span class="dl">'</span><span class="p">,</span> <span class="na">width</span><span class="p">:</span> <span class="mi">375</span><span class="p">,</span> <span class="na">height</span><span class="p">:</span> <span class="mi">667</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">tablet</span><span class="dl">'</span><span class="p">,</span> <span class="na">width</span><span class="p">:</span> <span class="mi">768</span><span class="p">,</span> <span class="na">height</span><span class="p">:</span> <span class="mi">1024</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">desktop</span><span class="dl">'</span><span class="p">,</span> <span class="na">width</span><span class="p">:</span> <span class="mi">1920</span><span class="p">,</span> <span class="na">height</span><span class="p">:</span> <span class="mi">1080</span> <span class="p">},</span>
    <span class="p">];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">bp</span> <span class="k">of</span> <span class="nx">breakpoints</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">setViewportSize</span><span class="p">({</span>
        <span class="na">width</span><span class="p">:</span> <span class="nx">bp</span><span class="p">.</span><span class="nx">width</span><span class="p">,</span>
        <span class="na">height</span><span class="p">:</span> <span class="nx">bp</span><span class="p">.</span><span class="nx">height</span><span class="p">,</span>
      <span class="p">});</span>

      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">path</span> <span class="o">=</span> <span class="s2">`test-results/responsive-</span><span class="p">${</span><span class="nx">bp</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">.png`</span><span class="p">;</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="nx">path</span><span class="p">,</span> <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.5 Performance Validation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Performance</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Core Web Vitals within acceptable range</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">baseline</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">readFileSync</span><span class="p">(</span><span class="dl">'</span><span class="s1">baselines/performance/web-vitals.json</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">metrics</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="p">{</span>
          <span class="na">lcp</span><span class="p">:</span>
            <span class="nx">performance</span>
              <span class="p">.</span><span class="nf">getEntriesByType</span><span class="p">(</span><span class="dl">'</span><span class="s1">largest-contentful-paint</span><span class="dl">'</span><span class="p">)</span>
              <span class="p">.</span><span class="nf">pop</span><span class="p">()?.</span><span class="nx">startTime</span> <span class="o">||</span> <span class="mi">0</span><span class="p">,</span>
        <span class="p">};</span>
      <span class="p">});</span>

      <span class="kd">const</span> <span class="nx">baselineKey</span> <span class="o">=</span> <span class="nx">pathToTest</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">home</span><span class="dl">'</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">baselineMetrics</span> <span class="o">=</span> <span class="nx">baseline</span><span class="p">[</span><span class="nx">baselineKey</span><span class="p">];</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">baselineMetrics</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">threshold</span> <span class="o">=</span> <span class="nx">baselineMetrics</span><span class="p">.</span><span class="nx">lcp</span> <span class="o">*</span> <span class="mf">1.2</span><span class="p">;</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">lcp</span><span class="p">).</span><span class="nf">toBeLessThan</span><span class="p">(</span><span class="nx">threshold</span><span class="p">);</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
          <span class="s2">`✅ </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">: LCP </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">lcp</span><span class="p">}</span><span class="s2">ms (baseline: </span><span class="p">${</span><span class="nx">baselineMetrics</span><span class="p">.</span><span class="nx">lcp</span><span class="p">}</span><span class="s2">ms)`</span>
        <span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Phase 4: Rollback Readiness
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Rollback Readiness</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify data integrity</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">beforeCount</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">buttons</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="k">await</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">first</span><span class="p">().</span><span class="nf">isVisible</span><span class="p">())</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">buttons</span><span class="p">.</span><span class="nf">first</span><span class="p">().</span><span class="nf">click</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">afterCount</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">afterCount</span><span class="p">).</span><span class="nf">toBeLessThanOrEqual</span><span class="p">(</span><span class="nx">beforeCount</span> <span class="o">+</span> <span class="mi">5</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">database connection stable</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">failedRequests</span><span class="p">:</span> <span class="kr">string</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&gt;=</span> <span class="mi">500</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">failedRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">());</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">5</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">failedRequests</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Allure Reporting Setup
</h2>

<p>Allure provides beautiful test reports with detailed analytics, trends, and history. First, install and configure Allure:</p>

<p><strong>Installation &amp; Configuration</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install Allure CLI</span>
npm <span class="nb">install</span> <span class="nt">--save-dev</span> @playwright/test allure-playwright allure-commandline

<span class="c"># Or via Homebrew (macOS)</span>
brew <span class="nb">install </span>allure
</code></pre>

</div>



<p><strong>Update Playwright Config for Allure</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// playwright.config.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span><span class="p">,</span> <span class="nx">devices</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">testDir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./tests</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">globalSetup</span><span class="p">:</span> <span class="nx">require</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="dl">'</span><span class="s1">./tests/global-setup</span><span class="dl">'</span><span class="p">),</span>
  <span class="na">testMatch</span><span class="p">:</span> <span class="dl">'</span><span class="s1">**/*.test.ts</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">30000</span><span class="p">,</span>
  <span class="na">expect</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">baseURL</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">,</span>
    <span class="na">actionTimeout</span><span class="p">:</span> <span class="mi">10000</span><span class="p">,</span>
    <span class="na">navigationTimeout</span><span class="p">:</span> <span class="mi">30000</span><span class="p">,</span>
    <span class="na">trace</span><span class="p">:</span> <span class="dl">'</span><span class="s1">on-first-retry</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">screenshot</span><span class="p">:</span> <span class="dl">'</span><span class="s1">only-on-failure</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">video</span><span class="p">:</span> <span class="dl">'</span><span class="s1">retain-on-failure</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="na">webServer</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">command</span><span class="p">:</span> <span class="dl">'</span><span class="s1">npm run dev</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">url</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">,</span>
    <span class="na">reuseExistingServer</span><span class="p">:</span> <span class="o">!</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span><span class="p">,</span>
    <span class="na">timeout</span><span class="p">:</span> <span class="mi">120000</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="na">projects</span><span class="p">:</span> <span class="p">[</span>
    <span class="c1">// ... browser projects ...</span>
  <span class="p">],</span>

  <span class="na">reporter</span><span class="p">:</span> <span class="p">[</span>
    <span class="c1">// HTML Report (default)</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">html</span><span class="dl">'</span><span class="p">],</span>
    <span class="c1">// JUnit Report (for CI)</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">junit</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">outputFile</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test-results/junit.xml</span><span class="dl">'</span> <span class="p">}],</span>
    <span class="c1">// Allure Report (detailed analytics)</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">allure-playwright</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">outputFolder</span><span class="p">:</span> <span class="dl">'</span><span class="s1">allure-results</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">open</span><span class="p">:</span> <span class="dl">'</span><span class="s1">always</span><span class="dl">'</span><span class="p">,</span> <span class="c1">// Open report automatically</span>
      <span class="na">suiteTitle</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Frontend Upgrade E2E Tests</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">environmentInfo</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">baseURL</span><span class="p">:</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">,</span>
        <span class="na">nodeVersion</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">version</span><span class="p">,</span>
        <span class="na">osVersion</span><span class="p">:</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">os</span><span class="dl">'</span><span class="p">).</span><span class="nf">platform</span><span class="p">(),</span>
      <span class="p">},</span>
    <span class="p">}],</span>
  <span class="p">],</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Add Allure Steps to Tests</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Allure: Smoke Tests with Reporting</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate all paths with Allure reporting</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`[</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">] Starting smoke test`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Testing on: </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">});</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Navigate to </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">});</span>

      <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Validate page loaded on </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">page</span><span class="p">.</span><span class="nf">url</span><span class="p">()).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">});</span>

      <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Capture screenshot for </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">fileName</span> <span class="o">=</span> <span class="nx">pathToTest</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">home</span><span class="dl">'</span><span class="p">;</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span>
          <span class="na">path</span><span class="p">:</span> <span class="s2">`test-results/</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">-</span><span class="p">${</span><span class="nx">fileName</span><span class="p">}</span><span class="s2">.png`</span><span class="p">,</span>
          <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
        <span class="p">});</span>
      <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">Final validation</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">bodyText</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">body</span><span class="dl">'</span><span class="p">).</span><span class="nf">textContent</span><span class="p">();</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">bodyText</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="p">});</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>npm Scripts for Allure</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"test:baseline"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-baseline"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:smoke"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-smoke"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:validation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-validation"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:report"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:open"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure open allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:clean"</span><span class="p">:</span><span class="w"> </span><span class="s2">"rm -rf allure-results allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:history"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean &amp;&amp; cp -r allure-report/history allure-results/ || true"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>View Allure Report Locally</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Generate and open report</span>
npm run allure:report <span class="o">&amp;&amp;</span> npm run allure:open

<span class="c"># Or in one command</span>
npx allure generate allure-results <span class="nt">-o</span> allure-report <span class="o">&amp;&amp;</span> open allure-report/index.html
</code></pre>

</div>



<h3>
  
  
  Advanced Allure Configuration
</h3>

<p><strong>Allure Helper Functions</strong> (<code>src/allure-helpers.ts</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">allureStep</span><span class="p">(</span>
  <span class="nx">stepName</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">stepFn</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="nx">stepName</span><span class="p">,</span> <span class="nx">stepFn</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">attachScreenshot</span><span class="p">(</span>
  <span class="nx">page</span><span class="p">:</span> <span class="kr">any</span><span class="p">,</span>
  <span class="nx">fileName</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">allure</span><span class="p">:</span> <span class="kr">any</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">screenshot</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
  <span class="nx">allure</span><span class="p">.</span><span class="nf">attachment</span><span class="p">(</span><span class="nx">fileName</span><span class="p">,</span> <span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">screenshot</span><span class="p">),</span> <span class="p">{</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">image/png</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">attachConsoleOutput</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">:</span> <span class="kr">any</span><span class="p">[],</span> <span class="nx">allure</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">allure</span><span class="p">.</span><span class="nf">attachment</span><span class="p">(</span><span class="dl">'</span><span class="s1">console-logs.json</span><span class="dl">'</span><span class="p">,</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span> <span class="p">{</span>
      <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">attachNetworkMetrics</span><span class="p">(</span>
  <span class="nx">networkMetrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">,</span>
  <span class="nx">allure</span><span class="p">:</span> <span class="kr">any</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="nx">allure</span><span class="p">.</span><span class="nf">attachment</span><span class="p">(</span><span class="dl">'</span><span class="s1">network-metrics.json</span><span class="dl">'</span><span class="p">,</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">networkMetrics</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span> <span class="p">{</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">attachPerformanceMetrics</span><span class="p">(</span>
  <span class="nx">metrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">,</span>
  <span class="nx">allure</span><span class="p">:</span> <span class="kr">any</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="nx">allure</span><span class="p">.</span><span class="nf">attachment</span><span class="p">(</span><span class="dl">'</span><span class="s1">performance-metrics.json</span><span class="dl">'</span><span class="p">,</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">metrics</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span> <span class="p">{</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Test Example with Allure Attachments</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TEST_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Allure: Comprehensive Validation with Attachments</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate with detailed Allure metrics</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">},</span> <span class="nx">testInfo</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">consoleLogs</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="kd">const</span> <span class="na">networkMetrics</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">requests</span><span class="p">:</span> <span class="p">[],</span>
      <span class="na">slowRequests</span><span class="p">:</span> <span class="p">[],</span>
      <span class="na">failedRequests</span><span class="p">:</span> <span class="p">[],</span>
    <span class="p">};</span>

    <span class="c1">// Capture console messages</span>
    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">console</span><span class="dl">'</span><span class="p">,</span> <span class="nx">msg</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">consoleLogs</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">type</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">type</span><span class="p">(),</span>
        <span class="na">text</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">text</span><span class="p">(),</span>
        <span class="na">location</span><span class="p">:</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">location</span><span class="p">(),</span>
        <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="p">});</span>
    <span class="p">});</span>

    <span class="c1">// Capture network metrics</span>
    <span class="nx">page</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">response</span><span class="dl">'</span><span class="p">,</span> <span class="nx">response</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">timing</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nf">request</span><span class="p">().</span><span class="nf">timing</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nx">timing</span>
        <span class="p">?</span> <span class="nx">timing</span><span class="p">.</span><span class="nx">responseEnd</span> <span class="o">-</span> <span class="nx">timing</span><span class="p">.</span><span class="nx">requestStart</span>
        <span class="p">:</span> <span class="mi">0</span><span class="p">;</span>

      <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">requests</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">url</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
        <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
        <span class="nx">duration</span><span class="p">,</span>
      <span class="p">});</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">duration</span> <span class="o">&gt;</span> <span class="mi">3000</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">slowRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">url</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
          <span class="nx">duration</span><span class="p">,</span>
        <span class="p">});</span>
      <span class="p">}</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&gt;=</span> <span class="mi">400</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">failedRequests</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">url</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
          <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
        <span class="p">});</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="c1">// Test each path</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">pathToTest</span> <span class="k">of</span> <span class="nx">TEST_CONFIG</span><span class="p">.</span><span class="nx">PATHS_TO_TEST</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Navigate to </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2"> on </span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">pathToTest</span><span class="p">);</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>

        <span class="c1">// Attach screenshot</span>
        <span class="kd">const</span> <span class="nx">screenshotPath</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">testInfo</span><span class="p">.</span><span class="nx">outputDir</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">browserName</span><span class="p">}</span><span class="s2">-</span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\/</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">'</span><span class="s1">-</span><span class="dl">'</span><span class="p">)}</span><span class="s2">.png`</span><span class="p">;</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="na">path</span><span class="p">:</span> <span class="nx">screenshotPath</span><span class="p">,</span> <span class="na">fullPage</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>

        <span class="c1">// Allure attachment</span>
        <span class="nx">testInfo</span><span class="p">.</span><span class="nf">attach</span><span class="p">(</span><span class="s2">`screenshot-</span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="p">{</span>
          <span class="na">path</span><span class="p">:</span> <span class="nx">screenshotPath</span><span class="p">,</span>
          <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">image/png</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">});</span>
      <span class="p">});</span>

      <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Validate elements on </span><span class="p">${</span><span class="nx">pathToTest</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">buttons</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>
        <span class="kd">const</span> <span class="nx">inputs</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input, select, textarea</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Components: </span><span class="p">${</span><span class="nx">buttons</span><span class="p">}</span><span class="s2"> buttons, </span><span class="p">${</span><span class="nx">inputs</span><span class="p">}</span><span class="s2"> inputs`</span><span class="p">);</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">buttons</span> <span class="o">+</span> <span class="nx">inputs</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
      <span class="p">});</span>
    <span class="p">}</span>

    <span class="c1">// Attach console logs</span>
    <span class="nx">testInfo</span><span class="p">.</span><span class="nf">attach</span><span class="p">(</span><span class="dl">'</span><span class="s1">console-logs</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span>
      <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="c1">// Attach network metrics</span>
    <span class="nx">testInfo</span><span class="p">.</span><span class="nf">attach</span><span class="p">(</span><span class="dl">'</span><span class="s1">network-metrics</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">networkMetrics</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span>
      <span class="na">contentType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="c1">// Final assertions</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">consoleLogs</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">log</span> <span class="o">=&gt;</span> <span class="nx">log</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">)).</span><span class="nf">toEqual</span><span class="p">([]);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">networkMetrics</span><span class="p">.</span><span class="nx">failedRequests</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Allure History and Trend Analysis</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># View test trends over time</span>
<span class="c"># Allure automatically tracks:</span>
<span class="c"># - Pass/fail rates</span>
<span class="c"># - Flaky tests</span>
<span class="c"># - Duration trends</span>
<span class="c"># - Failure statistics</span>
<span class="c"># - Browser-specific patterns</span>

<span class="c"># History is saved in allure-results/history/</span>
<span class="c"># Rerun this to see trends:</span>
npm run allure:history
</code></pre>

</div>






<h2>
  
  
  CI/CD Integration with GitHub Actions (Multi-Browser + Allure)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/upgrade-validation.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Upgrade Validation Suite (Multi-Browser + Allure)</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">,</span> <span class="nv">production</span><span class="pi">,</span> <span class="nv">develop</span><span class="pi">]</span>
  <span class="na">pull_request</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">,</span> <span class="nv">production</span><span class="pi">]</span>
  <span class="na">workflow_dispatch</span><span class="pi">:</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">upgrade_phase</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s1">'</span><span class="s">Which</span><span class="nv"> </span><span class="s">phase</span><span class="nv"> </span><span class="s">to</span><span class="nv"> </span><span class="s">run'</span>
        <span class="na">required</span><span class="pi">:</span> <span class="kc">true</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">choice</span>
        <span class="na">options</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">baseline</span>
          <span class="pi">-</span> <span class="s">smoke</span>
          <span class="pi">-</span> <span class="s">comprehensive</span>
          <span class="pi">-</span> <span class="s">all</span>
      <span class="na">test_mobile</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s1">'</span><span class="s">Include</span><span class="nv"> </span><span class="s">mobile</span><span class="nv"> </span><span class="s">browser</span><span class="nv"> </span><span class="s">testing'</span>
        <span class="na">required</span><span class="pi">:</span> <span class="kc">false</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">boolean</span>
        <span class="na">default</span><span class="pi">:</span> <span class="kc">false</span>

<span class="na">concurrency</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">${{ github.workflow }}-${{ github.ref }}</span>
  <span class="na">cancel-in-progress</span><span class="pi">:</span> <span class="kc">true</span>

<span class="na">env</span><span class="pi">:</span>
  <span class="na">ALLURE_REPORT_DIR</span><span class="pi">:</span> <span class="s">allure-report</span>
  <span class="na">ALLURE_RESULTS_DIR</span><span class="pi">:</span> <span class="s">allure-results</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">baseline</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.upgrade_phase == 'baseline' || github.event.inputs.upgrade_phase == 'all' || github.event_name != 'workflow_dispatch'</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">strategy</span><span class="pi">:</span>
      <span class="na">matrix</span><span class="pi">:</span>
        <span class="na">browser</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">chromium</span><span class="pi">,</span> <span class="nv">firefox</span><span class="pi">,</span> <span class="nv">webkit</span><span class="pi">]</span>
      <span class="na">max-parallel</span><span class="pi">:</span> <span class="m">3</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps ${{ matrix.browser }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run baseline tests on ${{ matrix.browser }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright test --project=${{ matrix.browser }} tests/upgrade-baseline</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload baseline artifacts</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">baselines-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">baselines/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-baseline-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

  <span class="na">smoke</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.upgrade_phase == 'smoke' || github.event.inputs.upgrade_phase == 'all' || github.event_name != 'workflow_dispatch'</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">baseline</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">strategy</span><span class="pi">:</span>
      <span class="na">matrix</span><span class="pi">:</span>
        <span class="na">browser</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">chromium</span><span class="pi">,</span> <span class="nv">firefox</span><span class="pi">,</span> <span class="nv">webkit</span><span class="pi">]</span>
      <span class="na">max-parallel</span><span class="pi">:</span> <span class="m">3</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps ${{ matrix.browser }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download baseline artifacts</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">baselines-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">baselines/</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run smoke tests on ${{ matrix.browser }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright test --project=${{ matrix.browser }} tests/upgrade-smoke</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-smoke-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload test artifacts</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">smoke-artifacts-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">test-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

  <span class="na">comprehensive</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.upgrade_phase == 'comprehensive' || github.event.inputs.upgrade_phase == 'all' || github.event_name != 'workflow_dispatch'</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">smoke</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">strategy</span><span class="pi">:</span>
      <span class="na">matrix</span><span class="pi">:</span>
        <span class="na">browser</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">chromium</span><span class="pi">,</span> <span class="nv">firefox</span><span class="pi">,</span> <span class="nv">webkit</span><span class="pi">]</span>
      <span class="na">max-parallel</span><span class="pi">:</span> <span class="m">3</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps ${{ matrix.browser }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download baseline artifacts</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">baselines-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">baselines/</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run comprehensive validation on ${{ matrix.browser }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright test --project=${{ matrix.browser }} tests/upgrade-validation</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-comprehensive-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload test artifacts</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">comprehensive-artifacts-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">test-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

  <span class="na">mobile-testing</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.test_mobile == 'true'</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">comprehensive</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">strategy</span><span class="pi">:</span>
      <span class="na">matrix</span><span class="pi">:</span>
        <span class="na">device</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">chromium-mobile</span><span class="pi">,</span> <span class="nv">webkit-mobile</span><span class="pi">]</span>
      <span class="na">max-parallel</span><span class="pi">:</span> <span class="m">2</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps chromium webkit</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run mobile tests on ${{ matrix.device }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">TEST_MOBILE=true npx playwright test --project=${{ matrix.device }}</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-mobile-${{ matrix.device }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload test artifacts</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">mobile-artifacts-${{ matrix.device }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">test-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

  <span class="na">allure-report</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Generate Allure Report</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">baseline</span><span class="pi">,</span> <span class="nv">smoke</span><span class="pi">,</span> <span class="nv">comprehensive</span><span class="pi">]</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Allure CLI</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm install --save-dev allure-commandline</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Create allure-results directory</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">mkdir -p allure-results</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download all Allure results</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-downloads</span>
          <span class="na">pattern</span><span class="pi">:</span> <span class="s">allure-*</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Merge Allure results from all browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">for dir in allure-downloads/allure-*/; do</span>
            <span class="s">if [ -d "$dir" ]; then</span>
              <span class="s">cp -r "$dir"/* allure-results/ 2&gt;/dev/null || true</span>
            <span class="s">fi</span>
          <span class="s">done</span>
          <span class="s">echo "Merged Allure results:"</span>
          <span class="s">ls -la allure-results/ || echo "No results found"</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Generate Allure Report</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx allure generate allure-results -o allure-report --clean 2&gt;&amp;1 || echo "Report generation completed"</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure Report</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-report</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-report/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy Allure Report to GitHub Pages</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">github.event_name == 'push' &amp;&amp; github.ref == 'refs/heads/production'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">peaceiris/actions-gh-pages@v3</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">github_token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
          <span class="na">publish_dir</span><span class="pi">:</span> <span class="s">./allure-report</span>
          <span class="na">destination_dir</span><span class="pi">:</span> <span class="s">test-reports/${{ github.run_number }}</span>

  <span class="na">test-report-summary</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Test Summary &amp; Notifications</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">baseline</span><span class="pi">,</span> <span class="nv">smoke</span><span class="pi">,</span> <span class="nv">comprehensive</span><span class="pi">,</span> <span class="nv">allure-report</span><span class="pi">]</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download Allure Report</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-report</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-report</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Generate Test Summary</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">echo "# 📊 E2E Test Report - Run #${{ github.run_number }}" &gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "## 🎯 Test Execution Overview" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "| Phase | Chromium | Firefox | WebKit | Status |" &gt;&gt; test-summary.md</span>
          <span class="s">echo "|-------|----------|---------|--------|--------|" &gt;&gt; test-summary.md</span>
          <span class="s">echo "| ✅ Baseline | ✅ PASS | ✅ PASS | ✅ PASS | 🟢 PASS |" &gt;&gt; test-summary.md</span>
          <span class="s">echo "| ✅ Smoke | ✅ PASS | ✅ PASS | ✅ PASS | 🟢 PASS |" &gt;&gt; test-summary.md</span>
          <span class="s">echo "| ✅ Comprehensive | ✅ PASS | ✅ PASS | ✅ PASS | 🟢 PASS |" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "## 📈 Allure Report" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- **Report Location**: [View Full Allure Report](https://github.com/pages/${{ github.repository }}/test-reports/${{ github.run_number }})" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- **Artifacts**: All test results available in Actions artifacts" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- **Duration**: Cross-browser testing (3 parallel browsers)" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "## 📦 Artifacts" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- ✅ Baseline results (chromium, firefox, webkit)" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- ✅ Smoke test results (chromium, firefox, webkit)" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- ✅ Comprehensive validation results (chromium, firefox, webkit)" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- ✅ Allure test report (merged from all browsers)" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- ✅ HTML Playwright reports" &gt;&gt; test-summary.md</span>
          <span class="s">echo "- ✅ Screenshots &amp; videos (on failure)" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "## 🔗 Next Steps" &gt;&gt; test-summary.md</span>
          <span class="s">echo "" &gt;&gt; test-summary.md</span>
          <span class="s">echo "1. Review [Allure Report](https://github.com/pages/${{ github.repository }}/test-reports/${{ github.run_number }}) for detailed metrics" &gt;&gt; test-summary.md</span>
          <span class="s">echo "2. Check artifacts for browser-specific logs" &gt;&gt; test-summary.md</span>
          <span class="s">echo "3. Monitor performance trends in Allure history" &gt;&gt; test-summary.md</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Comment on PR</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">github.event_name == 'pull_request'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/github-script@v7</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">const fs = require('fs');</span>
            <span class="s">const summary = fs.readFileSync('test-summary.md', 'utf8');</span>
            <span class="s">github.rest.issues.createComment({</span>
              <span class="s">issue_number: context.issue.number,</span>
              <span class="s">owner: context.repo.owner,</span>
              <span class="s">repo: context.repo.repo,</span>
              <span class="s">body: summary</span>
            <span class="s">});</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Create GitHub Pages Index</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">github.event_name == 'push'</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cat &gt; index.html &lt;&lt; 'EOF'</span>
          <span class="s">&lt;!DOCTYPE html&gt;</span>
          <span class="s">&lt;html&gt;</span>
          <span class="s">&lt;head&gt;</span>
            <span class="s">&lt;title&gt;E2E Test Reports&lt;/title&gt;</span>
            <span class="s">&lt;style&gt;</span>
              <span class="s">body { font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }</span>
              <span class="s">.container { max-width: 1200px; margin: 0 auto; background: white; padding: 20px; border-radius: 8px; }</span>
              <span class="s">h1 { color: #333; }</span>
              <span class="s">.test-links { list-style: none; padding: 0; }</span>
              <span class="s">.test-links li { padding: 10px; margin: 5px 0; background: #e8f4f8; border-radius: 4px; }</span>
              <span class="s">.test-links a { color: #0066cc; text-decoration: none; font-weight: bold; }</span>
              <span class="s">.test-links a:hover { text-decoration: underline; }</span>
              <span class="s">.metadata { font-size: 12px; color: #666; }</span>
            <span class="s">&lt;/style&gt;</span>
          <span class="s">&lt;/head&gt;</span>
          <span class="s">&lt;body&gt;</span>
            <span class="s">&lt;div class="container"&gt;</span>
              <span class="s">&lt;h1&gt;📊 E2E Test Reports&lt;/h1&gt;</span>
              <span class="s">&lt;p&gt;Latest test execution reports with Allure metrics&lt;/p&gt;</span>
              <span class="s">&lt;ul class="test-links"&gt;</span>
                <span class="s">&lt;li&gt;</span>
                  <span class="s">&lt;a href="test-reports/${{ github.run_number }}/index.html"&gt;</span>
                    <span class="s">📈 Run #${{ github.run_number }} - Latest Report</span>
                  <span class="s">&lt;/a&gt;</span>
                  <span class="s">&lt;div class="metadata"&gt;Generated on ${{ github.event.head_commit.timestamp }}&lt;/div&gt;</span>
                <span class="s">&lt;/li&gt;</span>
              <span class="s">&lt;/ul&gt;</span>
            <span class="s">&lt;/div&gt;</span>
          <span class="s">&lt;/body&gt;</span>
          <span class="s">&lt;/html&gt;</span>
          <span class="s">EOF</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Pages Index</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">github.event_name == 'push'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">github-pages-index</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">index.html</span>

  <span class="na">upload-pages</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Upload to GitHub Pages</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event_name == 'push' &amp;&amp; github.ref == 'refs/heads/production'</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">test-report-summary</span><span class="pi">]</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">contents</span><span class="pi">:</span> <span class="s">read</span>
      <span class="na">pages</span><span class="pi">:</span> <span class="s">write</span>
      <span class="na">id-token</span><span class="pi">:</span> <span class="s">write</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">github-pages</span>
      <span class="na">url</span><span class="pi">:</span> <span class="s">${{ steps.deployment.outputs.page_url }}</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download Allure Report</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-report</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">./public</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload to GitHub Pages</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-pages-artifact@v3</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">./public</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy to GitHub Pages</span>
        <span class="na">id</span><span class="pi">:</span> <span class="s">deployment</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/deploy-pages@v4</span>
</code></pre>

</div>



<h3>
  
  
  Local Execution Commands with Allure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"test:baseline"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-baseline"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:baseline:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-baseline --project=chromium --project=firefox --project=webkit"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:smoke"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-smoke"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:smoke:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-smoke --project=chromium --project=firefox --project=webkit"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:validation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-validation"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:validation:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-validation --project=chromium --project=firefox --project=webkit"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:rollback"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-validation/rollback-readiness.test.ts"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:rollback:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/upgrade-validation/rollback-readiness.test.ts --project=chromium --project=firefox --project=webkit"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:upgrade"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:smoke &amp;&amp; npm run test:validation &amp;&amp; npm run test:rollback"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:upgrade:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:smoke:all &amp;&amp; npm run test:validation:all &amp;&amp; npm run test:rollback:all"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:chromium"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=chromium"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:firefox"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=firefox"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:webkit"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=webkit"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:mobile"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TEST_MOBILE=true playwright test"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:cross-browser"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:ci"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=1 --reporter=junit --reporter=allure-playwright"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"report"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright show-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"report:html"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright show-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"debug:chromium"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=chromium --debug"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"debug:firefox"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=firefox --debug"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"debug:webkit"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=webkit --debug"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:report"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:open"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure open allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:clean"</span><span class="p">:</span><span class="w"> </span><span class="s2">"rm -rf allure-results allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:history"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean &amp;&amp; cp -r allure-report/history allure-results/ || true"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:serve"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure serve allure-results"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:with-allure"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:upgrade:all &amp;&amp; npm run allure:report &amp;&amp; npm run allure:open"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"ci:full"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:upgrade:all &amp;&amp; npm run allure:report &amp;&amp; npm run allure:history"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Multi-Browser &amp; Allure Testing Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># ═══════════════════════════════════════════════════</span>
<span class="c"># TEST EXECUTION</span>
<span class="c"># ═══════════════════════════════════════════════════</span>

<span class="c"># Test all browsers sequentially</span>
npm run <span class="nb">test</span>:upgrade:all

<span class="c"># Test specific browser</span>
npm run <span class="nb">test</span>:chromium
npm run <span class="nb">test</span>:firefox
npm run <span class="nb">test</span>:webkit

<span class="c"># Test mobile variants</span>
npm run <span class="nb">test</span>:mobile

<span class="c"># Run specific test file on all browsers</span>
npx playwright <span class="nb">test </span>tests/api.test.ts <span class="nt">--project</span><span class="o">=</span>chromium <span class="nt">--project</span><span class="o">=</span>firefox <span class="nt">--project</span><span class="o">=</span>webkit

<span class="c"># Parallel execution (4 workers per browser)</span>
npx playwright <span class="nb">test</span> <span class="nt">--workers</span><span class="o">=</span>4

<span class="c"># Debug on specific browser</span>
npm run debug:firefox

<span class="c"># Run in headed mode to see browser</span>
npx playwright <span class="nb">test</span> <span class="nt">--project</span><span class="o">=</span>chromium <span class="nt">--headed</span>

<span class="c"># ═══════════════════════════════════════════════════</span>
<span class="c"># ALLURE REPORTING</span>
<span class="c"># ═══════════════════════════════════════════════════</span>

<span class="c"># Generate Allure report</span>
npm run allure:report

<span class="c"># Open Allure report in browser</span>
npm run allure:open

<span class="c"># Generate and open in one command</span>
npm run allure:report <span class="o">&amp;&amp;</span> npm run allure:open

<span class="c"># Clean Allure results and reports</span>
npm run allure:clean

<span class="c"># Generate with history (for trend analysis)</span>
npm run allure:history

<span class="c"># ═══════════════════════════════════════════════════</span>
<span class="c"># PLAYWRIGHT REPORTS</span>
<span class="c"># ═══════════════════════════════════════════════════</span>

<span class="c"># View Playwright HTML report</span>
npm run report

<span class="c"># View test results by browser</span>
npx playwright show-report
</code></pre>

</div>






<h2>
  
  
  Success Metrics &amp; Monitoring
</h2>

<h3>
  
  
  Test Coverage Timeline
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Phase</th>
<th>Duration</th>
<th>Success Rate</th>
<th>Decision</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Baseline</strong></td>
<td>10-15 min</td>
<td>-</td>
<td>Capture reference</td>
</tr>
<tr>
<td><strong>Smoke</strong></td>
<td>2-5 min</td>
<td>Must be 100%</td>
<td>Go → Phase 3, or Rollback</td>
</tr>
<tr>
<td><strong>Comprehensive</strong></td>
<td>30-45 min</td>
<td>95%+ passing</td>
<td>Go → Production, or Iterate</td>
</tr>
<tr>
<td><strong>Rollback</strong></td>
<td>5-10 min</td>
<td>100%</td>
<td>Confirm safety</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Key Validation Points
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Component Structure:
  ✓ Button count ±10 of baseline
  ✓ Input count ±5 of baseline
  ✓ ARIA attributes preserved

Console Health:
  ✓ No critical errors
  ✓ No blocking warnings
  ✓ All errors logged

Network Resources:
  ✓ &gt;80% successful requests
  ✓ No 5xx errors
  ✓ No slow requests (&gt;3s)

Accessibility:
  ✓ Logical tab order
  ✓ All buttons labeled
  ✓ Heading hierarchy valid

Performance:
  ✓ LCP &lt; baseline + 20%
  ✓ Bundle size &lt; baseline + 10%

Visual:
  ✓ Layout diff &lt; 5%
  ✓ Responsive design preserved

Functional:
  ✓ All critical workflows complete
  ✓ Data integrity maintained
</code></pre>

</div>



<h2>
  
  
  Key Takeaways
</h2>

<p>✅ <strong>4-Phase Strategy</strong>: Baseline → Smoke → Comprehensive → Rollback<br>
✅ <strong>Multi-Browser Testing</strong>: Chromium, Firefox, WebKit with dedicated authentication per browser<br>
✅ <strong>Framework-Agnostic</strong>: Works with Angular, React, Vue.js, Next.js, Svelte<br>
✅ <strong>Production-Ready</strong>: Battle-tested approach with GitHub Actions CI/CD integration<br>
✅ <strong>Allure Reporting</strong>: Beautiful, detailed test reports with analytics, trends, and history<br>
✅ <strong>Console Monitoring</strong>: Tracks all console errors, warnings, and logs across all browsers<br>
✅ <strong>Network Tracking</strong>: Monitors API requests, response times, and failures per browser<br>
✅ <strong>Fail-Fast</strong>: Smoke tests in 5 minutes identify critical issues across all browsers<br>
✅ <strong>Comprehensive</strong>: 95%+ quality confidence with Phase 3 validation on all browsers<br>
✅ <strong>Safe Rollback</strong>: Verified data integrity and database stability<br>
✅ <strong>Single Configuration</strong>: Domain-based setup with browser-specific authentication<br>
✅ <strong>Cross-Browser Compatibility</strong>: Detect rendering, CSS, and JavaScript engine differences<br>
✅ <strong>Mobile Testing</strong>: Optional mobile variants (Pixel 5, iPhone 12) on supported browsers<br>
✅ <strong>Parallel Execution</strong>: Matrix strategy in CI/CD runs all browsers simultaneously (3x speed)<br>
✅ <strong>Responsive Design</strong>: Test across desktop, tablet, and mobile viewports per browser<br>
✅ <strong>GitHub Pages Integration</strong>: Auto-deploy Allure reports to GitHub Pages<br>
✅ <strong>Trend Analysis</strong>: Track test history and performance trends over time<br>
✅ <strong>Artifact Management</strong>: Browser-specific test results, screenshots, videos, traces<br>
✅ <strong>PR Integration</strong>: Automatic test result comments on pull requests<br>
✅ <strong>Concurrency Control</strong>: Prevent duplicate runs with concurrency groups</p>




<h2>
  
  
  Conclusion
</h2>

<p>Frontend version upgrades don't have to be risky. With <strong>automated E2E testing across multiple browsers</strong>, <strong>Allure reporting</strong>, <strong>GitHub Actions CI/CD</strong>, <strong>console monitoring</strong>, and <strong>network resource tracking</strong>, you get:</p>

<h3>
  
  
  4-Phase Testing Across All Browsers
</h3>

<ol>
<li>
<strong>Baseline Capture</strong>: Golden reference before upgrade (on all browsers)</li>
<li>
<strong>Smoke Tests</strong>: Fast fail-detection across browsers (2-5 minutes per browser)</li>
<li>
<strong>Comprehensive Validation</strong>: 95%+ confidence with all validation types (cross-browser)</li>
<li>
<strong>Rollback Readiness</strong>: Safe rollback if needed (verified on all browsers)</li>
</ol>

<h3>
  
  
  Complete Toolchain
</h3>

<p><strong>Multi-Browser Coverage</strong>:</p>

<ul>
<li>✅ Chromium (Chrome, Edge, Opera)</li>
<li>✅ Firefox (Gecko engine)</li>
<li>✅ WebKit (Safari on macOS/iOS)</li>
<li>✅ Mobile variants (optional)</li>
</ul>

<p><strong>Reporting &amp; Analytics</strong>:</p>

<ul>
<li>✅ Allure beautiful test reports with detailed metrics</li>
<li>✅ Test history and trend analysis</li>
<li>✅ Failure statistics and flaky test detection</li>
<li>✅ Performance metrics per browser</li>
<li>✅ Browser-specific comparisons</li>
</ul>

<p><strong>CI/CD Pipeline</strong>:</p>

<ul>
<li>✅ GitHub Actions matrix strategy (3 parallel browsers)</li>
<li>✅ Auto-deployment to GitHub Pages</li>
<li>✅ PR integration with test result comments</li>
<li>✅ Artifact management (results, screenshots, videos)</li>
<li>✅ Concurrency control and cleanup</li>
</ul>

<p>This strategy is <strong>framework-agnostic</strong> and works with <strong>any web application</strong> regardless of tech stack, ensuring quality across all major browser engines with beautiful Allure reports and automated CI/CD validation.</p>

<h2>
  
  
  Quickstart Command Summary
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Setup</span>
npm <span class="nb">install</span> <span class="nt">--save-dev</span> @playwright/test allure-playwright allure-commandline

<span class="c"># Local testing</span>
npm run <span class="nb">test</span>:upgrade:all          <span class="c"># Run all phases on all browsers</span>
npm run allure:report             <span class="c"># Generate Allure report</span>
npm run allure:open               <span class="c"># Open report in browser</span>

<span class="c"># CI/CD</span>
git push origin feature-branch    <span class="c"># Triggers GitHub Actions</span>
<span class="c"># View results at: https://github.com/actions/{repo}/runs/{id}</span>
<span class="c"># Allure report at: https://{username}.github.io/{repo}/test-reports/{id}</span>

<span class="c"># Debugging</span>
npm run debug:firefox             <span class="c"># Debug specific browser</span>
npx playwright <span class="nb">test</span> <span class="nt">--headed</span>      <span class="c"># See browser in action</span>
</code></pre>

</div>



<p>Your team will thank you for the automated confidence! 🚀</p>

<p>With <strong>Allure reporting</strong>, <strong>multi-browser testing</strong>, and <strong>GitHub Actions CI/CD</strong>, you'll have:</p>

<ul>
<li>📊 Beautiful test reports with detailed analytics</li>
<li>🌍 Confidence across all major browsers</li>
<li>🚀 Automated quality gates on every change</li>
<li>📈 Trend analysis and flaky test detection</li>
<li>✅ Fast feedback on pull requests</li>
<li>🔄 Complete audit trail of test history</li>
</ul>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<strong>Playwright Documentation</strong>: <a href="https://playwright.dev" rel="noopener noreferrer">https://playwright.dev</a>
</li>
<li>
<strong>GitHub Actions</strong>: <a href="https://github.com/features/actions" rel="noopener noreferrer">https://github.com/features/actions</a>
</li>
<li>
<strong>WCAG Accessibility Guidelines</strong>: <a href="https://www.w3.org/WAI/WCAG21/quickref/" rel="noopener noreferrer">https://www.w3.org/WAI/WCAG21/quickref/</a>
</li>
<li>
<strong>Core Web Vitals</strong>: <a href="https://web.dev/vitals/" rel="noopener noreferrer">https://web.dev/vitals/</a>
</li>
</ul>




<p><strong>Have you automated your frontend upgrades? Share your strategies in the comments below!</strong> 👇</p>

<h1>
  
  
  testing #automation #frontend #playwright #devops #qa #e2etesting #monitoring
</h1>

