---
Title: # How to Automate a Real-World Container Charter Platform with Cypress: Best Practices for QA…
Description: 
Author: mohamed Said Ibrahim
Date: 2025-10-02T22:11:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>How to Automate a Real-World Container Charter Platform with Cypress: Best Practices for QA Engineers</p>

<p><em>Tags</em>: Cypress, End-to-End Testing, QA Automation, Container Charter Platform, Dashboard Testing, JavaScript, CI/CD, Test Best Practices</p>




<h1>
  
  
  Introduction
</h1>

<p>This article provides a complete guide to automating a <strong>container charter platform</strong> using <strong>Cypress</strong>, a leading JavaScript end-to-end testing framework. The platform includes a shipping logistics dashboard with features like route tracking, dynamic pricing, statistical charts, and port listings. We’ll explore how to build a scalable, maintainable Cypress testing suite using best practices, including POM, API mocking, custom commands, and CI integration.</p>




<h1>
  
  
  Why Cypress Is Ideal for This Platform
</h1>

<p>Cypress is perfectly suited for modern web apps because:</p>

<ul>
<li>It runs directly in the browser, providing native access to elements.
</li>
<li>It supports real-time reloads and instant feedback.
</li>
<li>It offers powerful tools for network stubbing and mocking.
</li>
<li>It integrates smoothly with CI pipelines.</li>
</ul>

<p>For a platform reliant on real-time data and API-driven UIs like this container charter system, Cypress ensures robustness, speed, and maintainability.</p>




<h1>
  
  
  Understanding the Platform
</h1>

<p>Key modules to test:</p>

<p><strong>Routes</strong>: Origin–destination containers, ETA predictions, live route mapping.<br><br>
<strong>Prices</strong>: Dynamic price updates per container line or location.<br><br>
<strong>Statistics</strong>: Charts showing shipping trends, volume, and delays.<br><br>
<strong>Ports</strong>: Real-time active port listings with filtering options.</p>


<h2>
  
  
  Cypress Setup and Project Structure
</h2>

<p>Install Cypress:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm init <span class="nt">-y</span>  
npm <span class="nb">install </span>cypress <span class="nt">--save-dev</span>  
npx cypress open  
</code></pre>

</div>



<p>Recommended directory structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cypress/  
├── fixtures/ # Test data (e.g., mock responses)  
├── integration/  
│ └── dashboard/ # Test files for dashboard modules  
├── pages/ # Page Object files (POM pattern)  
├── support/  
│ ├── commands.js # Custom Cypress commands  
│ └── index.js  
cypress.config.js  
</code></pre>

</div>






<h1>
  
  
  Using Page Object Model (POM)
</h1>

<p>Encapsulate dashboard selectors and actions in <code>pages/DashboardPage.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">class</span> <span class="nc">DashboardPage</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nf">visit</span><span class="p">()</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nx">cy</span><span class="p">.</span><span class="nf">visit</span><span class="p">(</span><span class="dl">'</span><span class="s1">/dashboard</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">}</span>  
<span class="err"> </span> <span class="nf">getRouteInfo</span><span class="p">()</span> <span class="p">{</span>  
<span class="err"> </span> <span class="k">return</span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">[data-testid="route-info"</span><span class="se">\</span><span class="s1">]</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">}</span>  
<span class="err"> </span> <span class="nf">getPriceSection</span><span class="p">()</span> <span class="p">{</span>  
<span class="err"> </span> <span class="k">return</span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">[data-testid="price-section"</span><span class="se">\</span><span class="s1">]</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">}</span>  
<span class="err"> </span> <span class="nf">getStatsChart</span><span class="p">()</span> <span class="p">{</span>  
<span class="err"> </span> <span class="k">return</span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">[data-testid="stats-chart"</span><span class="se">\</span><span class="s1">]</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">}</span>  
<span class="err"> </span> <span class="nf">getPortList</span><span class="p">()</span> <span class="p">{</span>  
<span class="err"> </span> <span class="k">return</span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">[data-testid="port-list"</span><span class="se">\</span><span class="s1">] li</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">}</span>  
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">new</span> <span class="nc">DashboardPage</span><span class="p">();</span>  
</code></pre>

</div>






<h1>
  
  
  Writing Clean Functional Tests
</h1>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">DashboardPage</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../../pages/DashboardPage</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Dashboard UI Tests</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nx">cy</span><span class="p">.</span><span class="nf">loginAs</span><span class="p">(</span><span class="dl">'</span><span class="s1">admin</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="nx">DashboardPage</span><span class="p">.</span><span class="nf">visit</span><span class="p">();</span>  
<span class="err"> </span> <span class="p">});</span>

<span class="err"> </span> <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">Displays correct route information</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nx">DashboardPage</span><span class="p">.</span><span class="nf">getRouteInfo</span><span class="p">().</span><span class="nf">should</span><span class="p">(</span><span class="dl">'</span><span class="s1">contain</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Hamburg</span><span class="dl">'</span><span class="p">).</span><span class="nf">and</span><span class="p">(</span><span class="dl">'</span><span class="s1">contain</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Shanghai</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">});</span>

<span class="err"> </span> <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">Displays valid pricing data</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nx">DashboardPage</span><span class="p">.</span><span class="nf">getPriceSection</span><span class="p">().</span><span class="nf">invoke</span><span class="p">(</span><span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">).</span><span class="nf">then</span><span class="p">(</span><span class="nx">price</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nf">expect</span><span class="p">(</span><span class="nf">parseFloat</span><span class="p">(</span><span class="nx">price</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="dl">'</span><span class="s1">€</span><span class="dl">'</span><span class="p">,</span> <span class="dl">''</span><span class="p">))).</span><span class="nx">to</span><span class="p">.</span><span class="nx">be</span><span class="p">.</span><span class="nf">gt</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">});</span>  
<span class="err"> </span> <span class="p">});</span>

<span class="err"> </span> <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">Renders the statistics chart</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nx">DashboardPage</span><span class="p">.</span><span class="nf">getStatsChart</span><span class="p">().</span><span class="nf">should</span><span class="p">(</span><span class="dl">'</span><span class="s1">be.visible</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">});</span>

<span class="err"> </span> <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">Shows a list of available ports</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="nx">DashboardPage</span><span class="p">.</span><span class="nf">getPortList</span><span class="p">().</span><span class="nf">should</span><span class="p">(</span><span class="dl">'</span><span class="s1">have.length.greaterThan</span><span class="dl">'</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>  
<span class="err"> </span> <span class="p">});</span>  
<span class="p">});</span>  
</code></pre>

</div>






<h1>
  
  
  Mocking APIs for Speed &amp; Reliability
</h1>

<p>Avoid flaky tests and gain speed using <code>cy.intercept()\</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">cy</span><span class="p">.</span><span class="nf">intercept</span><span class="p">(</span><span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">/api/dashboard/prices</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>  
<span class="err"> </span> <span class="na">statusCode</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span>  
<span class="err"> </span> <span class="na">body</span><span class="p">:</span> <span class="p">{</span> <span class="na">currentPrice</span><span class="p">:</span> <span class="mi">1800</span> <span class="p">}</span>  
<span class="p">}).</span><span class="k">as</span><span class="p">(</span><span class="dl">'</span><span class="s1">mockPrice</span><span class="dl">'</span><span class="p">);</span>  
</code></pre>

</div>



<p>Use this before tests that rely on <code>/prices\</code> data.</p>




<h1>
  
  
  Custom Cypress Commands
</h1>

<p>Define reusable login or setup tasks in <code>support/commands.js\</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">Cypress</span><span class="p">.</span><span class="nx">Commands</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="dl">'</span><span class="s1">loginAs</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">role</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>  
<span class="err"> </span> <span class="kd">const</span> <span class="nx">users</span> <span class="o">=</span> <span class="p">{</span>  
<span class="err"> </span> <span class="na">admin</span><span class="p">:</span> <span class="p">{</span> <span class="na">username</span><span class="p">:</span> <span class="dl">'</span><span class="s1">admin</span><span class="dl">'</span><span class="p">,</span> <span class="na">password</span><span class="p">:</span> <span class="dl">'</span><span class="s1">admin123</span><span class="dl">'</span> <span class="p">},</span>  
<span class="err"> </span> <span class="na">viewer</span><span class="p">:</span> <span class="p">{</span> <span class="na">username</span><span class="p">:</span> <span class="dl">'</span><span class="s1">viewer</span><span class="dl">'</span><span class="p">,</span> <span class="na">password</span><span class="p">:</span> <span class="dl">'</span><span class="s1">viewer123</span><span class="dl">'</span> <span class="p">}</span>  
<span class="err"> </span> <span class="p">};</span>

<span class="err"> </span> <span class="nx">cy</span><span class="p">.</span><span class="nf">visit</span><span class="p">(</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">);</span>  
<span class="err"> </span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">#username</span><span class="dl">'</span><span class="p">).</span><span class="nf">type</span><span class="p">(</span><span class="nx">users</span><span class="err">\</span><span class="p">[</span><span class="nx">role</span><span class="err">\</span><span class="p">].</span><span class="nx">username</span><span class="p">);</span>  
<span class="err"> </span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">#password</span><span class="dl">'</span><span class="p">).</span><span class="nf">type</span><span class="p">(</span><span class="nx">users</span><span class="err">\</span><span class="p">[</span><span class="nx">role</span><span class="err">\</span><span class="p">].</span><span class="nx">password</span><span class="p">);</span>  
<span class="err"> </span> <span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="se">\</span><span class="s1">[type="submit"</span><span class="se">\</span><span class="s1">]</span><span class="dl">'</span><span class="p">).</span><span class="nf">click</span><span class="p">();</span>  
<span class="p">});</span>  
</code></pre>

</div>






<h2>
  
  
  Handling Edge Cases and Fallback UI
</h2>

<p>Ensure the dashboard handles API failures gracefully:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">cy</span><span class="p">.</span><span class="nf">intercept</span><span class="p">(</span><span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">/api/dashboard/stats</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">statusCode</span><span class="p">:</span> <span class="mi">500</span> <span class="p">}).</span><span class="k">as</span><span class="p">(</span><span class="dl">'</span><span class="s1">failStats</span><span class="dl">'</span><span class="p">);</span>  
<span class="nx">cy</span><span class="p">.</span><span class="nf">visit</span><span class="p">(</span><span class="dl">'</span><span class="s1">/dashboard</span><span class="dl">'</span><span class="p">);</span>  
<span class="nx">cy</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">[data-testid="stats-error"</span><span class="se">\</span><span class="s1">]</span><span class="dl">'</span><span class="p">)</span>  
<span class="err">  </span><span class="p">.</span><span class="nf">should</span><span class="p">(</span><span class="dl">'</span><span class="s1">be.visible</span><span class="dl">'</span><span class="p">)</span>  
<span class="err">  </span><span class="p">.</span><span class="nf">and</span><span class="p">(</span><span class="dl">'</span><span class="s1">contain</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Unable to load statistics</span><span class="dl">'</span><span class="p">);</span>  
</code></pre>

</div>






<h1>
  
  
  CI/CD Integration with GitHub Actions
</h1>

<p>Enable Cypress tests to run on every pull request:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">jobs</span><span class="pi">:</span>  
<span class="na">  e2e-tests</span><span class="pi">:</span>  
<span class="na">  runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>  
<span class="na">  steps</span><span class="pi">:</span>  
<span class="na">  - uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>  
<span class="na">  - uses</span><span class="pi">:</span> <span class="s">cypress-io/github-action@v5</span>  
<span class="na">  with</span><span class="pi">:</span>  
<span class="na">  start</span><span class="pi">:</span> <span class="s">npm start</span>  
<span class="na">  wait-on</span><span class="pi">:</span> <span class="s1">'</span><span class="s">http://localhost:3000'</span>  
<span class="na">  wait-on-timeout</span><span class="pi">:</span> <span class="s">120</span>  
<span class="na">  command</span><span class="pi">:</span> <span class="s">npm run test:e2e</span>  
</code></pre>

</div>



<p>Also add reporting tools like **Mochawesome** or **Allure**:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx cypress run <span class="nt">--reporter</span> mochawesome  
</code></pre>

</div>






<h2>
  
  
  Final Thoughts
</h2>

<p>If you’re working on a data-driven dashboard or shipping application, Cypress can dramatically improve your test quality and speed to release.</p>




<h2>
  
  
  About the Author
</h2>

<p>I’m Mohamed, a passionate QA Automation Engineer specializing in modern web automation with Cypress, Selenium, and Playwright. I share weekly articles on practical QA strategies and tools.</p>

<p>✅ Follow me on Medium for more Cypress insights<br><br>
✅ Let’s connect on [LinkedIn](<a href="https://www.linkedin.com/in/your-profile" rel="noopener noreferrer">https://www.linkedin.com/in/your-profile</a>)</p>

<p><em>Happy Testing!</em> 🚀</p>

<p><a href="https://medium.com/p/88f5476bae46" rel="noopener noreferrer">View original.</a></p>

<p>Exported from <a href="https://medium.com" rel="noopener noreferrer">Medium</a> on October 2, 2025.</p>

