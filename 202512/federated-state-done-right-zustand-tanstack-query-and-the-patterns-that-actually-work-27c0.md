---
Title: Federated State Done Right: Zustand, TanStack Query, and the Patterns That Actually Work
Description: 
Author: martin rojas
Date: 2025-12-16T20:53:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>We've all been there: you set up Module Federation, split your app into micro-frontends, and suddenly your Zustand store updates in one module but not another. Or worse—your TanStack Query cache fetches the same user profile three times because each remote thinks it's alone in the world.</p>

<p>The patterns that work beautifully in monolithic React apps break down in federated architectures. Context providers don't cross module boundaries. Stores instantiate twice. Cache invalidation becomes a distributed systems problem you didn't sign up for.</p>

<p>This guide covers the patterns that actually work in production—singleton configuration that prevents duplicate instances, cache-sharing strategies that don't create tight coupling, and the critical separation between client state (Zustand) and server state (TanStack Query) that makes federated apps maintainable. These aren't theoretical recommendations; they're lessons from teams at Zalando, PayPal, and other organizations running Module Federation at scale.</p>

<h2>
  
  
  Why Federated State Breaks: The Memory Model Problem
</h2>

<p>In a monolithic SPA, memory is a contiguous, shared resource. A Redux store or React Context provider instantiated at the root is universally accessible. In a federated system, the application is composed of distinct JavaScript bundles—often developed by different teams, deployed at different times, and loaded asynchronously at runtime. These bundles execute within the same browser tab, yet they're separated by distinct closure scopes and dependency trees.</p>

<p>The root cause of most Module Federation state issues is deceptively simple: without explicit singleton configuration, each federated module gets its own instance of React, Redux, or Zustand. Users experience this as authentication that works in one section but not another, theme toggles that affect only part of the interface, or cart items that vanish when navigating between micro-frontends.</p>

<h3>
  
  
  The Share Scope Engine
</h3>

<p>The engine powering state sharing is the <code>__webpack_share_scopes__</code> global object—an internal Webpack API that acts as a registry for all shared modules in the browser session.</p>

<p>When the Host application bootstraps, it initializes entries in <code>__webpack_share_scopes__.default</code> for every library marked as <code>shared</code>. Each entry contains the version number and factory function to load the module. When a Remote application initializes, it performs a handshake: inspecting the share scope, comparing available versions against its requirements, and using semantic versioning resolution to determine compatibility.</p>

<p>If the Host provides React 18.2.0 and the Remote requires <code>^18.0.0</code>, the runtime determines compatibility and the Remote uses the Host's reference. This "Reference Sharing" ensures that when the Remote calls <code>React.useContext</code>, it accesses the exact same Context Registry as the Host. If this handshake fails, the Remote loads its own React, creating a parallel universe where the Host's providers don't exist.</p>

<h3>
  
  
  Configuration for Singleton Enforcement
</h3>

<p>The fix requires explicit singleton configuration in webpack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// webpack.config.js - Every federated module needs this</span>
<span class="kd">const</span> <span class="nx">deps</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">./package.json</span><span class="dl">'</span><span class="p">).</span><span class="nx">dependencies</span><span class="p">;</span>

<span class="k">new</span> <span class="nc">ModuleFederationPlugin</span><span class="p">({</span>
  <span class="na">shared</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">react</span><span class="p">:</span> <span class="p">{</span> 
      <span class="na">singleton</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> 
      <span class="na">requiredVersion</span><span class="p">:</span> <span class="nx">deps</span><span class="p">.</span><span class="nx">react</span><span class="p">,</span>
      <span class="na">strictVersion</span><span class="p">:</span> <span class="kc">true</span> 
    <span class="p">},</span>
    <span class="dl">'</span><span class="s1">react-dom</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> 
      <span class="na">singleton</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> 
      <span class="na">requiredVersion</span><span class="p">:</span> <span class="nx">deps</span><span class="p">[</span><span class="dl">'</span><span class="s1">react-dom</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">strictVersion</span><span class="p">:</span> <span class="kc">true</span> 
    <span class="p">},</span>
    <span class="na">zustand</span><span class="p">:</span> <span class="p">{</span> <span class="na">singleton</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">requiredVersion</span><span class="p">:</span> <span class="nx">deps</span><span class="p">.</span><span class="nx">zustand</span> <span class="p">},</span>
    <span class="dl">'</span><span class="s1">@tanstack/react-query</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> 
      <span class="na">singleton</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> 
      <span class="na">requiredVersion</span><span class="p">:</span> <span class="nx">deps</span><span class="p">[</span><span class="dl">'</span><span class="s1">@tanstack/react-query</span><span class="dl">'</span><span class="p">]</span> 
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">})</span>
</code></pre>

</div>



<p>Three configuration properties matter most: <code>singleton: true</code> ensures only one instance exists across all federated modules, <code>strictVersion: true</code> throws errors when versions conflict rather than silently loading duplicates, and <code>requiredVersion</code> with explicit semver ranges prevents deployment failures. Dynamically loading versions from <code>package.json</code> ensures configuration matches installed packages.</p>

<h3>
  
  
  The Async Boundary Pattern
</h3>

<p>The "Shared module is not available for eager consumption" error plagues new Module Federation setups. Standard entry points import React synchronously, but shared modules load asynchronously. The runtime hasn't initialized the share scope before the import executes.</p>

<p>Setting <code>eager: true</code> forces the library into the initial bundle, solving the error but adding ~100-150KB gzipped to Time to Interactive. The architectural best practice is establishing an asynchronous boundary:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// index.js - Simple wrapper enabling async loading</span>
<span class="k">import</span><span class="p">(</span><span class="dl">'</span><span class="s1">./bootstrap</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// bootstrap.js - Your actual application entry</span>
<span class="k">import</span> <span class="nx">React</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">ReactDOM</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react-dom</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">App</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./App</span><span class="dl">'</span><span class="p">;</span>
<span class="nx">ReactDOM</span><span class="p">.</span><span class="nf">render</span><span class="p">(</span><span class="o">&lt;</span><span class="nx">App</span> <span class="o">/&gt;</span><span class="p">,</span> <span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">'</span><span class="s1">root</span><span class="dl">'</span><span class="p">));</span>
</code></pre>

</div>



<p>When Webpack sees <code>import('./bootstrap')</code>, it creates a promise. During resolution, the Federation runtime initializes <code>__webpack_share_scopes__</code>, checks remote entry points, and ensures shared dependencies are ready. By the time <code>bootstrap.js</code> executes, React is available in the shared scope.</p>

<h2>
  
  
  Client State: Zustand vs Redux in Federated Architectures
</h2>

<p><strong>Zustand has emerged as the preferred client state library for micro-frontends</strong> due to its 1KB bundle size, singleton-friendly architecture, and absence of provider hierarchy requirements. Unlike Redux or Context, a Zustand store doesn't require wrapping components in providers that must align across module boundaries.</p>

<h3>
  
  
  The "Initial Value Only" Bug
</h3>

<p>Developers migrating to federated architectures often encounter a baffling bug: the Remote application renders initial state correctly, but when the Host updates state, the Remote fails to re-render. It appears "stuck" on the initial value.</p>

<p>This highlights the difference between sharing code and sharing instances. If both Host and Remote import a store via build-time aliases, the bundler includes the code in both bundles. At runtime, the Host creates <code>Store_Instance_A</code>, the Remote creates <code>Store_Instance_B</code>. The Host updates Instance A; the Remote listens to Instance B. No update propagates.</p>

<h3>
  
  
  The Exported Store Pattern
</h3>

<p>To guarantee synchronization, ensure the exact same JavaScript object in memory is used by both Host and Remote:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Remote module exposes its store</span>
<span class="c1">// cart-remote/webpack.config.js</span>
<span class="nx">exposes</span><span class="p">:</span> <span class="p">{</span>
  <span class="dl">'</span><span class="s1">./CartStore</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./src/stores/cartStore</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">./CartComponent</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./src/components/Cart</span><span class="dl">'</span><span class="p">,</span>
<span class="p">}</span>

<span class="c1">// libs/shared/data-access/src/lib/cart.store.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">create</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zustand</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">persist</span><span class="p">,</span> <span class="nx">devtools</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zustand/middleware</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">useCartStore</span> <span class="o">=</span> <span class="nf">create</span><span class="p">(</span>
  <span class="nf">devtools</span><span class="p">(</span>
    <span class="nf">persist</span><span class="p">(</span>
      <span class="p">(</span><span class="kd">set</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
        <span class="na">items</span><span class="p">:</span> <span class="p">[],</span>
        <span class="na">addItem</span><span class="p">:</span> <span class="p">(</span><span class="nx">item</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span> 
          <span class="na">items</span><span class="p">:</span> <span class="p">[...</span><span class="nx">state</span><span class="p">.</span><span class="nx">items</span><span class="p">,</span> <span class="nx">item</span><span class="p">]</span> 
        <span class="p">})),</span>
        <span class="na">clearCart</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">set</span><span class="p">({</span> <span class="na">items</span><span class="p">:</span> <span class="p">[]</span> <span class="p">}),</span>
      <span class="p">}),</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">cart-storage</span><span class="dl">'</span> <span class="p">}</span>
    <span class="p">),</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CartStore</span><span class="dl">'</span> <span class="p">}</span>
  <span class="p">)</span>
<span class="p">);</span>

<span class="c1">// Export atomic selectors (prevents unnecessary re-renders)</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">useCartItems</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">useCartStore</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">items</span><span class="p">);</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">useCartActions</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">useCartStore</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">actions</span><span class="p">);</span>
</code></pre>

</div>



<p>The Remote imports from the federation namespace, not the local library:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// remote/src/App.tsx</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useCartStore</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">host/CartStore</span><span class="dl">'</span><span class="p">;</span> 

<span class="k">export</span> <span class="kd">const</span> <span class="nx">RemoteApp</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">items</span> <span class="o">=</span> <span class="nf">useCartStore</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">items</span><span class="p">);</span>
  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="p">{</span><span class="nx">items</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span> <span class="nx">items</span> <span class="k">in</span> <span class="nx">cart</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span><span class="p">};</span>
</code></pre>

</div>



<p>When <code>remote/App.tsx</code> imports <code>host/CartStore</code>, Webpack delegates loading to the Federation runtime, which returns the module reference the Host already loaded. The <code>useCartStore</code> refers to the exact closure created in the Host.</p>

<h3>
  
  
  The Dependency Injection Pattern for Redux
</h3>

<p>For Redux-based architectures with Provider requirements, wrapping Remotes in their own <code>&lt;Provider store={store}&gt;</code> creates dangerous nested providers. A cleaner pattern is dependency injection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Remote declares store as a prop contract</span>
<span class="kr">interface</span> <span class="nx">Props</span> <span class="p">{</span>
  <span class="nl">store</span><span class="p">:</span> <span class="nx">StoreType</span><span class="p">;</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">RemoteWidget</span> <span class="o">=</span> <span class="p">({</span> <span class="nx">store</span> <span class="p">}:</span> <span class="nx">Props</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">state</span> <span class="o">=</span> <span class="nf">useStore</span><span class="p">(</span><span class="nx">store</span><span class="p">);</span> 
  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="p">{</span><span class="nx">state</span><span class="p">.</span><span class="nx">value</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span><span class="p">};</span>
<span class="k">export</span> <span class="k">default</span> <span class="nx">RemoteWidget</span><span class="p">;</span>

<span class="c1">// Host passes its store instance</span>
<span class="kd">const</span> <span class="nx">RemoteWidget</span> <span class="o">=</span> <span class="nx">React</span><span class="p">.</span><span class="nf">lazy</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">import</span><span class="p">(</span><span class="dl">'</span><span class="s1">remote/Widget</span><span class="dl">'</span><span class="p">));</span>

<span class="kd">const</span> <span class="nx">App</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">Suspense</span> <span class="nx">fallback</span><span class="o">=</span><span class="dl">"</span><span class="s2">Loading...</span><span class="dl">"</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">RemoteWidget</span> <span class="nx">store</span><span class="o">=</span><span class="p">{</span><span class="nx">myStore</span><span class="p">}</span> <span class="sr">/</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/Suspense</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<p>This decouples the Remote from store location, enables testing with mock stores, and ensures ownership remains with the Host.</p>

<h2>
  
  
  Server State: Cache Sharing Strategies
</h2>

<p><strong>TanStack Query v5 fundamentally changed cache sharing</strong> by removing the <code>contextSharing</code> prop. The current approach requires explicit QueryClient sharing via Module Federation exposes.</p>

<h3>
  
  
  Strategy A: Shared QueryClient (Monolithic Cache)
</h3>

<p>The Host initializes a single QueryClient and shares it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// host/src/queryClient.ts (exposed via Module Federation)</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">queryClient</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">QueryClient</span><span class="p">({</span>
  <span class="na">defaultOptions</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">queries</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">staleTime</span><span class="p">:</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span>
      <span class="na">gcTime</span><span class="p">:</span> <span class="mi">5</span> <span class="o">*</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span>
      <span class="na">refetchOnWindowFocus</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="c1">// Remote modules consume the shared client</span>
<span class="kd">const</span> <span class="nx">queryClient</span> <span class="o">=</span> <span class="k">await</span> <span class="k">import</span><span class="p">(</span><span class="dl">'</span><span class="s1">host/QueryClient</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">function</span> <span class="nf">RemoteApp</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">QueryClientProvider</span> <span class="nx">client</span><span class="o">=</span><span class="p">{</span><span class="nx">queryClient</span><span class="p">.</span><span class="k">default</span><span class="p">}</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">ProductList</span> <span class="o">/&gt;</span>
    <span class="o">&lt;</span><span class="sr">/QueryClientProvider</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Advantages</strong>: Instant cache deduplication—if the Host fetches <code>['user', 'me']</code> and the Remote needs the same data, TanStack Query serves it from cache without a network request. Global invalidation means mutations in one module update all consumers.</p>

<p><strong>Risk</strong>: This requires successful Context sharing. If React instances differ, the Remote throws "No QueryClient set" errors.</p>

<h3>
  
  
  Strategy B: Isolated QueryClients (Distributed Cache)
</h3>

<p>Each Remote creates its own QueryClient, ensuring true independence.</p>

<p><strong>Advantages</strong>: Remotes can use different React Query versions. A crash in the Host's cache doesn't affect Remotes.</p>

<p><strong>Disadvantages</strong>: Both Host and Remote fetch the same data (double network traffic). If the Remote updates user data, the Host's cache remains stale until reload.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Shared QueryClient</th>
<th>Isolated QueryClient</th>
</tr>
</thead>
<tbody>
<tr>
<td>Data Reuse</td>
<td>High (instant cache hits)</td>
<td>Low (relies on HTTP cache)</td>
</tr>
<tr>
<td>Coupling</td>
<td>Tight (must share React)</td>
<td>Loose (independent instances)</td>
</tr>
<tr>
<td>Invalidation</td>
<td>Global (one mutation updates all)</td>
<td>Local (manual sync required)</td>
</tr>
<tr>
<td>Robustness</td>
<td>Brittle (context issues fatal)</td>
<td>Robust (fail-safe)</td>
</tr>
<tr>
<td><strong>Use When</strong></td>
<td>Internal, trusted MFEs</td>
<td>3rd-party or distinct domains</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Cross-MFE Cache Invalidation
</h3>

<p>For distributed caches requiring coordination, use BroadcastChannel:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">channel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">BroadcastChannel</span><span class="p">(</span><span class="dl">'</span><span class="s1">query-cache-sync</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">function</span> <span class="nf">useCreateProduct</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">queryClient</span> <span class="o">=</span> <span class="nf">useQueryClient</span><span class="p">();</span>

  <span class="k">return</span> <span class="nf">useMutation</span><span class="p">({</span>
    <span class="na">mutationFn</span><span class="p">:</span> <span class="nx">createProduct</span><span class="p">,</span>
    <span class="na">onSuccess</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">queryClient</span><span class="p">.</span><span class="nf">invalidateQueries</span><span class="p">({</span> <span class="na">queryKey</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">products</span><span class="dl">'</span><span class="p">]</span> <span class="p">});</span>
      <span class="nx">channel</span><span class="p">.</span><span class="nf">postMessage</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">INVALIDATE</span><span class="dl">'</span><span class="p">,</span> <span class="na">queryKey</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">products</span><span class="dl">'</span><span class="p">]</span> <span class="p">});</span>
    <span class="p">},</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="c1">// Subscriber in each MFE</span>
<span class="nx">channel</span><span class="p">.</span><span class="nx">onmessage</span> <span class="o">=</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">INVALIDATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">queryClient</span><span class="p">.</span><span class="nf">invalidateQueries</span><span class="p">({</span> <span class="na">queryKey</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">queryKey</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  The Emerging Paradigm: Local-First Databases
</h2>

<p>TanStack DB, currently in beta, represents a paradigm shift from "Caching API Responses" to "Syncing a Database Replica." It's particularly transformative for distributed architectures because it solves synchronization at the protocol level.</p>

<p>State is organized into typed Collections rather than arbitrary string keys, acting as rigid contracts. Multiple MFEs can query the same dataset with different filters without coordinating who fetches what—the database acts as the central state hub.</p>

<p>For multi-tab synchronization, TanStack Query's <code>broadcastQueryClient</code> plugin provides a bridge:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">broadcastQueryClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@tanstack/query-broadcast-client-experimental</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">broadcastQueryClient</span><span class="p">({</span>
  <span class="nx">queryClient</span><span class="p">,</span>
  <span class="na">broadcastChannel</span><span class="p">:</span> <span class="dl">'</span><span class="s1">my-app-session</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Event-Driven Architecture: When State Isn't the Answer
</h2>

<p>Not all communication requires shared state. For transactional "fire-and-forget" interactions, events reduce coupling:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Shared event bus</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">authChannel</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">BroadcastChannel</span><span class="p">(</span><span class="dl">'</span><span class="s1">auth_events</span><span class="dl">'</span><span class="p">);</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">sendLogout</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">authChannel</span><span class="p">.</span><span class="nf">postMessage</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">LOGOUT</span><span class="dl">'</span> <span class="p">});</span>
<span class="p">};</span>

<span class="c1">// For same-document communication, use CustomEvent</span>
<span class="nb">window</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">(</span><span class="k">new</span> <span class="nc">CustomEvent</span><span class="p">(</span><span class="dl">'</span><span class="s1">cart:open</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">detail</span><span class="p">:</span> <span class="p">{</span> <span class="nx">productId</span> <span class="p">}</span> <span class="p">}));</span>
</code></pre>

</div>



<p>CustomEvents leverage the browser's native event loop with zero library dependencies—ideal for UI signals like a "Buy Now" button triggering a cart drawer.</p>

<h2>
  
  
  Production Hardening
</h2>

<h3>
  
  
  Three-Layer Error Handling
</h3>

<p>Module Federation 2.0's RetryPlugin handles transient network failures:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">RetryPlugin</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@module-federation/retry-plugin</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">mf</span> <span class="o">=</span> <span class="nf">createInstance</span><span class="p">({</span>
  <span class="na">plugins</span><span class="p">:</span> <span class="p">[</span>
    <span class="nc">RetryPlugin</span><span class="p">({</span>
      <span class="na">retryTimes</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>
      <span class="na">retryDelay</span><span class="p">:</span> <span class="mi">1000</span><span class="p">,</span>
      <span class="na">manifestDomains</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">https://cdn1.example.com</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">https://cdn2.example.com</span><span class="dl">'</span><span class="p">],</span>
    <span class="p">}),</span>
  <span class="p">],</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Combine with <code>errorLoadRemote</code> hooks for load fallbacks and React Error Boundaries for runtime errors.</p>

<h3>
  
  
  Memory Leak Prevention
</h3>

<p>In federated architectures, MFEs mount and unmount as users navigate. Listeners registered on global stores or event buses that aren't cleaned up accumulate, causing duplicate API calls and unpredictable behavior. Strict linting enforcing cleanup functions in <code>useEffect</code> is mandatory.</p>

<h2>
  
  
  Monorepo Structure with Turborepo
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>my-monorepo/
├── apps/
│   ├── host/                    # Shell application
│   └── product-remote/          # Federated product module
├── packages/
│   ├── state/                   # Shared stores and query hooks
│   │   ├── src/
│   │   │   ├── stores/          # Zustand stores
│   │   │   ├── hooks/           # React Query hooks
│   │   │   └── api/             # API clients
│   └── typescript-config/
├── turbo.json
└── pnpm-workspace.yaml
</code></pre>

</div>



<p>Prevent duplicate instances with pnpm configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="c"># .npmrc
</span><span class="py">resolve-peers-from-workspace-root</span><span class="p">=</span><span class="s">true</span>
<span class="py">dedupe-peer-dependents</span><span class="p">=</span><span class="s">true</span>
</code></pre>

</div>



<h2>
  
  
  Recommendations by Team Structure
</h2>

<p><strong>Small teams (2-5 developers)</strong>: Use Zustand + TanStack Query as singletons. Build-time sharing via Turborepo internal packages. Module Federation adds unnecessary complexity.</p>

<p><strong>Medium teams (5-15 developers)</strong>: Hybrid approach—shared packages for core functionality (auth, session), Module Federation for feature modules. Invest in three-layer error handling.</p>

<p><strong>Large organizations (multiple teams)</strong>: Props-based interfaces between MFEs, centralized authentication via shell, strict version alignment. Debugging distributed state exceeds coordination overhead.</p>

<p><strong>The essential rule</strong>: Never store server data in client state libraries. TanStack Query handles caching, refetching, and invalidation—duplicating in Zustand creates synchronization bugs.</p>

<h2>
  
  
  Production Checklist
</h2>

<ul>
<li>[ ] Async boundary pattern (<code>import('./bootstrap')</code>) in every federated module entry</li>
<li>[ ] Singleton configuration for React, react-dom, and all state libraries with explicit <code>requiredVersion</code>
</li>
<li>[ ] State logic extracted to shared <code>libs/data-access</code> packages</li>
<li>[ ] Host exposes state instances; Remotes consume via <code>import from 'host/Store'</code>
</li>
<li>[ ] All event listeners return cleanup functions in <code>useEffect</code>
</li>
<li>[ ] BroadcastChannel-based invalidation for cross-MFE coordination</li>
<li>[ ] Three-layer error handling (retry, fallback, boundary)</li>
<li>[ ] pnpm/yarn resolutions enforcing identical library versions</li>
</ul>




<h2>
  
  
  Further Reading &amp; Resources
</h2>

<h3>
  
  
  Official Documentation
</h3>

<ul>
<li><a href="https://webpack.js.org/plugins/module-federation-plugin/" rel="noopener noreferrer">Webpack Module Federation Plugin</a></li>
<li><a href="https://webpack.js.org/concepts/module-federation/" rel="noopener noreferrer">Module Federation Concepts</a></li>
<li><a href="https://module-federation.io/configure/shared" rel="noopener noreferrer">Module Federation Shared Configuration</a></li>
<li><a href="https://tanstack.com/query/v5/docs/react/guides/migrating-to-v5" rel="noopener noreferrer">TanStack Query v5 Migration Guide</a></li>
</ul>

<h3>
  
  
  Error Handling &amp; Production Patterns
</h3>

<ul>
<li><a href="https://module-federation.io/blog/error-load-remote" rel="noopener noreferrer">Error Handling for Remote Module Rendering</a></li>
<li><a href="https://github.com/module-federation/core/discussions/2397" rel="noopener noreferrer">Module Federation 2.0 Release Notes</a></li>
</ul>

<h3>
  
  
  State Management Patterns
</h3>

<ul>
<li><a href="https://dev.to/manomano-tech-team/how-to-work-with-redux-on-module-federation-13gm">Redux with Module Federation</a></li>
<li><a href="https://www.humansecurity.com/tech-engineering-blog/how-ive-integrated-redux-with-micro-frontends/" rel="noopener noreferrer">Integrating Redux with Micro-Frontends</a></li>
<li><a href="https://medium.com/@ancilartech/large-scale-apps-101-redux-zustand-jotai-or-recoil-for-scalable-react-state-management-cebcd77e24a3" rel="noopener noreferrer">Scalable React State Management Comparison</a></li>
<li><a href="https://medium.com/@freeyeon96/zustand-react-query-new-state-management-7aad6090af56" rel="noopener noreferrer">Zustand + React Query Patterns</a></li>
</ul>

<h3>
  
  
  Singleton &amp; Context Issues
</h3>

<ul>
<li><a href="https://smashingtips.com/programming/mastering-webpack-module-federation-solve-singleton-issues-code-snippets/" rel="noopener noreferrer">Solving Singleton Issues in Module Federation</a></li>
<li><a href="https://medium.com/@robbiedelavictoria/ensuring-module-uniqueness-in-webpack-module-federation-c17c01c68c6c" rel="noopener noreferrer">Ensuring Module Uniqueness</a></li>
<li><a href="https://github.com/module-federation/module-federation-examples/discussions/2146" rel="noopener noreferrer">React Context Between Exposed Components</a></li>
</ul>

<h3>
  
  
  Advanced Topics
</h3>

<ul>
<li><a href="https://medium.com/@siddhesh.shirdhankar18/error-boundaries-in-micro-frontend-architecture-5b5dd2c71541" rel="noopener noreferrer">Error Boundaries in Micro-frontend Architecture</a></li>
<li><a href="https://medium.com/@alexuxui/introducing-fmr-federated-module-reloading-820ec56256db" rel="noopener noreferrer">Federated Module Reloading (HMR)</a></li>
</ul>

