---
Title: Getting Started with eslint-plugin-secure-coding
Description: 
Author: Ofri Peretz
Date: 2025-12-31T21:31:41.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>75 security rules. 60 seconds to install. Full OWASP coverage.</strong></p>

<h2>
  
  
  Quick Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> eslint-plugin-secure-coding
</code></pre>

</div>



<h2>
  
  
  Flat Config (ESLint 9+)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// eslint.config.js</span>
<span class="k">import</span> <span class="nx">secureCoding</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">eslint-plugin-secure-coding</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="p">[</span><span class="nx">secureCoding</span><span class="p">.</span><span class="nx">configs</span><span class="p">.</span><span class="nx">recommended</span><span class="p">];</span>
</code></pre>

</div>



<h2>
  
  
  Run ESLint
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx eslint <span class="nb">.</span>
</code></pre>

</div>



<p>You'll see output like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>src/auth.ts
  15:3  error  🔒 CWE-798 OWASP:A02 CVSS:7.5 | Hardcoded credential detected
               Fix: Use environment variable: process.env.DATABASE_PASSWORD

src/utils.ts
  42:5  error  🔒 CWE-95 OWASP:A03 CVSS:9.8 | Dangerous <span class="nb">eval</span><span class="o">()</span> with expression
               Fix: Replace <span class="nb">eval</span><span class="o">()</span> with safer alternatives like JSON.parse<span class="o">()</span>
</code></pre>

</div>



<h2>
  
  
  Available Presets
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Balanced for most projects</span>
<span class="nx">secureCoding</span><span class="p">.</span><span class="nx">configs</span><span class="p">.</span><span class="nx">recommended</span><span class="p">;</span>

<span class="c1">// Maximum security (all 75 rules as errors)</span>
<span class="nx">secureCoding</span><span class="p">.</span><span class="nx">configs</span><span class="p">.</span><span class="nx">strict</span><span class="p">;</span>

<span class="c1">// Web application compliance</span>
<span class="nx">secureCoding</span><span class="p">.</span><span class="nx">configs</span><span class="p">[</span><span class="dl">'</span><span class="s1">owasp-top-10</span><span class="dl">'</span><span class="p">];</span>

<span class="c1">// Mobile apps (React Native)</span>
<span class="nx">secureCoding</span><span class="p">.</span><span class="nx">configs</span><span class="p">[</span><span class="dl">'</span><span class="s1">owasp-mobile-top-10</span><span class="dl">'</span><span class="p">];</span>
</code></pre>

</div>



<h2>
  
  
  Rule Overview
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Category</th>
<th>Rules</th>
<th>Examples</th>
</tr>
</thead>
<tbody>
<tr>
<td>Injection Prevention</td>
<td>11</td>
<td>eval(), command injection, GraphQL</td>
</tr>
<tr>
<td>Cryptography</td>
<td>6</td>
<td>Weak hashes, random, timing attacks</td>
</tr>
<tr>
<td>Authentication</td>
<td>3</td>
<td>Hardcoded credentials, weak passwords</td>
</tr>
<tr>
<td>Session/Cookies</td>
<td>3</td>
<td>Insecure cookies, session fixation</td>
</tr>
<tr>
<td>Data Exposure</td>
<td>5</td>
<td>PII in logs, debug code, secrets</td>
</tr>
<tr>
<td>Input Validation</td>
<td>8</td>
<td>XSS, path traversal, prototype pollution</td>
</tr>
<tr>
<td>OWASP Mobile</td>
<td>30</td>
<td>Insecure storage, certificate validation</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Customizing Rules
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// eslint.config.js</span>
<span class="k">import</span> <span class="nx">secureCoding</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">eslint-plugin-secure-coding</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="p">[</span>
  <span class="nx">secureCoding</span><span class="p">.</span><span class="nx">configs</span><span class="p">.</span><span class="nx">recommended</span><span class="p">,</span>

  <span class="c1">// Override specific rules</span>
  <span class="p">{</span>
    <span class="na">rules</span><span class="p">:</span> <span class="p">{</span>
      <span class="c1">// Downgrade to warning</span>
      <span class="dl">'</span><span class="s1">secure-coding/no-pii-in-logs</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">warn</span><span class="dl">'</span><span class="p">,</span>

      <span class="c1">// Disable if not applicable</span>
      <span class="dl">'</span><span class="s1">secure-coding/detect-non-literal-fs-filename</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">off</span><span class="dl">'</span><span class="p">,</span>

      <span class="c1">// Configure options</span>
      <span class="dl">'</span><span class="s1">secure-coding/no-hardcoded-credentials</span><span class="dl">'</span><span class="p">:</span> <span class="p">[</span>
        <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">{</span>
          <span class="na">allowTestFiles</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">],</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">];</span>
</code></pre>

</div>



<h2>
  
  
  Ignoring False Positives
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// eslint-disable-next-line secure-coding/no-hardcoded-credentials</span>
<span class="kd">const</span> <span class="nx">EXAMPLE_KEY</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">pk_test_example</span><span class="dl">'</span><span class="p">;</span> <span class="c1">// Test fixture</span>
</code></pre>

</div>



<p>Or in config:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">files</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">**/*.test.ts</span><span class="dl">'</span><span class="p">],</span>
  <span class="nx">rules</span><span class="p">:</span> <span class="p">{</span>
    <span class="dl">'</span><span class="s1">secure-coding/no-hardcoded-credentials</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">off</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  CI/CD Integration
</h2>

<h3>
  
  
  GitHub Actions
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/security.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Security Lint</span>

<span class="na">on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">push</span><span class="pi">,</span> <span class="nv">pull_request</span><span class="pi">]</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">lint</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-node@v4</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">npm ci</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">npx eslint . --max-warnings </span><span class="m">0</span>
</code></pre>

</div>



<h3>
  
  
  Pre-commit Hook
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> husky lint-staged
npx husky init
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">package.json</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"lint-staged"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"*.{js,ts}"</span><span class="p">:</span><span class="w"> </span><span class="s2">"eslint --max-warnings 0"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  IDE Integration
</h2>

<h3>
  
  
  VS Code
</h3>

<p>ESLint extension will show errors inline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🔒 CWE-798 | Hardcoded credential detected
</code></pre>

</div>



<h3>
  
  
  Cursor/Copilot
</h3>

<p>AI assistants read the structured errors and can auto-fix:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CWE-89 → Parameterized query fix
CWE-798 → Environment variable fix
</code></pre>

</div>



<h2>
  
  
  Quick Reference
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install</span>
npm <span class="nb">install</span> <span class="nt">--save-dev</span> eslint-plugin-secure-coding

<span class="c"># Config (eslint.config.js)</span>
import secureCoding from <span class="s1">'eslint-plugin-secure-coding'</span><span class="p">;</span>
<span class="nb">export </span>default <span class="o">[</span>secureCoding.configs.recommended]<span class="p">;</span>

<span class="c"># Run</span>
npx eslint <span class="nb">.</span>

<span class="c"># Fix auto-fixable issues</span>
npx eslint <span class="nb">.</span> <span class="nt">--fix</span>
</code></pre>

</div>



<h2>
  
  
  Next Steps
</h2>

<ol>
<li>
<strong>Read the rules</strong>: Each rule has detailed docs with examples</li>
<li>
<strong>Try strict mode</strong>: <code>secureCoding.configs.strict</code>
</li>
<li>
<strong>Add to CI</strong>: Block PRs with security issues</li>
<li>
<strong>Combine plugins</strong>: Add <code>eslint-plugin-pg</code>, <code>eslint-plugin-jwt</code> for specialized coverage</li>
</ol>




<p>📦 <a href="https://www.npmjs.com/package/eslint-plugin-secure-coding" rel="noopener noreferrer">npm: eslint-plugin-secure-coding</a><br>
📖 <a href="https://github.com/ofri-peretz/eslint/tree/main/packages/eslint-plugin-secure-coding/docs/rules" rel="noopener noreferrer">Full Rule List</a></p>

<p><a href="https://github.com/ofri-peretz/eslint" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">⭐ Star on GitHub</a>
<br>
📖 <a href="https://github.com/ofri-peretz/eslint/tree/main/packages/eslint-plugin-secure-coding#owasp-coverage-matrix" rel="noopener noreferrer">OWASP Coverage Matrix</a></p>

<p><a href="https://github.com/ofri-peretz/eslint" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">⭐ Star on GitHub</a>
</p>




<p>🚀 <strong>Questions? Open an issue on GitHub!</strong></p>

<p><a href="https://github.com/ofri-peretz" rel="noopener noreferrer">GitHub</a> | <a href="https://linkedin.com/in/ofri-peretz" rel="noopener noreferrer">LinkedIn</a> | <a href="https://dev.to/ofri-peretz">Dev.to</a></p>

