---
Title: Code Your Own Virtual DOM in 100 Lines of JavaScript
Description: 
Author: Luiz Claudio Garcia
Date: 2026-03-25T22:10:54.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Code Your Own Virtual DOM in 100 Lines of JavaScript
</h1>




<h2>
  
  
  Introduction: Open the Black Box
</h2>

<p>You've been using React for years. You know how to lift state, split components, and reach for <code>useMemo</code> when things get slow. But ask yourself: what does React's <code>render()</code> actually do?</p>

<p>Most developers never look. The framework works, so they don't have to. That's fine — until they hit a bug they can't explain, a performance profile that makes no sense, or a mental model with a hole in it the size of a reconciliation algorithm.</p>

<p>Here's the thing that most introductions get wrong: the Virtual DOM is not a performance optimization. Rich Harris, creator of Svelte, made this argument in 2018 and it still holds. A VDOM doesn't make DOM operations faster — it adds a layer of work <em>on top</em> of them. What it gives you is something more valuable: a programming model where you describe what the UI should look like, not how to get there. The framework figures out the minimal set of changes. You just write the output.</p>

<p>That abstraction is worth understanding from the inside.</p>

<p>In about 100 lines of vanilla JavaScript — no build step, no npm, no JSX — we're going to build the engine that powers it. Four functions. A complete, working Virtual DOM that can mount a tree to the DOM, diff two trees, and apply the minimum necessary changes. When you're done, React's internals won't feel like magic. They'll feel like an engineering tradeoff you understand.</p>

<p>Open a text editor and a browser tab. That's all you need.</p>




<h2>
  
  
  What We're Building
</h2>

<p>You already write UI in HTML: <code>&lt;div id="app"&gt;&lt;p&gt;Count: 0&lt;/p&gt;&lt;/div&gt;</code>. The problem is HTML is a string — your JavaScript can't inspect the tree, compare two versions, or compute what changed. What if you described the same structure as a plain JavaScript object instead? That single idea is what the Virtual DOM is built on.</p>

<p>Before writing a single function, here's the API we'll end up with:</p>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Create virtual nodes</span>
<span class="kd">const</span> <span class="nx">vdom1</span> <span class="o">=</span> <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">div</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">app</span><span class="dl">'</span> <span class="p">},</span>
  <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">p</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span> <span class="dl">'</span><span class="s1">Count: 0</span><span class="dl">'</span><span class="p">)</span>
<span class="p">);</span>

<span class="c1">// Mount to the real DOM</span>
<span class="kd">const</span> <span class="nx">container</span> <span class="o">=</span> <span class="nf">render</span><span class="p">(</span><span class="nx">vdom1</span><span class="p">);</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nx">container</span><span class="p">);</span>

<span class="c1">// Later: the count changes</span>
<span class="kd">const</span> <span class="nx">vdom2</span> <span class="o">=</span> <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">div</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">app</span><span class="dl">'</span> <span class="p">},</span>
  <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">p</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span> <span class="dl">'</span><span class="s1">Count: 1</span><span class="dl">'</span><span class="p">)</span>
<span class="p">);</span>

<span class="c1">// Compute what changed</span>
<span class="kd">const</span> <span class="nx">patches</span> <span class="o">=</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">vdom1</span><span class="p">,</span> <span class="nx">vdom2</span><span class="p">);</span>

<span class="c1">// Apply only the changes</span>
<span class="nf">patch</span><span class="p">(</span><span class="nx">container</span><span class="p">,</span> <span class="nx">patches</span><span class="p">);</span>
<span class="c1">// Result: only the text node is updated. The div and p stay untouched.</span>
</code></pre>

</div>



<p>Four functions. No classes, no event bus, no reactivity system. Just <code>h</code>, <code>render</code>, <code>diff</code>, and <code>patch</code>.</p>

<p>A vnode — what <code>h()</code> returns — is a plain JavaScript object:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">vnode</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">p</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">props</span><span class="p">:</span> <span class="p">{},</span>
  <span class="na">children</span><span class="p">:</span> <span class="p">[{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">,</span> <span class="na">props</span><span class="p">:</span> <span class="p">{</span> <span class="na">nodeValue</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Count: 0</span><span class="dl">'</span> <span class="p">}</span> <span class="p">}]</span>
<span class="p">};</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fss90t6kntksyuku4hqri.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fss90t6kntksyuku4hqri.png" alt="diagram-01" width="784" height="312"></a></p>

<p><em>A vnode tree mirrors the DOM structure — but lives entirely in memory as plain JS objects.</em></p>

<p>That's it. No special prototype, no framework magic. A vnode is a description of a DOM node — nothing more. The real DOM knows nothing about it until <code>render()</code> walks the tree and builds actual elements.</p>




<h2>
  
  
  The Mental Model
</h2>

<p>The engine works in two phases. Get this picture in your head before reading a single line of implementation.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wscjhusk9dnwjmkwhsw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wscjhusk9dnwjmkwhsw.png" alt="diagram-02" width="451" height="541"></a></p>

<p><em>The engine is four functions with a clean separation of concerns. Only <code>patch()</code> touches the DOM.</em></p>

<p><strong>Phase 1 — Initial render:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F71jp9edusbqsh8582gjx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F71jp9edusbqsh8582gjx.png" alt="diagram-03" width="784" height="275"></a></p>

<p><em>Both phases start with <code>h()</code> — but one creates, the other updates.</em></p>

<p>When something changes, you call <code>h()</code> again to describe what the UI should look like now. <code>diff()</code> compares the old vnode tree to the new one and produces a list of patch descriptors — plain objects describing what changed. Then <code>patch()</code> applies those changes to the real DOM.</p>

<p>Notice the division of responsibility:</p>

<ul>
<li>
<code>diff()</code> never touches the DOM. It only compares two trees and produces data.</li>
<li>
<code>patch()</code> never reads vnodes. It only applies the patch descriptors to real nodes.</li>
</ul>

<p>This separation is why the engine is testable and predictable. The diffing logic is a pure function — same inputs, same output, every time. The DOM mutation is isolated to <code>patch()</code>.</p>

<p>One more thing to lock in: <code>diff()</code> compares trees by <strong>position</strong>. Child at index 0 in the old tree is compared against child at index 0 in the new tree. This is a deliberate simplification. Real React uses <code>key</code> props to track element identity across reorders — we won't. We'll cover what that costs in section 9.</p>




<h2>
  
  
  Step 1 — <code>h()</code>: Creating Virtual Nodes
</h2>

<p>You've been looking at vnodes since the previous section — plain objects with <code>type</code>, <code>props</code>, and <code>children</code>. Now you need a function to create them. You could write those objects by hand, but nesting three levels of them gets unreadable fast. <code>h()</code> is the shorthand.</p>

<p><code>h()</code> is the hyperscript function — the same pattern used by React's <code>createElement</code>, Vue's <code>h</code>, and Snabbdom. It takes a type, props, and children and returns a plain object.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">h</span><span class="p">(</span><span class="nx">type</span><span class="p">,</span> <span class="nx">props</span><span class="p">,</span> <span class="p">...</span><span class="nx">children</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="nx">type</span><span class="p">,</span>
    <span class="na">props</span><span class="p">:</span> <span class="nx">props</span> <span class="o">||</span> <span class="p">{},</span>
    <span class="na">children</span><span class="p">:</span> <span class="nx">children</span><span class="p">.</span><span class="nf">flat</span><span class="p">().</span><span class="nf">map</span><span class="p">(</span><span class="nx">child</span> <span class="o">=&gt;</span>
      <span class="k">typeof</span> <span class="nx">child</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">child</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">number</span><span class="dl">'</span>
        <span class="p">?</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">,</span> <span class="na">props</span><span class="p">:</span> <span class="p">{</span> <span class="na">nodeValue</span><span class="p">:</span> <span class="nc">String</span><span class="p">(</span><span class="nx">child</span><span class="p">)</span> <span class="p">}</span> <span class="p">}</span>
        <span class="p">:</span> <span class="nx">child</span>
    <span class="p">)</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p>A few things happening here:</p>

<p><strong><code>children.flat()</code></strong> — callers sometimes pass arrays of children (e.g. from <code>.map()</code>). Flattening one level keeps the tree clean without recursive complexity.</p>

<p><strong>Text node normalization</strong> — the DOM distinguishes between element nodes and text nodes, and so does our engine. Strings and numbers become <code>{ type: 'TEXT_NODE', props: { nodeValue: '...' } }</code> so <code>render()</code> and <code>diff()</code> can treat all children uniformly without special-casing strings.</p>

<p><strong><code>props || {}</code></strong> — callers often pass <code>null</code> for props when there are none: <code>h('br', null)</code>. The fallback prevents <code>Object.entries(null)</code> errors downstream.</p>

<p>That's 11 lines. Test it:</p>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">p</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">class</span><span class="p">:</span> <span class="dl">'</span><span class="s1">greeting</span><span class="dl">'</span> <span class="p">},</span> <span class="dl">'</span><span class="s1">Hello</span><span class="dl">'</span><span class="p">));</span>
<span class="c1">// {</span>
<span class="c1">//   type: 'p',</span>
<span class="c1">//   props: { class: 'greeting' },</span>
<span class="c1">//   children: [{ type: 'TEXT_NODE', props: { nodeValue: 'Hello' } }]</span>
<span class="c1">// }</span>
</code></pre>

</div>



<p>The output is a plain object you can inspect, serialize, or pass around. Nothing framework-specific about it.</p>




<h2>
  
  
  Step 2 — <code>render()</code>: Mounting to the DOM
</h2>

<p>At this point, your vnode tree is a JavaScript object sitting in memory — the browser can't display it. Open DevTools right now and you'll find nothing in the Elements panel. To put anything on screen, you need to walk that tree and call real browser APIs: <code>createElement</code>, <code>setAttribute</code>, <code>createTextNode</code>. That's all <code>render()</code> does.</p>

<p><code>render()</code> takes a vnode and returns a real DOM node. It's a recursive tree walk.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">render</span><span class="p">(</span><span class="nx">vnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createTextNode</span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">el</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">type</span><span class="p">);</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">props</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">key</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">on</span><span class="dl">'</span><span class="p">)</span> <span class="o">&amp;&amp;</span> <span class="k">typeof</span> <span class="nx">value</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">function</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">el</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="nx">key</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">2</span><span class="p">).</span><span class="nf">toLowerCase</span><span class="p">(),</span> <span class="nx">value</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="nx">el</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">child</span> <span class="k">of</span> <span class="nx">vnode</span><span class="p">.</span><span class="nx">children</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">el</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">child</span><span class="p">));</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">el</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Text nodes take a separate path — <code>document.createTextNode()</code> instead of <code>createElement()</code>. For everything else: create the element, apply props, recurse on children.</p>

<p>The event handler convention (<code>onClick</code> → <code>addEventListener('click', ...)</code>) lets you write <code>h('button', { onClick: handleClick }, 'Click me')</code> without any event delegation machinery. It's not as efficient as a production implementation, but it's correct and readable at this scale.</p>

<p>At this point you have a working static renderer. Try it:</p>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">vnode</span> <span class="o">=</span> <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">ul</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span>
  <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">li</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span> <span class="dl">'</span><span class="s1">Apples</span><span class="dl">'</span><span class="p">),</span>
  <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">li</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span> <span class="dl">'</span><span class="s1">Oranges</span><span class="dl">'</span><span class="p">),</span>
  <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">li</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span> <span class="dl">'</span><span class="s1">Bananas</span><span class="dl">'</span><span class="p">)</span>
<span class="p">);</span>

<span class="nb">document</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">vnode</span><span class="p">));</span>
</code></pre>

</div>



<p>Open your browser. You'll see a list. The VDOM engine is half done — <code>render()</code> handles the initial mount. Now we need to handle updates.</p>




<h2>
  
  
  Step 3 — <code>diff()</code>: Comparing Two Trees
</h2>

<p>The user clicked the button. <code>count</code> is now 1. You call <code>h()</code> again and get a new vnode tree — but the DOM still reflects the old one. You have two trees: what is and what should be. You could call <code>render()</code> again and replace the whole DOM, but that defeats the point. Instead, you need to compare the two trees and produce a precise description of only what changed. That's <code>diff()</code>.</p>

<p>This is the core of the engine. <code>diff()</code> takes two vnode trees and returns a patch descriptor — a plain object describing what changed. It never touches the DOM.</p>

<p>There are three cases to handle.</p>

<p>Start with the function signature and the two edge cases: a node that didn't exist before, and a node that no longer exists.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">,</span> <span class="nx">newVnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">oldVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">newVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span> <span class="p">};</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>When <code>diff()</code> recurses on children, the old and new trees may have different lengths. An index that exists in the new tree but not the old means a node was added — <code>CREATE</code>. The reverse means it was removed — <code>REMOVE</code>.</p>

<p><strong>Case 1: Different types</strong></p>

<p>If the old node is a <code>div</code> and the new one is a <code>span</code>, we don't attempt reconciliation. We replace the whole subtree. This is the same heuristic React uses: different types produce different trees, so tearing down and rebuilding is cheaper than diffing across types.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">,</span> <span class="nx">newVnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">oldVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">newVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span> <span class="p">};</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REPLACE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="p">}</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Case 2: Both text nodes</strong></p>

<p>Text nodes have no children or props to recurse into — just compare <code>nodeValue</code>. If it changed, return an update. If not, return <code>null</code>: no patch needed, skip the DOM entirely.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">,</span> <span class="nx">newVnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">oldVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">newVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span> <span class="p">};</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REPLACE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span>
      <span class="p">?</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="p">}</span>
      <span class="p">:</span> <span class="kc">null</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Case 3: Same element type</strong></p>

<p>If we reach this point, both nodes are the same element type. We need to do two things: diff the props and diff the children.</p>

<p>For props, collect all keys from both old and new, and record any that changed. A key present in old but missing in new gets <code>undefined</code> — that's our signal to remove it in <code>patch()</code>.</p>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>  <span class="c1">// Inside diff() — same element type case:</span>
  <span class="kd">const</span> <span class="nx">propPatches</span> <span class="o">=</span> <span class="p">{};</span>
  <span class="kd">const</span> <span class="nx">allKeys</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Set</span><span class="p">([</span>
    <span class="p">...</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">),</span>
    <span class="p">...</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">)</span>
  <span class="p">]);</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">key</span> <span class="k">of</span> <span class="nx">allKeys</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">!==</span> <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">])</span> <span class="p">{</span>
      <span class="nx">propPatches</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">=</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">];</span>
    <span class="p">}</span>
  <span class="p">}</span>
</code></pre>

</div>



<p>For children, iterate up to the longest of the two child arrays and call <code>diff()</code> recursively on each pair by index:</p>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>  <span class="c1">// Inside diff() — same element type case (continued):</span>
  <span class="kd">const</span> <span class="nx">childrenLen</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span>
    <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
    <span class="nx">newVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">.</span><span class="nx">length</span>
  <span class="p">);</span>
  <span class="kd">const</span> <span class="nx">childPatches</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">({</span> <span class="na">length</span><span class="p">:</span> <span class="nx">childrenLen</span> <span class="p">},</span> <span class="p">(</span><span class="nx">_</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span>
    <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">])</span>
  <span class="p">);</span>
</code></pre>

</div>



<p>If nothing changed — no prop diffs, no child diffs — return <code>null</code>. No patch needed at this node.</p>

<p><strong>The complete function:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">,</span> <span class="nx">newVnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">oldVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">newVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span> <span class="p">};</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REPLACE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span>
      <span class="p">?</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="p">}</span>
      <span class="p">:</span> <span class="kc">null</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">propPatches</span> <span class="o">=</span> <span class="p">{};</span>
  <span class="kd">const</span> <span class="nx">allKeys</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Set</span><span class="p">([</span>
    <span class="p">...</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">),</span>
    <span class="p">...</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">)</span>
  <span class="p">]);</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">key</span> <span class="k">of</span> <span class="nx">allKeys</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">!==</span> <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">])</span> <span class="p">{</span>
      <span class="nx">propPatches</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">=</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">];</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">childrenLen</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span>
    <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
    <span class="nx">newVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">.</span><span class="nx">length</span>
  <span class="p">);</span>
  <span class="kd">const</span> <span class="nx">childPatches</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">({</span> <span class="na">length</span><span class="p">:</span> <span class="nx">childrenLen</span> <span class="p">},</span> <span class="p">(</span><span class="nx">_</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span>
    <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">])</span>
  <span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">propPatches</span><span class="p">).</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">childPatches</span><span class="p">.</span><span class="nf">every</span><span class="p">(</span><span class="nx">p</span> <span class="o">=&gt;</span> <span class="nx">p</span> <span class="o">===</span> <span class="kc">null</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UPDATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">propPatches</span><span class="p">,</span> <span class="nx">childPatches</span> <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn6swm74fwk2at99y2l40.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn6swm74fwk2at99y2l40.png" alt="diagram-04" width="654" height="1259"></a></p>

<p><em><code>diff()</code> makes exactly four decisions. Every case returns a plain object — or <code>null</code> if nothing changed.</em></p>

<p>The child diffing is index-based: child 0 in the old tree is always compared to child 0 in the new tree. If you reorder a list of 100 items, the engine generates up to 100 UPDATE patches instead of recognising the move. That's the cost of skipping <code>key</code> support. For a counter demo it's invisible; for a sortable table it would matter. We'll come back to this in section 9.</p>

<p><strong>What React does differently:</strong> React uses <code>key</code> props for list identity, a Fiber architecture that makes reconciliation interruptible, and batched state updates. Each adds significant complexity. Our engine has none of them — and for most UIs under a few hundred nodes, the difference is undetectable.</p>




<h2>
  
  
  Step 4 — <code>patch()</code>: Applying Changes
</h2>

<p><code>diff()</code> returned a patch descriptor — a plain object saying "this text node's value changed", "this child was removed". But the DOM hasn't moved yet. Nothing has. Someone has to carry out those instructions against real browser nodes. That's <code>patch()</code>.</p>

<p><code>patch()</code> takes a real DOM node and a patch descriptor from <code>diff()</code>, and mutates the DOM. This is the only function in the engine that touches real nodes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">patch</span><span class="p">(</span><span class="nx">domNode</span><span class="p">,</span> <span class="nx">patchDescriptor</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span> <span class="o">===</span> <span class="kc">null</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">parent</span> <span class="o">=</span> <span class="nx">domNode</span><span class="p">.</span><span class="nx">parentNode</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">parent</span><span class="p">.</span><span class="nf">removeChild</span><span class="p">(</span><span class="nx">domNode</span><span class="p">);</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">parent</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">newVnode</span><span class="p">));</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">REPLACE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">parent</span><span class="p">.</span><span class="nf">replaceChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">newVnode</span><span class="p">),</span> <span class="nx">domNode</span><span class="p">);</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">domNode</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="o">=</span> <span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">UPDATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">propPatches</span><span class="p">))</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">key</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">on</span><span class="dl">'</span><span class="p">)</span> <span class="o">&amp;&amp;</span> <span class="k">typeof</span> <span class="nx">value</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">function</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Event handler updates are out of scope — skip for now</span>
      <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">value</span> <span class="o">===</span> <span class="kc">undefined</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">domNode</span><span class="p">.</span><span class="nf">removeAttribute</span><span class="p">(</span><span class="nx">key</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="nx">domNode</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="nx">children</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">domNode</span><span class="p">.</span><span class="nx">childNodes</span><span class="p">);</span>
    <span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">childPatches</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">childPatch</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">childPatch</span> <span class="o">&amp;&amp;</span> <span class="nx">childPatch</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">domNode</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">childPatch</span><span class="p">.</span><span class="nx">newVnode</span><span class="p">));</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="nf">patch</span><span class="p">(</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">childPatch</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Each <code>kind</code> maps directly to a DOM operation:</p>

<ul>
<li>
<code>REMOVE</code> — the node was in the old tree but not the new. Remove it.</li>
<li>
<code>CREATE</code> — the node is in the new tree but wasn't in the old. Render and append it.</li>
<li>
<code>REPLACE</code> — the types diverged. Render the new vnode and swap it in.</li>
<li>
<code>TEXT</code> — set <code>nodeValue</code> directly on the text node. No element recreation needed.</li>
<li>
<code>UPDATE</code> — apply changed props, then recurse on children using their index positions in <code>domNode.childNodes</code>.</li>
</ul>

<p>One honest limitation worth calling out: event handler updates (changing an <code>onClick</code> from one function to another) are skipped here. Handling them correctly requires tracking the old listener to call <code>removeEventListener</code> before adding the new one — which means storing refs to functions, which adds state. It's solvable, but not in our line budget. For the demo ahead, event handlers are set once at render time and never change.</p>




<h2>
  
  
  Putting It Together
</h2>

<p>All four functions are done. Here's a counter that updates the DOM on every click — the demo code lives outside the 100-line engine:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
<span class="nt">&lt;body&gt;</span>
<span class="nt">&lt;script&gt;</span>
<span class="c1">// ── engine (71 lines) ──────────────────────────────────</span>
<span class="kd">function</span> <span class="nf">h</span><span class="p">(</span><span class="nx">type</span><span class="p">,</span> <span class="nx">props</span><span class="p">,</span> <span class="p">...</span><span class="nx">children</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="nx">type</span><span class="p">,</span>
    <span class="na">props</span><span class="p">:</span> <span class="nx">props</span> <span class="o">||</span> <span class="p">{},</span>
    <span class="na">children</span><span class="p">:</span> <span class="nx">children</span><span class="p">.</span><span class="nf">flat</span><span class="p">().</span><span class="nf">map</span><span class="p">(</span><span class="nx">child</span> <span class="o">=&gt;</span>
      <span class="k">typeof</span> <span class="nx">child</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">child</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">number</span><span class="dl">'</span>
        <span class="p">?</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">,</span> <span class="na">props</span><span class="p">:</span> <span class="p">{</span> <span class="na">nodeValue</span><span class="p">:</span> <span class="nc">String</span><span class="p">(</span><span class="nx">child</span><span class="p">)</span> <span class="p">}</span> <span class="p">}</span>
        <span class="p">:</span> <span class="nx">child</span>
    <span class="p">)</span>
  <span class="p">};</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">render</span><span class="p">(</span><span class="nx">vnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createTextNode</span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span><span class="p">);</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">el</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">type</span><span class="p">);</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span><span class="nx">vnode</span><span class="p">.</span><span class="nx">props</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">key</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">on</span><span class="dl">'</span><span class="p">)</span> <span class="o">&amp;&amp;</span> <span class="k">typeof</span> <span class="nx">value</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">function</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">el</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="nx">key</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">2</span><span class="p">).</span><span class="nf">toLowerCase</span><span class="p">(),</span> <span class="nx">value</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="nx">el</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">child</span> <span class="k">of</span> <span class="nx">vnode</span><span class="p">.</span><span class="nx">children</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">el</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">child</span><span class="p">));</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="nx">el</span><span class="p">;</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">,</span> <span class="nx">newVnode</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">oldVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">newVnode</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span><span class="p">)</span> <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">REPLACE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">newVnode</span> <span class="p">};</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT_NODE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="o">!==</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span>
      <span class="p">?</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="p">}</span>
      <span class="p">:</span> <span class="kc">null</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">propPatches</span> <span class="o">=</span> <span class="p">{};</span>
  <span class="kd">const</span> <span class="nx">allKeys</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Set</span><span class="p">([...</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">),</span> <span class="p">...</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">)]);</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">key</span> <span class="k">of</span> <span class="nx">allKeys</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">!==</span> <span class="nx">oldVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">])</span> <span class="nx">propPatches</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">=</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">props</span><span class="p">[</span><span class="nx">key</span><span class="p">];</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">childrenLen</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">childPatches</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">({</span> <span class="na">length</span><span class="p">:</span> <span class="nx">childrenLen</span> <span class="p">},</span> <span class="p">(</span><span class="nx">_</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span>
    <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">newVnode</span><span class="p">.</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">])</span>
  <span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">propPatches</span><span class="p">).</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">childPatches</span><span class="p">.</span><span class="nf">every</span><span class="p">(</span><span class="nx">p</span> <span class="o">=&gt;</span> <span class="nx">p</span> <span class="o">===</span> <span class="kc">null</span><span class="p">))</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">kind</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UPDATE</span><span class="dl">'</span><span class="p">,</span> <span class="nx">propPatches</span><span class="p">,</span> <span class="nx">childPatches</span> <span class="p">};</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">patch</span><span class="p">(</span><span class="nx">domNode</span><span class="p">,</span> <span class="nx">patchDescriptor</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span> <span class="o">===</span> <span class="kc">null</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">parent</span> <span class="o">=</span> <span class="nx">domNode</span><span class="p">.</span><span class="nx">parentNode</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">REMOVE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span> <span class="nx">parent</span><span class="p">.</span><span class="nf">removeChild</span><span class="p">(</span><span class="nx">domNode</span><span class="p">);</span> <span class="k">return</span><span class="p">;</span> <span class="p">}</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span> <span class="nx">parent</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">newVnode</span><span class="p">));</span> <span class="k">return</span><span class="p">;</span> <span class="p">}</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">REPLACE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span> <span class="nx">parent</span><span class="p">.</span><span class="nf">replaceChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">newVnode</span><span class="p">),</span> <span class="nx">domNode</span><span class="p">);</span> <span class="k">return</span><span class="p">;</span> <span class="p">}</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span> <span class="nx">domNode</span><span class="p">.</span><span class="nx">nodeValue</span> <span class="o">=</span> <span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span> <span class="k">return</span><span class="p">;</span> <span class="p">}</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">UPDATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="p">[</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">]</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">entries</span><span class="p">(</span><span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">propPatches</span><span class="p">))</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">key</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">on</span><span class="dl">'</span><span class="p">)</span> <span class="o">&amp;&amp;</span> <span class="k">typeof</span> <span class="nx">value</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">function</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// skip — event handler updates not supported</span>
      <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">value</span> <span class="o">===</span> <span class="kc">undefined</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">domNode</span><span class="p">.</span><span class="nf">removeAttribute</span><span class="p">(</span><span class="nx">key</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="nx">domNode</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="nx">value</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
    <span class="kd">const</span> <span class="nx">children</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">domNode</span><span class="p">.</span><span class="nx">childNodes</span><span class="p">);</span>
    <span class="nx">patchDescriptor</span><span class="p">.</span><span class="nx">childPatches</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">childPatch</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">childPatch</span> <span class="o">&amp;&amp;</span> <span class="nx">childPatch</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">CREATE</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">domNode</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nf">render</span><span class="p">(</span><span class="nx">childPatch</span><span class="p">.</span><span class="nx">newVnode</span><span class="p">));</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="nf">patch</span><span class="p">(</span><span class="nx">children</span><span class="p">[</span><span class="nx">i</span><span class="p">],</span> <span class="nx">childPatch</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
<span class="c1">// ── end engine ──────────────────────────────────────────</span>

<span class="c1">// Demo: counter (not counted in the 79 lines)</span>
<span class="kd">let</span> <span class="nx">count</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">view</span><span class="p">(</span><span class="nx">n</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">div</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span>
    <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">p</span><span class="dl">'</span><span class="p">,</span> <span class="p">{},</span> <span class="s2">`Count: </span><span class="p">${</span><span class="nx">n</span><span class="p">}</span><span class="s2">`</span><span class="p">),</span>
    <span class="nf">h</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">onClick</span><span class="p">:</span> <span class="nx">increment</span> <span class="p">},</span> <span class="dl">'</span><span class="s1">Increment</span><span class="dl">'</span><span class="p">)</span>
  <span class="p">);</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">increment</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">oldVdom</span> <span class="o">=</span> <span class="nf">view</span><span class="p">(</span><span class="nx">count</span><span class="p">);</span>
  <span class="nx">count</span><span class="o">++</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">newVdom</span> <span class="o">=</span> <span class="nf">view</span><span class="p">(</span><span class="nx">count</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">patches</span> <span class="o">=</span> <span class="nf">diff</span><span class="p">(</span><span class="nx">oldVdom</span><span class="p">,</span> <span class="nx">newVdom</span><span class="p">);</span>
  <span class="nf">patch</span><span class="p">(</span><span class="nx">container</span><span class="p">,</span> <span class="nx">patches</span><span class="p">);</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">initialVdom</span> <span class="o">=</span> <span class="nf">view</span><span class="p">(</span><span class="nx">count</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">container</span> <span class="o">=</span> <span class="nf">render</span><span class="p">(</span><span class="nx">initialVdom</span><span class="p">);</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nx">container</span><span class="p">);</span>
<span class="nt">&lt;/script&gt;</span>
<span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<p>Save this as <code>index.html</code> and open it in a browser. Click the button. Open DevTools and watch the Elements panel — only the text node inside <code>&lt;p&gt;</code> flashes on each click. The <code>&lt;div&gt;</code>, <code>&lt;p&gt;</code>, and <code>&lt;button&gt;</code> elements are never recreated.</p>

<p><strong>Line count audit:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Function</th>
<th>Lines</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>h()</code></td>
<td>11</td>
</tr>
<tr>
<td><code>render()</code></td>
<td>17</td>
</tr>
<tr>
<td><code>diff()</code></td>
<td>21</td>
</tr>
<tr>
<td><code>patch()</code></td>
<td>22</td>
</tr>
<tr>
<td><strong>Total</strong></td>
<td><strong>71 lines</strong></td>
</tr>
</tbody>
</table></div>

<p>71 lines. The title promised 100 and we came in under budget — because every line had to earn its place.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszylqatb01bwch06nmh7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszylqatb01bwch06nmh7.png" alt="diagram-05" width="213" height="840"></a></p>

<p><em><code>patch()</code> only touches what changed. The <code>div</code> and <code>p</code> are never recreated.</em></p>




<h2>
  
  
  What We Skipped — and Why
</h2>

<p>The engine works. It's also deliberately incomplete. Here's what didn't make the cut and why each omission was a real decision, not an oversight.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Why we skipped it</th>
<th>Where to go next</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Keyed diffing</strong></td>
<td>Tracking element identity across reorders requires a map-based algorithm — roughly 40 extra lines just for this case. Without keys, reordering a list of 10 items produces 10 patches instead of 1 move operation. Acceptable for small lists; not for production.</td>
<td>
<a href="https://github.com/snabbdom/snabbdom" rel="noopener noreferrer">Snabbdom source</a> — the keyed diff is readable and well-commented</td>
</tr>
<tr>
<td><strong>Event handler updates</strong></td>
<td>Correctly swapping a listener requires storing the previous function to call <code>removeEventListener</code>. That means tracking state outside the vnode, which breaks the stateless-function model of our engine.</td>
<td>Preact handles this cleanly — worth reading</td>
</tr>
<tr>
<td><strong>Components</strong></td>
<td>A component is just a function that returns a vnode: <code>const Button = (props) =&gt; h('button', props, 'Click')</code>. Adding component support means resolving these functions during <code>diff()</code> — another ~20 lines.</td>
<td>
<a href="https://pomb.us/build-your-own-react/" rel="noopener noreferrer">Didact</a> adds components in step 5</td>
</tr>
<tr>
<td><strong>Fiber / async rendering</strong></td>
<td>React's Fiber architecture makes reconciliation interruptible — it can pause diffing mid-tree and yield to the browser. This prevents long trees from blocking the main thread. Our synchronous diff would freeze on trees with thousands of nodes.</td>
<td><a href="https://github.com/acdlite/react-fiber-architecture" rel="noopener noreferrer">React Fiber Architecture</a></td>
</tr>
<tr>
<td><strong>SVG support</strong></td>
<td>SVG elements require <code>document.createElementNS('http://www.w3.org/2000/svg', tag)</code> instead of <code>createElement</code>. A two-line change, but it complicates the <code>render()</code> function's element creation path.</td>
<td>MDN: <code>createElementNS</code>
</td>
</tr>
<tr>
<td><strong>Batched updates</strong></td>
<td>Calling <code>diff</code> + <code>patch</code> synchronously on every state change means 10 rapid updates = 10 separate DOM mutations. React batches these into one.</td>
<td>React 18 automatic batching docs</td>
</tr>
</tbody>
</table></div>

<p>The point isn't that our engine is naive — it's that each of these features exists because a real problem demanded it. Keyed diffing came from list performance bugs. Fiber came from animation jank in complex trees. Every line in React or Vue's codebase is a solved problem that our 79 lines haven't encountered yet.</p>

<p>That's not a failure. That's scope.</p>




<h2>
  
  
  Conclusion
</h2>

<p>71 lines of vanilla JavaScript. Four functions. A working Virtual DOM that mounts trees, diffs them, and applies the minimum necessary changes to the real DOM.</p>

<p>That's all React does at its core too. The rest — hooks, Fiber, concurrent features, the dev tools, the ecosystem — is years of engineering on top of this same foundation. Understanding the foundation doesn't make the rest trivial, but it makes it legible.</p>

<p>If you want to keep going, here's a path with increasing depth:</p>

<ol>
<li><p><strong><a href="https://github.com/snabbdom/snabbdom" rel="noopener noreferrer">Snabbdom</a></strong> — A production-grade Virtual DOM in ~800 lines. Modular, readable, and the library Vue's original VDOM was based on. Start here to see how keyed diffing and a module system extend what we built.</p></li>
<li><p><strong><a href="https://github.com/preactjs/preact" rel="noopener noreferrer">Preact</a></strong> — A React-compatible implementation in ~4KB gzipped. The source is approachable and shows how components, hooks, and event handling layer on top of a minimal VDOM core.</p></li>
<li><p><strong><a href="https://pomb.us/build-your-own-react/" rel="noopener noreferrer">Build your own React</a></strong> by Rodrigo Pombo — The definitive deep-dive. Walks through Fiber, reconciliation, and hooks from scratch. Pick this up when you're ready to understand React's scheduler.</p></li>
</ol>

<p>One last thing worth holding onto: the 100-line constraint wasn't a gimmick. It forced every function to do exactly one job and nothing else. <code>diff()</code> doesn't touch the DOM. <code>patch()</code> doesn't read vnodes. <code>h()</code> doesn't know what <code>render()</code> does. That separation is why each piece is understandable in isolation.</p>

<p>Constraints are how you find out what actually matters.</p>

