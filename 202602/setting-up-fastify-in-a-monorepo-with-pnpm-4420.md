---
Title: Setting Up Fastify in a Monorepo with pnpm
Description: 
Author: Antonio Tripodi
Date: 2026-02-18T21:37:10.000Z
Robots: noindex,nofollow
Template: index
---
<p>A complete guide to create and configure a Fastify project within a monorepo managed with pnpm workspace.</p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>Node.js (v22 or higher)</li>
<li>pnpm installed</li>
</ul>

<h2>
  
  
  Monorepo Structure
</h2>

<p>The final structure will look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>app-monorepo/
├── package.json
├── pnpm-workspace.yaml
├── apps/
│   └── api/
│       ├── package.json
│       ├── tsconfig.json
│       └── src/
│           ├── app.ts           
│           ├── server.ts        
│           ├── routes/
│           │   ├── root.ts
│           │   └── users/
│           │       └── index.ts
│           └── plugins/
│               ├── cors.ts
│               ├── helmet.ts
│               └── sensible.ts
└── tsconfig.json
</code></pre>

</div>



<blockquote>
<p><strong>Note:</strong> Separating <code>app.ts</code> from <code>server.ts</code> is an official Fastify convention. It allows you to test the app without starting the HTTP server.</p>
</blockquote>




<h2>
  
  
  Initialize the Monorepo
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>app-monorepo
<span class="nb">cd </span>app-monorepo
pnpm init
</code></pre>

</div>



<h2>
  
  
  Configure pnpm Workspace
</h2>

<p>Create the <code>pnpm-workspace.yaml</code> file in the root:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">packages</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s1">'</span><span class="s">apps/*'</span>
</code></pre>

</div>



<h2>
  
  
  Configure TypeScript (Root)
</h2>

<p>Create the <code>tsconfig.json</code> file in the root:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"compilerOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"target"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ES2022"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"module"</span><span class="p">:</span><span class="w"> </span><span class="s2">"commonjs"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"lib"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"ES2022"</span><span class="p">],</span><span class="w">
    </span><span class="nl">"moduleResolution"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"esModuleInterop"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"strict"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"skipLibCheck"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"forceConsistentCasingInFileNames"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"resolveJsonModule"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"declaration"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"declarationMap"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"sourceMap"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"composite"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"exclude"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"**/node_modules"</span><span class="p">,</span><span class="w"> </span><span class="s2">"**/dist"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Create the API Package
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> apps/api/src/<span class="o">{</span>routes/users,plugins<span class="o">}</span>
<span class="nb">cd </span>apps/api
pnpm init
</code></pre>

</div>



<p><code>apps/api/package.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"@monorepo/api"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"commonjs"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"dev"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tsx watch src/server.ts"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"build"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tsc"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node dist/server.js"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node --test"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"type-check"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tsc --noEmit"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"dependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"fastify"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^5.7.4"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@fastify/autoload"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^6.3.1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@fastify/cors"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^11.2.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@fastify/helmet"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^13.0.2"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@fastify/sensible"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^6.0.4"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"close-with-grace"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^2.4.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"fastify-plugin"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^5.1.0"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"devDependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"@types/node"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^25.2.3"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"pino-pretty"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^13.1.3"</span><span class="w">
    </span><span class="nl">"tsx"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^4.21.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"typescript"</span><span class="p">:</span><span class="w"> </span><span class="s2">"~5.9.3"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Now run this command from the terminal<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm <span class="nb">install</span>
</code></pre>

</div>



<h2>
  
  
  Configure TypeScript for the API
</h2>

<p><code>apps/api/tsconfig.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"extends"</span><span class="p">:</span><span class="w"> </span><span class="s2">"../../tsconfig.json"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"compilerOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"outDir"</span><span class="p">:</span><span class="w"> </span><span class="s2">"./dist"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"rootDir"</span><span class="p">:</span><span class="w"> </span><span class="s2">"./src"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"tsBuildInfoFile"</span><span class="p">:</span><span class="w"> </span><span class="s2">"./dist/.tsbuildinfo"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"composite"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"include"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"src/**/*"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"exclude"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"node_modules"</span><span class="p">,</span><span class="w"> </span><span class="s2">"dist"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">

</span></code></pre>

</div>



<h2>
  
  
  Create the App Factory
</h2>

<p>The official Fastify convention is to separate <code>app.ts</code> (the factory that registers plugins and routes) from <code>server.ts</code> (the entry point that starts the server).</p>

<h2>
  
  
  Create the Server Entry Point
</h2>

<p><code>close-with-grace</code> is the package recommended by the Fastify team (created by Matteo Collina, core maintainer) for graceful shutdown. It handles <code>SIGINT</code>, <code>SIGTERM</code>, and <code>uncaughtException</code> in a single place, and the configurable <code>delay</code> gives in-flight requests time to complete before the server closes.</p>

<p>Install it first:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm <span class="nt">--filter</span> @monorepo/api add close-with-grace
</code></pre>

</div>



<p><code>apps/api/src/server.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">Fastify</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">closeWithGrace</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">close-with-grace</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">app</span><span class="p">,</span> <span class="p">{</span> <span class="nx">options</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./app</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Load .env file</span>
<span class="k">try</span> <span class="p">{</span>
  <span class="nx">process</span><span class="p">.</span><span class="nf">loadEnvFile</span><span class="p">();</span>
<span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
  <span class="c1">// .env is optional</span>
<span class="p">}</span>

<span class="c1">// Instantiate Fastify with options exported from app.ts (logger included)</span>
<span class="kd">const</span> <span class="nx">server</span> <span class="o">=</span> <span class="nc">Fastify</span><span class="p">(</span><span class="nx">options</span><span class="p">);</span>

<span class="c1">// Register the app as a plugin</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">app</span><span class="p">);</span>

<span class="c1">// Graceful shutdown configuration</span>
<span class="kd">const</span> <span class="nx">closeListeners</span> <span class="o">=</span> <span class="nf">closeWithGrace</span><span class="p">(</span>
  <span class="p">{</span> <span class="na">delay</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">FASTIFY_CLOSE_GRACE_DELAY</span> <span class="o">??</span> <span class="dl">'</span><span class="s1">500</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="mi">500</span> <span class="p">},</span>
  <span class="k">async</span> <span class="nf">function </span><span class="p">({</span> <span class="nx">err</span> <span class="p">})</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">server</span><span class="p">.</span><span class="nx">log</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">await</span> <span class="nx">server</span><span class="p">.</span><span class="nf">close</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">);</span>

<span class="nx">server</span><span class="p">.</span><span class="nf">addHook</span><span class="p">(</span><span class="dl">'</span><span class="s1">onClose</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">closeListeners</span><span class="p">.</span><span class="nf">uninstall</span><span class="p">();</span>
<span class="p">});</span>

<span class="c1">// Start listening</span>
<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">FASTIFY_PORT</span> <span class="o">??</span> <span class="dl">'</span><span class="s1">3000</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">listen</span><span class="p">({</span> <span class="na">port</span><span class="p">:</span> <span class="nx">PORT</span><span class="p">,</span> <span class="na">host</span><span class="p">:</span> <span class="dl">'</span><span class="s1">0.0.0.0</span><span class="dl">'</span> <span class="p">},</span> <span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">server</span><span class="p">.</span><span class="nx">log</span><span class="p">.</span><span class="nf">error</span><span class="p">({</span> <span class="nx">err</span> <span class="p">},</span> <span class="dl">'</span><span class="s1">Server shutdown due to an error</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>For this to work, <code>app.ts</code> must also export a <code>options</code> object with the Fastify configuration (including the logger):</p>

<p><code>apps/api/src/app.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">AutoLoad</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@fastify/autoload</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">FastifyInstance</span><span class="p">,</span> <span class="nx">FastifyPluginOptions</span><span class="p">,</span> <span class="nx">FastifyServerOptions</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Fastify server options</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">options</span><span class="p">:</span> <span class="nx">FastifyServerOptions</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">logger</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">level</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LOG_LEVEL</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">debug</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">transport</span><span class="p">:</span>
      <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LOG_LEVEL</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">debug</span><span class="dl">'</span>
        <span class="p">?</span> <span class="p">{</span>
            <span class="na">target</span><span class="p">:</span> <span class="dl">'</span><span class="s1">pino-pretty</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">options</span><span class="p">:</span> <span class="p">{</span>
              <span class="na">translateTime</span><span class="p">:</span> <span class="dl">'</span><span class="s1">HH:MM:ss Z</span><span class="dl">'</span><span class="p">,</span>
              <span class="na">ignore</span><span class="p">:</span> <span class="dl">'</span><span class="s1">pid,hostname</span><span class="dl">'</span><span class="p">,</span>
              <span class="na">colorize</span><span class="p">:</span> <span class="kc">true</span>
            <span class="p">}</span>
          <span class="p">}</span>
        <span class="p">:</span> <span class="kc">undefined</span>
  <span class="p">}</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">app</span><span class="p">(</span>
  <span class="nx">fastify</span><span class="p">:</span> <span class="nx">FastifyInstance</span><span class="p">,</span>
  <span class="nx">opts</span><span class="p">:</span> <span class="nx">FastifyPluginOptions</span>
<span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Automatically load all plugins from the plugins/ folder</span>
  <span class="k">await</span> <span class="nx">fastify</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">AutoLoad</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">dir</span><span class="p">:</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">__dirname</span><span class="p">,</span> <span class="dl">'</span><span class="s1">plugins</span><span class="dl">'</span><span class="p">),</span>
    <span class="na">options</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">opts</span> <span class="p">},</span>
  <span class="p">});</span>

  <span class="c1">// Automatically load all routes from the routes/ folder</span>
  <span class="k">await</span> <span class="nx">fastify</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">AutoLoad</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">dir</span><span class="p">:</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">__dirname</span><span class="p">,</span> <span class="dl">'</span><span class="s1">routes</span><span class="dl">'</span><span class="p">),</span>
    <span class="na">options</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">opts</span> <span class="p">},</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p><strong>Note:</strong> <code>pino-pretty</code> is only enabled outside of production. In production, raw JSON logs are more efficient and better supported by log aggregators.</p>
</blockquote>

<h2>
  
  
  Create Plugins
</h2>

<p>Plugins follow the convention of using <code>fastify-plugin</code> to expose decorations to the global context.</p>

<p><code>apps/api/src/plugins/cors.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">fp</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify-plugin</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">FastifyInstance</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">Cors</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@fastify/cors</span><span class="dl">'</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">corsPlugin</span><span class="p">(</span><span class="nx">fastify</span><span class="p">:</span> <span class="nx">FastifyInstance</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">fastify</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">Cors</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">origin</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">methods</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">PUT</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">PATCH</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">DELETE</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">credentials</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">fp</span><span class="p">(</span><span class="nx">corsPlugin</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">cors</span><span class="dl">'</span>
<span class="p">});</span>
</code></pre>

</div>



<p><code>apps/api/src/plugins/helmet.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">fp</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify-plugin</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">FastifyInstance</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">helmet</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@fastify/helmet</span><span class="dl">'</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">helmetPlugin</span><span class="p">(</span><span class="nx">fastify</span><span class="p">:</span> <span class="nx">FastifyInstance</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">fastify</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">helmet</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">crossOriginResourcePolicy</span><span class="p">:</span> <span class="p">{</span> <span class="na">policy</span><span class="p">:</span> <span class="dl">'</span><span class="s1">same-origin</span><span class="dl">'</span> <span class="p">},</span>
    <span class="na">crossOriginEmbedderPolicy</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="c1">// Other Helmet options </span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">fp</span><span class="p">(</span><span class="nx">helmetPlugin</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">helmet</span><span class="dl">'</span>
<span class="p">});</span>

</code></pre>

</div>



<p><code>apps/api/src/plugins/sensible.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">fp</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify-plugin</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">sensible</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@fastify/sensible</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">fp</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">fastify</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">fastify</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="nx">sensible</span><span class="p">);</span>
<span class="p">});</span>

</code></pre>

</div>



<blockquote>
<p><strong>Why <code>fastify-plugin</code>?</strong> Without it, each plugin has its own encapsulated scope. With <code>fastify-plugin</code>, decorations and hooks are exposed to the parent context, making them globally available.</p>
</blockquote>

<h2>
  
  
  Create Routes
</h2>

<p>Each file in the <code>routes/</code> folder is automatically loaded by <code>@fastify/autoload</code>. The folder structure mirrors the route paths.</p>

<p><code>apps/api/src/routes/root.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">FastifyInstance</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">async</span> <span class="nf">function </span><span class="p">(</span><span class="nx">fastify</span><span class="p">:</span> <span class="nx">FastifyInstance</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">fastify</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="nx">reply</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Hello from Fastify in monorepo!</span><span class="dl">'</span> <span class="p">};</span>
  <span class="p">});</span>

  <span class="nx">fastify</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/health</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="nx">reply</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ok</span><span class="dl">'</span><span class="p">,</span> <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">()</span> <span class="p">};</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p><code>apps/api/src/routes/users/index.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">FastifyInstance</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fastify</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">async</span> <span class="nf">function </span><span class="p">(</span><span class="nx">fastify</span><span class="p">:</span> <span class="nx">FastifyInstance</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">fastify</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="nx">reply</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Alice</span><span class="dl">'</span><span class="p">,</span> <span class="na">email</span><span class="p">:</span> <span class="dl">'</span><span class="s1">alice@example.com</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Bob</span><span class="dl">'</span><span class="p">,</span> <span class="na">email</span><span class="p">:</span> <span class="dl">'</span><span class="s1">bob@example.com</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">];</span>
  <span class="p">});</span>

  <span class="nx">fastify</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/:id</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="nx">reply</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="nx">id</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">params</span> <span class="k">as</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span> <span class="p">};</span>

    <span class="k">if </span><span class="p">(</span><span class="nf">isNaN</span><span class="p">(</span><span class="nf">parseInt</span><span class="p">(</span><span class="nx">id</span><span class="p">)))</span> <span class="p">{</span>
      <span class="k">return</span> <span class="nx">reply</span><span class="p">.</span><span class="nf">badRequest</span><span class="p">(</span><span class="dl">'</span><span class="s1">ID must be a number</span><span class="dl">'</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="nf">parseInt</span><span class="p">(</span><span class="nx">id</span><span class="p">),</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">User </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">id</span> <span class="p">};</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p><strong>Note:</strong> <code>@fastify/autoload</code> automatically maps <code>routes/users/index.ts</code> to the <code>/users</code> path. No need to register routes manually in <code>app.ts</code>.</p>
</blockquote>

<h2>
  
  
  Root Scripts
</h2>

<p>Root <code>package.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"dev"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pnpm --filter @monorepo/api dev"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"build"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pnpm -r build"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pnpm --filter @monorepo/api start"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"test"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pnpm --filter @monorepo/api test"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"type-check"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pnpm -r type-check"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Example of Useful Commands
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Development</span>
pnpm dev

<span class="c"># Build all packages</span>
pnpm build

<span class="c"># Production</span>
pnpm start

<span class="c"># Test</span>
pnpm <span class="nb">test</span>

<span class="c"># Add a dependency to the API</span>
pnpm <span class="nt">--filter</span> @monorepo/api add fastify-plugin

<span class="c"># Add a dev dependency</span>
pnpm <span class="nt">--filter</span> @monorepo/api add <span class="nt">-D</span> @types/node
</code></pre>

</div>



<p>I hope this guide helps you set up a Fastify monorepo. This post will be continuously updated as improvements are made.</p>

