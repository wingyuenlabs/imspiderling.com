---
Title: Angular 20: Querying Data with `rxResource` — from `request/loader` to `params/stream`
Description: 
Author: Cristian Sifuentes
Date: 2025-10-23T22:07:24.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fol7la78772kpijzzr29t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fol7la78772kpijzzr29t.png" alt="Angular 20: Querying Data with  raw `rxResource` endraw  — from  raw `request/loader` endraw  to  raw `params/stream` endraw " width="800" height="800"></a></p>

<h2>
  
  
  Angular 20: Querying Data with <code>rxResource</code> — from <code>request/loader</code> to <code>params/stream</code>
</h2>

<p><strong>Angular 20</strong> ships a refined Resource API. If you used <code>resource()</code>/<code>rxResource()</code> in v19, two names changed and status got simpler:</p>

<ul>
<li>
<code>request</code> → <strong><code>params</code></strong>
</li>
<li>
<code>loader</code> → <strong><code>stream</code></strong> (used for both querying and streaming)</li>
<li>
<code>ResourceStatus</code> → <strong>string union</strong> (<code>'idle' | 'loading' | 'reloading' | 'resolved' | 'error'</code>)</li>
</ul>

<p>This post is a hands‑on guide to querying/paginating data with <strong><code>rxResource</code></strong> in Angular 20 using <strong>Signals</strong>, with <strong>typed results</strong>, <strong>status UIs</strong>, and a <strong>one‑file demo</strong> you can paste.</p>

<blockquote>
<p>Part 8 of the series: <em>Resource API Changes in Angular 20 — Query Resource in rxResource</em>.</p>
</blockquote>




<h2>
  
  
  TL;DR
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">rxResource</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/rxjs-interop</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">pageRef</span> <span class="o">=</span> <span class="nx">rxResource</span><span class="o">&lt;</span><span class="nx">Page</span><span class="p">,</span> <span class="nx">QueryParams</span><span class="o">&gt;</span><span class="p">({</span>
  <span class="na">params</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">queryParams</span><span class="p">(),</span>        <span class="c1">// reactive input (was `request`)</span>
  <span class="na">stream</span><span class="p">:</span> <span class="p">({</span> <span class="nx">params</span><span class="p">,</span> <span class="nx">abortSignal</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="c1">// Observable/Promise factory (was `loader`)</span>
    <span class="nx">http</span><span class="p">.</span><span class="kd">get</span><span class="o">&lt;</span><span class="nx">Page</span><span class="o">&gt;</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/items</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">params</span><span class="p">:</span> <span class="nf">params</span><span class="p">(),</span> <span class="na">signal</span><span class="p">:</span> <span class="nx">abortSignal</span> <span class="p">}),</span>
  <span class="na">defaultValue</span><span class="p">:</span> <span class="p">{</span> <span class="na">items</span><span class="p">:</span> <span class="p">[],</span> <span class="na">total</span><span class="p">:</span> <span class="mi">0</span> <span class="p">},</span> <span class="c1">// avoid undefined in templates</span>
<span class="p">});</span>
</code></pre>

</div>



<ul>
<li>Use <strong><code>params</code></strong> to derive fetch inputs from <strong>signals</strong>.</li>
<li>Implement <strong><code>stream</code></strong> to return an <strong>Observable</strong> or <strong>Promise</strong> of your typed data.</li>
<li>Read <strong><code>.value()</code></strong>, <strong><code>.status()</code></strong>, <strong><code>.error()</code></strong>, <strong><code>.isLoading()</code></strong>, and call <strong><code>.reload()</code></strong>.</li>
</ul>




<h2>
  
  
  Complete Example — <em>Pokémon Pagination with <code>rxResource</code></em>
</h2>

<p>This component paginates Pokémons using <a href="https://pokeapi.co/" rel="noopener noreferrer"><code>pokeapi.co</code></a>. It demonstrates:</p>

<ul>
<li>URL‑like <strong>query signals</strong> (<code>q</code>, <code>limit</code>, <code>offset</code>)</li>
<li>A <strong>typed</strong> <code>rxResource&lt;Page, Query&gt;</code>
</li>
<li>UI for <strong>loading/error/resolved</strong> states</li>
<li>
<strong>Prev/Next</strong> pagination with status transitions</li>
</ul>

<blockquote>
<p>Paste this into a fresh standalone component, import it in a route, and go.</p>
</blockquote>

<h3>
  
  
  <code>pokemon-page.component.ts</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span><span class="p">,</span> <span class="nx">computed</span><span class="p">,</span> <span class="nx">effect</span><span class="p">,</span> <span class="nx">inject</span><span class="p">,</span> <span class="nx">signal</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HttpClient</span><span class="p">,</span> <span class="nx">HttpParams</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">rxResource</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/rxjs-interop</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">NgIf</span><span class="p">,</span> <span class="nx">NgFor</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">map</span><span class="p">,</span> <span class="nx">mergeMap</span><span class="p">,</span> <span class="nx">forkJoin</span><span class="p">,</span> <span class="k">of</span><span class="p">,</span> <span class="nx">catchError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>

<span class="cm">/** Domain models (trim as needed) */</span>
<span class="kr">interface</span> <span class="nx">PrePokemon</span> <span class="p">{</span> <span class="nl">id</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">sprites</span><span class="p">?:</span> <span class="p">{</span> <span class="nx">front_default</span><span class="p">?:</span> <span class="kr">string</span> <span class="p">}</span> <span class="p">}</span>
<span class="kr">interface</span> <span class="nx">PageUrlResponse</span> <span class="p">{</span> <span class="nl">count</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">next</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="p">;</span> <span class="nl">previous</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="p">;</span> <span class="nl">results</span><span class="p">:</span> <span class="nb">Array</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">url</span><span class="p">:</span> <span class="kr">string</span> <span class="p">}</span><span class="o">&gt;</span> <span class="p">}</span>
<span class="kr">interface</span> <span class="nx">Page</span> <span class="p">{</span> <span class="nl">count</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">next</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="p">;</span> <span class="nl">previous</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="p">;</span> <span class="nl">results</span><span class="p">:</span> <span class="nx">PrePokemon</span><span class="p">[]</span> <span class="p">}</span>

<span class="p">@</span><span class="nd">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="dl">'</span><span class="s1">app-pokemon-page</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">standalone</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">NgIf</span><span class="p">,</span> <span class="nx">NgFor</span><span class="p">],</span>
  <span class="na">templateUrl</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./pokemon-page.component.html</span><span class="dl">'</span><span class="p">,</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">PokemonPageComponent</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">http</span> <span class="o">=</span> <span class="nf">inject</span><span class="p">(</span><span class="nx">HttpClient</span><span class="p">);</span>

  <span class="cm">/** Query signals */</span>
  <span class="k">readonly</span> <span class="nx">limit</span> <span class="o">=</span> <span class="nf">signal</span><span class="p">(</span><span class="mi">6</span><span class="p">);</span>
  <span class="k">readonly</span> <span class="nx">page</span>  <span class="o">=</span> <span class="nf">signal</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
  <span class="k">readonly</span> <span class="nx">offset</span> <span class="o">=</span> <span class="nf">computed</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nf">page</span><span class="p">()</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nf">limit</span><span class="p">());</span>

  <span class="cm">/** Compute the API URL like a router would do */</span>
  <span class="k">readonly</span> <span class="nx">url</span> <span class="o">=</span> <span class="nf">computed</span><span class="p">(</span>
    <span class="p">()</span> <span class="o">=&gt;</span> <span class="s2">`https://pokeapi.co/api/v2/pokemon?limit=</span><span class="p">${</span><span class="k">this</span><span class="p">.</span><span class="nf">limit</span><span class="p">()}</span><span class="s2">&amp;offset=</span><span class="p">${</span><span class="k">this</span><span class="p">.</span><span class="nf">offset</span><span class="p">()}</span><span class="s2">`</span>
  <span class="p">);</span>

  <span class="cm">/** Helper: hydrate list of Pokémon URLs into full Pokémon objects */</span>
  <span class="k">private</span> <span class="nx">paginate</span><span class="nf">$</span><span class="p">(</span><span class="nx">url</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">http</span><span class="p">.</span><span class="kd">get</span><span class="o">&lt;</span><span class="nx">PageUrlResponse</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">url</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">mergeMap</span><span class="p">((</span><span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">pokemonUrls</span> <span class="o">=</span> <span class="nx">res</span><span class="p">.</span><span class="nx">results</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">url</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">pokemons$</span> <span class="o">=</span> <span class="nx">pokemonUrls</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">u</span> <span class="o">=&gt;</span>
          <span class="k">this</span><span class="p">.</span><span class="nx">http</span><span class="p">.</span><span class="kd">get</span><span class="o">&lt;</span><span class="nx">PrePokemon</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">u</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">catchError</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">of</span><span class="p">(</span><span class="kc">undefined</span> <span class="k">as</span> <span class="nx">unknown</span> <span class="k">as</span> <span class="nx">PrePokemon</span><span class="p">)))</span>
        <span class="p">);</span>
        <span class="k">return</span> <span class="nf">forkJoin</span><span class="p">(</span><span class="nx">pokemons$</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
          <span class="nf">map</span><span class="p">(</span><span class="nx">list</span> <span class="o">=&gt;</span> <span class="nx">list</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nb">Boolean</span><span class="p">)),</span>
          <span class="nf">map</span><span class="p">(</span><span class="nx">results</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="p">...</span><span class="nx">res</span><span class="p">,</span> <span class="nx">results</span> <span class="p">}</span> <span class="k">as</span> <span class="nx">Page</span><span class="p">))</span>
        <span class="p">);</span>
      <span class="p">})</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="cm">/** The Resource (v20 API: params + stream) */</span>
  <span class="k">readonly</span> <span class="nx">pageRef</span> <span class="o">=</span> <span class="nx">rxResource</span><span class="o">&lt;</span><span class="nx">Page</span><span class="p">,</span> <span class="kr">string</span><span class="o">&gt;</span><span class="p">({</span>
    <span class="na">params</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nf">url</span><span class="p">(),</span>
    <span class="na">stream</span><span class="p">:</span> <span class="p">({</span> <span class="na">params</span><span class="p">:</span> <span class="nx">url</span><span class="p">,</span> <span class="nx">abortSignal</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">paginate</span><span class="nf">$</span><span class="p">(</span><span class="nx">url</span><span class="p">),</span> <span class="c1">// HttpClient respects signal</span>
    <span class="na">defaultValue</span><span class="p">:</span> <span class="p">{</span> <span class="na">count</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="na">next</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span> <span class="na">previous</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span> <span class="na">results</span><span class="p">:</span> <span class="p">[]</span> <span class="p">},</span>
  <span class="p">});</span>

  <span class="cm">/** Convenience getters derived from the resource's value */</span>
  <span class="k">readonly</span> <span class="nx">value</span>   <span class="o">=</span> <span class="nf">computed</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">pageRef</span><span class="p">.</span><span class="nf">value</span><span class="p">());</span>
  <span class="k">readonly</span> <span class="nx">nextUrl</span> <span class="o">=</span> <span class="nf">computed</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nf">value</span><span class="p">().</span><span class="nx">next</span> <span class="o">??</span> <span class="kc">undefined</span><span class="p">);</span>
  <span class="k">readonly</span> <span class="nx">prevUrl</span> <span class="o">=</span> <span class="nf">computed</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nf">value</span><span class="p">().</span><span class="nx">previous</span> <span class="o">??</span> <span class="kc">undefined</span><span class="p">);</span>

  <span class="cm">/** Keep page in range when count changes */</span>
  <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
    <span class="nf">effect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">total</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">value</span><span class="p">().</span><span class="nx">count</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">pages</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">ceil</span><span class="p">(</span><span class="nx">total</span> <span class="o">/</span> <span class="k">this</span><span class="p">.</span><span class="nf">limit</span><span class="p">()));</span>
      <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nf">page</span><span class="p">()</span> <span class="o">&gt;</span> <span class="nx">pages</span><span class="p">)</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">pages</span><span class="p">);</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="cm">/** UI handlers */</span>
  <span class="nf">next</span><span class="p">()</span> <span class="p">{</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">p</span> <span class="o">=&gt;</span> <span class="nx">p</span> <span class="o">+</span> <span class="mi">1</span><span class="p">);</span> <span class="p">}</span>
  <span class="nf">prev</span><span class="p">()</span> <span class="p">{</span> <span class="k">this</span><span class="p">.</span><span class="nx">page</span><span class="p">.</span><span class="nf">update</span><span class="p">(</span><span class="nx">p</span> <span class="o">=&gt;</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nx">p</span> <span class="o">-</span> <span class="mi">1</span><span class="p">));</span> <span class="p">}</span>
  <span class="nf">reload</span><span class="p">()</span> <span class="p">{</span> <span class="k">this</span><span class="p">.</span><span class="nx">pageRef</span><span class="p">.</span><span class="nf">reload</span><span class="p">();</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  <code>pokemon-page.component.html</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;section</span> <span class="na">class=</span><span class="s">"p-6 space-y-4"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;header</span> <span class="na">class=</span><span class="s">"flex items-center gap-3"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"text-sm opacity-70"</span><span class="nt">&gt;</span>Limit:<span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;select</span> <span class="na">class=</span><span class="s">"select select-bordered select-sm"</span>
            <span class="na">(change)=</span><span class="s">"limit.set(+$any($event.target).value)"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;option</span> <span class="na">[selected]=</span><span class="s">"limit()===6"</span> <span class="na">value=</span><span class="s">"6"</span><span class="nt">&gt;</span>6<span class="nt">&lt;/option&gt;</span>
      <span class="nt">&lt;option</span> <span class="na">[selected]=</span><span class="s">"limit()===12"</span> <span class="na">value=</span><span class="s">"12"</span><span class="nt">&gt;</span>12<span class="nt">&lt;/option&gt;</span>
      <span class="nt">&lt;option</span> <span class="na">[selected]=</span><span class="s">"limit()===24"</span> <span class="na">value=</span><span class="s">"24"</span><span class="nt">&gt;</span>24<span class="nt">&lt;/option&gt;</span>
    <span class="nt">&lt;/select&gt;</span>

    <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"btn btn-sm"</span> <span class="na">(click)=</span><span class="s">"reload()"</span> <span class="na">[disabled]=</span><span class="s">"pageRef.isLoading()"</span><span class="nt">&gt;</span>Reload<span class="nt">&lt;/button&gt;</span>

    <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"ml-auto text-sm opacity-70"</span><span class="nt">&gt;</span>
      Status: {{ pageRef.status() }}
    <span class="nt">&lt;/span&gt;</span>
  <span class="nt">&lt;/header&gt;</span>

  @if (pageRef.status() === 'loading' || pageRef.status() === 'reloading') {
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"flex items-center gap-2"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"loading loading-spinner"</span><span class="nt">&gt;&lt;/span&gt;</span>
      <span class="nt">&lt;span&gt;</span>Loading…<span class="nt">&lt;/span&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  }

  @if (pageRef.status() === 'error') {
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"alert alert-error"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;span&gt;</span>Failed to load: {{ pageRef.error()?.message ?? 'Unknown error' }}<span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"btn btn-sm ml-auto"</span> <span class="na">(click)=</span><span class="s">"reload()"</span><span class="nt">&gt;</span>Retry<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  }

  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-4"</span><span class="nt">&gt;</span>
    @for (p of value().results; track p.id) {
      <span class="nt">&lt;article</span> <span class="na">class=</span><span class="s">"card bg-base-200"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;figure</span> <span class="na">class=</span><span class="s">"aspect-square bg-base-300"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;img</span> <span class="na">[src]=</span><span class="s">"p.sprites?.front_default || 'https://placehold.co/160x160?text=PKMN'"</span>
               <span class="na">[alt]=</span><span class="s">"p.name"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;/figure&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body p-3"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title text-sm"</span><span class="nt">&gt;</span>{{ p.name | titlecase }}<span class="nt">&lt;/h3&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/article&gt;</span>
    }
  <span class="nt">&lt;/div&gt;</span>

  <span class="nt">&lt;footer</span> <span class="na">class=</span><span class="s">"flex items-center gap-3"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"btn"</span> <span class="na">(click)=</span><span class="s">"prev()"</span> <span class="na">[disabled]=</span><span class="s">"!prevUrl() || pageRef.isLoading()"</span><span class="nt">&gt;</span>Prev<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"opacity-70"</span><span class="nt">&gt;</span>Page {{ page() }}<span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"btn"</span> <span class="na">(click)=</span><span class="s">"next()"</span> <span class="na">[disabled]=</span><span class="s">"!nextUrl() || pageRef.isLoading()"</span><span class="nt">&gt;</span>Next<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/footer&gt;</span>
<span class="nt">&lt;/section&gt;</span>
</code></pre>

</div>






<h2>
  
  
  Migration (v19 → v20)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>v19 name</th>
<th>v20 name</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>request</code></td>
<td><strong><code>params</code></strong></td>
<td>Reactive function returning the fetch inputs (often built from signals).</td>
</tr>
<tr>
<td><code>loader</code></td>
<td><strong><code>stream</code></strong></td>
<td>Returns an <strong>Observable</strong> or <strong>Promise</strong>. Also used for server‑sent/streaming scenarios.</td>
</tr>
<tr>
<td>
<code>ResourceStatus</code> (enum-ish)</td>
<td><strong>string union</strong></td>
<td>`'idle'</td>
</tr>
<tr>
<td>{% raw %}<code>hasValue()</code> checks</td>
<td>still valid</td>
<td>You can also rely on <code>defaultValue</code> to avoid <code>undefined</code> altogether.</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Tip:</strong> If you had <code>request/loader</code> in multiple files, a mechanical rename to <strong><code>params/stream</code></strong> covers most of the migration.</p>
</blockquote>




<h2>
  
  
  Expert Tips &amp; Pitfalls
</h2>

<ul>
<li>
<strong>Always type your resource</strong>: <code>rxResource&lt;Result, Params&gt;</code>. This prevents <code>{}.foo</code> errors in templates.</li>
<li>
<strong>Provide <code>defaultValue</code></strong> to eliminate <code>undefined</code> guards in HTML.</li>
<li>
<strong>Abort stale requests</strong>: HttpClient supports <code>{ signal: abortSignal }</code> — Angular cancels when params change.</li>
<li>
<strong>Prefer stable <code>track</code> keys</strong> in <code>@for</code> (use an <code>id</code> field, not <code>$index</code>).</li>
<li>
<strong>Keep side effects out of <code>stream</code></strong>; it should be a pure factory for the request Observable/Promise.</li>
<li>
<strong>Reload UX</strong>: <code>status() === 'reloading'</code> is perfect for subtle spinners on a refresh action.</li>
</ul>




<h2>
  
  
  When to use <code>resource()</code> vs <code>rxResource()</code>
</h2>

<ul>
<li>Use <strong><code>resource()</code></strong> when your fetch returns a <strong>Promise</strong> or you don’t need RxJS operators.</li>
<li>Use <strong><code>rxResource()</code></strong> when your source is an <strong>Observable</strong> or you benefit from RxJS composition (debounce, mergeMap, retry, etc.).</li>
</ul>

<p>Both share the <strong>same mental model</strong> and the <code>params/stream</code> names in v20.</p>




<h2>
  
  
  Appendix: Minimal Service Version
</h2>

<p>If you prefer a service wrapper instead of composing inside the component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span><span class="p">,</span> <span class="nx">inject</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HttpClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">map</span><span class="p">,</span> <span class="nx">mergeMap</span><span class="p">,</span> <span class="nx">forkJoin</span><span class="p">,</span> <span class="k">of</span><span class="p">,</span> <span class="nx">catchError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span> <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span> <span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">PokemonPageService</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">http</span> <span class="o">=</span> <span class="nf">inject</span><span class="p">(</span><span class="nx">HttpClient</span><span class="p">);</span>

  <span class="nf">paginate</span><span class="p">(</span><span class="nx">url</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">http</span><span class="p">.</span><span class="kd">get</span><span class="o">&lt;</span><span class="nx">PageUrlResponse</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">url</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">mergeMap</span><span class="p">((</span><span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">pokemonUrls</span> <span class="o">=</span> <span class="nx">res</span><span class="p">.</span><span class="nx">results</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">url</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">pokemons$</span> <span class="o">=</span> <span class="nx">pokemonUrls</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">u</span> <span class="o">=&gt;</span>
          <span class="k">this</span><span class="p">.</span><span class="nx">http</span><span class="p">.</span><span class="kd">get</span><span class="o">&lt;</span><span class="nx">PrePokemon</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">u</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">catchError</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">of</span><span class="p">(</span><span class="kc">undefined</span> <span class="k">as</span> <span class="nx">unknown</span> <span class="k">as</span> <span class="nx">PrePokemon</span><span class="p">)))</span>
        <span class="p">);</span>
        <span class="k">return</span> <span class="nf">forkJoin</span><span class="p">(</span><span class="nx">pokemons$</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
          <span class="nf">map</span><span class="p">(</span><span class="nx">list</span> <span class="o">=&gt;</span> <span class="nx">list</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nb">Boolean</span><span class="p">)),</span>
          <span class="nf">map</span><span class="p">(</span><span class="nx">results</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="p">...</span><span class="nx">res</span><span class="p">,</span> <span class="nx">results</span> <span class="p">}</span> <span class="k">as</span> <span class="nx">Page</span><span class="p">))</span>
        <span class="p">);</span>
      <span class="p">})</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Resources
</h2>

<ul>
<li>Angular docs — <strong>Resource</strong>: <a href="https://next.angular.dev/guide/signals/resource#" rel="noopener noreferrer">https://next.angular.dev/guide/signals/resource#</a>
</li>
<li>API: <strong><code>resource</code></strong>: <a href="https://next.angular.dev/api/core/resource#" rel="noopener noreferrer">https://next.angular.dev/api/core/resource#</a>
</li>
<li>API: <strong><code>rxResource</code></strong>: <a href="https://next.angular.dev/api/core/rxjs-interop/rxResource#" rel="noopener noreferrer">https://next.angular.dev/api/core/rxjs-interop/rxResource#</a>
</li>
</ul>




<p><strong>Happy querying!</strong> If you’d like the <strong>streaming</strong> counterpart (server events, websockets) with <code>rxResource</code> in Angular 20, I can publish the follow‑up <em>(Part 9)</em> next.</p>

