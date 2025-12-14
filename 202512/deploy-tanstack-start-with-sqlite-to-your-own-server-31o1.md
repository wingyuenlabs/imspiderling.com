---
Title: Deploy TanStack Start with SQLite to Your Own Server
Description: 
Author: Andreas M
Date: 2025-12-14T21:33:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>This guide walks you through deploying a TanStack Start application with SQLite database to your own server. Any Linux-based VPS or dedicated server will work.</p>

<p>We'll use <a href="https://haloy.dev" rel="noopener noreferrer">Haloy</a> for deployment. A open source tool that deploys and orcestrates Docker containers on your own servers with automatic HTTPS.</p>

<p>The complete source code is available at <a href="https://github.com/haloydev/examples/tree/main/tanstack-start-sqlite" rel="noopener noreferrer">github.com/haloydev/examples/tanstack-start-sqlite</a>.</p>

<h2>
  
  
  Why This Stack?
</h2>

<p>There is no such thing as the perfect stack, but one combination I'm super excited about right now is TanStack Start and SQLite. Here's why:</p>

<ul>
<li><p>Simplicity is King. You are building an app, not managing a database cluster. SQLite runs in the same process as your app. There’s no network latency between your web server and your database. Development setup is non-existent. Run pnpm install and start coding. It’s glorious.</p></li>
<li><p>Performance out of the box. Because the database lives right next to your server code, reads and writes are incredibly fast. You get local-file-system speeds which often blow network-attached database servers out of the water for typical web requests.</p></li>
<li><p>Deployment is easy. You deploy your app server and the database file (mounted as a persistent volume) to a single region with <a href="https://haloy.dev" rel="noopener noreferrer">Haloy</a></p></li>
</ul>

<h2>
  
  
  What You'll Build
</h2>

<p>A full-stack React application using:</p>

<ul>
<li>
<strong>TanStack Start</strong> - React meta-framework with file-based routing and server functions</li>
<li>
<strong>SQLite</strong> - Lightweight, file-based database</li>
<li>
<strong>Drizzle ORM</strong> - TypeScript ORM for type-safe database queries</li>
<li>
<strong>Haloy</strong> - Simple deployment to your own server</li>
</ul>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>Node.js 20+ installed</li>
<li>Haloy installed (<a href="https://haloy.dev/docs/quickstart" rel="noopener noreferrer">Quickstart</a>)</li>
<li>A linux server (VPS or dedicated server)</li>
<li>A domain or a subdomain</li>
<li>Basic familiarity with React and TypeScript</li>
</ul>

<h2>
  
  
  Project Setup
</h2>

<h3>
  
  
  1. Initialize the Project
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>my-tanstack-app
<span class="nb">cd </span>my-tanstack-app
pnpm init
</code></pre>

</div>



<h3>
  
  
  2. Configure TypeScript
</h3>

<p>Create <code>tsconfig.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"compilerOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"jsx"</span><span class="p">:</span><span class="w"> </span><span class="s2">"react-jsx"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"moduleResolution"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Bundler"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"module"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ESNext"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"target"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ES2022"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"skipLibCheck"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"strictNullChecks"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  3. Install Dependencies
</h3>

<p>Install TanStack Start and React:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add @tanstack/react-start @tanstack/react-router react react-dom nitro
</code></pre>

</div>



<p>Install dev dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add <span class="nt">-D</span> vite @vitejs/plugin-react typescript @types/react @types/react-dom @types/node vite-tsconfig-paths
</code></pre>

</div>



<p>Install Drizzle and SQLite:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add drizzle-orm @libsql/client dotenv drizzle-kit
</code></pre>

</div>



<p><strong>Note:</strong> <code>drizzle-kit</code> is installed as a production dependency (not <code>-D</code>) because we need it available in the Docker container to run migrations at startup.</p>

<h3>
  
  
  4. Update package.json
</h3>

<p>Update your <code>package.json</code> with the required configuration and scripts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">...</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"module"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"dev"</span><span class="p">:</span><span class="w"> </span><span class="s2">"vite dev"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"build"</span><span class="p">:</span><span class="w"> </span><span class="s2">"vite build"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node .output/server/index.mjs"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"db:generate"</span><span class="p">:</span><span class="w"> </span><span class="s2">"drizzle-kit generate"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"db:migrate"</span><span class="p">:</span><span class="w"> </span><span class="s2">"drizzle-kit migrate"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  5. Create Vite Configuration
</h3>

<p>Create <code>vite.config.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">vite</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">nitro</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">nitro/vite</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">tsConfigPaths</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">vite-tsconfig-paths</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">tanstackStart</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tanstack/react-start/plugin/vite</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">viteReact</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@vitejs/plugin-react</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">server</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">port</span><span class="p">:</span> <span class="mi">3000</span><span class="p">,</span>
  <span class="p">},</span>
  <span class="na">plugins</span><span class="p">:</span> <span class="p">[</span>
    <span class="nf">tsConfigPaths</span><span class="p">(),</span>
    <span class="nf">tanstackStart</span><span class="p">(),</span>
    <span class="nf">nitro</span><span class="p">(),</span>
    <span class="c1">// react's vite plugin must come after start's vite plugin</span>
    <span class="nf">viteReact</span><span class="p">(),</span>
  <span class="p">],</span>
  <span class="na">nitro</span><span class="p">:</span> <span class="p">{},</span>
<span class="p">});</span>
</code></pre>

</div>



<h4>
  
  
  About Nitro
</h4>

<p>TanStack Start uses <a href="https://nitro.unjs.io/" rel="noopener noreferrer">Nitro</a> as its server engine. For this deployment, we're using the default Node.js preset, which works perfectly with Haloy. No additional configuration is needed. The empty <code>nitro: {}</code> object is sufficient.</p>

<h2>
  
  
  Database Setup
</h2>

<h3>
  
  
  1. Configure Drizzle
</h3>

<p>Create <code>drizzle.config.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">config</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">defineConfig</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">drizzle-kit</span><span class="dl">"</span><span class="p">;</span>

<span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">databaseUrl</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DATABASE_URL</span><span class="p">;</span>
<span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">databaseUrl</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">DATABASE_URL is not set</span><span class="dl">"</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineConfig</span><span class="p">({</span>
  <span class="na">out</span><span class="p">:</span> <span class="dl">"</span><span class="s2">./drizzle</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">schema</span><span class="p">:</span> <span class="dl">"</span><span class="s2">./src/db/schema.ts</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">dialect</span><span class="p">:</span> <span class="dl">"</span><span class="s2">sqlite</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">dbCredentials</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">url</span><span class="p">:</span> <span class="nx">databaseUrl</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  2. Create Database Client
</h3>

<p>Create <code>src/db/index.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="dl">"</span><span class="s2">dotenv/config</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@libsql/client</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">drizzle</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">drizzle-orm/libsql</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">databaseUrl</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DATABASE_URL</span><span class="p">;</span>
<span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">databaseUrl</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">DATABASE_URL is not set</span><span class="dl">"</span><span class="p">);</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="nf">createClient</span><span class="p">({</span> <span class="na">url</span><span class="p">:</span> <span class="nx">databaseUrl</span> <span class="p">});</span>
<span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="nf">drizzle</span><span class="p">({</span> <span class="nx">client</span> <span class="p">});</span>
<span class="k">export</span> <span class="p">{</span> <span class="nx">client</span><span class="p">,</span> <span class="nx">db</span> <span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  3. Define Your Schema
</h3>

<p>Create <code>src/db/schema.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">sql</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">drizzle-orm</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">integer</span><span class="p">,</span> <span class="nx">sqliteTable</span><span class="p">,</span> <span class="nx">text</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">drizzle-orm/sqlite-core</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">todos</span> <span class="o">=</span> <span class="nf">sqliteTable</span><span class="p">(</span><span class="dl">"</span><span class="s2">todos</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="nf">integer</span><span class="p">(</span><span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">mode</span><span class="p">:</span> <span class="dl">"</span><span class="s2">number</span><span class="dl">"</span> <span class="p">}).</span><span class="nf">primaryKey</span><span class="p">({</span>
    <span class="na">autoIncrement</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">}),</span>
  <span class="na">title</span><span class="p">:</span> <span class="nf">text</span><span class="p">(</span><span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">).</span><span class="nf">notNull</span><span class="p">(),</span>
  <span class="na">createdAt</span><span class="p">:</span> <span class="nf">integer</span><span class="p">(</span><span class="dl">"</span><span class="s2">created_at</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">mode</span><span class="p">:</span> <span class="dl">"</span><span class="s2">timestamp</span><span class="dl">"</span> <span class="p">}).</span><span class="k">default</span><span class="p">(</span>
    <span class="nx">sql</span><span class="s2">`(unixepoch())`</span>
  <span class="p">),</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  4. Create Environment File
</h3>

<p>Create <code>.env</code> for local development:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">DATABASE_URL</span><span class="o">=</span>file:local.db
</code></pre>

</div>



<h3>
  
  
  5. Generate and Run Migrations
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm db:generate
pnpm db:migrate
</code></pre>

</div>



<p>This creates migration files in the <code>drizzle/</code> directory that will be used in production.</p>

<h2>
  
  
  Application Code
</h2>

<h3>
  
  
  1. Create the Router
</h3>

<p>Create <code>src/router.tsx</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createRouter</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tanstack/react-router</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">routeTree</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./routeTree.gen</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">getRouter</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">createRouter</span><span class="p">({</span>
    <span class="nx">routeTree</span><span class="p">,</span>
    <span class="na">scrollRestoration</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">defaultNotFoundComponent</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>404 - not found<span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;,</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="nx">router</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Note:</strong> You might see a TypeScript error about <code>./routeTree.gen</code> not being found. This is expected. TanStack Start automatically generates this file when you run the dev server in the next steps.</p>

<h3>
  
  
  2. Create the Root Route
</h3>

<p>Create <code>src/routes/__root.tsx</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">/// &lt;reference types="vite/client" /&gt;</span>

<span class="k">import</span> <span class="p">{</span>
  <span class="nx">createRootRoute</span><span class="p">,</span>
  <span class="nx">HeadContent</span><span class="p">,</span>
  <span class="nx">Outlet</span><span class="p">,</span>
  <span class="nx">Scripts</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tanstack/react-router</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">ReactNode</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">react</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">Route</span> <span class="o">=</span> <span class="nf">createRootRoute</span><span class="p">({</span>
  <span class="na">head</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span>
    <span class="na">meta</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span>
        <span class="na">charSet</span><span class="p">:</span> <span class="dl">"</span><span class="s2">utf-8</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">},</span>
      <span class="p">{</span>
        <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">viewport</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">content</span><span class="p">:</span> <span class="dl">"</span><span class="s2">width=device-width, initial-scale=1</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">},</span>
      <span class="p">{</span>
        <span class="na">title</span><span class="p">:</span> <span class="dl">"</span><span class="s2">TanStack Start Starter</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">],</span>
  <span class="p">}),</span>
  <span class="na">component</span><span class="p">:</span> <span class="nx">RootComponent</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">function</span> <span class="nf">RootComponent</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">RootDocument</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">Outlet</span> <span class="p">/&gt;</span>
    <span class="p">&lt;/</span><span class="nc">RootDocument</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">RootDocument</span><span class="p">({</span> <span class="nx">children</span> <span class="p">}:</span> <span class="nb">Readonly</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">children</span><span class="p">:</span> <span class="nx">ReactNode</span> <span class="p">}</span><span class="o">&gt;</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">html</span> <span class="na">lang</span><span class="p">=</span><span class="s">"en"</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">head</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nc">HeadContent</span> <span class="p">/&gt;</span>
      <span class="p">&lt;/</span><span class="nt">head</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">body</span><span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">children</span><span class="si">}</span>
        <span class="p">&lt;</span><span class="nc">Scripts</span> <span class="p">/&gt;</span>
      <span class="p">&lt;/</span><span class="nt">body</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">html</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  3. Create the Index Route
</h3>

<p>Create <code>src/routes/index.tsx</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createFileRoute</span><span class="p">,</span> <span class="nx">useRouter</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tanstack/react-router</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createServerFn</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tanstack/react-start</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">eq</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">drizzle-orm</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">db</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../db</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">todos</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../db/schema</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">getTodos</span> <span class="o">=</span> <span class="nf">createServerFn</span><span class="p">({</span>
  <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GET</span><span class="dl">"</span><span class="p">,</span>
<span class="p">}).</span><span class="nf">handler</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">select</span><span class="p">().</span><span class="k">from</span><span class="p">(</span><span class="nx">todos</span><span class="p">));</span>

<span class="kd">const</span> <span class="nx">addTodo</span> <span class="o">=</span> <span class="nf">createServerFn</span><span class="p">({</span> <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span> <span class="p">})</span>
  <span class="p">.</span><span class="nf">inputValidator</span><span class="p">((</span><span class="nx">data</span><span class="p">:</span> <span class="nx">FormData</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="p">(</span><span class="nx">data</span> <span class="k">instanceof</span> <span class="nx">FormData</span><span class="p">))</span> <span class="p">{</span>
      <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Expected FormData</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">title</span><span class="p">:</span> <span class="nx">data</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">title</span><span class="dl">"</span><span class="p">)?.</span><span class="nf">toString</span><span class="p">()</span> <span class="o">||</span> <span class="dl">""</span><span class="p">,</span>
    <span class="p">};</span>
  <span class="p">})</span>
  <span class="p">.</span><span class="nf">handler</span><span class="p">(</span><span class="k">async </span><span class="p">({</span> <span class="nx">data</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">insert</span><span class="p">(</span><span class="nx">todos</span><span class="p">).</span><span class="nf">values</span><span class="p">({</span> <span class="na">title</span><span class="p">:</span> <span class="nx">data</span><span class="p">.</span><span class="nx">title</span> <span class="p">});</span>
  <span class="p">});</span>

<span class="kd">const</span> <span class="nx">deleteTodo</span> <span class="o">=</span> <span class="nf">createServerFn</span><span class="p">({</span> <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span> <span class="p">})</span>
  <span class="p">.</span><span class="nf">inputValidator</span><span class="p">((</span><span class="nx">data</span><span class="p">:</span> <span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">data</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">handler</span><span class="p">(</span><span class="k">async </span><span class="p">({</span> <span class="nx">data</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="k">delete</span><span class="p">(</span><span class="nx">todos</span><span class="p">).</span><span class="nf">where</span><span class="p">(</span><span class="nf">eq</span><span class="p">(</span><span class="nx">todos</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">data</span><span class="p">));</span>
  <span class="p">});</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">Route</span> <span class="o">=</span> <span class="nf">createFileRoute</span><span class="p">(</span><span class="dl">"</span><span class="s2">/</span><span class="dl">"</span><span class="p">)({</span>
  <span class="na">component</span><span class="p">:</span> <span class="nx">RouteComponent</span><span class="p">,</span>
  <span class="na">loader</span><span class="p">:</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="k">await</span> <span class="nf">getTodos</span><span class="p">(),</span>
<span class="p">});</span>

<span class="kd">function</span> <span class="nf">RouteComponent</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">useRouter</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">todos</span> <span class="o">=</span> <span class="nx">Route</span><span class="p">.</span><span class="nf">useLoaderData</span><span class="p">();</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">ul</span><span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">todos</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">todo</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
          <span class="p">&lt;</span><span class="nt">li</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">todo</span><span class="p">.</span><span class="nx">id</span><span class="si">}</span><span class="p">&gt;</span>
            <span class="si">{</span><span class="nx">todo</span><span class="p">.</span><span class="nx">title</span><span class="si">}</span>
            <span class="p">&lt;</span><span class="nt">button</span>
              <span class="na">type</span><span class="p">=</span><span class="s">"button"</span>
              <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
                  <span class="k">await</span> <span class="nf">deleteTodo</span><span class="p">({</span> <span class="na">data</span><span class="p">:</span> <span class="nx">todo</span><span class="p">.</span><span class="nx">id</span> <span class="p">});</span>
                  <span class="nx">router</span><span class="p">.</span><span class="nf">invalidate</span><span class="p">();</span>
                <span class="p">}</span><span class="si">}</span>
              <span class="p">&gt;</span>
                X
              <span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span>
            <span class="p">&lt;/</span><span class="nt">li</span><span class="p">&gt;</span>
          <span class="p">))</span><span class="si">}</span>
        <span class="p">&lt;/</span><span class="nt">ul</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">h2</span><span class="p">&gt;</span>Add todo<span class="p">&lt;/</span><span class="nt">h2</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">form</span>
          <span class="na">onSubmit</span><span class="p">=</span><span class="si">{</span><span class="k">async </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
            <span class="kd">const</span> <span class="nx">form</span> <span class="o">=</span> <span class="nx">e</span><span class="p">.</span><span class="nx">currentTarget</span><span class="p">;</span>
            <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">FormData</span><span class="p">(</span><span class="nx">form</span><span class="p">);</span>
            <span class="k">await</span> <span class="nf">addTodo</span><span class="p">({</span> <span class="nx">data</span> <span class="p">});</span>
            <span class="nx">router</span><span class="p">.</span><span class="nf">invalidate</span><span class="p">();</span>
            <span class="nx">form</span><span class="p">.</span><span class="nf">reset</span><span class="p">();</span>
          <span class="p">}</span><span class="si">}</span>
        <span class="p">&gt;</span>
          <span class="p">&lt;</span><span class="nt">input</span> <span class="na">name</span><span class="p">=</span><span class="s">"title"</span> <span class="na">placeholder</span><span class="p">=</span><span class="s">"Enter a new todo..."</span> <span class="p">/&gt;</span>
          <span class="p">&lt;</span><span class="nt">button</span> <span class="na">type</span><span class="p">=</span><span class="s">"submit"</span><span class="p">&gt;</span>Add<span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span>
        <span class="p">&lt;/</span><span class="nt">form</span><span class="p">&gt;</span>
      <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
    <span class="p">);</span>
  <span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Docker Configuration
</h2>

<h3>
  
  
  1. Create Dockerfile
</h3>

<p>Create <code>Dockerfile</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">node:24-slim</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>
<span class="k">ENV</span><span class="s"> PNPM_HOME="/pnpm"</span>
<span class="k">ENV</span><span class="s"> PATH="$PNPM_HOME:$PATH"</span>
<span class="k">RUN </span>corepack <span class="nb">enable</span>
<span class="k">COPY</span><span class="s"> . /app</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="k">FROM</span><span class="w"> </span><span class="s">base</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">prod-deps</span>
<span class="k">RUN </span><span class="nt">--mount</span><span class="o">=</span><span class="nb">type</span><span class="o">=</span>cache,id<span class="o">=</span>pnpm,target<span class="o">=</span>/pnpm/store pnpm <span class="nb">install</span> <span class="nt">--prod</span> <span class="nt">--frozen-lockfile</span>

<span class="k">FROM</span><span class="w"> </span><span class="s">base</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">build</span>
<span class="k">RUN </span><span class="nt">--mount</span><span class="o">=</span><span class="nb">type</span><span class="o">=</span>cache,id<span class="o">=</span>pnpm,target<span class="o">=</span>/pnpm/store pnpm <span class="nb">install</span> <span class="nt">--frozen-lockfile</span>
<span class="k">RUN </span>pnpm run build

<span class="k">FROM</span><span class="s"> base</span>
<span class="k">COPY</span><span class="s"> --from=prod-deps /app/node_modules /app/node_modules</span>
<span class="k">COPY</span><span class="s"> --from=build /app/.output /app/.output</span>
<span class="k">CMD</span><span class="s"> [ "sh", "-c", "pnpm db:migrate &amp;&amp; pnpm start" ]</span>
</code></pre>

</div>



<p>Key points:</p>

<ul>
<li>Uses multi-stage builds for smaller final image</li>
<li>Runs database migrations at container startup. This is safe because Drizzle migrations are idempotent (running them multiple times has no effect if the database is already up to date)</li>
<li>Production dependencies include <code>drizzle-kit</code> for migrations</li>
</ul>

<h3>
  
  
  2. Create .dockerignore
</h3>

<p>Create <code>.dockerignore</code>:</p>

<p>We need to exclude files from the built Docker image. Notice <code>*.db</code> is excluded. Your local database should never be copied to production. The production database lives in the persistent volume.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>node_modules
.git
.gitignore
*.md
dist
.DS_Store
*.db
</code></pre>

</div>



<h2>
  
  
  Haloy Configuration
</h2>

<p>Create <code>haloy.yml</code>:</p>

<p>This file tells the <code>haloy</code> CLI tool how to deploy your app. It's pretty simple and straightforward.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">my-tanstack-app</span>
<span class="na">server</span><span class="pi">:</span> <span class="s">your-server.haloy.dev</span>
<span class="na">domains</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">domain</span><span class="pi">:</span> <span class="s">my-app.example.com</span>
<span class="na">port</span><span class="pi">:</span> <span class="m">3000</span>
<span class="na">env</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">NODE_ENV</span>
    <span class="na">value</span><span class="pi">:</span> <span class="s">production</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">DATABASE_URL</span>
    <span class="na">value</span><span class="pi">:</span> <span class="s2">"</span><span class="s">file:/app/db-data/production.db"</span>
<span class="na">volumes</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">db-data:/app/db-data"</span>
</code></pre>

</div>



<h3>
  
  
  Configuration Explained
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Field</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>name</code></td>
<td>Unique identifier for your application</td>
</tr>
<tr>
<td><code>server</code></td>
<td>Your Haloy server domain</td>
</tr>
<tr>
<td><code>domains</code></td>
<td>Public domain(s) for your app (HTTPS is automatic)</td>
</tr>
<tr>
<td><code>port</code></td>
<td>The port your app listens on inside the container. Nitro defaults to port 3000, which matches the Vite config</td>
</tr>
<tr>
<td><code>env</code></td>
<td>Environment variables passed to your container</td>
</tr>
<tr>
<td><code>volumes</code></td>
<td>Persistent storage - <strong>critical for SQLite data</strong>
</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Volume Configuration
</h3>

<p>The <code>volumes</code> configuration is critical for SQLite. See <a href="https://haloy.dev/docs/volumes" rel="noopener noreferrer">Volumes</a> for more details on persistent storage.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">volumes</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">db-data:/app/db-data"</span>
</code></pre>

</div>



<p>This creates a named volume <code>db-data</code> mounted at <code>/app/db-data</code> inside the container. The <code>DATABASE_URL</code> points to a file in this directory, ensuring your database persists across deployments and container restarts.</p>

<h2>
  
  
  Deploy
</h2>

<h3>
  
  
  1. Test Locally
</h3>

<p>Before deploying, verify everything works locally. If you haven't already, make sure you've completed the database setup steps above (create <code>.env</code>, generate and run migrations).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm dev
</code></pre>

</div>



<p>Visit <code>http://localhost:3000</code> and try adding a todo to verify both the app and database are working correctly.</p>

<h3>
  
  
  2. Deploy with Haloy
</h3>

<p>If everything is working locally, you can now deploy to your server. Make sure you have Haloy installed and have configured your domain's DNS to point to your server. Check out the <a href="https://haloy.dev/docs/quickstart" rel="noopener noreferrer">Quickstart</a> if you haven't set it up yet.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>haloy deploy
</code></pre>

</div>



<p>Haloy will:</p>

<ol>
<li>Build your Docker image locally</li>
<li>Push it to your server</li>
<li>Run the container with your configuration</li>
<li>Set up HTTPS automatically</li>
<li>Route traffic to your app</li>
</ol>

<h3>
  
  
  3. Verify Deployment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check status</span>
haloy status

<span class="c"># View logs</span>
haloy logs
</code></pre>

</div>



<p>Your app should now be live.</p>

<h2>
  
  
  Production Considerations
</h2>

<h3>
  
  
  Database Backups
</h3>

<p>SQLite stores all data in a single file. To back up your database:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Execute a backup command in the container</span>
haloy <span class="nb">exec</span> <span class="nt">--</span> <span class="nb">cp</span> /app/db-data/production.db /app/db-data/backup-<span class="si">$(</span><span class="nb">date</span> +%Y%m%d<span class="si">)</span>.db
</code></pre>

</div>



<p>Consider setting up automated backups using a cron job or scheduled task.</p>

<h3>
  
  
  Monitoring
</h3>

<p>View your application logs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Stream logs</span>
haloy logs

<span class="c"># Check application status</span>
haloy status
</code></pre>

</div>



<h2>
  
  
  Troubleshooting
</h2>

<h3>
  
  
  Database Not Persisting
</h3>

<p>Ensure your <code>volumes</code> configuration matches your <code>DATABASE_URL</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">env</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">DATABASE_URL</span>
    <span class="na">value</span><span class="pi">:</span> <span class="s2">"</span><span class="s">file:/app/db-data/production.db"</span>  <span class="c1"># Must be inside the volume mount</span>
<span class="na">volumes</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">db-data:/app/db-data"</span>  <span class="c1"># Volume mounted here</span>
</code></pre>

</div>



<p>You can use <code>haloy exec</code> to run commands inside your container for debugging. Verify the database file exists and is being written to the correct location:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>haloy <span class="nb">exec</span> <span class="nt">--</span> <span class="nb">ls</span> <span class="nt">-la</span> /app/db-data/
</code></pre>

</div>



<h3>
  
  
  Migration Errors
</h3>

<p>If migrations fail at startup, check:</p>

<ol>
<li>The <code>drizzle/</code> directory is included in your Docker image</li>
<li>
<code>drizzle-kit</code> is a production dependency (not devDependency)</li>
<li>Logs for specific error messages: <code>haloy logs</code>
</li>
</ol>

<p>Verify the migration files are present in the container:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>haloy <span class="nb">exec</span> <span class="nt">--</span> <span class="nb">ls</span> <span class="nt">-la</span> /app/drizzle/
</code></pre>

</div>



<h3>
  
  
  Connection Issues
</h3>

<p>If you can't connect to your deployed app:</p>

<ol>
<li>Verify the domain is correctly configured: <code>haloy status</code>
</li>
<li>Check the app is running: <code>haloy logs</code>
</li>
<li>Ensure port 3000 matches your app's listening port</li>
</ol>

<h3>
  
  
  Scaling Limitations
</h3>

<p>SQLite is designed for single-server deployments. If you need to run multiple replicas of your application, you have two options:</p>

<ol>
<li>
<strong>Switch to a client-server database</strong> like PostgreSQL or MySQL</li>
<li>
<strong>Use a distributed SQLite solution</strong> like <a href="https://turso.tech/" rel="noopener noreferrer">Turso</a> or <a href="https://fly.io/docs/litefs/" rel="noopener noreferrer">LiteFS</a>
</li>
</ol>

<p>For most applications, a single replica with SQLite can handle significant traffic. Often more than you'd expect.</p>

