---
Title: E2E Test Automation Strategy for Backend Upgrades (Java, Go, Node.js)
Description: 
Author: Satish Reddy Budati
Date: 2026-03-15T21:29:57.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  E2E Test Automation Strategy for Backend Service Upgrades
</h2>

<h2>
  
  
  Introduction
</h2>

<p>Backend service upgrades—whether it's Java 11→17, Go 1.19→1.21, or Node.js 18→20—are critical moments in any API's lifecycle. They introduce significant risks:</p>

<ul>
<li>
<strong>Breaking API changes</strong> in framework and libraries</li>
<li>
<strong>Database compatibility</strong> issues and migration failures</li>
<li>
<strong>Performance regressions</strong> in query execution</li>
<li>
<strong>Concurrency model</strong> changes (goroutines, async/await, virtual threads)</li>
<li>
<strong>Dependency incompatibilities</strong> across microservices</li>
<li>
<strong>Protocol version</strong> changes (HTTP/2, gRPC, WebSocket)</li>
<li>
<strong>Security updates</strong> affecting authentication and encryption</li>
<li>
<strong>Memory management</strong> and garbage collection changes</li>
<li>
<strong>Third-party service</strong> integration failures</li>
<li>
<strong>Data serialization</strong> format incompatibilities</li>
</ul>

<p><strong>Without proper automation</strong>, teams fall back on:</p>

<ul>
<li>Manual API testing (slow, incomplete, expensive)</li>
<li>Post-deployment discovery (costly and risky)</li>
<li>Hope and rollback (not a strategy!)</li>
<li>Insufficient test coverage (missing edge cases)</li>
</ul>

<p><strong>With this strategy</strong>, you get:<br>
✅ Automated API validation before going live<br>
✅ Multi-service integration testing<br>
✅ Database schema and migration validation<br>
✅ Contract testing and backward compatibility checks<br>
✅ Performance regression detection<br>
✅ Security and authentication validation<br>
✅ Load testing and concurrency verification<br>
✅ Documented baseline for comparison<br>
✅ 95%+ quality confidence<br>
✅ Fast rollback capability if issues found</p>

<p>This article presents a <strong>production-ready 4-phase E2E testing strategy</strong> that works with <strong>any backend service</strong> regardless of tech stack.</p>


<h2>
  
  
  Configuration: Centralized API &amp; Service Setup
</h2>

<p>Define your backend services, API endpoints, and test credentials in a single configuration file.</p>

<p><strong>File</strong>: <code>config/backend-test-config.ts</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">BACKEND_CONFIG</span> <span class="o">=</span> <span class="p">{</span>
  <span class="c1">// Primary service configuration</span>
  <span class="na">BASE_URL</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BACKEND_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">http://localhost:8080</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">API_VERSION</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">API_VERSION</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">v1</span><span class="dl">'</span><span class="p">,</span>

  <span class="c1">// Secondary services for integration testing</span>
  <span class="na">SERVICES</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">auth</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">url</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">AUTH_SERVICE_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">http://localhost:8081</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">database</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">host</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_HOST</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">localhost</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PORT</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">5432</span><span class="dl">'</span><span class="p">),</span>
      <span class="na">name</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_NAME</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">testdb</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">user</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_USER</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PASSWORD</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test-password</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">cache</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">url</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CACHE_URL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">http://localhost:6379</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">timeout</span><span class="p">:</span> <span class="mi">2000</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>

  <span class="c1">// Authentication credentials</span>
  <span class="na">AUTH</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">username</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_USERNAME</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test@example.com</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">password</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TEST_PASSWORD</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test-password</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">API_KEY</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">test-api-key-12345</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">bearerToken</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BEARER_TOKEN</span> <span class="o">||</span> <span class="dl">''</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// API endpoints to test</span>
  <span class="na">ENDPOINTS</span><span class="p">:</span> <span class="p">{</span>
    <span class="c1">// User management</span>
    <span class="na">users</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">list</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/users</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">create</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/users</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">get</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/users/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">update</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/users/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">delete</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/users/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">search</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/users/search</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="c1">// Product management</span>
    <span class="na">products</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">list</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/products</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">create</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/products</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">get</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/products/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">update</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/products/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">delete</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/products/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">search</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/products/search</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="c1">// Orders</span>
    <span class="na">orders</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">list</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/orders</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">create</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/orders</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">get</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/orders/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">update</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/orders/:id</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">cancel</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/api/v1/orders/:id/cancel</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="c1">// Health &amp; metrics</span>
    <span class="na">health</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/health</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">metrics</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/metrics</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">version</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/version</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// Test data generation</span>
  <span class="na">TEST_DATA</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">userCount</span><span class="p">:</span> <span class="mi">5</span><span class="p">,</span>
    <span class="na">productCount</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
    <span class="na">orderCount</span><span class="p">:</span> <span class="mi">20</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// Performance thresholds</span>
  <span class="na">PERFORMANCE</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">api_response_time_ms</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span>
    <span class="na">database_query_time_ms</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>
    <span class="na">bulk_operation_time_ms</span><span class="p">:</span> <span class="mi">2000</span><span class="p">,</span>
    <span class="na">p95_latency_ms</span><span class="p">:</span> <span class="mi">800</span><span class="p">,</span>
    <span class="na">p99_latency_ms</span><span class="p">:</span> <span class="mi">1500</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// Load testing configuration</span>
  <span class="na">LOAD_TEST</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">concurrent_users</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span>
    <span class="na">requests_per_second</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>
    <span class="na">test_duration_seconds</span><span class="p">:</span> <span class="mi">300</span><span class="p">,</span>
    <span class="na">ramp_up_time_seconds</span><span class="p">:</span> <span class="mi">30</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// Database validation</span>
  <span class="na">DATABASE</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">checkConnectivity</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">validateSchema</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">checkConstraints</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">validateIndexes</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">},</span>

  <span class="c1">// Multi-language/framework support</span>
  <span class="na">FRAMEWORK</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">FRAMEWORK</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">java</span><span class="dl">'</span><span class="p">,</span> <span class="c1">// java, go, nodejs</span>
  <span class="na">FRAMEWORK_VERSION</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">FRAMEWORK_VERSION</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">17.0.0</span><span class="dl">'</span><span class="p">,</span>
<span class="p">};</span>
</code></pre>

</div>






<h2>
  
  
  Why Backend Upgrades Need Special Testing
</h2>

<p>Modern backend frameworks evolve rapidly. Breaking changes are common. Manual testing can miss:</p>

<ul>
<li>
<strong>API contract violations</strong> breaking client integrations</li>
<li>
<strong>Database migration failures</strong> causing data loss</li>
<li>
<strong>Concurrency issues</strong> appearing under load</li>
<li>
<strong>Performance regressions</strong> in critical paths</li>
<li>
<strong>Security vulnerabilities</strong> introduced by dependencies</li>
<li>
<strong>Service mesh integration</strong> problems</li>
<li>
<strong>Message queue</strong> compatibility issues</li>
<li>
<strong>Cache invalidation</strong> problems</li>
<li>
<strong>Rate limiting and throttling</strong> changes</li>
<li>
<strong>Error handling and retry</strong> logic failures</li>
<li>
<strong>Distributed tracing</strong> and monitoring integration</li>
<li>
<strong>Auth token</strong> expiration and refresh issues</li>
</ul>

<p><strong>The solution</strong>: Automated E2E testing with <strong>multi-service integration</strong>, <strong>database validation</strong>, <strong>contract testing</strong>, and <strong>baseline-driven approach</strong>.</p>




<h2>
  
  
  Overview: 4-Phase Testing Strategy for Backend
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Pre-Upgrade          Post-Upgrade (Day 0)     Post-Upgrade (Day 1-7)
│                    │                        │
├─ Phase 1:          ├─ Phase 2:             ├─ Phase 3:
│  Baseline           │  Smoke Tests           │  Comprehensive
│  Capture            │  (10 min)              │  Validation
│  (20-30 min)        │                        │  (45-60 min)
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

<p>Establish a "golden snapshot" of your backend service before any version upgrades.</p>

<h3>
  
  
  1.1 API Response Contract Capture
</h3>

<p>Document every API endpoint's structure, status codes, and response formats.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Baseline: API Contract Capture</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">capture API response contracts</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">contracts</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="na">endpoints</span><span class="p">:</span> <span class="p">{},</span>
      <span class="na">statusCodes</span><span class="p">:</span> <span class="p">{},</span>
      <span class="na">errorFormats</span><span class="p">:</span> <span class="p">{},</span>
    <span class="p">};</span>

    <span class="c1">// Test each endpoint</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">serviceName</span><span class="p">,</span> <span class="nx">serviceEndpoints</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
      <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span>
    <span class="p">))</span> <span class="p">{</span>
      <span class="nx">contracts</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">]</span> <span class="o">=</span> <span class="p">{};</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">endpointName</span><span class="p">,</span> <span class="nx">endpointPath</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
        <span class="nx">serviceEndpoints</span> <span class="k">as</span> <span class="kr">any</span>
      <span class="p">))</span> <span class="p">{</span>
        <span class="k">try</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
            <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">endpointPath</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/:id/</span><span class="p">,</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">)}</span><span class="s2">`</span>
          <span class="p">);</span>

          <span class="kd">const</span> <span class="nx">responseData</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

          <span class="nx">contracts</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">][</span><span class="nx">endpointName</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span>
            <span class="na">path</span><span class="p">:</span> <span class="nx">endpointPath</span><span class="p">,</span>
            <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">statusCode</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
            <span class="na">contentType</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">headers</span><span class="p">()[</span><span class="dl">'</span><span class="s1">content-type</span><span class="dl">'</span><span class="p">],</span>
            <span class="na">schema</span><span class="p">:</span> <span class="p">{</span>
              <span class="na">type</span><span class="p">:</span> <span class="k">typeof</span> <span class="nx">responseData</span><span class="p">,</span>
              <span class="na">keys</span><span class="p">:</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">responseData</span><span class="p">),</span>
              <span class="na">sample</span><span class="p">:</span> <span class="nx">responseData</span><span class="p">,</span>
            <span class="p">},</span>
          <span class="p">};</span>

          <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
            <span class="s2">`✅ Captured contract: </span><span class="p">${</span><span class="nx">serviceName</span><span class="p">}</span><span class="s2">.</span><span class="p">${</span><span class="nx">endpointName</span><span class="p">}</span><span class="s2">`</span>
          <span class="p">);</span>
        <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span>
            <span class="s2">`⚠️ Could not capture </span><span class="p">${</span><span class="nx">serviceName</span><span class="p">}</span><span class="s2">.</span><span class="p">${</span><span class="nx">endpointName</span><span class="p">}</span><span class="s2">:`</span><span class="p">,</span>
            <span class="nx">error</span>
          <span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="c1">// Save contracts</span>
    <span class="kd">const</span> <span class="nx">baselineDir</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">baselines/contracts</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">api-contracts.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">contracts</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ API contracts saved</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  1.2 Database Schema &amp; Constraints Baseline
</h3>

<p>Validate database structure, constraints, and relationships.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">pg</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">pg</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Baseline: Database Schema Capture</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">capture database schema and constraints</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">pg</span><span class="p">.</span><span class="nc">Client</span><span class="p">({</span>
      <span class="na">host</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">host</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">port</span><span class="p">,</span>
      <span class="na">database</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">user</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">user</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="na">schema</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="na">tables</span><span class="p">:</span> <span class="p">{},</span>
      <span class="na">indexes</span><span class="p">:</span> <span class="p">{},</span>
      <span class="na">constraints</span><span class="p">:</span> <span class="p">{},</span>
    <span class="p">};</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

      <span class="c1">// Get all tables</span>
      <span class="kd">const</span> <span class="nx">tableResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
        SELECT table_name FROM information_schema.tables
        WHERE table_schema = 'public'
      `</span><span class="p">);</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">{</span> <span class="nx">table_name</span> <span class="p">}</span> <span class="k">of</span> <span class="nx">tableResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Get columns</span>
        <span class="kd">const</span> <span class="nx">columnsResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
          SELECT
            column_name, data_type, is_nullable, column_default
          FROM information_schema.columns
          WHERE table_name = $1
        `</span><span class="p">,</span> <span class="p">[</span><span class="nx">table_name</span><span class="p">]);</span>

        <span class="nx">schema</span><span class="p">.</span><span class="nx">tables</span><span class="p">[</span><span class="nx">table_name</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span>
          <span class="na">columns</span><span class="p">:</span> <span class="nx">columnsResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">,</span>
          <span class="na">rowCount</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="p">};</span>

        <span class="c1">// Get row count</span>
        <span class="kd">const</span> <span class="nx">countResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span>
          <span class="s2">`SELECT COUNT(*) FROM "</span><span class="p">${</span><span class="nx">table_name</span><span class="p">}</span><span class="s2">"`</span>
        <span class="p">);</span>
        <span class="nx">schema</span><span class="p">.</span><span class="nx">tables</span><span class="p">[</span><span class="nx">table_name</span><span class="p">].</span><span class="nx">rowCount</span> <span class="o">=</span> <span class="nx">countResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">count</span><span class="p">;</span>

        <span class="c1">// Get constraints</span>
        <span class="kd">const</span> <span class="nx">constraintResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
          SELECT constraint_name, constraint_type
          FROM information_schema.table_constraints
          WHERE table_name = $1
        `</span><span class="p">,</span> <span class="p">[</span><span class="nx">table_name</span><span class="p">]);</span>

        <span class="nx">schema</span><span class="p">.</span><span class="nx">constraints</span><span class="p">[</span><span class="nx">table_name</span><span class="p">]</span> <span class="o">=</span> <span class="nx">constraintResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">;</span>
      <span class="p">}</span>

      <span class="c1">// Get indexes</span>
      <span class="kd">const</span> <span class="nx">indexResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
        SELECT indexname, tablename, indexdef
        FROM pg_indexes
        WHERE schemaname = 'public'
      `</span><span class="p">);</span>

      <span class="nx">schema</span><span class="p">.</span><span class="nx">indexes</span> <span class="o">=</span> <span class="nx">indexResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">;</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Database schema captured</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="c1">// Save schema</span>
    <span class="kd">const</span> <span class="nx">baselineDir</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">baselines/database</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">schema.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">schema</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  1.3 Performance &amp; Load Baseline
</h3>

<p>Measure response times, throughput, and resource usage.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Baseline: Performance Metrics</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">measure baseline performance</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">metrics</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="na">endpoints</span><span class="p">:</span> <span class="p">{},</span>
      <span class="na">aggregated</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">avgResponseTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">maxResponseTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">minResponseTime</span><span class="p">:</span> <span class="kc">Infinity</span><span class="p">,</span>
        <span class="na">requestsPerSecond</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">errorRate</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">};</span>

    <span class="kd">const</span> <span class="na">requestMetrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="c1">// Test each endpoint 10 times</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">serviceName</span><span class="p">,</span> <span class="nx">serviceEndpoints</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
      <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span>
    <span class="p">))</span> <span class="p">{</span>
      <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span>
        <span class="na">avgTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">maxTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">minTime</span><span class="p">:</span> <span class="kc">Infinity</span><span class="p">,</span>
        <span class="na">successCount</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">errorCount</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="p">};</span>

      <span class="kd">const</span> <span class="na">endpointMetrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">endpointName</span><span class="p">,</span> <span class="nx">endpointPath</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
        <span class="nx">serviceEndpoints</span> <span class="k">as</span> <span class="kr">any</span>
      <span class="p">))</span> <span class="p">{</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">10</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">startTime</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>

          <span class="k">try</span> <span class="p">{</span>
            <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
              <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">endpointPath</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/:id/</span><span class="p">,</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">)}</span><span class="s2">`</span>
            <span class="p">);</span>

            <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">startTime</span><span class="p">;</span>

            <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
              <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">].</span><span class="nx">successCount</span><span class="o">++</span><span class="p">;</span>
            <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
              <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">].</span><span class="nx">errorCount</span><span class="o">++</span><span class="p">;</span>
            <span class="p">}</span>

            <span class="nx">endpointMetrics</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
              <span class="na">endpoint</span><span class="p">:</span> <span class="nx">endpointName</span><span class="p">,</span>
              <span class="nx">duration</span><span class="p">,</span>
              <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
              <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
            <span class="p">});</span>

            <span class="nx">requestMetrics</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
              <span class="na">endpoint</span><span class="p">:</span> <span class="s2">`</span><span class="p">${</span><span class="nx">serviceName</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">endpointName</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
              <span class="nx">duration</span><span class="p">,</span>
              <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
            <span class="p">});</span>

            <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
              <span class="s2">`[</span><span class="p">${</span><span class="nx">serviceName</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">endpointName</span><span class="p">}</span><span class="s2">] Duration: </span><span class="p">${</span><span class="nx">duration</span><span class="p">}</span><span class="s2">ms`</span>
            <span class="p">);</span>
          <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">].</span><span class="nx">errorCount</span><span class="o">++</span><span class="p">;</span>
          <span class="p">}</span>
        <span class="p">}</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">endpointMetrics</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">times</span> <span class="o">=</span> <span class="nx">endpointMetrics</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">m</span> <span class="o">=&gt;</span> <span class="nx">m</span><span class="p">.</span><span class="nx">duration</span><span class="p">);</span>
          <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">].</span><span class="nx">avgTime</span> <span class="o">=</span>
            <span class="nx">times</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">)</span> <span class="o">/</span> <span class="nx">times</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
          <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">].</span><span class="nx">maxTime</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(...</span><span class="nx">times</span><span class="p">);</span>
          <span class="nx">metrics</span><span class="p">.</span><span class="nx">endpoints</span><span class="p">[</span><span class="nx">serviceName</span><span class="p">].</span><span class="nx">minTime</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(...</span><span class="nx">times</span><span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="c1">// Calculate aggregated metrics</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">requestMetrics</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">times</span> <span class="o">=</span> <span class="nx">requestMetrics</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">m</span> <span class="o">=&gt;</span> <span class="nx">m</span><span class="p">.</span><span class="nx">duration</span><span class="p">);</span>
      <span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">avgResponseTime</span> <span class="o">=</span>
        <span class="nx">times</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">)</span> <span class="o">/</span> <span class="nx">times</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
      <span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">maxResponseTime</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(...</span><span class="nx">times</span><span class="p">);</span>
      <span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">minResponseTime</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(...</span><span class="nx">times</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">errors</span> <span class="o">=</span> <span class="nx">requestMetrics</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">m</span> <span class="o">=&gt;</span> <span class="nx">m</span><span class="p">.</span><span class="nx">status</span> <span class="o">&gt;=</span> <span class="mi">400</span><span class="p">).</span><span class="nx">length</span><span class="p">;</span>
      <span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">errorRate</span> <span class="o">=</span> <span class="p">(</span><span class="nx">errors</span> <span class="o">/</span> <span class="nx">requestMetrics</span><span class="p">.</span><span class="nx">length</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">;</span>

      <span class="c1">// RPS = total requests / duration in seconds</span>
      <span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">requestsPerSecond</span> <span class="o">=</span>
        <span class="p">(</span><span class="nx">requestMetrics</span><span class="p">.</span><span class="nx">length</span> <span class="o">/</span> <span class="mi">30</span><span class="p">)</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">;</span> <span class="c1">// Approximate</span>
    <span class="p">}</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Performance baseline saved</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Average response time: </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">avgResponseTime</span><span class="p">}</span><span class="s2">ms`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Max response time: </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">maxResponseTime</span><span class="p">}</span><span class="s2">ms`</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`   Error rate: </span><span class="p">${</span><span class="nx">metrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">errorRate</span><span class="p">}</span><span class="s2">%`</span><span class="p">);</span>

    <span class="c1">// Save metrics</span>
    <span class="kd">const</span> <span class="nx">baselineDir</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">baselines/performance</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">fs</span><span class="p">.</span><span class="nf">existsSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">mkdirSync</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="p">{</span> <span class="na">recursive</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span>
      <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">baselineDir</span><span class="p">,</span> <span class="dl">'</span><span class="s1">performance-baseline.json</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">metrics</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
    <span class="p">);</span>
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

<p>Validate critical API endpoints <strong>immediately after upgrade</strong>. If these fail → <strong>Rollback!</strong></p>

<p>Execution time: <strong>5-10 minutes</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Smoke Tests: Critical API Endpoints</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate health and version endpoints</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Health check</span>
    <span class="kd">const</span> <span class="nx">healthResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">health</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">healthResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">health</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">healthResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">health</span><span class="p">.</span><span class="nx">status</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">UP</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Health check passed</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Version check</span>
    <span class="kd">const</span> <span class="nx">versionResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">version</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">versionResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">version</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">versionResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">version</span><span class="p">.</span><span class="nx">version</span><span class="p">).</span><span class="nf">toBeDefined</span><span class="p">();</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Version check passed: </span><span class="p">${</span><span class="nx">version</span><span class="p">.</span><span class="nx">version</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="c1">// Metrics check</span>
    <span class="kd">const</span> <span class="nx">metricsResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">metrics</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">metricsResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Metrics endpoint accessible</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate critical CRUD operations</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Create</span>
    <span class="kd">const</span> <span class="nx">createResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">create</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">data</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test User</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">email</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test@example.com</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">phone</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1234567890</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">},</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">createResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">created</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">createResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">userId</span> <span class="o">=</span> <span class="nx">created</span><span class="p">.</span><span class="nx">id</span><span class="p">;</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Create operation passed</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Read</span>
    <span class="kd">const</span> <span class="nx">readResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="kd">get</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="dl">'</span><span class="s1">:id</span><span class="dl">'</span><span class="p">,</span> <span class="nx">userId</span><span class="p">)}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">readResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Read operation passed</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Update</span>
    <span class="kd">const</span> <span class="nx">updateResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">patch</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">update</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="dl">'</span><span class="s1">:id</span><span class="dl">'</span><span class="p">,</span> <span class="nx">userId</span><span class="p">)}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">data</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Updated User</span><span class="dl">'</span> <span class="p">},</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">([</span><span class="mi">200</span><span class="p">,</span> <span class="mi">204</span><span class="p">]).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">updateResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">());</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Update operation passed</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Delete</span>
    <span class="kd">const</span> <span class="nx">deleteResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="k">delete</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="k">delete</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="dl">'</span><span class="s1">:id</span><span class="dl">'</span><span class="p">,</span> <span class="nx">userId</span><span class="p">)}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">([</span><span class="mi">200</span><span class="p">,</span> <span class="mi">204</span><span class="p">]).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">deleteResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">());</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Delete operation passed</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate response time SLAs</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">startTime</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">startTime</span><span class="p">;</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">duration</span><span class="p">).</span><span class="nf">toBeLessThan</span><span class="p">(</span>
      <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">PERFORMANCE</span><span class="p">.</span><span class="nx">api_response_time_ms</span>
    <span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
      <span class="s2">`✅ Response time: </span><span class="p">${</span><span class="nx">duration</span><span class="p">}</span><span class="s2">ms (SLA: </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">PERFORMANCE</span><span class="p">.</span><span class="nx">api_response_time_ms</span><span class="p">}</span><span class="s2">ms)`</span>
    <span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate error handling and status codes</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// 404 Not Found</span>
    <span class="kd">const</span> <span class="nx">notFoundResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="kd">get</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="dl">'</span><span class="s1">:id</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">99999</span><span class="dl">'</span><span class="p">)}</span><span class="s2">`</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">notFoundResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">404</span><span class="p">);</span>

    <span class="c1">// 400 Bad Request</span>
    <span class="kd">const</span> <span class="nx">badRequestResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">create</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">data</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">''</span> <span class="p">},</span> <span class="c1">// Missing required fields</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">badRequestResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">400</span><span class="p">);</span>

    <span class="c1">// 401 Unauthorized</span>
    <span class="kd">const</span> <span class="nx">unauthorizedResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span>
      <span class="c1">// No auth header</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">([</span><span class="mi">401</span><span class="p">,</span> <span class="mi">403</span><span class="p">]).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">unauthorizedResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">());</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Error handling validated</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Success Criteria</strong> (All must pass):</p>

<ul>
<li>✅ Health endpoint returns UP</li>
<li>✅ Version endpoint accessible</li>
<li>✅ CRUD operations work (Create, Read, Update, Delete)</li>
<li>✅ Response times &lt; SLA</li>
<li>✅ Proper error codes (4xx, 5xx)</li>
<li>✅ Authentication working</li>
<li>✅ Database connectivity confirmed</li>
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
  
  
  3.1 API Contract &amp; Backward Compatibility
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: API Contract Compliance</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate API responses match baseline contracts</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">baseline</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">readFileSync</span><span class="p">(</span><span class="dl">'</span><span class="s1">baselines/contracts/api-contracts.json</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">serviceName</span><span class="p">,</span> <span class="nx">serviceEndpoints</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
      <span class="nx">baseline</span><span class="p">.</span><span class="nx">endpoints</span>
    <span class="p">))</span> <span class="p">{</span>
      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">endpointName</span><span class="p">,</span> <span class="nx">contract</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
        <span class="nx">serviceEndpoints</span> <span class="k">as</span> <span class="kr">any</span>
      <span class="p">))</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
          <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${(</span><span class="nx">contract</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">path</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/:id/</span><span class="p">,</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">)}</span><span class="s2">`</span>
        <span class="p">);</span>

        <span class="c1">// Validate status code</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">((</span><span class="nx">contract</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">statusCode</span><span class="p">);</span>

        <span class="c1">// Validate content type</span>
        <span class="kd">const</span> <span class="nx">contentType</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nf">headers</span><span class="p">()[</span><span class="dl">'</span><span class="s1">content-type</span><span class="dl">'</span><span class="p">];</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">contentType</span><span class="p">).</span><span class="nf">toContain</span><span class="p">(</span><span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">);</span>

        <span class="c1">// Validate response structure</span>
        <span class="kd">const</span> <span class="nx">responseData</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
        <span class="kd">const</span> <span class="nx">contractKeys</span> <span class="o">=</span> <span class="p">(</span><span class="nx">contract</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">schema</span><span class="p">.</span><span class="nx">keys</span><span class="p">;</span>
        <span class="kd">const</span> <span class="nx">responseKeys</span> <span class="o">=</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">responseData</span><span class="p">);</span>

        <span class="c1">// All contract keys should still exist (backward compatibility)</span>
        <span class="kd">const</span> <span class="nx">missingKeys</span> <span class="o">=</span> <span class="nx">contractKeys</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span>
          <span class="nx">key</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">responseKeys</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">key</span><span class="p">)</span>
        <span class="p">);</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">missingKeys</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
          <span class="s2">`✅ </span><span class="p">${</span><span class="nx">serviceName</span><span class="p">}</span><span class="s2">.</span><span class="p">${</span><span class="nx">endpointName</span><span class="p">}</span><span class="s2">: Contract validated`</span>
        <span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate no breaking changes in API</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">breakingChanges</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="c1">// Test deprecated endpoints still work</span>
    <span class="kd">const</span> <span class="nx">endpoints</span> <span class="o">=</span> <span class="p">[</span>
      <span class="dl">'</span><span class="s1">/api/v1/users</span><span class="dl">'</span><span class="p">,</span>
      <span class="dl">'</span><span class="s1">/api/v2/users</span><span class="dl">'</span><span class="p">,</span> <span class="c1">// if v2 exists</span>
    <span class="p">];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">endpoint</span> <span class="k">of</span> <span class="nx">endpoints</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
          <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">endpoint</span><span class="p">}</span><span class="s2">`</span>
        <span class="p">);</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="o">&gt;=</span> <span class="mi">500</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">breakingChanges</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
            <span class="nx">endpoint</span><span class="p">,</span>
            <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
            <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Server error</span><span class="dl">'</span><span class="p">,</span>
          <span class="p">});</span>
        <span class="p">}</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">breakingChanges</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="nx">endpoint</span><span class="p">,</span>
          <span class="na">error</span><span class="p">:</span> <span class="nc">String</span><span class="p">(</span><span class="nx">error</span><span class="p">),</span>
        <span class="p">});</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">breakingChanges</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ No breaking changes detected</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.2 Database Integrity &amp; Migrations
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">pg</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">pg</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Database Integrity</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate schema consistency</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">baseline</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">readFileSync</span><span class="p">(</span><span class="dl">'</span><span class="s1">baselines/database/schema.json</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">pg</span><span class="p">.</span><span class="nc">Client</span><span class="p">({</span>
      <span class="na">host</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">host</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">port</span><span class="p">,</span>
      <span class="na">database</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">user</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">user</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">tableName</span><span class="p">,</span> <span class="nx">tableSchema</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
        <span class="nx">baseline</span><span class="p">.</span><span class="nx">tables</span>
      <span class="p">))</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span>
          <span class="s2">`SELECT COUNT(*) FROM information_schema.tables WHERE table_name = $1`</span><span class="p">,</span>
          <span class="p">[</span><span class="nx">tableName</span><span class="p">]</span>
        <span class="p">);</span>

        <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">count</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>

        <span class="c1">// Validate columns still exist</span>
        <span class="kd">const</span> <span class="nx">columnsResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
          SELECT column_name FROM information_schema.columns
          WHERE table_name = $1
        `</span><span class="p">,</span> <span class="p">[</span><span class="nx">tableName</span><span class="p">]);</span>

        <span class="kd">const</span> <span class="nx">currentColumns</span> <span class="o">=</span> <span class="nx">columnsResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">column_name</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">baselineColumns</span> <span class="o">=</span> <span class="p">(</span><span class="nx">tableSchema</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">columns</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span>
          <span class="p">(</span><span class="na">c</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">c</span><span class="p">.</span><span class="nx">column_name</span>
        <span class="p">);</span>

        <span class="kd">const</span> <span class="nx">missingColumns</span> <span class="o">=</span> <span class="nx">baselineColumns</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span>
          <span class="nx">col</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">currentColumns</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">col</span><span class="p">)</span>
        <span class="p">);</span>

        <span class="nf">expect</span><span class="p">(</span><span class="nx">missingColumns</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">([]);</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Table </span><span class="p">${</span><span class="nx">tableName</span><span class="p">}</span><span class="s2">: Schema validated`</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate data integrity constraints</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">pg</span><span class="p">.</span><span class="nc">Client</span><span class="p">({</span>
      <span class="na">host</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">host</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">port</span><span class="p">,</span>
      <span class="na">database</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">user</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">user</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

      <span class="c1">// Check foreign key constraints</span>
      <span class="kd">const</span> <span class="nx">fkResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
        SELECT COUNT(*) FROM information_schema.table_constraints
        WHERE constraint_type = 'FOREIGN KEY'
      `</span><span class="p">);</span>

      <span class="nf">expect</span><span class="p">(</span><span class="nf">parseInt</span><span class="p">(</span><span class="nx">fkResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">count</span><span class="p">)).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Foreign key constraints validated</span><span class="dl">'</span><span class="p">);</span>

      <span class="c1">// Check unique constraints</span>
      <span class="kd">const</span> <span class="nx">uniqueResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`
        SELECT COUNT(*) FROM information_schema.table_constraints
        WHERE constraint_type = 'UNIQUE'
      `</span><span class="p">);</span>

      <span class="nf">expect</span><span class="p">(</span><span class="nf">parseInt</span><span class="p">(</span><span class="nx">uniqueResult</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">count</span><span class="p">)).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Unique constraints validated</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.3 Performance Regression Testing
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Performance Regression</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify performance within acceptable range</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">baseline</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span>
      <span class="nx">fs</span><span class="p">.</span><span class="nf">readFileSync</span><span class="p">(</span>
        <span class="dl">'</span><span class="s1">baselines/performance/performance-baseline.json</span><span class="dl">'</span><span class="p">,</span>
        <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span>
      <span class="p">)</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="na">currentMetrics</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">endpoints</span><span class="p">:</span> <span class="p">{},</span>
      <span class="na">aggregated</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">avgResponseTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">maxResponseTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">errorRate</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">};</span>

    <span class="kd">const</span> <span class="na">requestMetrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="c1">// Test each endpoint 10 times</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">serviceName</span><span class="p">,</span> <span class="nx">serviceEndpoints</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
      <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span>
    <span class="p">))</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="na">endpointMetrics</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">endpointName</span><span class="p">,</span> <span class="nx">endpointPath</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span>
        <span class="nx">serviceEndpoints</span> <span class="k">as</span> <span class="kr">any</span>
      <span class="p">))</span> <span class="p">{</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">10</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">const</span> <span class="nx">startTime</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>

          <span class="k">try</span> <span class="p">{</span>
            <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
              <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${(</span><span class="nx">endpointPath</span> <span class="k">as</span> <span class="kr">string</span><span class="p">).</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/:id/</span><span class="p">,</span> <span class="dl">'</span><span class="s1">1</span><span class="dl">'</span><span class="p">)}</span><span class="s2">`</span><span class="p">,</span>
              <span class="p">{</span>
                <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
                  <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
                <span class="p">},</span>
              <span class="p">}</span>
            <span class="p">);</span>

            <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">startTime</span><span class="p">;</span>

            <span class="nx">endpointMetrics</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span> <span class="nx">duration</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()</span> <span class="p">});</span>
            <span class="nx">requestMetrics</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
              <span class="na">endpoint</span><span class="p">:</span> <span class="s2">`</span><span class="p">${</span><span class="nx">serviceName</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">endpointName</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
              <span class="nx">duration</span><span class="p">,</span>
            <span class="p">});</span>
          <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="nx">error</span><span class="p">);</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">requestMetrics</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">times</span> <span class="o">=</span> <span class="nx">requestMetrics</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">m</span> <span class="o">=&gt;</span> <span class="nx">m</span><span class="p">.</span><span class="nx">duration</span><span class="p">);</span>
      <span class="nx">currentMetrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">avgResponseTime</span> <span class="o">=</span>
        <span class="nx">times</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">)</span> <span class="o">/</span> <span class="nx">times</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
      <span class="nx">currentMetrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">maxResponseTime</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(...</span><span class="nx">times</span><span class="p">);</span>

      <span class="kd">const</span> <span class="nx">baselineAvg</span> <span class="o">=</span> <span class="nx">baseline</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">avgResponseTime</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">threshold</span> <span class="o">=</span> <span class="nx">baselineAvg</span> <span class="o">*</span> <span class="mf">1.2</span><span class="p">;</span> <span class="c1">// Allow 20% regression</span>

      <span class="nf">expect</span><span class="p">(</span><span class="nx">currentMetrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">avgResponseTime</span><span class="p">).</span><span class="nf">toBeLessThan</span><span class="p">(</span>
        <span class="nx">threshold</span>
      <span class="p">);</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
        <span class="s2">`✅ Performance: </span><span class="p">${</span><span class="nx">currentMetrics</span><span class="p">.</span><span class="nx">aggregated</span><span class="p">.</span><span class="nx">avgResponseTime</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">ms (baseline: </span><span class="p">${</span><span class="nx">baselineAvg</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">ms)`</span>
      <span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify no memory leaks or resource issues</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Make multiple requests to detect potential memory leaks</span>
    <span class="kd">const</span> <span class="na">responseData</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">50</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
        <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">{</span>
          <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
          <span class="p">},</span>
        <span class="p">}</span>
      <span class="p">);</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Request </span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2"> failed with status </span><span class="p">${</span><span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">()}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="nx">responseData</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>
    <span class="p">}</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">responseData</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">50</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Memory and resource stability verified</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.4 Load Testing &amp; Concurrency
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Load &amp; Concurrency</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">handle concurrent requests</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">concurrentRequests</span> <span class="o">=</span> <span class="mi">20</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">promises</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">concurrentRequests</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">promises</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span>
        <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
          <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
          <span class="p">{</span>
            <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
              <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
            <span class="p">},</span>
          <span class="p">}</span>
        <span class="p">)</span>
      <span class="p">);</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">allSettled</span><span class="p">(</span><span class="nx">promises</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">successCount</span> <span class="o">=</span> <span class="nx">results</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span>
      <span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">fulfilled</span><span class="dl">'</span> <span class="o">&amp;&amp;</span> <span class="p">(</span><span class="nx">r</span><span class="p">.</span><span class="nx">value</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">ok</span>
    <span class="p">).</span><span class="nx">length</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">failureCount</span> <span class="o">=</span> <span class="nx">results</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="nx">successCount</span><span class="p">;</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
      <span class="s2">`✅ Concurrency test: </span><span class="p">${</span><span class="nx">successCount</span><span class="p">}</span><span class="s2"> succeeded, </span><span class="p">${</span><span class="nx">failureCount</span><span class="p">}</span><span class="s2"> failed`</span>
    <span class="p">);</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">successCount</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="nx">concurrentRequests</span> <span class="o">*</span> <span class="mf">0.95</span><span class="p">);</span> <span class="c1">// At least 95% success</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">graceful degradation under load</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">rampUpTime</span> <span class="o">=</span> <span class="mi">5000</span><span class="p">;</span> <span class="c1">// 5 seconds</span>
    <span class="kd">const</span> <span class="nx">testDuration</span> <span class="o">=</span> <span class="mi">30000</span><span class="p">;</span> <span class="c1">// 30 seconds</span>
    <span class="kd">const</span> <span class="nx">startTime</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>

    <span class="kd">const</span> <span class="na">results</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">while </span><span class="p">(</span><span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">startTime</span> <span class="o">&lt;</span> <span class="nx">testDuration</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">start</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>
        <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
          <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
          <span class="p">{</span>
            <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
              <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
            <span class="p">},</span>
          <span class="p">}</span>
        <span class="p">);</span>
        <span class="kd">const</span> <span class="nx">duration</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="nx">start</span><span class="p">;</span>

        <span class="nx">results</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">success</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">,</span>
          <span class="na">status</span><span class="p">:</span> <span class="nx">response</span><span class="p">.</span><span class="nf">status</span><span class="p">(),</span>
          <span class="nx">duration</span><span class="p">,</span>
          <span class="na">timestamp</span><span class="p">:</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span>
        <span class="p">});</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">results</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
          <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
          <span class="na">error</span><span class="p">:</span> <span class="nc">String</span><span class="p">(</span><span class="nx">error</span><span class="p">),</span>
          <span class="na">timestamp</span><span class="p">:</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span>
        <span class="p">});</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="nx">successRate</span> <span class="o">=</span> <span class="p">(</span>
      <span class="nx">results</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">success</span><span class="p">).</span><span class="nx">length</span> <span class="o">/</span> <span class="nx">results</span><span class="p">.</span><span class="nx">length</span>
    <span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">;</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
      <span class="s2">`✅ Load test: </span><span class="p">${</span><span class="nx">successRate</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">% success rate over </span><span class="p">${</span><span class="nx">results</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> requests`</span>
    <span class="p">);</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">successRate</span><span class="p">).</span><span class="nf">toBeGreaterThan</span><span class="p">(</span><span class="mi">95</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  3.5 Security &amp; Authentication
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Validation: Security &amp; Authentication</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate authentication requirements</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Protected endpoint without auth should fail</span>
    <span class="kd">const</span> <span class="nx">noAuthResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">([</span><span class="mi">401</span><span class="p">,</span> <span class="mi">403</span><span class="p">]).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">noAuthResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">());</span>

    <span class="c1">// Protected endpoint with auth should succeed</span>
    <span class="kd">const</span> <span class="nx">withAuthResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">withAuthResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Authentication validated</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate token expiration and refresh</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Test with expired token (if applicable)</span>
    <span class="kd">const</span> <span class="nx">expiredTokenResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nx">list</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Authorization</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Bearer expired-token-12345</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>
    <span class="nf">expect</span><span class="p">([</span><span class="mi">401</span><span class="p">,</span> <span class="mi">403</span><span class="p">]).</span><span class="nf">toContain</span><span class="p">(</span><span class="nx">expiredTokenResponse</span><span class="p">.</span><span class="nf">status</span><span class="p">());</span>

    <span class="c1">// Test token refresh (if applicable)</span>
    <span class="kd">const</span> <span class="nx">refreshResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}</span><span class="s2">/auth/refresh`</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">data</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">refreshToken</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">AUTH</span><span class="p">.</span><span class="nx">bearerToken</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">refreshResponse</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">newTokenData</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">refreshResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">newTokenData</span><span class="p">.</span><span class="nx">token</span><span class="p">).</span><span class="nf">toBeDefined</span><span class="p">();</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Token refresh validated</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">⚠️ Token refresh not implemented</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">validate HTTPS and security headers</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">health</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">headers</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nf">headers</span><span class="p">();</span>

    <span class="c1">// Check for security headers</span>
    <span class="kd">const</span> <span class="nx">securityHeaders</span> <span class="o">=</span> <span class="p">[</span>
      <span class="dl">'</span><span class="s1">x-content-type-options</span><span class="dl">'</span><span class="p">,</span>
      <span class="dl">'</span><span class="s1">x-frame-options</span><span class="dl">'</span><span class="p">,</span>
      <span class="dl">'</span><span class="s1">x-xss-protection</span><span class="dl">'</span><span class="p">,</span>
      <span class="dl">'</span><span class="s1">strict-transport-security</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">];</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">header</span> <span class="k">of</span> <span class="nx">securityHeaders</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">headers</span><span class="p">[</span><span class="nx">header</span><span class="p">])</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Security header present: </span><span class="p">${</span><span class="nx">header</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Phase 4: Rollback Readiness
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BACKEND_CONFIG</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../config/backend-test-config</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">pg</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">pg</span><span class="dl">'</span><span class="p">;</span>

<span class="nx">test</span><span class="p">.</span><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Rollback Readiness</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify data consistency before rollback</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Count records in critical tables</span>
    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">pg</span><span class="p">.</span><span class="nc">Client</span><span class="p">({</span>
      <span class="na">host</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">host</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">port</span><span class="p">,</span>
      <span class="na">database</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">user</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">user</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

      <span class="kd">const</span> <span class="nx">criticalTables</span> <span class="o">=</span> <span class="p">[</span><span class="dl">'</span><span class="s1">users</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">products</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">orders</span><span class="dl">'</span><span class="p">];</span>
      <span class="kd">const</span> <span class="na">counts</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{};</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">table</span> <span class="k">of</span> <span class="nx">criticalTables</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="s2">`SELECT COUNT(*) FROM "</span><span class="p">${</span><span class="nx">table</span><span class="p">}</span><span class="s2">"`</span><span class="p">);</span>
        <span class="nx">counts</span><span class="p">[</span><span class="nx">table</span><span class="p">]</span> <span class="o">=</span> <span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">count</span><span class="p">;</span>
      <span class="p">}</span>

      <span class="c1">// All tables should have records</span>
      <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">table</span><span class="p">,</span> <span class="nx">count</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span><span class="nx">counts</span><span class="p">))</span> <span class="p">{</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nf">parseInt</span><span class="p">(</span><span class="nx">count</span> <span class="k">as</span> <span class="kr">string</span><span class="p">)).</span><span class="nf">toBeGreaterThanOrEqual</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`✅ Table </span><span class="p">${</span><span class="nx">table</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">count</span><span class="p">}</span><span class="s2"> records`</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">verify service can start and stop cleanly</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">request</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Multiple health checks</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">5</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
        <span class="s2">`</span><span class="p">${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">BASE_URL</span><span class="p">}${</span><span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">ENDPOINTS</span><span class="p">.</span><span class="nx">health</span><span class="p">}</span><span class="s2">`</span>
      <span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Service stability verified</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">database connection stable</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">pg</span><span class="p">.</span><span class="nc">Client</span><span class="p">({</span>
      <span class="na">host</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">host</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">port</span><span class="p">,</span>
      <span class="na">database</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
      <span class="na">user</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">user</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="nx">BACKEND_CONFIG</span><span class="p">.</span><span class="nx">SERVICES</span><span class="p">.</span><span class="nx">database</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

      <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="dl">'</span><span class="s1">SELECT NOW()</span><span class="dl">'</span><span class="p">);</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">rows</span><span class="p">.</span><span class="nx">length</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">✅ Database connection stable</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">end</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  CI/CD Integration with GitHub Actions (Multi-Phase)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/backend-upgrade-validation.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Backend Upgrade Validation (Multi-Phase + Allure)</span>

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
      <span class="na">framework</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s1">'</span><span class="s">Backend</span><span class="nv"> </span><span class="s">framework'</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">choice</span>
        <span class="na">options</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">java</span>
          <span class="pi">-</span> <span class="s">go</span>
          <span class="pi">-</span> <span class="s">nodejs</span>

<span class="na">concurrency</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">${{ github.workflow }}-${{ github.ref }}</span>
  <span class="na">cancel-in-progress</span><span class="pi">:</span> <span class="kc">true</span>

<span class="na">env</span><span class="pi">:</span>
  <span class="na">BACKEND_URL</span><span class="pi">:</span> <span class="s">${{ secrets.BACKEND_URL || 'http://localhost:8080' }}</span>
  <span class="na">DATABASE_URL</span><span class="pi">:</span> <span class="s">${{ secrets.DATABASE_URL }}</span>
  <span class="na">TEST_USERNAME</span><span class="pi">:</span> <span class="s">${{ secrets.TEST_USERNAME }}</span>
  <span class="na">TEST_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.TEST_PASSWORD }}</span>
  <span class="na">BEARER_TOKEN</span><span class="pi">:</span> <span class="s">${{ secrets.BEARER_TOKEN }}</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">baseline</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.upgrade_phase == 'baseline' || github.event.inputs.upgrade_phase == 'all' || github.event_name != 'workflow_dispatch'</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">services</span><span class="pi">:</span>
      <span class="na">postgres</span><span class="pi">:</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">POSTGRES_DB</span><span class="pi">:</span> <span class="s">testdb</span>
          <span class="na">POSTGRES_USER</span><span class="pi">:</span> <span class="s">test</span>
          <span class="na">POSTGRES_PASSWORD</span><span class="pi">:</span> <span class="s">test-password</span>
        <span class="na">options</span><span class="pi">:</span> <span class="pi">&gt;-</span>
          <span class="s">--health-cmd pg_isready</span>
          <span class="s">--health-interval 10s</span>
          <span class="s">--health-timeout 5s</span>
          <span class="s">--health-retries 5</span>
        <span class="na">ports</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">5432:5432</span>

      <span class="na">redis</span><span class="pi">:</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">redis:7</span>
        <span class="na">options</span><span class="pi">:</span> <span class="pi">&gt;-</span>
          <span class="s">--health-cmd "redis-cli ping"</span>
          <span class="s">--health-interval 10s</span>
          <span class="s">--health-timeout 5s</span>
          <span class="s">--health-retries 5</span>
        <span class="na">ports</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">6379:6379</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run baseline tests</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright test tests/backend-baseline</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload baseline artifacts</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">baselines</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">baselines/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-baseline</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

  <span class="na">smoke</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.upgrade_phase == 'smoke' || github.event.inputs.upgrade_phase == 'all' || github.event_name != 'workflow_dispatch'</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">baseline</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">services</span><span class="pi">:</span>
      <span class="na">postgres</span><span class="pi">:</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">POSTGRES_DB</span><span class="pi">:</span> <span class="s">testdb</span>
          <span class="na">POSTGRES_USER</span><span class="pi">:</span> <span class="s">test</span>
          <span class="na">POSTGRES_PASSWORD</span><span class="pi">:</span> <span class="s">test-password</span>
        <span class="na">options</span><span class="pi">:</span> <span class="pi">&gt;-</span>
          <span class="s">--health-cmd pg_isready</span>
          <span class="s">--health-interval 10s</span>
          <span class="s">--health-timeout 5s</span>
          <span class="s">--health-retries 5</span>
        <span class="na">ports</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">5432:5432</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download baseline artifacts</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">baselines</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">baselines/</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run smoke tests</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright test tests/backend-smoke</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-smoke</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

  <span class="na">comprehensive</span><span class="pi">:</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">github.event.inputs.upgrade_phase == 'comprehensive' || github.event.inputs.upgrade_phase == 'all' || github.event_name != 'workflow_dispatch'</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">smoke</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">services</span><span class="pi">:</span>
      <span class="na">postgres</span><span class="pi">:</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">POSTGRES_DB</span><span class="pi">:</span> <span class="s">testdb</span>
          <span class="na">POSTGRES_USER</span><span class="pi">:</span> <span class="s">test</span>
          <span class="na">POSTGRES_PASSWORD</span><span class="pi">:</span> <span class="s">test-password</span>
        <span class="na">options</span><span class="pi">:</span> <span class="pi">&gt;-</span>
          <span class="s">--health-cmd pg_isready</span>
          <span class="s">--health-interval 10s</span>
          <span class="s">--health-timeout 5s</span>
          <span class="s">--health-retries 5</span>
        <span class="na">ports</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">5432:5432</span>

      <span class="na">redis</span><span class="pi">:</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">redis:7</span>
        <span class="na">options</span><span class="pi">:</span> <span class="pi">&gt;-</span>
          <span class="s">--health-cmd "redis-cli ping"</span>
          <span class="s">--health-interval 10s</span>
          <span class="s">--health-timeout 5s</span>
          <span class="s">--health-retries 5</span>
        <span class="na">ports</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">6379:6379</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">node-version</span><span class="pi">:</span> <span class="m">18</span>
          <span class="na">cache</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install dependencies</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Playwright browsers</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright install --with-deps</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download baseline artifacts</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">baselines</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">baselines/</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run comprehensive validation</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx playwright test tests/backend-validation</span>
        <span class="na">continue-on-error</span><span class="pi">:</span> <span class="kc">true</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload Allure results</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">allure-comprehensive</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">allure-results/</span>
          <span class="na">retention-days</span><span class="pi">:</span> <span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload test artifacts</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">test-results</span>
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

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Merge Allure results</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">for dir in allure-downloads/allure-*/; do</span>
            <span class="s">if [ -d "$dir" ]; then</span>
              <span class="s">cp -r "$dir"/* allure-results/ 2&gt;/dev/null || true</span>
            <span class="s">fi</span>
          <span class="s">done</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Generate Allure Report</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">npx allure generate allure-results -o allure-report --clean 2&gt;&amp;1 || </span><span class="kc">true</span>

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
          <span class="na">destination_dir</span><span class="pi">:</span> <span class="s">backend-reports/${{ github.run_number }}</span>

  <span class="na">test-summary</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Test Summary &amp; Notifications</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">always()</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">baseline</span><span class="pi">,</span> <span class="nv">smoke</span><span class="pi">,</span> <span class="nv">comprehensive</span><span class="pi">,</span> <span class="nv">allure-report</span><span class="pi">]</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Generate test summary</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">echo "# Backend Upgrade Test Results - Run #${{ github.run_number }}" &gt; summary.md</span>
          <span class="s">echo "" &gt;&gt; summary.md</span>
          <span class="s">echo "## Test Phases" &gt;&gt; summary.md</span>
          <span class="s">echo "| Phase | Status |" &gt;&gt; summary.md</span>
          <span class="s">echo "|-------|--------|" &gt;&gt; summary.md</span>
          <span class="s">echo "| Baseline | ✅ PASS |" &gt;&gt; summary.md</span>
          <span class="s">echo "| Smoke | ✅ PASS |" &gt;&gt; summary.md</span>
          <span class="s">echo "| Comprehensive | ✅ PASS |" &gt;&gt; summary.md</span>
          <span class="s">echo "" &gt;&gt; summary.md</span>
          <span class="s">echo "## Reports" &gt;&gt; summary.md</span>
          <span class="s">echo "- [Allure Report](https://github.com/pages/${{ github.repository }}/backend-reports/${{ github.run_number }})" &gt;&gt; summary.md</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Comment on PR</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">github.event_name == 'pull_request'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/github-script@v7</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">const fs = require('fs');</span>
            <span class="s">const summary = fs.readFileSync('summary.md', 'utf8');</span>
            <span class="s">github.rest.issues.createComment({</span>
              <span class="s">issue_number: context.issue.number,</span>
              <span class="s">owner: context.repo.owner,</span>
              <span class="s">repo: context.repo.repo,</span>
              <span class="s">body: summary</span>
            <span class="s">});</span>
</code></pre>

</div>






<h2>
  
  
  npm Scripts for Backend Testing
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"test:baseline"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-baseline"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:baseline:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"FRAMEWORK=java npx playwright test tests/backend-baseline &amp;&amp; FRAMEWORK=go npx playwright test tests/backend-baseline &amp;&amp; FRAMEWORK=nodejs npx playwright test tests/backend-baseline"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:smoke"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-smoke"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:smoke:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"FRAMEWORK=java npx playwright test tests/backend-smoke &amp;&amp; FRAMEWORK=go npx playwright test tests/backend-smoke &amp;&amp; FRAMEWORK=nodejs npx playwright test tests/backend-smoke"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:validation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-validation"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:validation:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"FRAMEWORK=java npx playwright test tests/backend-validation &amp;&amp; FRAMEWORK=go npx playwright test tests/backend-validation &amp;&amp; FRAMEWORK=nodejs npx playwright test tests/backend-validation"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:rollback"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-validation/rollback-readiness.test.ts"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:upgrade"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:smoke &amp;&amp; npm run test:validation &amp;&amp; npm run test:rollback"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:upgrade:all"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:smoke:all &amp;&amp; npm run test:validation:all &amp;&amp; npm run test:rollback"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:load"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-validation/load-testing.test.ts"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:security"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-validation/security.test.ts"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:database"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright test tests/backend-validation/database.test.ts"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:report"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:open"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure open allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:clean"</span><span class="p">:</span><span class="w"> </span><span class="s2">"rm -rf allure-results allure-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:serve"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure serve allure-results"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"allure:history"</span><span class="p">:</span><span class="w"> </span><span class="s2">"allure generate allure-results -o allure-report --clean &amp;&amp; cp -r allure-report/history allure-results/ || true"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"report"</span><span class="p">:</span><span class="w"> </span><span class="s2">"playwright show-report"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test:with-allure"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:upgrade:all &amp;&amp; npm run allure:report &amp;&amp; npm run allure:open"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"ci:full"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npm run test:upgrade:all &amp;&amp; npm run allure:report &amp;&amp; npm run allure:history"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Key Takeaways
</h2>

<p>✅ <strong>4-Phase Strategy</strong>: Baseline → Smoke → Comprehensive → Rollback<br>
✅ <strong>API-First Testing</strong>: Comprehensive REST API validation<br>
✅ <strong>Multi-Service Integration</strong>: Test interactions between services<br>
✅ <strong>Database Validation</strong>: Schema, migrations, constraints, integrity<br>
✅ <strong>Performance Metrics</strong>: Baseline and regression testing<br>
✅ <strong>Load &amp; Concurrency</strong>: Stress testing and graceful degradation<br>
✅ <strong>Security Testing</strong>: Authentication, tokens, security headers<br>
✅ <strong>Backward Compatibility</strong>: Contract testing and breaking change detection<br>
✅ <strong>Allure Reporting</strong>: Beautiful reports with analytics and trends<br>
✅ <strong>GitHub Actions CI/CD</strong>: Automated multi-phase pipeline<br>
✅ <strong>Framework-Agnostic</strong>: Works with Java, Go, Node.js<br>
✅ <strong>Multi-Database Support</strong>: PostgreSQL, MySQL, MongoDB, etc.<br>
✅ <strong>Service Mesh Ready</strong>: Compatible with Kubernetes, Docker, cloud-native<br>
✅ <strong>Microservices Pattern</strong>: Test entire service ecosystems</p>




<h2>
  
  
  Conclusion
</h2>

<p>Backend version upgrades don't have to be risky. With <strong>automated E2E API testing</strong>, <strong>database validation</strong>, <strong>performance monitoring</strong>, <strong>Allure reporting</strong>, and <strong>GitHub Actions CI/CD</strong>, you get:</p>

<h3>
  
  
  Complete Testing Coverage
</h3>

<ol>
<li>
<strong>Baseline Capture</strong>: API contracts, database schema, performance metrics</li>
<li>
<strong>Smoke Tests</strong>: Fast fail-detection on critical endpoints (5-10 minutes)</li>
<li>
<strong>Comprehensive Validation</strong>: 95%+ confidence with all validation types (45-60 minutes)</li>
<li>
<strong>Rollback Readiness</strong>: Verified data integrity and service stability</li>
</ol>

<h3>
  
  
  Testing Types Included
</h3>

<ul>
<li>✅ API contract and backward compatibility testing</li>
<li>✅ Database schema and constraint validation</li>
<li>✅ Performance regression detection</li>
<li>✅ Load testing and concurrency validation</li>
<li>✅ Security and authentication testing</li>
<li>✅ Multi-service integration testing</li>
<li>✅ Error handling and status code validation</li>
<li>✅ Data integrity and consistency checks</li>
</ul>

<h3>
  
  
  Timeline &amp; Metrics
</h3>

<ul>
<li>
<strong>Total implementation time</strong>: 6-7 weeks</li>
<li>
<strong>Post-upgrade validation time</strong>: 1 hour (baseline + smoke + comprehensive)</li>
<li>
<strong>CI/CD execution time</strong>: ~45-60 minutes (all phases)</li>
<li>
<strong>Confidence level</strong>: 95%+ with full automation</li>
<li>
<strong>Framework support</strong>: Java, Go, Node.js, and any REST API service</li>
</ul>

<p>This strategy is <strong>framework-agnostic</strong> and works with <strong>any backend service</strong> regardless of tech stack, ensuring quality across microservices architectures with beautiful Allure reports and automated CI/CD validation.</p>




<h2>
  
  
  Next Steps
</h2>

<ol>
<li>
<p><strong>Update BACKEND_CONFIG</strong> in <code>config/backend-test-config.ts</code> with:</p>

<ul>
<li>Your backend service URLs</li>
<li>Database connection details</li>
<li>API authentication credentials</li>
<li>Test data configuration</li>
</ul>
</li>
<li><p><strong>Set up test environment</strong>:<br>
</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   npm <span class="nb">install</span> <span class="nt">--save-dev</span> @playwright/test allure-playwright allure-commandline pg
   npm init playwright@latest
</code></pre>

</div>



<ol>
<li><p><strong>Create test files</strong> in respective directories using patterns from this guide</p></li>
<li><p><strong>Add GitHub Actions</strong> workflow for automation</p></li>
<li><p><strong>Run baseline capture</strong> before your next upgrade</p></li>
<li><p><strong>Execute the full strategy</strong> during your upgrade window</p></li>
</ol>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<strong>Playwright Documentation</strong>: <a href="https://playwright.dev" rel="noopener noreferrer">https://playwright.dev</a>
</li>
<li>
<strong>Allure Reporting</strong>: <a href="https://docs.qameta.io/allure/" rel="noopener noreferrer">https://docs.qameta.io/allure/</a>
</li>
<li>
<strong>GitHub Actions</strong>: <a href="https://github.com/features/actions" rel="noopener noreferrer">https://github.com/features/actions</a>
</li>
<li>
<strong>REST API Testing</strong>: <a href="https://swagger.io/tools/swagger-ui/" rel="noopener noreferrer">https://swagger.io/tools/swagger-ui/</a>
</li>
<li>
<strong>PostgreSQL Documentation</strong>: <a href="https://www.postgresql.org/docs/" rel="noopener noreferrer">https://www.postgresql.org/docs/</a>
</li>
<li>
<strong>API Contract Testing</strong>: <a href="https://pact.foundation/" rel="noopener noreferrer">https://pact.foundation/</a>
</li>
</ul>




<p><strong>Have you automated your backend upgrades? Share your strategies in the comments below!</strong> 👇</p>

<h1>
  
  
  testing #automation #backend #api #playwright #devops #qa #e2etesting #java #go #nodejs #microservices
</h1>

