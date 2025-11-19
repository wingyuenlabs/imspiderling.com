---
Title: Nuxt Tutorial 5 - Middleware
Description: 
Author: Alois Sečkár
Date: 2025-11-19T21:57:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>This part of the Nuxt tutorial focuses on middleware — handlers that can be invoked automatically before rendering a given page on the <a href="https://dev.to/aloisseckar/nuxt-tutorial-2-components-pages-432g">frontend</a> or before processing data on the <a href="https://dev.to/aloisseckar/nuxt-tutorial-4-server-side-2dal">server side</a>.</p>

<h2>
  
  
  /app/middleware
</h2>

<p>Client-side middleware files live in the <code>/app/middleware</code> folder, from which Nuxt automatically loads them.</p>

<p>Middleware runs during navigation (routing). For a file to be recognized as middleware, it must <code>export default</code> the <code>defineNuxtRouteMiddleware</code> method. The handler receives <code>to</code> (destination) and <code>from</code> (origin) route objects. The syntax looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineNuxtRouteMiddleware</span><span class="p">((</span><span class="nx">to</span><span class="p">,</span> <span class="k">from</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// logic</span>
<span class="p">})</span>
</code></pre>

</div>



<p>The type of <code>to</code> and <code>from</code> params comes from <a href="https://router.vuejs.org/" rel="noopener noreferrer">Vue Router</a> and is called <a href="https://router.vuejs.org/api/interfaces/RouteLocationNormalizedGeneric.html" rel="noopener noreferrer"><code>RouteLocationNormalizedGeneric</code></a>. Through these objects you can access all key navigation info — URL, query parameters (<code>?</code>), hash (<code>#</code>), etc. Inside the function you can also use any <a href="https://dev.to/aloisseckar/nuxt-tutorial-3-utils-composables-3o51">composables and util functions</a>.</p>

<h3>
  
  
  Attaching middleware
</h3>

<p>Once you have a handler, you need to tell the framework which page(s) it should run for. Typically, this is done in a page’s <code>&lt;script setup&gt;</code> block via the <code>definePageMeta</code> macro. A brief side-note on “macros”: they look like ordinary functions, but are special compile-time constructs — essentially keywords — that the Nuxt compiler understands and replaces with actual JS code.</p>

<p>Here’s an example of assigning middleware to a specific page:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight vue"><code><span class="nt">&lt;</span><span class="k">script</span> <span class="na">setup</span> <span class="na">lang=</span><span class="s">"ts"</span><span class="nt">&gt;</span>
<span class="nf">definePageMeta</span><span class="p">({</span>
  <span class="na">middleware</span><span class="p">:</span> <span class="dl">'</span><span class="s1">foo</span><span class="dl">'</span>
<span class="p">})</span>
<span class="nt">&lt;/</span><span class="k">script</span><span class="nt">&gt;</span>
</code></pre>

</div>



<p>This ensures the middleware defined in <code>/app/middleware/foo.ts</code> runs before the page renders.</p>

<p>You can also attach multiple middleware handlers by passing an array instead of a single string:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight vue"><code><span class="nt">&lt;</span><span class="k">script</span> <span class="na">setup</span> <span class="na">lang=</span><span class="s">"ts"</span><span class="nt">&gt;</span>
<span class="nf">definePageMeta</span><span class="p">({</span>
  <span class="na">middleware</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">foo1</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">foo2</span><span class="dl">'</span><span class="p">]</span>
<span class="p">})</span>
<span class="nt">&lt;/</span><span class="k">script</span><span class="nt">&gt;</span>
</code></pre>

</div>



<p>It’s even possible to define “inline” middleware directly <a href="https://nuxt.com/docs/4.x/api/utils/define-page-meta#defining-middleware" rel="noopener noreferrer">within <code>definePageMeta</code></a>, but I believe this tends to obscure things since you have to remember it’s there.</p>

<p>If you want a handler to run on every page, writing <code>definePageMeta</code> on each page would quickly get tedious. Instead, you can create a “global” middleware by adding the <code>.global</code> suffix to the filename (quite similar to defining HTTP method in API endpoints). While <code>/app/middleware/foo.ts</code> must be explicitly referenced, <code>/app/middleware/foo.global.ts</code> executes automatically on routing to any page.</p>

<p>Since Nuxt <strong>3.11</strong>, you can also configure middleware centrally in <code>nuxt.config.ts</code> via <code>routeRules</code> (client-side only, not for the Nitro server routes).</p>

<h3>
  
  
  Example use case
</h3>

<p>A typical use case is authorization — checking whether the current user may view the page. If not, redirect them to login. Use the Nuxt helper <code>navigateTo</code> and be sure to <code>return</code> it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineNuxtRouteMiddleware</span><span class="p">((</span><span class="nx">to</span><span class="p">,</span> <span class="k">from</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nf">userCanView</span><span class="p">(</span><span class="nx">to</span><span class="p">.</span><span class="nx">path</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nf">navigateTo</span><span class="p">(</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">)</span>
  <span class="p">}</span>
<span class="p">})</span>

<span class="kd">function</span> <span class="nf">userCanView</span><span class="p">(</span><span class="nx">page</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">boolean</span> <span class="p">{</span>
  <span class="c1">// access control logic</span>
  <span class="k">return</span> <span class="kc">true</span>
<span class="p">}</span>
</code></pre>

</div>



<p>If you <strong>don't want</strong> to redirect, just let the function end without returning anything. Nuxt will then continue and navigate to the page.</p>

<p>Other uses include audit logging or initializing resources required by the target page.</p>

<p><strong>Final note on order:</strong> Global middleware runs first (alphabetically), followed by page-specific middleware (in the order listed for the page). Keep this in mind if ordering matters.</p>

<p>📗<a href="https://nuxt.com/docs/4.x/directory-structure/app/middleware" rel="noopener noreferrer">Official documentation</a></p>

<h2>
  
  
  /server/middleware
</h2>

<p>There’s a similar mechanism for the Nuxt server. Files go into <code>/server/middleware</code> folder, where they are automatically registered and executed.</p>

<p>Since <code>to</code> and <code>from</code> aren’t meaningful here, server middleware is defined differently:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineEventHandler</span><span class="p">((</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// logic</span>
<span class="p">})</span>
</code></pre>

</div>



<p>The <code>event</code> parameter contains the JS server runtime context and the incoming HTTP request data (<code>event.node.req</code>).</p>

<p>Unlike client-side middleware, server middleware runs before <strong>every</strong> API request. It executes before <strong>any</strong> API endpoint starts handling the request. It shouldn’t return a value, and should at most augment the incoming request rather than changing original data.</p>

<p>📗<a href="https://nuxt.com/docs/4.x/directory-structure/server#server-middleware" rel="noopener noreferrer">Official documentation</a></p>

<h2>
  
  
  Case Study
</h2>

<p>In a project we've built at work, client middleware is used, for example, to:</p>

<ul>
<li>verify login status and redirect unauthenticated users to the login page</li>
<li>set the window title based on the current URL</li>
</ul>

<h2>
  
  
  Demo Project
</h2>

<p>You can find the reference implementation here:<br>
<a href="https://github.com/AloisSeckar/demos-nuxt/tree/main/nuxt-middleware" rel="noopener noreferrer">nuxt-middleware @ GitHub</a></p>

<p>Four middleware functions are defined:</p>

<ul>
<li>
<code>/app/middleware/allRoutes.global.ts</code> — runs before every navigation; logs origin and destination to the console</li>
<li>
<code>/app/middleware/onlySecond.ts</code> — runs only before loading the <code>/second</code> page; logs an explanation to the console</li>
<li>
<code>/app/middleware/onlyThird.ts</code> — runs only before loading the <code>/third</code> page; logs an explanation to the console; configured via <code>nuxt.config.ts</code>
</li>
<li>
<code>/server/middleware/server.ts</code> — runs before processing each API request; logs the URL (on the server, not in the browser)</li>
</ul>

<p>The simple client lets you switch between links and observe console output. Triggering an API call is done by clicking the buttons on <code>/first</code> or <code>/second</code>.</p>

<h2>
  
  
  Summary
</h2>

<p>Middleware is great for implementing logic that should run BEFORE server API processing or client page rendering. Client middleware can be bound to specific pages or configured to run globally before any page load. Server middleware always runs before every API call.</p>

<p>By now we’ve covered a large part of Nuxt’s fundamentals. We’ve intentionally spent little time on the underlying JavaScript framework, <a href="https://vuejs.org/" rel="noopener noreferrer">Vue.js</a>, on which everything is built. We’ve touched on <a href="https://dev.to/aloisseckar/nuxt-tutorial-2-components-pages-432g#components-in-general">components</a> and <a href="https://dev.to/aloisseckar/nuxt-tutorial-3-utils-composables-3o51">composables</a> without going deep. While we won’t match the depth of the <a href="https://vuejs.org/guide/introduction.html" rel="noopener noreferrer">official Vue.js documentation</a>, we’ll address the essentials in the forthcomming <em>Vue.js intermezzo</em>.</p>

