---
Title: I built a data grid with web components – here's what I learned
Description: 
Author: Øystein Amundsen
Date: 2026-01-24T21:01:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>A while back I needed a data grid for a project. You know the drill – looked at AG Grid (too heavy, enterprise pricing), TanStack Table (great but headless, didn't want to build all the UI), various React/Vue-specific options (locked into one framework).</p>

<p>I was also working across multiple projects – some Angular, some React, some just vanilla JS. The idea of maintaining different grid implementations for each was... not appealing.</p>

<p>So I did what any reasonable developer does: I spent way more time building my own thing than I would have spent just using an existing solution. 😅</p>

<h2>
  
  
  The idea
</h2>

<p>Web components have been "almost ready" for years, but they're actually pretty solid now. Custom elements, CSS nesting, adoptedStyleSheets – the browser APIs are there.</p>

<p>So I built <code>@toolbox-web/grid</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;tbw-grid&gt;&lt;/tbw-grid&gt;</span>

<span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"module"</span><span class="nt">&gt;</span>
  <span class="k">import</span> <span class="dl">'</span><span class="s1">@toolbox-web/grid</span><span class="dl">'</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">grid</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">tbw-grid</span><span class="dl">'</span><span class="p">);</span>
  <span class="nx">grid</span><span class="p">.</span><span class="nx">columns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">name</span><span class="dl">'</span><span class="p">,</span> <span class="na">header</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Name</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span> <span class="na">header</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Email</span><span class="dl">'</span> <span class="p">},</span>
  <span class="p">];</span>
  <span class="nx">grid</span><span class="p">.</span><span class="nx">rows</span> <span class="o">=</span> <span class="nx">data</span><span class="p">;</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>



<p>No framework. Just a custom element. Works in React, Angular, Vue, or plain HTML.</p>

<h2>
  
  
  What's in it
</h2>

<p>The basics you'd expect:</p>

<ul>
<li>Sorting, filtering, inline editing</li>
<li>Column resizing, reordering, pinning</li>
<li>Row selection (single, multi, range)</li>
<li>Row virtualization (handles 100k+ rows)</li>
</ul>

<p>And some stuff I needed for specific projects:</p>

<ul>
<li>Row grouping and tree data</li>
<li>Master-detail panels</li>
<li>Clipboard support (copy/paste)</li>
<li>Undo/redo for edits</li>
<li>CSV/Excel export</li>
</ul>

<p>Everything is a plugin, so you only ship what you use. The core is pretty small (~40KB gzipped with common plugins).</p>

<h2>
  
  
  Things I learned building this
</h2>

<h3>
  
  
  Shadow DOM is a pain
</h3>

<p>I started with Shadow DOM for style encapsulation. Seemed like the "right" way. But:</p>

<ul>
<li>Users couldn't style the grid with their existing CSS</li>
<li>Server-side rendering was a mess</li>
<li>DevTools debugging was awkward</li>
</ul>

<p>Switched to light DOM with CSS nesting. You get scoping without the isolation headaches.</p>

<h3>
  
  
  One render loop to rule them all
</h3>

<p>Early versions had render calls scattered everywhere. Change a property? Render. Resize a column? Render. Sort? Render. This caused layout thrashing and flickering.</p>

<p>Now everything goes through a single scheduler:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>state change → queue → batch → one RAF → render
</code></pre>

</div>



<p>Doesn't matter how many things change in a frame, there's one render pass.</p>

<h3>
  
  
  Plugins are just classes
</h3>

<p>I went back and forth on the plugin architecture. Ended up with simple classes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">class</span> <span class="nc">MyPlugin</span> <span class="kd">extends</span> <span class="nc">BaseGridPlugin</span><span class="o">&lt;</span><span class="nx">MyConfig</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">readonly</span> <span class="nx">name</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">myPlugin</span><span class="dl">'</span><span class="p">;</span>

  <span class="nf">processRows</span><span class="p">(</span><span class="nx">rows</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// transform data before render</span>
    <span class="k">return</span> <span class="nx">rows</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nf">afterRender</span><span class="p">()</span> <span class="p">{</span>
    <span class="c1">// do stuff with the DOM</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Constructor injection for config, lifecycle hooks for different phases. Nothing fancy, but it works well.</p>

<h2>
  
  
  Framework adapters
</h2>

<p>The raw custom element works everywhere, but framework-specific wrappers make life nicer.</p>

<p><strong>React:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">DataGrid</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@toolbox-web/grid-react</span><span class="dl">'</span><span class="p">;</span>

<span class="p">&lt;</span><span class="nc">DataGrid</span>
  <span class="na">rows</span><span class="p">=</span><span class="si">{</span><span class="nx">employees</span><span class="si">}</span>
  <span class="na">gridConfig</span><span class="p">=</span><span class="si">{</span><span class="p">{</span>
    <span class="na">columns</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">name</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">status</span><span class="dl">'</span><span class="p">,</span> <span class="na">renderer</span><span class="p">:</span> <span class="p">(</span><span class="nx">ctx</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">&lt;</span><span class="nc">Badge</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">ctx</span><span class="p">.</span><span class="nx">value</span><span class="si">}</span> <span class="p">/&gt;</span> <span class="p">},</span>
    <span class="p">],</span>
  <span class="p">}</span><span class="si">}</span>
<span class="p">/&gt;</span>
</code></pre>

</div>



<p><strong>Angular:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="o">&lt;</span><span class="nx">tbw</span><span class="o">-</span><span class="nx">grid</span> <span class="p">[</span><span class="nx">rows</span><span class="p">]</span><span class="o">=</span><span class="dl">"</span><span class="s2">data</span><span class="dl">"</span> <span class="p">[</span><span class="nx">gridConfig</span><span class="p">]</span><span class="o">=</span><span class="dl">"</span><span class="s2">config</span><span class="dl">"</span><span class="o">&gt;</span>
  <span class="o">&lt;</span><span class="nx">tbw</span><span class="o">-</span><span class="nx">grid</span><span class="o">-</span><span class="nx">column</span> <span class="nx">field</span><span class="o">=</span><span class="dl">"</span><span class="s2">status</span><span class="dl">"</span><span class="o">&gt;</span>
    <span class="o">&lt;</span><span class="nx">my</span><span class="o">-</span><span class="nx">badge</span> <span class="o">*</span><span class="nx">tbwRenderer</span><span class="o">=</span><span class="dl">"</span><span class="s2">let value</span><span class="dl">"</span> <span class="p">[</span><span class="nx">status</span><span class="p">]</span><span class="o">=</span><span class="dl">"</span><span class="s2">value</span><span class="dl">"</span> <span class="o">/&gt;</span>
  <span class="o">&lt;</span><span class="sr">/tbw-grid-column</span><span class="err">&gt;
</span><span class="o">&lt;</span><span class="sr">/tbw-grid</span><span class="err">&gt;
</span></code></pre>

</div>



<p>You get proper types, native component renderers, and framework-idiomatic APIs.</p>

<h2>
  
  
  Rough comparison
</h2>

<p>Just for context – not trying to say mine is "better", they're different tools:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th></th>
<th>@toolbox-web/grid</th>
<th>AG Grid</th>
<th>TanStack Table</th>
</tr>
</thead>
<tbody>
<tr>
<td>Bundle</td>
<td>~40KB</td>
<td>~500KB</td>
<td>~15KB (headless)</td>
</tr>
<tr>
<td>UI included</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Framework</td>
<td>Any</td>
<td>Wrappers</td>
<td>React/Vue/Solid</td>
</tr>
<tr>
<td>License</td>
<td>MIT</td>
<td>MIT + paid</td>
<td>MIT</td>
</tr>
</tbody>
</table></div>

<p>AG Grid is way more mature and feature-complete. TanStack is brilliant if you want full control over rendering. This sits somewhere in between – batteries included but not as heavy.</p>

<h2>
  
  
  Try it out
</h2>

<p>It's MIT licensed, free, no tracking, no enterprise upsell. Just a thing I built that might be useful to others.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @toolbox-web/grid
</code></pre>

</div>



<ul>
<li>
<strong>Docs:</strong> <a href="https://toolboxjs.com" rel="noopener noreferrer">https://toolboxjs.com</a>
</li>
<li>
<strong>GitHub:</strong> <a href="https://github.com/OysteinAmundsen/toolbox" rel="noopener noreferrer">https://github.com/OysteinAmundsen/toolbox</a>
</li>
<li>
<strong>npm:</strong> <a href="https://www.npmjs.com/package/@toolbox-web/grid" rel="noopener noreferrer">https://www.npmjs.com/package/@toolbox-web/grid</a>
</li>
</ul>

<p>Would love feedback, bug reports, or feature ideas. I'm actively working on it and trying to make it as useful as possible.</p>




<p><em>What data grid do you use? Curious what features people actually care about.</em></p>

