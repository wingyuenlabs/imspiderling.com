---
Title: End-to-End Testing with Playwright: Complete Guide with Page Object Model
Description: 
Author: Satish Reddy Budati
Date: 2026-03-15T21:31:45.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  End-to-End Testing with Playwright: Complete Guide with Page Object Model
</h1>

<p>Master production-ready E2E testing with Playwright, Page Object Model, multi-browser testing, and CI/CD integration.</p>




<h2>
  
  
  Table of Contents
</h2>

<ol>
<li>Why Playwright?</li>
<li>Project Setup</li>
<li>Page Object Model Pattern</li>
<li>Complete E-Commerce Example</li>
<li>Multi-Browser Testing</li>
<li>Parallel Execution</li>
<li>Retry &amp; Flakiness Management</li>
<li>Advanced Reporting with Allure</li>
<li>Common Mistakes &amp; Solutions</li>
<li>CI/CD Integration (GitHub Actions)</li>
<li>Best Practices</li>
<li>Running Tests</li>
</ol>




<h2>
  
  
  Why Playwright?
</h2>

<p>Playwright has become the industry standard for E2E testing. Here's why:</p>

<h3>
  
  
  ✅ Cross-Browser Testing Out of the Box
</h3>

<p>Test on <strong>Chrome, Firefox, Safari, and Edge</strong> with the same API. No more managing separate drivers.</p>

<h3>
  
  
  ✅ Fast &amp; Reliable
</h3>

<ul>
<li>
<strong>Auto-waiting</strong>: Elements automatically wait to be ready</li>
<li>
<strong>Parallel execution</strong>: Run hundreds of tests simultaneously</li>
<li>
<strong>Low overhead</strong>: Tests run in seconds</li>
<li>
<strong>Minimal flakiness</strong>: Built-in resilience reduces false failures</li>
</ul>

<h3>
  
  
  ✅ First-Class Debugging
</h3>

<ul>
<li>
<strong>Playwright Inspector</strong>: Step-by-step debugging</li>
<li>
<strong>Trace Viewer</strong>: Record and replay execution</li>
<li>
<strong>Screenshots &amp; Videos</strong>: Automatic captures on failure</li>
<li>
<strong>Locator Playground</strong>: Find elements interactively</li>
</ul>

<h3>
  
  
  ✅ Production-Ready Features
</h3>

<ul>
<li>Mobile &amp; device testing</li>
<li>Network control &amp; interception</li>
<li>Multiple language support</li>
<li>Strong community &amp; Microsoft backing</li>
</ul>




<h2>
  
  
  Project Setup
</h2>

<h3>
  
  
  Step 1: Initialize Project
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>playwright-ecommerce-tests
<span class="nb">cd </span>playwright-ecommerce-tests
npm init <span class="nt">-y</span>
</code></pre>

</div>



<h3>
  
  
  Step 2: Install Dependencies
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-D</span> @playwright/test
npm <span class="nb">install</span> <span class="nt">-D</span> @playwright/test-docker  <span class="c"># Optional: for containerized testing</span>
npm <span class="nb">install </span>dotenv                       <span class="c"># Environment variables</span>
npx playwright <span class="nb">install</span>                   <span class="c"># Install browsers</span>
</code></pre>

</div>



<h3>
  
  
  Step 3: Install Allure Reporter
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-D</span> allure-playwright
npm <span class="nb">install</span> <span class="nt">-D</span> @playwright/test
npm <span class="nb">install</span> <span class="nt">-g</span> allure-commandline
</code></pre>

</div>



<h3>
  
  
  Step 4: Project Structure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>playwright-ecommerce-tests/
├── tests/
│   ├── fixtures/
│   │   ├── base.ts              # Base page class
│   │   └── auth.ts              # Auth fixture
│   ├── pages/
│   │   ├── login-page.ts
│   │   ├── product-search-page.ts
│   │   ├── product-details-page.ts
│   │   ├── shopping-cart-page.ts
│   │   ├── checkout-page.ts
│   │   └── order-confirmation-page.ts
│   └── e2e/
│       ├── ecommerce-flow.test.ts
│       ├── auth.test.ts
│       └── cart.test.ts
├── playwright.config.ts
├── allure-results/              # Auto-generated
├── allure-report/               # Auto-generated
├── .env.example
├── .env
├── .github/
│   └── workflows/
│       └── playwright.yml
└── package.json
</code></pre>

</div>



<h3>
  
  
  Step 5: Configuration Files
</h3>

<p><strong>playwright.config.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span><span class="p">,</span> <span class="nx">devices</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">testDir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./tests/e2e</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">testMatch</span><span class="p">:</span> <span class="dl">'</span><span class="s1">**/*.test.ts</span><span class="dl">'</span><span class="p">,</span>

  <span class="c1">// ✅ Timeouts</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">30000</span> <span class="p">:</span> <span class="mi">10000</span><span class="p">,</span>
  <span class="na">expect</span><span class="p">:</span> <span class="p">{</span> <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span> <span class="p">},</span>

  <span class="c1">// ✅ Parallel &amp; Retry Configuration</span>
  <span class="na">fullyParallel</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">workers</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">4</span> <span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span>  <span class="c1">// 4 workers in CI, auto in local</span>
  <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>          <span class="c1">// Retry failed tests in CI only</span>
  <span class="na">forbidOnly</span><span class="p">:</span> <span class="o">!!</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span><span class="p">,</span>

  <span class="c1">// ✅ Reporting</span>
  <span class="na">reporter</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">html</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">outputFolder</span><span class="p">:</span> <span class="dl">'</span><span class="s1">playwright-report</span><span class="dl">'</span> <span class="p">}],</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">json</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">outputFile</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test-results/results.json</span><span class="dl">'</span> <span class="p">}],</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">junit</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">outputFile</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test-results/junit.xml</span><span class="dl">'</span> <span class="p">}],</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">list</span><span class="dl">'</span><span class="p">],</span>  <span class="c1">// Console output</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">allure-playwright</span><span class="dl">'</span><span class="p">],</span>  <span class="c1">// Allure reporting</span>
  <span class="p">],</span>

  <span class="c1">// ✅ Global settings for all projects</span>
  <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">baseURL</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BASE_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">https://demo.ecommerce.com</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">trace</span><span class="p">:</span> <span class="dl">'</span><span class="s1">on-first-retry</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">screenshot</span><span class="p">:</span> <span class="dl">'</span><span class="s1">only-on-failure</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">video</span><span class="p">:</span> <span class="dl">'</span><span class="s1">retain-on-failure</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// ✅ Browser Projects with retry override</span>
  <span class="na">projects</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">chromium</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Chrome</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">firefox</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Firefox</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">3</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>  <span class="c1">// More retries for Firefox</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">webkit</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Safari</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">3</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>  <span class="c1">// More retries for Safari</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile-chrome</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Pixel 5</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile-safari</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">iPhone 12</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">],</span>

  <span class="c1">// ✅ Web server configuration</span>
  <span class="na">webServer</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">command</span><span class="p">:</span> <span class="dl">'</span><span class="s1">npm run start</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">url</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BASE_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">https://demo.ecommerce.com</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">reuseExistingServer</span><span class="p">:</span> <span class="o">!</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span><span class="p">,</span>
    <span class="na">timeout</span><span class="p">:</span> <span class="mi">120</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span>  <span class="c1">// 2 minutes to start server</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>.env.example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>BASE_URL=https://demo.ecommerce.com
TEST_USER_EMAIL=testuser@example.com
TEST_USER_PASSWORD=SecurePassword123!
TEST_PRODUCT_NAME=Laptop
TIMEOUT=30000
ALLURE_RESULTS_DIR=./allure-results
</code></pre>

</div>



<p><strong>package.json:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright-ecommerce-tests"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"test"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:headed"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --headed"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:debug"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --debug"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:ui"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --ui"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:chrome"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=chromium"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:firefox"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=firefox"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:webkit"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=webkit"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:mobile"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --project=mobile-chrome"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:serial"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:parallel-2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=2"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:parallel-4"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=4"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:parallel-8"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=8"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:retry"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --retries=2"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:ci"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test --workers=4 --retries=2"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"report"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright show-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:generate"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:open"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure open allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:serve"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure serve allure-results"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"devDependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"@playwright/test"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^1.40.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@playwright/test-docker"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^1.40.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure-playwright"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^2.13.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"dotenv"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^16.3.1"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Page Object Model Pattern
</h2>

<h3>
  
  
  Why Page Object Model?
</h3>

<p><strong>❌ Without POM (Scattered Locators):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Login</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[name="email"]</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">user@example.com</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[name="password"]</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">password123</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button:has-text("Log In")</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForNavigation</span><span class="p">();</span>
<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Another test</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Duplicated locators!</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[name="email"]</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">user@example.com</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>✅ With POM (Clean &amp; Maintainable):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Login</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">loginPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LoginPage</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">login</span><span class="p">(</span><span class="dl">'</span><span class="s1">user@example.com</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">password123</span><span class="dl">'</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Base Page Class
</h3>

<p><strong>tests/fixtures/base.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Page</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">BasePage</span> <span class="p">{</span>
  <span class="k">readonly</span> <span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">page</span> <span class="o">=</span> <span class="nx">page</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">goto</span><span class="p">(</span><span class="nx">path</span><span class="p">:</span> <span class="kr">string</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="nx">path</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">waitForPageLoad</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">getPageTitle</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">title</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">takeScreenshot</span><span class="p">(</span><span class="nx">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="na">path</span><span class="p">:</span> <span class="s2">`screenshots/</span><span class="p">${</span><span class="nx">name</span><span class="p">}</span><span class="s2">.png`</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nx">retryAction</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="nx">action</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="nx">maxRetries</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">3</span><span class="p">,</span>
    <span class="nx">delayMs</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">1000</span>
  <span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="na">lastError</span><span class="p">:</span> <span class="nb">Error</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">await</span> <span class="nf">action</span><span class="p">();</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">lastError</span> <span class="o">=</span> <span class="nx">error</span> <span class="k">as</span> <span class="nb">Error</span><span class="p">;</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="nx">delayMs</span><span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">throw</span> <span class="nx">lastError</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Complete E-Commerce Example
</h2>

<h3>
  
  
  Login Page Object
</h3>

<p><strong>tests/pages/login-page.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Page</span><span class="p">,</span> <span class="nx">Locator</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BasePage</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../fixtures/base</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">LoginPage</span> <span class="kd">extends</span> <span class="nc">BasePage</span> <span class="p">{</span>
  <span class="k">readonly</span> <span class="nx">emailInput</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>
  <span class="k">readonly</span> <span class="nx">passwordInput</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>
  <span class="k">readonly</span> <span class="nx">loginButton</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>
  <span class="k">readonly</span> <span class="nx">errorMessage</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">super</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">emailInput</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="email"]</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">passwordInput</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="password"]</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">loginButton</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button:has-text("Log In")</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">errorMessage</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="error-message"]</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">goto</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">super</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">waitForPageLoad</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">login</span><span class="p">(</span><span class="nx">email</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">password</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">emailInput</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="nx">email</span><span class="p">);</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">passwordInput</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="nx">password</span><span class="p">);</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">loginButton</span><span class="p">.</span><span class="nf">click</span><span class="p">();</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForNavigation</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">getErrorMessage</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return </span><span class="p">(</span><span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">errorMessage</span><span class="p">.</span><span class="nf">textContent</span><span class="p">())</span> <span class="o">||</span> <span class="dl">''</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">isLoginSuccessful</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">boolean</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">url</span><span class="p">().</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">/dashboard</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Product Search Page Object
</h3>

<p><strong>tests/pages/product-search-page.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Page</span><span class="p">,</span> <span class="nx">Locator</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BasePage</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../fixtures/base</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">ProductSearchPage</span> <span class="kd">extends</span> <span class="nc">BasePage</span> <span class="p">{</span>
  <span class="k">readonly</span> <span class="nx">searchInput</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>
  <span class="k">readonly</span> <span class="nx">searchButton</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>
  <span class="k">readonly</span> <span class="nx">productList</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>
  <span class="k">readonly</span> <span class="nx">firstProduct</span><span class="p">:</span> <span class="nx">Locator</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">super</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">searchInput</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[placeholder="Search products..."]</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">searchButton</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button:has-text("Search")</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">productList</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="product-item"]</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">firstProduct</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">productList</span><span class="p">.</span><span class="nf">first</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">goto</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">super</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/products</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">waitForPageLoad</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">searchProduct</span><span class="p">(</span><span class="nx">productName</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">searchInput</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="nx">productName</span><span class="p">);</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">searchButton</span><span class="p">.</span><span class="nf">click</span><span class="p">();</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">getProductCount</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">number</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">productList</span><span class="p">.</span><span class="nf">count</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">clickFirstProduct</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">firstProduct</span><span class="p">.</span><span class="nf">click</span><span class="p">();</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">waitForPageLoad</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">hasResults</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">boolean</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return </span><span class="p">(</span><span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getProductCount</span><span class="p">())</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Complete Test Suite
</h3>

<p><strong>tests/e2e/ecommerce-flow.test.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">LoginPage</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../pages/login-page</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">ProductSearchPage</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../pages/product-search-page</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">TEST_EMAIL</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_USER_EMAIL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">testuser@example.com</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">TEST_PASSWORD</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_USER_PASSWORD</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">SecurePassword123!</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">PRODUCT_NAME</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_PRODUCT_NAME</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">Laptop</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">E-Commerce Platform</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User completes full purchase flow</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Step 1: Login</span>
    <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">User logs in</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">loginPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LoginPage</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">goto</span><span class="p">();</span>
      <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">login</span><span class="p">(</span><span class="nx">TEST_EMAIL</span><span class="p">,</span> <span class="nx">TEST_PASSWORD</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">isLoginSuccessful</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="p">});</span>

    <span class="c1">// Step 2: Search product</span>
    <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">User searches for product</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">searchPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ProductSearchPage</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
      <span class="k">await</span> <span class="nx">searchPage</span><span class="p">.</span><span class="nf">goto</span><span class="p">();</span>
      <span class="k">await</span> <span class="nx">searchPage</span><span class="p">.</span><span class="nf">searchProduct</span><span class="p">(</span><span class="nx">PRODUCT_NAME</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="k">await</span> <span class="nx">searchPage</span><span class="p">.</span><span class="nf">hasResults</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="p">});</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Login fails with invalid credentials</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">loginPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LoginPage</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">goto</span><span class="p">();</span>
    <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">login</span><span class="p">(</span><span class="dl">'</span><span class="s1">invalid@example.com</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">wrongpassword</span><span class="dl">'</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">errorMessage</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">getErrorMessage</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">errorMessage</span><span class="p">).</span><span class="nf">toContain</span><span class="p">(</span><span class="dl">'</span><span class="s1">Invalid email or password</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Multi-Browser Testing
</h2>

<h3>
  
  
  Browser Configuration
</h3>

<p>Playwright automatically runs tests across <strong>all configured browsers</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">projects</span><span class="p">:</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">chromium</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Chrome</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">firefox</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Firefox</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">webkit</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Safari</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile-chrome</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Pixel 5</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile-safari</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">iPhone 12</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
<span class="p">],</span>
</code></pre>

</div>



<h3>
  
  
  Running Specific Browsers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run on Chrome only</span>
npm run <span class="nb">test</span>:chrome

<span class="c"># Run on Firefox only</span>
npm run <span class="nb">test</span>:firefox

<span class="c"># Run on mobile</span>
npm run <span class="nb">test</span>:mobile

<span class="c"># Run on all browsers (default)</span>
npm <span class="nb">test</span>
</code></pre>

</div>



<h3>
  
  
  Browser-Specific Assertions
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Should work on all browsers</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">browserName</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Run only on specific browsers</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">browserName</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">chromium</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Chrome-specific test</span>
  <span class="p">}</span>

  <span class="c1">// Browser-specific viewport handling</span>
  <span class="kd">const</span> <span class="nx">viewport</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">viewportSize</span><span class="p">();</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">browserName</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">webkit</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">viewport</span><span class="p">?.</span><span class="nx">width</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">980</span><span class="p">);</span> <span class="c1">// Safari viewport</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Parallel Execution
</h2>

<p>Playwright runs tests in <strong>parallel by default</strong>, significantly reducing total test duration. This is one of its key advantages over sequential test execution.</p>

<h3>
  
  
  Understanding Parallel Execution
</h3>

<p><strong>How it works:</strong></p>

<ul>
<li>Tests are distributed across multiple <strong>worker processes</strong>
</li>
<li>Each worker executes tests independently</li>
<li>No shared state between workers</li>
<li>Tests complete in fraction of sequential time</li>
</ul>

<h3>
  
  
  Parallel Configuration
</h3>

<p><strong>playwright.config.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="c1">// Enable parallel execution</span>
  <span class="na">fullyParallel</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>

  <span class="c1">// Number of parallel workers (default: number of CPU cores)</span>
  <span class="na">workers</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">4</span> <span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span>

  <span class="c1">// Timeout for entire test</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">30</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span>

  <span class="c1">// Timeout for each expect assertion</span>
  <span class="na">expect</span><span class="p">:</span> <span class="p">{</span> <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span> <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Worker Configuration
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="c1">// ✅ GOOD - Unlimited workers (uses all CPU cores)</span>
  <span class="na">workers</span><span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span>  <span class="c1">// Default behavior</span>

  <span class="c1">// ✅ GOOD - Fixed number for consistency</span>
  <span class="na">workers</span><span class="p">:</span> <span class="mi">4</span><span class="p">,</span>

  <span class="c1">// ✅ GOOD - Dynamic based on environment</span>
  <span class="na">workers</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">8</span><span class="p">,</span>

  <span class="c1">// ❌ BAD - Single worker (no parallelization)</span>
  <span class="na">workers</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Running Tests in Parallel
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run with default parallelism (all CPU cores)</span>
npm <span class="nb">test</span>

<span class="c"># Run with specific number of workers</span>
npx playwright <span class="nb">test</span> <span class="nt">--workers</span><span class="o">=</span>4

<span class="c"># Run with 1 worker (sequential)</span>
npx playwright <span class="nb">test</span> <span class="nt">--workers</span><span class="o">=</span>1

<span class="c"># Run with 2 workers</span>
npx playwright <span class="nb">test</span> <span class="nt">--workers</span><span class="o">=</span>2
</code></pre>

</div>



<h3>
  
  
  Parallel Execution Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// These 4 tests run in parallel (if workers &gt;= 4)</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User login</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// ~2 seconds</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Product search</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// ~2 seconds</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/products</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Add to cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// ~2 seconds</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/cart</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Checkout flow</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// ~2 seconds</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/checkout</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">});</span>

<span class="c1">// Sequential execution: 4 tests × 2 seconds = 8 seconds</span>
<span class="c1">// Parallel execution (4 workers): All 4 tests = 2 seconds</span>
</code></pre>

</div>



<h3>
  
  
  Controlling Parallelism
</h3>

<p><strong>Run tests sequentially within a file:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nx">describe</span><span class="p">.</span><span class="nf">serial</span><span class="p">(</span><span class="dl">'</span><span class="s1">User Account</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// These tests run sequentially (in order)</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User signs up</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Runs first</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User logs in</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Runs second (after signup completes)</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User updates profile</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Runs third (after login completes)</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Mix serial and parallel tests:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">E-Commerce</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Parallel tests</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Browse products</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Search functionality</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Filter by price</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>

  <span class="nx">test</span><span class="p">.</span><span class="nx">describe</span><span class="p">.</span><span class="nf">serial</span><span class="p">(</span><span class="dl">'</span><span class="s1">Checkout Process</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Serial within this describe block</span>
    <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Add to cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>
    <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Apply coupon</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>
    <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Complete payment</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Avoiding Test Conflicts in Parallel Execution
</h3>

<p><strong>❌ Problem: Shared state causes failures</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">let</span> <span class="nx">userId</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Create user</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Test A creates user</span>
  <span class="nx">userId</span> <span class="o">=</span> <span class="mi">123</span><span class="p">;</span>
<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Update user</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Test B runs in parallel and userId is undefined!</span>
  <span class="k">await</span> <span class="nf">updateUser</span><span class="p">(</span><span class="nx">userId</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>✅ Solution: Use fixtures or beforeEach</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">test</span><span class="p">.</span><span class="nf">beforeEach</span><span class="p">(</span><span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Each test gets fresh data</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/users</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">data</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test User</span><span class="dl">'</span> <span class="p">}</span>
  <span class="p">});</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">id</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
  <span class="c1">// Available in test context</span>
<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Update user</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">context</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Each test has its own user ID</span>
  <span class="kd">const</span> <span class="nx">userId</span> <span class="o">=</span> <span class="nx">context</span><span class="p">.</span><span class="nx">userId</span><span class="p">;</span>
  <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Performance Tuning for Parallel Execution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="c1">// ✅ Reduce timeout when running parallel</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">30</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span>
  <span class="na">expect</span><span class="p">:</span> <span class="p">{</span> <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span> <span class="p">},</span>

  <span class="c1">// ✅ More workers = faster (but uses more resources)</span>
  <span class="na">workers</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span>  <span class="c1">// For CI</span>

  <span class="c1">// ✅ Retry on failure (helps with flakiness)</span>
  <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>

  <span class="c1">// ✅ Full parallelism across files and tests</span>
  <span class="na">fullyParallel</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>

  <span class="c1">// ✅ Start web server once for all workers</span>
  <span class="na">webServer</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">command</span><span class="p">:</span> <span class="dl">'</span><span class="s1">npm run start</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">http://localhost:3000</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">reuseExistingServer</span><span class="p">:</span> <span class="o">!</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Monitoring Parallel Execution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Show detailed output with worker info</span>
npx playwright <span class="nb">test</span> <span class="nt">--reporter</span><span class="o">=</span>list

<span class="c"># HTML report shows which worker ran which test</span>
npm run report

<span class="c"># Verbose mode</span>
npx playwright <span class="nb">test</span> <span class="nt">--reporter</span><span class="o">=</span>verbose

<span class="c"># Debug with trace for parallel runs</span>
npx playwright <span class="nb">test</span> <span class="nt">--trace</span><span class="o">=</span>on
</code></pre>

</div>



<h3>
  
  
  Real-World Parallel Execution Results
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Sequential Execution (1 worker):
├─ Test 1: 2.5s
├─ Test 2: 2.3s
├─ Test 3: 2.1s
├─ Test 4: 2.4s
└─ Total: 9.3 seconds ❌

Parallel Execution (4 workers):
├─ Worker 1: Test 1 (2.5s)
├─ Worker 2: Test 2 (2.3s)
├─ Worker 3: Test 3 (2.1s)
├─ Worker 4: Test 4 (2.4s)
└─ Total: 2.5 seconds ✅ (3.7x faster!)
</code></pre>

</div>






<h2>
  
  
  Retry &amp; Flakiness Management
</h2>

<h3>
  
  
  Understanding Test Flakiness
</h3>

<p><strong>Flaky tests</strong> are tests that pass and fail unpredictably. Common causes:</p>

<ul>
<li>Network timeouts</li>
<li>Race conditions</li>
<li>Slow animations</li>
<li>Database delays</li>
<li>API rate limiting</li>
<li>Timing-dependent assertions</li>
</ul>

<h3>
  
  
  Built-in Retry Mechanism
</h3>

<p><strong>playwright.config.ts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="c1">// Retry failed tests 2 times</span>
  <span class="na">retries</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">2</span> <span class="p">:</span> <span class="mi">0</span><span class="p">,</span>

  <span class="c1">// Retry with different config</span>
  <span class="na">fullyParallel</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>

  <span class="c1">// Fast fail in local development</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span> <span class="p">?</span> <span class="mi">30000</span> <span class="p">:</span> <span class="mi">10000</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  How Retries Work
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Test Run:
├─ Attempt 1: FAILED ❌
├─ Attempt 2: FAILED ❌
└─ Attempt 3: PASSED ✅ → Test marked as PASSED
</code></pre>

</div>



<p><strong>HTML Report shows:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Test: "User checkout flow"
  ❌ Attempt 1 (2.3s) - Element not found
  ❌ Attempt 2 (2.1s) - Timeout waiting for navigation
  ✅ Attempt 3 (2.5s) - PASSED
  Duration: 6.9s (including retries)
</code></pre>

</div>



<h3>
  
  
  Conditional Retries
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Only retry specific tests</span>
<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Flaky Features</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">test</span><span class="p">.</span><span class="nx">describe</span><span class="p">.</span><span class="nf">configure</span><span class="p">({</span> <span class="na">retries</span><span class="p">:</span> <span class="mi">2</span> <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Slow API endpoint</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Will retry up to 2 times</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Animation heavy page</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Will retry up to 2 times</span>
  <span class="p">});</span>
<span class="p">});</span>

<span class="c1">// No retry for other tests</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Critical stable test</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// No retries (uses default: 0)</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Manual Retry Logic with Exponential Backoff
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Page</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">RetryableAction</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">)</span> <span class="p">{}</span>

  <span class="k">async</span> <span class="nx">retryWithBackoff</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="nx">action</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="nx">maxRetries</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">3</span><span class="p">,</span>
    <span class="nx">initialDelayMs</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">100</span>
  <span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="na">lastError</span><span class="p">:</span> <span class="nb">Error</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">attempt</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">attempt</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span><span class="p">;</span> <span class="nx">attempt</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">await</span> <span class="nf">action</span><span class="p">();</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">lastError</span> <span class="o">=</span> <span class="nx">error</span> <span class="k">as</span> <span class="nb">Error</span><span class="p">;</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">attempt</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
          <span class="c1">// Exponential backoff: 100ms, 200ms, 400ms</span>
          <span class="kd">const</span> <span class="nx">delay</span> <span class="o">=</span> <span class="nx">initialDelayMs</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">pow</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="nx">attempt</span><span class="p">);</span>
          <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Attempt </span><span class="p">${</span><span class="nx">attempt</span> <span class="o">+</span> <span class="mi">1</span><span class="p">}</span><span class="s2"> failed. Retrying in </span><span class="p">${</span><span class="nx">delay</span><span class="p">}</span><span class="s2">ms...`</span><span class="p">);</span>
          <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="nx">delay</span><span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span>
      <span class="s2">`Failed after </span><span class="p">${</span><span class="nx">maxRetries</span><span class="p">}</span><span class="s2"> attempts: </span><span class="p">${</span><span class="nx">lastError</span><span class="p">?.</span><span class="nx">message</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Using Retry in Tests
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">RetryableAction</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./retryable-action</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Fetch order with retry</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">retry</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RetryableAction</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>

  <span class="kd">const</span> <span class="nx">orderId</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">retry</span><span class="p">.</span><span class="nf">retryWithBackoff</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// API call that might fail</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/orders/latest</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Order not found</span><span class="dl">'</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

    <span class="k">return</span> <span class="nx">data</span><span class="p">.</span><span class="nx">id</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="nf">expect</span><span class="p">(</span><span class="nx">orderId</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Handling Network Timeouts
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">class</span> <span class="nc">NetworkRetry</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">page</span><span class="p">:</span> <span class="nx">Page</span><span class="p">)</span> <span class="p">{}</span>

  <span class="k">async</span> <span class="nf">fetchWithRetry</span><span class="p">(</span>
    <span class="nx">url</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
    <span class="nx">maxRetries</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">3</span>
  <span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">attempt</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">attempt</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span><span class="p">;</span> <span class="nx">attempt</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">url</span><span class="p">,</span> <span class="p">{</span>
          <span class="na">timeout</span><span class="p">:</span> <span class="mi">10000</span><span class="p">,</span> <span class="c1">// 10 second timeout</span>
        <span class="p">});</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="mi">429</span><span class="p">)</span> <span class="p">{</span>
          <span class="c1">// Rate limited - wait longer</span>
          <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="mi">2000</span> <span class="o">*</span> <span class="p">(</span><span class="nx">attempt</span> <span class="o">+</span> <span class="mi">1</span><span class="p">));</span>
          <span class="k">continue</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`HTTP </span><span class="p">${</span><span class="nx">response</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">attempt</span> <span class="o">===</span> <span class="nx">maxRetries</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span> <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>

        <span class="kd">const</span> <span class="nx">backoffMs</span> <span class="o">=</span> <span class="mi">500</span> <span class="o">*</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">pow</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="nx">attempt</span><span class="p">);</span>
        <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="nx">backoffMs</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Flakiness Detection &amp; Analysis
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Detect flaky test</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Use tag for analysis</span>
  <span class="nx">test</span><span class="p">.</span><span class="nf">info</span><span class="p">().</span><span class="nx">tags</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="dl">'</span><span class="s1">@flaky-detector</span><span class="dl">'</span><span class="p">);</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="c1">// Your test here</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">getByRole</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">click</span><span class="p">();</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Log detailed info for flakiness analysis</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Test failed:</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">url</span><span class="p">:</span> <span class="nx">page</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="na">error</span><span class="p">:</span> <span class="nx">error</span> <span class="k">instanceof</span> <span class="nb">Error</span> <span class="p">?</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span> <span class="p">:</span> <span class="dl">'</span><span class="s1">Unknown error</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">screenshot</span><span class="p">:</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">screenshot</span><span class="p">({</span> <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">flaky.png</span><span class="dl">'</span> <span class="p">}),</span>
    <span class="p">});</span>

    <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Retry with Different Browser Settings
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">projects</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">chromium</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Chrome</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">firefox</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Firefox</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span>  <span class="c1">// More retries for Firefox (more flaky)</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">webkit</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Safari</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">retries</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span>  <span class="c1">// More retries for Safari</span>
    <span class="p">},</span>
  <span class="p">],</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Monitoring Retry Statistics
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run tests with detailed output</span>
npx playwright <span class="nb">test</span> <span class="nt">--reporter</span><span class="o">=</span>verbose

<span class="c"># Shows:</span>
<span class="c"># ✓ test1 (2 attempts)</span>
<span class="c"># ✓ test2 (1 attempt - passed first try)</span>
<span class="c"># ✓ test3 (3 attempts)</span>
</code></pre>

</div>



<h3>
  
  
  Best Practices for Reducing Flakiness
</h3>

<p><strong>1. Use auto-waiting instead of manual waits:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ Flaky - hardcoded wait</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="mi">1000</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// ✅ Reliable - auto-waiting</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>2. Wait for specific conditions:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ Flaky - might not be ready</span>
<span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">.message</span><span class="dl">'</span><span class="p">).</span><span class="nf">textContent</span><span class="p">();</span>

<span class="c1">// ✅ Reliable - wait for visibility</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">.message</span><span class="dl">'</span><span class="p">).</span><span class="nf">waitFor</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">.message</span><span class="dl">'</span><span class="p">).</span><span class="nf">textContent</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>3. Use network waits for API calls:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ Flaky - API might be slow</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/products</span><span class="dl">'</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="mi">500</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">count</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-product]</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>

<span class="c1">// ✅ Reliable - wait for network</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/products</span><span class="dl">'</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">count</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-product]</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">();</span>
</code></pre>

</div>



<h3>
  
  
  Retry Strategy Summary
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Scenario</th>
<th>Retry Count</th>
<th>Reason</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>CI Environment</strong></td>
<td>2-3</td>
<td>Network delays, resource constraints</td>
</tr>
<tr>
<td><strong>Flaky Endpoints</strong></td>
<td>3</td>
<td>API timeouts, rate limiting</td>
</tr>
<tr>
<td><strong>Mobile Tests</strong></td>
<td>2</td>
<td>Slower devices, animations</td>
</tr>
<tr>
<td><strong>Local Development</strong></td>
<td>0</td>
<td>Immediate feedback</td>
</tr>
<tr>
<td><strong>Critical Tests</strong></td>
<td>0</td>
<td>Must be reliable, no masking</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Advanced Reporting with Allure
</h2>

<h3>
  
  
  Generate Allure Reports
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Generate HTML report from results</span>
npm run allure:generate

<span class="c"># Open in browser</span>
npm run allure:open

<span class="c"># Serve live report</span>
npm run allure:serve
</code></pre>

</div>



<h3>
  
  
  Custom Allure Configuration
</h3>

<p><strong>allure.properties (create in root):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="py">allure.results.directory</span><span class="p">=</span><span class="s">allure-results</span>
<span class="py">allure.report.directory</span><span class="p">=</span><span class="s">allure-report</span>
</code></pre>

</div>



<h3>
  
  
  Allure Decorators in Tests
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">allure</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">allure-playwright</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Add product to cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">allure</span><span class="p">.</span><span class="nf">label</span><span class="p">(</span><span class="dl">'</span><span class="s1">feature</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Shopping Cart</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">allure</span><span class="p">.</span><span class="nf">label</span><span class="p">(</span><span class="dl">'</span><span class="s1">severity</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">critical</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">allure</span><span class="p">.</span><span class="nf">label</span><span class="p">(</span><span class="dl">'</span><span class="s1">owner</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">QA Team</span><span class="dl">'</span><span class="p">);</span>

  <span class="k">await</span> <span class="nx">allure</span><span class="p">.</span><span class="nf">link</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://jira.example.com/PROJ-123</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">JIRA</span><span class="dl">'</span><span class="p">);</span>

  <span class="c1">// Test code...</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Allure Report Features
</h3>

<ul>
<li><strong>Test execution timeline</strong></li>
<li><strong>Failure analysis with trends</strong></li>
<li><strong>Historical data tracking</strong></li>
<li><strong>Environment configuration</strong></li>
<li><strong>Test categorization by features</strong></li>
<li>
<strong>Attachment support</strong> (screenshots, videos)</li>
</ul>




<h2>
  
  
  Common Mistakes &amp; Solutions
</h2>

<h3>
  
  
  ❌ Mistake #1: Hard Waits (Using waitForTimeout)
</h3>

<p><strong>Problem:</strong> Tests become flaky and slow<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Waits full 2 seconds regardless</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="mi">2000</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Use Playwright's built-in waiting<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Waits for element to be ready</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">waitFor</span><span class="p">();</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// ✅ BETTER - Auto-waiting (built-in)</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span> <span class="c1">// Automatically waits!</span>

<span class="c1">// ✅ BEST - Wait for specific state</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForLoadState</span><span class="p">(</span><span class="dl">'</span><span class="s1">networkidle</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #2: Brittle Selectors
</h3>

<p><strong>Problem:</strong> Tests break when UI changes<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Index-based selector</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">).</span><span class="nf">nth</span><span class="p">(</span><span class="mi">2</span><span class="p">).</span><span class="nf">click</span><span class="p">();</span>

<span class="c1">// ❌ BAD - CSS selectors are fragile</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">div &gt; div &gt; button</span><span class="dl">'</span><span class="p">).</span><span class="nf">click</span><span class="p">();</span>

<span class="c1">// ❌ BAD - Xpath is hard to maintain</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">//button[@class="btn"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">click</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Use semantic locators<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - By role (most resilient)</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">getByRole</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Add to Cart</span><span class="dl">'</span> <span class="p">}).</span><span class="nf">click</span><span class="p">();</span>

<span class="c1">// ✅ GOOD - By test ID</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">getByTestId</span><span class="p">(</span><span class="dl">'</span><span class="s1">add-to-cart-button</span><span class="dl">'</span><span class="p">).</span><span class="nf">click</span><span class="p">();</span>

<span class="c1">// ✅ GOOD - By label</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">getByLabel</span><span class="p">(</span><span class="dl">'</span><span class="s1">Email Address</span><span class="dl">'</span><span class="p">).</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">test@example.com</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// ✅ GOOD - By placeholder</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">getByPlaceholder</span><span class="p">(</span><span class="dl">'</span><span class="s1">Search...</span><span class="dl">'</span><span class="p">).</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">laptop</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #3: Not Handling Async Operations
</h3>

<p><strong>Problem:</strong> Tests fail due to race conditions<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Doesn't wait for navigation</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">Checkout</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">orderNum</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="order-number"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">textContent</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Wait for expected state changes<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Wait for navigation</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">Checkout</span><span class="dl">'</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">waitForNavigation</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">orderNum</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="order-number"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">textContent</span><span class="p">();</span>

<span class="c1">// ✅ BETTER - Wait for specific element</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">Checkout</span><span class="dl">'</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="order-number"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">waitFor</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">orderNum</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="order-number"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">textContent</span><span class="p">();</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #4: Not Cleaning Up Test Data
</h3>

<p><strong>Problem:</strong> Test data accumulates; tests interfere with each other<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - No cleanup</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Create user</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[name="email"]</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">test@example.com</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// User left in database!</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Clean up in afterEach<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Cleanup</span>
<span class="nx">test</span><span class="p">.</span><span class="nf">afterEach</span><span class="p">(</span><span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Delete created test data</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="k">delete</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/users/test@example.com</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// Clear local storage</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">context</span><span class="p">().</span><span class="nf">clearCookies</span><span class="p">();</span>
<span class="p">});</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #5: Vague Test Names &amp; Assertions
</h3>

<p><strong>Problem:</strong> Hard to understand what failed<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Unclear test name</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">test1</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Descriptive names &amp; messages<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Clear test name and assertion message</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User can add laptop to cart with quantity 2</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// ... test code</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">cartTotal</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="nx">expectedPrice</span><span class="p">);</span>
<span class="p">});</span>

<span class="c1">// ✅ BETTER - With message</span>
<span class="nf">expect</span><span class="p">(</span><span class="nx">cartTotal</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="nx">expectedPrice</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Total should reflect 2 laptops at $999 each</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #6: Not Using Test Fixtures
</h3>

<p><strong>Problem:</strong> Duplicated setup code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Auth code repeated in every test</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">View cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="email"]</span><span class="dl">'</span><span class="p">,</span> <span class="nx">TEST_EMAIL</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="dl">'</span><span class="s1">input[type="password"]</span><span class="dl">'</span><span class="p">,</span> <span class="nx">TEST_PASSWORD</span><span class="p">);</span>
  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// Now test actual feature</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Use fixtures for setup<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Fixture handles auth</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">test</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../fixtures/auth</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">View cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">authenticatedPage</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Already logged in!</span>
  <span class="k">await</span> <span class="nx">authenticatedPage</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="dl">'</span><span class="s1">/cart</span><span class="dl">'</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #7: Ignoring Mobile Testing
</h3>

<p><strong>Problem:</strong> App works on desktop but breaks on mobile<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Only desktop testing</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Checkout flow</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Only runs on desktop browsers</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Include mobile in configuration<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Runs on desktop AND mobile</span>
<span class="nx">projects</span><span class="p">:</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">chromium</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Desktop Chrome</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile-chrome</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">Pixel 5</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mobile-safari</span><span class="dl">'</span><span class="p">,</span> <span class="na">use</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">devices</span><span class="p">[</span><span class="dl">'</span><span class="s1">iPhone 12</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span> <span class="p">},</span>
<span class="p">],</span>

<span class="c1">// ✅ MOBILE-SPECIFIC TESTS</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Mobile checkout</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">isMobile</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">isMobile</span><span class="p">)</span> <span class="nx">test</span><span class="p">.</span><span class="nf">skip</span><span class="p">();</span>

  <span class="c1">// Mobile-specific assertions</span>
  <span class="kd">const</span> <span class="nx">viewport</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">viewportSize</span><span class="p">();</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">viewport</span><span class="p">?.</span><span class="nx">width</span><span class="p">).</span><span class="nf">toBeLessThan</span><span class="p">(</span><span class="mi">600</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #8: Not Using test.step() for Organization
</h3>

<p><strong>Problem:</strong> Long tests are hard to debug<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - No logical steps</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Full purchase</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// 20+ lines of code</span>
  <span class="c1">// Hard to find where it failed</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Use test.step()<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Clear step breakdown</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Full purchase</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">User logs in</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Login code</span>
  <span class="p">});</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">User searches for product</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Search code</span>
  <span class="p">});</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">User adds to cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Cart code</span>
  <span class="p">});</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">User completes checkout</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Checkout code</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #9: Not Retrying Flaky Operations
</h3>

<p><strong>Problem:</strong> Intermittent failures fail the entire test suite<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - No retry logic</span>
<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/orders</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// Fails if API is slow</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Add retry logic<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Retry with exponential backoff</span>
<span class="k">async</span> <span class="nx">retryAction</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span>
  <span class="nx">action</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="nx">maxRetries</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">3</span><span class="p">,</span>
  <span class="nx">delayMs</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">1000</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">maxRetries</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">try</span> <span class="p">{</span>
      <span class="k">return</span> <span class="k">await</span> <span class="nf">action</span><span class="p">();</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">i</span> <span class="o">===</span> <span class="nx">maxRetries</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span> <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
      <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">waitForTimeout</span><span class="p">(</span><span class="nx">delayMs</span> <span class="o">*</span> <span class="p">(</span><span class="nx">i</span> <span class="o">+</span> <span class="mi">1</span><span class="p">));</span> <span class="c1">// Exponential backoff</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Max retries exceeded</span><span class="dl">'</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>






<h3>
  
  
  ❌ Mistake #10: Not Validating Both UI and Data
</h3>

<p><strong>Problem:</strong> UI passes but backend data is wrong<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ BAD - Only UI validation</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Order created</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">expect</span><span class="p">(</span><span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="success"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">isVisible</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
  <span class="c1">// What about the database?</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Solution:</strong> Validate UI and backend<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ✅ GOOD - Validate both</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Order created</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// UI validation</span>
  <span class="nf">expect</span><span class="p">(</span><span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[data-testid="success"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">isVisible</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>

  <span class="c1">// API/Database validation</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/orders</span><span class="dl">'</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">orders</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">orders</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">orders</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">status</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">pending</span><span class="dl">'</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  CI/CD Integration (GitHub Actions)
</h2>

<h3>
  
  
  GitHub Actions Workflow
</h3>

<p>Create <code>.github/workflows/playwright.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">🎭 Playwright Tests</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span> <span class="nv">main</span><span class="pi">,</span> <span class="nv">develop</span> <span class="pi">]</span>
  <span class="na">pull_request</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span> <span class="nv">main</span> <span class="pi">]</span>
  <span class="na">schedule</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0</span><span class="nv"> </span><span class="s">2</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*'</span>  <span class="c1"># Run daily at 2 AM</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">test</span><span class="pi">:</span>
    <span class="na">timeout-minutes</span><span class="pi">:</span> <span class="m">60</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>

    <span class="na">strategy</span><span class="pi">:</span>
      <span class="na">fail-fast</span><span class="pi">:</span> <span class="kc">false</span>
      <span class="na">matrix</span><span class="pi">:</span>
        <span class="na">browser</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">chromium</span><span class="pi">,</span> <span class="nv">firefox</span><span class="pi">,</span> <span class="nv">webkit</span><span class="pi">]</span>
        <span class="na">node-version</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">18.x</span><span class="pi">,</span> <span class="nv">20.x</span><span class="pi">]</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📥 Check out code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📦 Set up Node.js ${{ matrix.node-version }}</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="s">${{ matrix.node-version }}</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📚 Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">🎭 Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">🚀 Run Playwright tests</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm test -- --project=${{ matrix.browser }} --workers=4</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">BASE_URL</span><span class="pi">:</span> <span class="s">${{ secrets.BASE_URL }}</span>
          <span class="na">TEST_USER_EMAIL</span><span class="pi">:</span> <span class="s">${{ secrets.TEST_USER_EMAIL }}</span>
          <span class="na">TEST_USER_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.TEST_USER_PASSWORD }}</span>
          <span class="na">CI</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📊 Generate Allure report</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm run allure:generate</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📤 Upload HTML report</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">playwright-report-${{ matrix.browser }}-${{ matrix.node-version }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">playwright-report/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📤 Upload Allure report</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-report-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-report/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">📤 Upload test results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">test-results-${{ matrix.browser }}</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">test-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">💬 Comment PR with results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always() &amp;&amp; github.event_name == 'pull_request'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/github-script@v7</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">const fs = require('fs');</span>
            <span class="s">const testResults = JSON.parse(fs.readFileSync('test-results/results.json', 'utf8'));</span>
            <span class="s">const comment = `</span>
            <span class="s">## 🎭 Playwright Test Results - ${{ matrix.browser }}</span>

            <span class="s">- ✅ Passed: ${testResults.stats.expected}</span>
            <span class="s">- ❌ Failed: ${testResults.stats.unexpected}</span>
            <span class="s">- ⏭️ Skipped: ${testResults.stats.skipped}</span>
            <span class="s">- ⏱️ Duration: ${(testResults.stats.duration / 1000).toFixed(2)}s</span>
            <span class="s">`;</span>
            <span class="s">github.rest.issues.createComment({</span>
              <span class="s">issue_number: context.issue.number,</span>
              <span class="s">owner: context.repo.owner,</span>
              <span class="s">repo: context.repo.repo,</span>
              <span class="s">body: comment</span>
            <span class="s">});</span>
</code></pre>

</div>



<h3>
  
  
  GitHub Secrets Setup
</h3>

<p>Add these secrets in <strong>Settings → Secrets and variables → Actions</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="py">BASE_URL</span><span class="p">=</span><span class="s">https://demo.ecommerce.com</span>
<span class="py">TEST_USER_EMAIL</span><span class="p">=</span><span class="s">testuser@example.com</span>
<span class="py">TEST_USER_PASSWORD</span><span class="p">=</span><span class="s">SecurePassword123!</span>
</code></pre>

</div>



<h3>
  
  
  Viewing Results
</h3>

<ol>
<li>Go to <strong>Actions</strong> tab</li>
<li>Click the workflow run</li>
<li>Download artifacts:

<ul>
<li>
<code>playwright-report</code> - HTML test report</li>
<li>
<code>allure-report</code> - Allure report</li>
<li>
<code>test-results</code> - Raw JSON results</li>
</ul>
</li>
</ol>




<h2>
  
  
  Best Practices
</h2>

<h3>
  
  
  1. Use Semantic Locators (Priority Order)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// 1️⃣ BEST - By role (accessibility-driven)</span>
<span class="nx">page</span><span class="p">.</span><span class="nf">getByRole</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Submit</span><span class="dl">'</span> <span class="p">});</span>

<span class="c1">// 2️⃣ GOOD - By test ID</span>
<span class="nx">page</span><span class="p">.</span><span class="nf">getByTestId</span><span class="p">(</span><span class="dl">'</span><span class="s1">submit-button</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// 3️⃣ GOOD - By label</span>
<span class="nx">page</span><span class="p">.</span><span class="nf">getByLabel</span><span class="p">(</span><span class="dl">'</span><span class="s1">Email</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// 4️⃣ ACCEPTABLE - By placeholder</span>
<span class="nx">page</span><span class="p">.</span><span class="nf">getByPlaceholder</span><span class="p">(</span><span class="dl">'</span><span class="s1">Enter email</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// 5️⃣ LAST RESORT - CSS selector</span>
<span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">css=button.submit</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  2. Organize Tests by Feature
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>tests/e2e/
├── auth/
│   ├── login.test.ts
│   ├── logout.test.ts
│   └── password-reset.test.ts
├── products/
│   ├── search.test.ts
│   ├── details.test.ts
│   └── reviews.test.ts
├── checkout/
│   ├── cart.test.ts
│   ├── shipping.test.ts
│   └── payment.test.ts
└── orders/
    ├── create.test.ts
    ├── tracking.test.ts
    └── returns.test.ts
</code></pre>

</div>



<h3>
  
  
  3. Use Fixtures for Common Setup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// tests/fixtures/auth.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">test</span> <span class="k">as</span> <span class="nx">base</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">LoginPage</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../pages/login-page</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">test</span> <span class="o">=</span> <span class="nx">base</span><span class="p">.</span><span class="nf">extend</span><span class="p">({</span>
  <span class="na">authenticatedPage</span><span class="p">:</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">},</span> <span class="nx">use</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">loginPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LoginPage</span><span class="p">(</span><span class="nx">page</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">goto</span><span class="p">();</span>
    <span class="k">await</span> <span class="nx">loginPage</span><span class="p">.</span><span class="nf">login</span><span class="p">(</span>
      <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_USER_EMAIL</span> <span class="o">||</span> <span class="dl">''</span><span class="p">,</span>
      <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_USER_PASSWORD</span> <span class="o">||</span> <span class="dl">''</span>
    <span class="p">);</span>
    <span class="k">await</span> <span class="nf">use</span><span class="p">();</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="k">export</span> <span class="p">{</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  4. Environment-Specific Configuration
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Load environment variables</span>
<span class="k">import</span> <span class="nx">dotenv</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">dotenv</span><span class="dl">'</span><span class="p">;</span>
<span class="nx">dotenv</span><span class="p">.</span><span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">getBaseURL</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CI</span><span class="p">)</span> <span class="k">return</span> <span class="dl">'</span><span class="s1">https://staging.example.com</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">return</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BASE_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">https://demo.ecommerce.com</span><span class="dl">'</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">use</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">baseURL</span><span class="p">:</span> <span class="nf">getBaseURL</span><span class="p">(),</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  5. Meaningful Test Names
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ❌ Unclear</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">test1</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>

<span class="c1">// ✅ Clear &amp; descriptive</span>
<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">User can add product with custom quantity to cart and see updated total</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{});</span>
</code></pre>

</div>






<h2>
  
  
  Running Tests
</h2>

<h3>
  
  
  Basic Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run all tests</span>
npm <span class="nb">test</span>

<span class="c"># Run with browser visible</span>
npm run <span class="nb">test</span>:headed

<span class="c"># Debug mode (step-by-step)</span>
npm run <span class="nb">test</span>:debug

<span class="c"># Interactive UI mode</span>
npm run <span class="nb">test</span>:ui

<span class="c"># Run specific browser</span>
npm run <span class="nb">test</span>:chrome

<span class="c"># Run specific file</span>
npx playwright <span class="nb">test </span>tests/e2e/auth.test.ts

<span class="c"># Run tests matching pattern</span>
npx playwright <span class="nb">test</span> <span class="nt">-g</span> <span class="s2">"login"</span>
</code></pre>

</div>



<h3>
  
  
  Parallel Execution Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run with default parallel workers (uses all CPU cores)</span>
npm <span class="nb">test</span>

<span class="c"># Run with 2 parallel workers</span>
npm run <span class="nb">test</span>:parallel-2

<span class="c"># Run with 4 parallel workers (recommended for CI)</span>
npm run <span class="nb">test</span>:parallel-4

<span class="c"># Run with 8 parallel workers (for large test suites)</span>
npm run <span class="nb">test</span>:parallel-8

<span class="c"># Run sequentially (1 worker - no parallelism)</span>
npm run <span class="nb">test</span>:serial
</code></pre>

</div>



<h3>
  
  
  Retry Configuration Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run with default retries (0 local, 2 in CI)</span>
npm <span class="nb">test</span>

<span class="c"># Force retries (retry failed tests 2 times)</span>
npm run <span class="nb">test</span>:retry

<span class="c"># Run with specific retry count</span>
npx playwright <span class="nb">test</span> <span class="nt">--retries</span><span class="o">=</span>3

<span class="c"># No retries (fail fast)</span>
npx playwright <span class="nb">test</span> <span class="nt">--retries</span><span class="o">=</span>0
</code></pre>

</div>



<h3>
  
  
  CI/CD Command
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run with 4 workers + 2 retries (production)</span>
npm run <span class="nb">test</span>:ci
</code></pre>

</div>



<h3>
  
  
  Advanced Combinations
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Parallel + retries + headed</span>
npx playwright <span class="nb">test</span> <span class="nt">--workers</span><span class="o">=</span>4 <span class="nt">--retries</span><span class="o">=</span>2 <span class="nt">--headed</span>

<span class="c"># Parallel + retries + specific browser</span>
npx playwright <span class="nb">test</span> <span class="nt">--project</span><span class="o">=</span>chromium <span class="nt">--workers</span><span class="o">=</span>4 <span class="nt">--retries</span><span class="o">=</span>2

<span class="c"># Parallel + retries + pattern matching</span>
npx playwright <span class="nb">test</span> <span class="nt">-g</span> <span class="s2">"checkout"</span> <span class="nt">--workers</span><span class="o">=</span>4 <span class="nt">--retries</span><span class="o">=</span>2

<span class="c"># Debug with trace on all workers</span>
npx playwright <span class="nb">test</span> <span class="nt">--trace</span><span class="o">=</span>on <span class="nt">--workers</span><span class="o">=</span>4
</code></pre>

</div>



<h3>
  
  
  Reporting Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># View HTML report</span>
npm run report

<span class="c"># View Allure report</span>
npm run allure:open

<span class="c"># Serve Allure report</span>
npm run allure:serve

<span class="c"># Generate Allure report</span>
npm run allure:generate
</code></pre>

</div>



<h3>
  
  
  Performance Monitoring
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Show test execution with timings</span>
npx playwright <span class="nb">test</span> <span class="nt">--reporter</span><span class="o">=</span>verbose

<span class="c"># Show test execution as list</span>
npx playwright <span class="nb">test</span> <span class="nt">--reporter</span><span class="o">=</span>list

<span class="c"># Show test execution as table</span>
npx playwright <span class="nb">test</span> <span class="nt">--reporter</span><span class="o">=</span>table
</code></pre>

</div>



<h3>
  
  
  Example: Production Test Run
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># production-test.sh</span>

<span class="c"># Parallel execution with retries and reporting</span>
<span class="nb">echo</span> <span class="s2">"🚀 Starting production test suite..."</span>

npm run <span class="nb">test</span>:ci <span class="se">\</span>
  <span class="nt">--reporter</span><span class="o">=</span>html <span class="se">\</span>
  <span class="nt">--reporter</span><span class="o">=</span>json <span class="se">\</span>
  <span class="nt">--reporter</span><span class="o">=</span>allure-playwright

<span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"✅ All tests passed!"</span>
  npm run allure:serve
<span class="k">else
  </span><span class="nb">echo</span> <span class="s2">"❌ Tests failed. Review report:"</span>
  npm run report
<span class="k">fi</span>
</code></pre>

</div>



<h3>
  
  
  Performance Comparison
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Running 100 tests:

Sequential (1 worker):
  Total time: 5 minutes 0 seconds

Parallel (4 workers):
  Total time: 1 minute 15 seconds
  Speed up: 4x faster ✅

Parallel (8 workers):
  Total time: 45 seconds
  Speed up: 6.7x faster ✅

With retries (2 retries × 4 workers):
  Initial pass: 1 min 15 sec
  Retries (5 flaky tests): 15 sec
  Total: 1 min 30 sec (better than sequential!)
</code></pre>

</div>






<h2>
  
  
  Conclusion
</h2>

<h3>
  
  
  Key Takeaways
</h3>

<p>✅ <strong>Playwright is production-ready</strong> - Fast, reliable, multi-browser support<br>
✅ <strong>Page Object Model keeps tests maintainable</strong> - DRY, reusable, scalable<br>
✅ <strong>Multi-browser testing catches real bugs</strong> - Desktop, mobile, all browsers<br>
✅ <strong>Allure reporting provides visibility</strong> - Trends, detailed analysis, history<br>
✅ <strong>Avoid common mistakes</strong> - Hard waits, brittle selectors, missing cleanup<br>
✅ <strong>CI/CD integration is seamless</strong> - GitHub Actions, Jenkins, GitLab</p>

<h3>
  
  
  Next Steps
</h3>

<ol>
<li>
<strong>Start simple</strong> - Write a login test</li>
<li>
<strong>Expand coverage</strong> - Add critical user flows</li>
<li>
<strong>Integrate CI/CD</strong> - Set up GitHub Actions</li>
<li>
<strong>Monitor quality</strong> - Use Allure for insights</li>
<li>
<strong>Iterate</strong> - Continuously improve tests</li>
</ol>

<h3>
  
  
  Resources
</h3>

<ul>
<li>📖 <a href="https://playwright.dev" rel="noopener noreferrer">Playwright Docs</a>
</li>
<li>🎓 <a href="https://playwright.dev/docs/best-practices" rel="noopener noreferrer">Best Practices</a>
</li>
<li>🐛 <a href="https://playwright.dev/docs/debug" rel="noopener noreferrer">Debugging Guide</a>
</li>
<li>📊 <a href="https://docs.qameta.io/allure/" rel="noopener noreferrer">Allure Reporting</a>
</li>
<li>👥 <a href="https://github.com/microsoft/playwright" rel="noopener noreferrer">GitHub Community</a>
</li>
</ul>




<p><strong>Happy Testing! 🎭🚀</strong></p>




<h2>
  
  
  Quick Reference Card
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>npm test</code></td>
<td>Run all tests</td>
</tr>
<tr>
<td><code>npm run test:headed</code></td>
<td>See browser</td>
</tr>
<tr>
<td><code>npm run test:debug</code></td>
<td>Debug step-by-step</td>
</tr>
<tr>
<td><code>npm run test:ui</code></td>
<td>Interactive mode</td>
</tr>
<tr>
<td><code>npm run report</code></td>
<td>View HTML report</td>
</tr>
<tr>
<td><code>npm run allure:open</code></td>
<td>View Allure report</td>
</tr>
<tr>
<td><code>npm run test:chrome</code></td>
<td>Test on Chrome</td>
</tr>
<tr>
<td><code>npm run test:mobile</code></td>
<td>Test on mobile</td>
</tr>
</tbody>
</table></div>




<p><em>This guide covers Playwright with Page Object Model, multi-browser testing, Allure reporting, common mistakes with solutions, and CI/CD integration. Perfect for both beginners and experienced QA engineers.</em></p>

