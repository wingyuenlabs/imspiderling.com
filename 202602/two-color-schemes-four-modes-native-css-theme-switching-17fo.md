---
Title: Two Color Schemes, Four Modes: Native CSS Theme Switching.
Description: 
Author: Olga Urentseva
Date: 2026-02-24T21:42:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>Frontend is finally moving toward vanillaization and that is the best thing that has happened to frontend since Wes Bos published his first courses. Native browser support for themes works absolutely fine and that can be enough for many projects, but I just wanted to have fun. I didn't want just one color scheme with light and dark variants. I wanted two, each supporting light and dark. Four variants in total. And the key is using native browser features. And it worked.</p>

<h3>
  
  
  The Requirements:
</h3>

<ul>
<li>Light and dark mode, handled by the browser natively</li>
<li>A second "spring" (or whatever) color scheme with a completely different palette</li>
<li>A toggle button to switch between default and spring</li>
<li>No flash of wrong theme on reload</li>
<li>No page reload needed to switch</li>
</ul>

<h3>
  
  
  The Specificity Hack:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nd">:root:root:root</span>
</code></pre>

</div>



<p>If you use styled-components, you've probably run into specificity wars where your global CSS variables get overridden by styled-components' injected styles. The fix is delightfully dumb, just repeat :root three times. Yes, I know, styled-components is outdated,  so my future desire would be to get rid of it, but for now it is not the main topic of the conversation :)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nd">:root:root:root</span> <span class="p">{</span>
  <span class="py">--color-background</span><span class="p">:</span> <span class="n">light-dark</span><span class="p">(</span><span class="n">oklch</span><span class="p">(</span><span class="m">0.99</span> <span class="m">0.0105</span> <span class="m">320.98</span><span class="p">),</span> <span class="n">oklch</span><span class="p">(</span><span class="m">13.709%</span> <span class="m">0.02553</span> <span class="m">268.319</span><span class="p">));</span>
  <span class="py">--color-primary</span><span class="p">:</span> <span class="n">light-dark</span><span class="p">(</span><span class="n">oklch</span><span class="p">(</span><span class="m">70.61%</span> <span class="m">0.085</span> <span class="m">271.69</span><span class="p">),</span> <span class="n">oklch</span><span class="p">(</span><span class="m">0.79</span> <span class="m">0.1233</span> <span class="m">266.14</span><span class="p">));</span>
  <span class="c">/* ...etc */</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The browser picks the right value automatically based on the user's system preference. No JavaScript involved. You do still need color-scheme declared for this to work:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>// index.html
<span class="cp">&lt;!doctype html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"UTF-8"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1.0"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"theme-color"</span> <span class="na">content=</span><span class="s">"#ffffff"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"color-scheme"</span> <span class="na">content=</span><span class="s">"light dark"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;title&gt;</span>Title<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"root"</span><span class="nt">&gt;&lt;/div&gt;</span>
    <span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"module"</span> <span class="na">src=</span><span class="s">"/src/main.tsx"</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>

</code></pre>

</div>



<p>Important thing. For browsers that don't support light-dark() yet, keep <a class="mentioned-user" href="https://dev.to/media">@media</a> (prefers-color-scheme) fallback blocks below. The cascade handles it gracefully.</p>

<p>But you probably notice that I just mentioned 2 themes of 1 color-scheme. And the whole approach above is something that was highly used before. But let's add another one. And it's gonna be tricky (not really).</p>

<h3>
  
  
  The Spring Theme: One CSS File, A Class on HTML Tag
</h3>

<p>For the second theme, I wanted to avoid dynamic CSS imports, because they're async, unreliable in production builds with Vite, and add complexity. I tried it, didn't work well. Instead, decided that both themes should be in one CSS file.<br>
The spring theme overrides default variables using a .spring class on the  element. The key insight is specificity: .spring:root:root:root beats :root:root:root because of the extra class:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nd">:root:root:root</span> <span class="p">{</span>
  <span class="py">--color-background</span><span class="p">:</span> <span class="n">light-dark</span><span class="p">(</span><span class="n">oklch</span><span class="p">(</span><span class="m">0.99</span> <span class="m">0.0105</span> <span class="m">320.98</span><span class="p">),</span> <span class="n">oklch</span><span class="p">(</span><span class="m">0.21</span> <span class="m">0.037</span> <span class="m">271.06</span><span class="p">));</span>
  <span class="py">--color-primary</span><span class="p">:</span> <span class="n">light-dark</span><span class="p">(</span><span class="n">oklch</span><span class="p">(</span><span class="m">70.61%</span> <span class="m">0.085</span> <span class="m">271.69</span><span class="p">),</span> <span class="n">oklch</span><span class="p">(</span><span class="m">0.79</span> <span class="m">0.1233</span> <span class="m">266.14</span><span class="p">));</span>
<span class="p">}</span>

<span class="c">/* this one wins when .spring is on &lt;html&gt; */</span>
<span class="nc">.spring</span><span class="nd">:root:root:root</span> <span class="p">{</span>
  <span class="py">--color-background</span><span class="p">:</span> <span class="n">light-dark</span><span class="p">(</span><span class="n">oklch</span><span class="p">(</span><span class="m">0.99</span> <span class="m">0.012</span> <span class="m">150</span><span class="p">),</span> <span class="n">oklch</span><span class="p">(</span><span class="m">0.18</span> <span class="m">0.05</span> <span class="m">145</span><span class="p">));</span>
  <span class="py">--color-primary</span><span class="p">:</span> <span class="n">light-dark</span><span class="p">(</span><span class="n">oklch</span><span class="p">(</span><span class="m">56.316%</span> <span class="m">0.10067</span> <span class="m">150.907</span><span class="p">),</span> <span class="n">oklch</span><span class="p">(</span><span class="m">0.72</span> <span class="m">0.2</span> <span class="m">145</span><span class="p">));</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Light and dark still work automatically inside both themes and light-dark() keeps doing its job regardless of which theme is active.</p>

<h3>
  
  
  The Toggle
</h3>

<p>classList.toggle() flips the class and returns the new boolean state. That's the entire toggle logic. No React state (or whatever you use), no context, no re-renders. The CSS reacts instantly because the class change is reflected immediately in the DOM.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// UI component</span>
<span class="kd">function</span> <span class="nf">handleToggle</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">isSpring</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">documentElement</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nf">toggle</span><span class="p">(</span><span class="dl">"</span><span class="s2">spring</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">localStorage</span><span class="p">.</span><span class="nf">setItem</span><span class="p">(</span><span class="dl">"</span><span class="s2">theme</span><span class="dl">"</span><span class="p">,</span> <span class="nx">isSpring</span> <span class="p">?</span> <span class="dl">"</span><span class="s2">spring</span><span class="dl">"</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">default</span><span class="dl">"</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Applying the Right Theme on Load
</h3>

<p>This runs synchronously before the first paint. No flash, no layout shift, no useEffect timing issues.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// main.tsx</span>
<span class="k">import</span> <span class="dl">"</span><span class="s2">./themes.css</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">savedTheme</span> <span class="o">=</span> <span class="nx">localStorage</span><span class="p">.</span><span class="nf">getItem</span><span class="p">(</span><span class="dl">"</span><span class="s2">theme</span><span class="dl">"</span><span class="p">);</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">documentElement</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nf">toggle</span><span class="p">(</span><span class="dl">"</span><span class="s2">spring</span><span class="dl">"</span><span class="p">,</span> <span class="nx">savedTheme</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">spring</span><span class="dl">"</span><span class="p">);</span>

<span class="nx">ReactDOM</span><span class="p">.</span><span class="nf">createRoot</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">root</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">).</span><span class="nf">render</span><span class="p">(</span>
  <span class="p">&lt;</span><span class="nc">React</span><span class="p">.</span><span class="nc">StrictMode</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nc">App</span> <span class="p">/&gt;</span>
  <span class="p">&lt;/</span><span class="nc">React</span><span class="p">.</span><span class="nc">StrictMode</span><span class="p">&gt;</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  What I Tried That Didn't Work
</h3>

<p>Before landing on this decision, I experimented with @container style() queries — a newer CSS feature that lets you apply styles based on the value of a custom property on a parent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="k">@container</span> <span class="n">style</span><span class="p">(</span><span class="n">--theme</span><span class="p">:</span> <span class="n">spring</span><span class="p">)</span> <span class="p">{</span>
  <span class="nt">body</span> <span class="p">{</span>
    <span class="py">--color-background</span><span class="p">:</span> <span class="p">...;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And it worked! But in Chrome. Not even in Firefox Developer Edition. Don't even talk about Safari. So I dropped it. But it is worth keeping an eye on for the future.  It would have been a much more elegant approach.</p>

<h3>
  
  
  The Result
</h3>

<ul>
<li>One CSS file with all four theme variants (light default, dark default, light spring, dark spring)</li>
<li>Zero JavaScript for color values, all colors live in CSS</li>
<li>No React context or state for theming</li>
<li>No page reload on toggle</li>
<li>No flash of wrong theme on load</li>
<li>Native browser dark mode support throughout</li>
</ul>

