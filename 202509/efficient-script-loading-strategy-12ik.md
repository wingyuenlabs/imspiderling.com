---
Title: Efficient script loading strategy
Description: 
Author: Yoriiis
Date: 2025-09-28T22:08:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>Loading JavaScript efficiently is critical for any website's performance.</p>

<p>At Prisma Media, we made this transition several years ago, and the solution we use today remains reliable and effective across all our sites.</p>

<p>Our pages must load scripts in a defined order while bundling resources efficiently.</p>

<p>The typical sequence is:</p>

<ol>
<li>Consent management</li>
<li>User connection</li>
<li>Site resources (e.g. styles, scripts)</li>
<li>Third-party scripts (e.g. Google Tag Manager, analytics)</li>
</ol>




<h2>
  
  
  Before 🦖: a JavaScript loader to control execution
</h2>

<p>Back in 2018 we relied on <a href="https://www.npmjs.com/package/scriptjs" rel="noopener noreferrer">scriptjs</a>, a small library that lets you load scripts programmatically and control execution order. We placed our calls to <code>scriptjs</code> in a <code>&lt;script&gt;</code> tag just before <code>&lt;/body&gt;</code> and listed each resource in the exact order we needed.</p>

<p>But two problems became clear over time:</p>

<ul>
<li>Performance: page rendering was delayed</li>
<li>Maintenance: the library was no longer actively maintained</li>
</ul>

<p>Those issues pushed us to rethink script loading entirely.</p>




<h2>
  
  
  Back to basics
</h2>

<p>After moving away from a JavaScript loader, our goal was to rely on the most <strong>native browser features</strong> possible. The reasoning was simple:</p>

<ul>
<li>The closer we stay to native HTML and browser standards, the more aligned we are with long-term best practices</li>
<li>We no longer wanted to depend on a third-party library that might require another migration in a few years</li>
<li>The solution had to be sustainable and easy to deploy across all our websites</li>
</ul>

<p>The obvious choice was to use regular <code>&lt;script src="…"&gt;</code> tags with the right attributes.</p>

<p>As explained brilliantly in <a href="https://flaviocopes.com/javascript-async-defer" rel="noopener noreferrer">Flavio Copes' article on <code>async</code> and <code>defer</code></a>, the best strategy is to place scripts in the <code>&lt;head&gt;</code> <strong>with</strong> the <code>defer</code> attribute.</p>

<p>This placement enables scripts to download in parallel while the browser continues parsing. It ensures scripts execute in order after parsing and makes the DOM interactive sooner.</p>




<h2>
  
  
  Handling our loading sequence
</h2>

<p>For scripts such as consent management, user connection, or third-party scripts served from a CDN, the transition was straightforward. We simply added <code>&lt;script defer src="…"&gt;&lt;/script&gt;</code> in the desired order.</p>

<p>The bigger challenge came from our own resources. Our platform is a multi-page application (MPA), not a single-page app (SPA). Our websites are server-rendered with Twig templates and we use Webpack to package all resources (styles, scripts, etc.).</p>

<p>Back in 2018 we had a single entry point for the entire site. That setup generated a large JavaScript file shared across all pages, even though each page had different needs. There was no code splitting or shared-chunk optimization.</p>

<p>When we decided to rely on native <code>&lt;script&gt;</code> tags, we wanted to align with our MPA architecture by producing page-specific bundles instead of a single monolithic file. Generating the right <code>&lt;link&gt;</code> and <code>&lt;script&gt;</code> tags for each page could not be done manually, so we needed a webpack plugin to generate them automatically at build time.</p>

<p>At that time the popular <a href="https://github.com/jantimon/html-webpack-plugin" rel="noopener noreferrer">html-webpack-plugin</a> assumed a JavaScript-based template and didn't fit our needs.</p>

<p>To bridge the gap I created <a href="https://github.com/yoriiis/chunks-webpack-plugin" rel="noopener noreferrer">chunks-webpack-plugin</a>: a Webpack plugin that analyzes entry point dependencies and generates HTML fragments with the appropriate <code>&lt;link&gt;</code> and <code>&lt;script&gt;</code> tags.</p>

<p>This kept the approach fully native, just standard <code>&lt;script&gt;</code> elements, while enabling per-page bundles with granular chunk splitting. For details on that optimization step you can read <a href="https://dev.to/yoriiis/granular-chunks-and-javascript-modules-for-faster-page-loads-4pd9">Granular chunks and JavaScript modules for faster page loads</a>.</p>




<h2>
  
  
  💡 Key notes:
</h2>

<ul>
<li>Modules: <code>&lt;script type="module"&gt;</code> are deferred by default, the <code>defer</code> attribute is unnecessary</li>
<li>Legacy fallback: use <code>nomodule</code> attribute for polyfills or legacy bundles; modern browsers will skip them automatically</li>
<li>Attribute precedence: if both <code>async</code> and <code>defer</code> are set, modern browsers give priority to <code>async</code>
</li>
<li>Inline scripts: <code>async</code> and <code>defer</code> have no effect on inline scripts (those without a <code>src</code> attribute)</li>
</ul>




<h2>
  
  
  HTML rendering
</h2>

<p>Below is a simplified example of the final markup. Each script follows the execution order described above.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;title&gt;</span>Home<span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;script </span><span class="na">defer</span> <span class="na">src=</span><span class="s">"consent.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script </span><span class="na">defer</span> <span class="na">src=</span><span class="s">"user-connection.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="c">&lt;!-- Page-specific bundle generated by Webpack --&gt;</span>
    <span class="nt">&lt;script </span><span class="na">defer</span> <span class="na">src=</span><span class="s">"home.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script </span><span class="na">defer</span> <span class="na">src=</span><span class="s">"gtm.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="c">&lt;!-- Page-specific content goes here --&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<h2>
  
  
  Maintaining efficient and reliable script loading
</h2>

<ul>
<li>Performance: measurable improvements in First Contentful Paint (FCP) and in Largest Contentful Paint (LCP), thanks to deferred, non-blocking script loading</li>
<li>Standards and maintainability: scripts are delivered as standard <code>&lt;script&gt;</code> elements, generated by Webpack, ensuring consistent behavior across all pages</li>
<li>Best practices: we consistently favor <code>&lt;script&gt;</code> tags with a <code>src</code> attribute over inline JavaScript blocks. This makes scripts cacheable, easier to debug, and aligned with browser parsing rules</li>
</ul>

<p>By combining native <code>&lt;script&gt;</code> tags and a controlled build-time generation, we ensure predictable execution, page-specific bundles, and a scalable, reliable strategy across all websites.</p>




