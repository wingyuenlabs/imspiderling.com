---
Title: We Fixed React's Context API: Introducing react-signal-context
Description: 
Author: pinniola
Date: 2025-10-07T21:59:35.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you're a React developer, you probably love its simplicity and power. But there's a good chance you've also fought against a silent enemy hiding in plain sight: the <strong>unnecessary re-render</strong>. And the most common culprit? Ironically, it's a tool we use every day to make our lives easier: the <strong>Context API</strong>.</p>

<p>For years, the community has accepted an uncomfortable trade-off: use Context to avoid "prop drilling," but then spend hours manually optimizing with <code>React.memo</code>, <code>useMemo</code>, and <code>useCallback</code> to hold back the tide of re-renders that follows.</p>

<p><strong>What if we told you that this trade-off is no longer necessary?</strong></p>

<h2>
  
  
  The Problem: Death by a Thousand Re-Renders
</h2>

<p>The internal workings of React's Context API are like a radio broadcast: when the value in the Provider changes, it transmits the new state to all listening components, <strong>indiscriminately</strong>.</p>

<p>Imagine a global state with user information, the app's theme, and a list of notifications. If only the theme changes from "light" to "dark," the component displaying the user's name and the one showing notifications are also forced to recalculate everything, even though the data they care about hasn't changed one bit. On a large scale, this leads to a slow but relentless performance degradation.</p>

<h2>
  
  
  The Solution: Stop Shouting, Start Subscribing
</h2>

<p>Inspired by the surgical precision of signals and the simplicity of libraries like Zustand, we created <strong>react-signal-context</strong>.</p>

<p>It's a drop-in library with an API that's almost identical to the one you already know and love, but with a completely different engine under the hood. Instead of broadcasting every update to everyone, it allows each component to "whisper" to the store exactly which piece of state it's actually interested in.</p>

<p><strong>The result?</strong> A component re-renders if and only if the specific data it selected has changed. Nothing more, nothing less.</p>

<h2>
  
  
  Seeing is Believing
</h2>

<p>Let's take a look at how familiar it is.</p>

<h3>
  
  
  1. Create your store:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// store.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createSignalContext</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react-signal-context</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="p">{</span> <span class="nx">Provider</span><span class="p">,</span> <span class="nx">useContext</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">createSignalContext</span><span class="p">((</span><span class="kd">set</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="na">count</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
  <span class="na">theme</span><span class="p">:</span> <span class="dl">'</span><span class="s1">light</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">increment</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">count</span><span class="p">:</span> <span class="nx">state</span><span class="p">.</span><span class="nx">count</span> <span class="o">+</span> <span class="mi">1</span> <span class="p">})),</span>
  <span class="na">toggleTheme</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span>
    <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
      <span class="na">theme</span><span class="p">:</span> <span class="nx">state</span><span class="p">.</span><span class="nx">theme</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">light</span><span class="dl">'</span> <span class="p">?</span> <span class="dl">'</span><span class="s1">dark</span><span class="dl">'</span> <span class="p">:</span> <span class="dl">'</span><span class="s1">light</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">})),</span>
<span class="p">}));</span>
</code></pre>

</div>



<h3>
  
  
  2. Consume state... selectively:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Counter.tsx</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useContext</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./store</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">React</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// This component will ONLY re-render if 'count' or 'increment' changes.</span>
<span class="c1">// It will completely ignore theme updates.</span>
<span class="kd">const</span> <span class="nx">Counter</span> <span class="o">=</span> <span class="nx">React</span><span class="p">.</span><span class="nf">memo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">count</span> <span class="o">=</span> <span class="nf">useContext</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">count</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">increment</span> <span class="o">=</span> <span class="nf">useContext</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">increment</span><span class="p">);</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Counter component is rendering!</span><span class="dl">'</span><span class="p">);</span>

  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">button</span> <span class="nx">onClick</span><span class="o">=</span><span class="p">{</span><span class="nx">increment</span><span class="p">}</span><span class="o">&gt;</span><span class="na">Count</span><span class="p">:</span> <span class="p">{</span><span class="nx">count</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/button&gt;</span><span class="err">;
</span><span class="p">});</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ThemeDisplay.tsx</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useContext</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./store</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">React</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// This component will ONLY re-render if 'theme' changes.</span>
<span class="kd">const</span> <span class="nx">ThemeDisplay</span> <span class="o">=</span> <span class="nx">React</span><span class="p">.</span><span class="nf">memo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">theme</span> <span class="o">=</span> <span class="nf">useContext</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">theme</span><span class="p">);</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Theme component is rendering!</span><span class="dl">'</span><span class="p">);</span>

  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">p</span><span class="o">&gt;</span><span class="nx">Current</span> <span class="na">theme</span><span class="p">:</span> <span class="p">{</span><span class="nx">theme</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/p&gt;</span><span class="err">;
</span><span class="p">});</span>
</code></pre>

</div>



<p>With this approach, <strong>performance becomes the default</strong>, not a manual optimization.</p>

<h2>
  
  
  Our Promise: The Performance of Zustand with the Simplicity of the Context API
</h2>

<p><code>react-signal-context</code> is designed to be a "pit of success." We want to make it easy to do the right thing.</p>

<p>✅ <strong>Zero Refactoring</strong>: Switching from the standard <code>React.useContext</code> just requires adding a selector.</p>

<p>✅ <strong>Lightweight</strong>: No external dependencies, just pure React.</p>

<p>✅ <strong>Modern &amp; Future-Proof</strong>: It leverages the <code>useSyncExternalStore</code> hook, the official React-recommended approach for integrating with external stores, ensuring full compatibility with concurrent features.</p>

<h2>
  
  
  Try it Now!
</h2>

<p>We are incredibly excited to share this tool with the community. We believe it can genuinely improve the lives of many React developers.</p>

<p>📦 <strong>Install via npm</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>react-signal-context
</code></pre>

</div>



<p>🔗 <strong>Explore the code on GitHub</strong>: [Link to your GitHub repository]</p>

<p>📖 <strong>Read the docs</strong>: [Link to your README.md on GitHub or a documentation site]</p>

<p>Try it in your next project, or introduce it into an existing one to solve a performance bottleneck. Let us know what you think by opening an issue or a discussion on GitHub.</p>




<p><strong>It's time to stop fighting re-renders and start building faster React applications, the easy way.</strong></p>

<p>What are your experiences with React Context performance issues? Have you found other solutions? Let's discuss in the comments! 👇</p>

