---
Title: Your Web App Keeps Running in Hidden Tabs — Here’s the Fix
Description: 
Author: Alkesh Gupta
Date: 2025-11-02T21:57:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Have you ever had music continue playing after switching tabs?<br>
Or dashboards keep polling APIs while you’re not even watching?</p>

<p>Modern web apps often continue working unnecessarily when the tab is hidden — leading to wasted resources, wrong analytics, and a worse user experience.</p>

<p>This article introduces a lightweight utility that solves that problem cleanly and consistently:<br>
<code>@alkeshgupta/tab-visibility</code></p>

<p>Let's deep-dive into why tab visibility matters, how the browser behaves behind the scenes, and how you can leverage it in React or vanilla JavaScript immediately.</p>

<blockquote>
<p>⭐ If you find this useful, consider leaving a star on GitHub!</p>
</blockquote>


<h3>
  
  
  🧠 The Hidden Problem You Don’t Notice (But Your Users Do)
</h3>

<p>When a browser tab goes into the background:</p>

<ul>
<li><p>Animations keep running</p></li>
<li><p>Videos continue playing</p></li>
<li><p>Timers continue ticking</p></li>
<li><p>API calls keep firing</p></li>
<li><p>Game loops keep looping</p></li>
</ul>

<p>This leads to:</p>

<ul>
<li><p>🔋 Battery drain</p></li>
<li><p>🐌 Performance degradation</p></li>
<li><p>📉 Incorrect business metrics</p></li>
<li><p>📊 Wasted backend server resources</p></li>
<li><p>😡 Annoying UX (media keeps playing when tabbed away)</p></li>
</ul>

<p>Developers often forget to pause/resume behavior based on tab visibility.</p>


<h2>
  
  
  🔍 The Browser’s Behavior: A Quick Primer
</h2>

<p>Browsers expose the Page Visibility API, which gives:</p>

<ul>
<li><p><code>document.visibilityState</code></p></li>
<li><p><code>visibilitychange event</code></p></li>
</ul>

<p>The state can be:</p>

<ul>
<li><p><code>visible</code></p></li>
<li><p><code>hidden</code></p></li>
</ul>

<p>When a user switches tabs, minimizes the browser, or switches focus — the event fires.</p>

<p>Manually handling this looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nb">document</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">visibilitychange</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">visibilityState</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">hidden</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">pauseThings</span><span class="p">();</span>
  <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
    <span class="nf">resumeThings</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>But as you scale your application:</p>

<ul>
<li>You need multiple listeners</li>
<li>You need to clean them up</li>
<li>You need to debounce/throttle events</li>
<li>You need media control logic</li>
<li>You need consistent behavior across browsers</li>
</ul>

<p>It gets messy!</p>




<h2>
  
  
  ❌ Traditional Implementation Pitfalls
</h2>

<p>Most developers:</p>

<ul>
<li>Forget to remove listeners</li>
<li>Duplicate logic across components</li>
<li>Handle blur/focus inconsistently</li>
<li>Mix UI and lifecycle logic incorrectly</li>
<li>Don’t centralize presence handling</li>
</ul>

<p>Even in React, it’s easy to introduce memory leaks.<br>
So we abstracted it.</p>


<h2>
  
  
  ✅ What <code>@alkeshgupta/tab-visibility</code> Solves
</h2>

<p>This utility:<br>
✔️ Detects tab visibility changes<br>
✔️ Provides clean callbacks<br>
✔️ Supports multiple handlers<br>
✔️ Automatically pauses <code>&lt;video&gt;</code> &amp; <code>&lt;audio&gt;</code> elements<br>
✔️ Works in React AND plain JavaScript<br>
✔️ Requires zero configuration</p>

<p>One API to rule them all.<br>
No boilerplate.</p>


<h2>
  
  
  ✨ Key Features
</h2>

<ul>
<li>onShow(callback)</li>
<li>onHide(callback)</li>
<li><p>autoPauseMedia()</p></li>
<li><p>Multi-callback support</p></li>
<li><p>Works globally</p></li>
<li><p>Simple and intuitive</p></li>
</ul>


<h2>
  
  
  📦 Installation
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @alkeshgupta/tab-visibility
</code></pre>

</div>


<p>or<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>yarn add @alkeshgupta/tab-visibility
</code></pre>

</div>






<h2>
  
  
  🧠 Basic Usage
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useTabVisibility</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@alkeshgupta/tab-visibility</span><span class="dl">"</span><span class="p">;</span>

<span class="nx">useTabVisibility</span><span class="p">.</span><span class="nf">onShow</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Tab is visible again!</span><span class="dl">"</span><span class="p">);</span>
<span class="p">});</span>

<span class="nx">useTabVisibility</span><span class="p">.</span><span class="nf">onHide</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">User switched tab!</span><span class="dl">"</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  ⚛️ React Usage Example
</h2>

<p>Works anywhere inside your components.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useEffect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">react</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useTabVisibilityHook</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@alkeshgupta/tab-visibility</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">MyComponent</span><span class="p">()</span> <span class="p">{</span>
  <span class="nf">useTabVisibilityHook</span><span class="p">({</span>
    <span class="na">onHide</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">User switched tab!</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">},</span>

    <span class="na">onShow</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Tab is visible again!</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">},</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="nx">Watching</span> <span class="nx">tab</span> <span class="nx">visibility</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span><span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">MyComponent</span><span class="p">;</span>
</code></pre>

</div>



<p>You can register as many event handlers as you want.</p>




<h2>
  
  
  🔊 Auto Pause Media (Video/Audio)
</h2>

<p>Automatically pause all <code>&lt;video&gt;</code> and <code>&lt;audio&gt;</code> elements when the tab becomes hidden:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">autoPauseMedia</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@alkeshgupta/tab-visibility</span><span class="dl">"</span><span class="p">;</span>

<span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">autoPauseMedia</span><span class="p">();</span>
<span class="p">},</span> <span class="p">[]);</span>
</code></pre>

</div>



<p><code>Optional - Ignore Muted Media:</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">autoPauseMedia</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@alkeshgupta/tab-visibility</span><span class="dl">"</span><span class="p">;</span>

<span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">autoPauseMedia</span><span class="p">({</span> <span class="na">ignoreMuted</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
<span class="p">},</span> <span class="p">[]);</span>
</code></pre>

</div>






<h2>
  
  
  🧩 Vanilla Example (HTML)
</h2>

<p>Works anywhere inside your components.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"module"</span><span class="nt">&gt;</span>
  <span class="k">import</span> <span class="p">{</span> <span class="nx">useTabVisibility</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">https://cdn.jsdelivr.net/npm/@alkeshgupta/tab-visibility@1.0.1/src/index.js</span><span class="dl">"</span><span class="p">;</span>

  <span class="nx">useTabVisibility</span><span class="p">.</span><span class="nf">onShow</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Tab is visible again!</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nx">useTabVisibility</span><span class="p">.</span><span class="nf">onHide</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">User switched tab!</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">});</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>






<h2>
  
  
  🧬 API Reference
</h2>

<h3>
  
  
  <code>onShow(callback: Function)</code>
</h3>

<p>Registers a function to run when tab becomes visible.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">onShow</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Welcome back!</span><span class="dl">"</span><span class="p">));</span>
</code></pre>

</div>



<h3>
  
  
  <code>onHide(callback: Function)</code>
</h3>

<p>Registers a function to run when tab becomes hidden.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">onHide</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Hey come back 👀</span><span class="dl">"</span><span class="p">));</span>
</code></pre>

</div>



<h3>
  
  
  <code>autoPauseMedia()</code>
</h3>

<p>Pauses all video and audio elements on the page.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">autoPauseMedia</span><span class="p">();</span>
</code></pre>

</div>



<h3>
  
  
  <code>autoPauseMedia({ ignoreMuted: true })</code>
</h3>

<p>Pauses all unmuted video and audio elements on the page.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">autoPauseMedia</span><span class="p">({</span> <span class="na">ignoreMuted</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  🧪 Testing Tip
</h2>

<p>To test locally:</p>

<ul>
<li>Open devtools</li>
<li>Start logging visibility events</li>
<li>Switch tabs</li>
<li>Observe:

<ul>
<li>event triggers</li>
<li>media pausing</li>
<li>batched handlers</li>
</ul>


</li>

</ul>

<p>No special instrumentation required.</p>




<h2>
  
  
  🔍 When Should You Use This?
</h2>

<p>Ideal for:</p>

<ul>
<li>Media streaming apps</li>
<li>Dashboards</li>
<li>Trading charts</li>
<li>Data visualizations</li>
<li>Pomodoro timers</li>
<li>Meeting &amp; call apps</li>
<li>Games</li>
<li>Chat presence indicators</li>
</ul>




<h2>
  
  
  ⚙️ Inside the Library (Technical Architecture)
</h2>

<p>Internally, the utility:</p>

<ul>
<li>Registers a single visibilitychange listener</li>
<li>Stores onShow/onHide callbacks in arrays</li>
<li>Executes them sequentially</li>
<li>Optionally inspects DOM media nodes</li>
<li>Avoids duplicate attachment</li>
<li>Works globally across modules</li>
</ul>

<p>This prevents event chaos.</p>




<h2>
  
  
  🛑 Common Pitfalls (Solved)
</h2>

<p>Without this:</p>

<ul>
<li>You may pause media incorrectly</li>
<li>Forget cleanup</li>
<li>Attach listeners multiple times</li>
<li>Cause memory leaks</li>
<li>Miss subtle visibility states</li>
</ul>

<p>This utility removes risk.</p>




<h2>
  
  
  📉 Browser Throttling Behavior
</h2>

<p>Background tabs aggressively throttle:</p>

<ul>
<li><code>setInterval</code></li>
<li><code>setTimeout</code></li>
<li>JavaScript execution priority</li>
<li><code>requestAnimationFrame</code></li>
</ul>

<p>So if you rely on timers — visibility is essential.</p>




<h2>
  
  
  📜 Browser Support
</h2>

<p>✅ Chrome\<br>
✅ Firefox\<br>
✅ Edge\<br>
✅ Safari\<br>
✅ Opera</p>

<p>Uses the standard visibilitychange API, widely supported.</p>




<h2>
  
  
  📏 Notes
</h2>

<ul>
<li>All callbacks can be registered multiple times.</li>
<li>Order of callbacks is preserved.</li>
<li>Works in frameworks (React, Next.js, etc.) after mount.</li>
</ul>




<h2>
  
  
  📎 Links
</h2>

<p>GitHub Repo:<br>
<a href="https://github.com/thealkeshgupta/tab-visibility" rel="noopener noreferrer">https://github.com/thealkeshgupta/tab-visibility</a></p>

<p>npm Package:<br>
<a href="https://www.npmjs.com/package/@alkeshgupta/tab-visibility" rel="noopener noreferrer">https://www.npmjs.com/package/@alkeshgupta/tab-visibility</a></p>

<p>If this saved your time — please ⭐ on GitHub!</p>




<h2>
  
  
  🤝 Contributing
</h2>

<p>PRs are welcome!<br>
Add features, fix edge cases, or open an issue.</p>




<h2>
  
  
  📜 License
</h2>

<p>MIT — free for commercial &amp; personal use.</p>




<h2>
  
  
  ✅ Conclusion
</h2>

<p>Handling tab visibility properly:</p>

<ul>
<li>Saves CPU</li>
<li>Reduces power consumption</li>
<li>Improves UX</li>
<li>Optimizes analytics</li>
<li>Keeps media behavior sane</li>
</ul>

<p>And instead of manually wiring event listeners + cleanup logic in every feature, you now have a reusable utility.</p>

<p>Try <code>@alkeshgupta/tab-visibility</code> in your next project 🚀</p>




