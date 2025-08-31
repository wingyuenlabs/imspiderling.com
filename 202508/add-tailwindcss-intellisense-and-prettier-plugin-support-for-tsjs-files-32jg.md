---
Title: Add TailwindCSS IntelliSense and Prettier Plugin Support for TS/JS Files
Description: 
Author: Ahmed Zougari
Date: 2025-08-31T22:05:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>When you're working with TailwindCSS, sometimes you need to write class strings in non-TSX files, like exporting variants when using cva (<a href="https://cva.style/docs?" rel="noopener noreferrer">class-variance-authority</a>.</p>

<p>The problem is that you get neither Tailwind IntelliSense nor class sorting from the <a href="https://www.npmjs.com/package/prettier-plugin-tailwindcss" rel="noopener noreferrer">Prettier Tailwind plugin</a>.</p>

<h2>
  
  
  What is the cn utility?
</h2>

<p>If you're familiar with shadcn, you already know it; if not, the snippet below explains it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">clsx</span><span class="p">,</span> <span class="kd">type</span> <span class="nx">ClassValue</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">clsx</span><span class="dl">"</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">twMerge</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">tailwind-merge</span><span class="dl">"</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">cn</span><span class="p">(...</span><span class="nx">inputs</span><span class="p">:</span> <span class="nx">ClassValue</span><span class="p">[])</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nf">twMerge</span><span class="p">(</span><span class="nf">clsx</span><span class="p">(</span><span class="nx">inputs</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Add TailwindCSS Intellisense to typescript files
</h2>

<p><strong>VSCode:</strong> In your <code>settings.json</code>, add this line (you can include any name for your functions).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"tailwindCSS.classFunctions"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"cva"</span><span class="p">,</span><span class="w"> </span><span class="s2">"cn"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>NeoVim:</strong> Make sure you have <a href="https://github.com/neovim/nvim-lspconfig" rel="noopener noreferrer">nvim-lspconfig</a> installed<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight lua"><code><span class="nb">require</span> <span class="s1">'lspconfig'</span><span class="p">.</span><span class="n">tailwindcss</span><span class="p">.</span><span class="n">setup</span><span class="p">({</span>
  <span class="n">settings</span> <span class="o">=</span> <span class="p">{</span>
    <span class="n">tailwindCSS</span> <span class="o">=</span> <span class="p">{</span>
      <span class="n">classFunctions</span> <span class="o">=</span> <span class="p">{</span> <span class="s2">"cva"</span><span class="p">,</span> <span class="s2">"cn"</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">})</span>
</code></pre>

</div>



<h2>
  
  
  Prettier TailwindCSS Plugin Support
</h2>

<p>Simply add this to your existing Prettier config file<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"plugins"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"prettier-plugin-tailwindcss"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"tailwindFunctions"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"cva"</span><span class="p">,</span><span class="w"> </span><span class="s2">"cn"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Bonus: Show the added utilities with the IntelliSense suggestions
</h2>

<p>If you're creating utilities in your CSS file, and they aren't showing with IntelliSense, instead of doing it like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="k">@layer</span> <span class="n">utilities</span> <span class="p">{</span>
  <span class="nc">.bg-gradient</span> <span class="p">{</span>
    <span class="nl">background</span><span class="p">:</span> <span class="n">linear-gradient</span><span class="p">(</span><span class="m">180deg</span><span class="p">,</span> <span class="m">#f5f5ff</span> <span class="m">73%</span><span class="p">,</span> <span class="m">#e0e0ff</span> <span class="m">100%</span><span class="p">);</span>
  <span class="p">}</span>
  <span class="nc">.txt-preset-1</span> <span class="p">{</span>
    <span class="err">@apply</span> <span class="err">text-[52px]</span> <span class="err">font-bold</span> <span class="err">leading-[140%]</span> <span class="err">tracking-[-2px];</span>
  <span class="p">}</span>

  <span class="nc">.txt-preset-1-mobile</span> <span class="p">{</span>
    <span class="err">@apply</span> <span class="err">text-[46px]</span> <span class="err">font-bold</span> <span class="err">leading-[120%]</span> <span class="err">tracking-[-2px];</span>
  <span class="p">}</span>

  <span class="nc">.txt-preset-2</span> <span class="p">{</span>
    <span class="err">@apply</span> <span class="err">text-[40px]</span> <span class="err">font-bold</span> <span class="err">leading-[120%]</span> <span class="err">tracking-[-0.3px];</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Do this; it not only shows in IntelliSense but also works with variants like <code>hover</code>, <code>focus</code>, and <code>lg:</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="k">@utility</span> <span class="n">bg-gradient</span> <span class="p">{</span>
  <span class="nl">background</span><span class="p">:</span> <span class="n">linear-gradient</span><span class="p">(</span><span class="m">180deg</span><span class="p">,</span> <span class="m">#f5f5ff</span> <span class="m">73%</span><span class="p">,</span> <span class="m">#e0e0ff</span> <span class="m">100%</span><span class="p">);</span>
<span class="p">}</span>
<span class="k">@utility</span> <span class="n">txt-preset-1</span> <span class="p">{</span>
  <span class="k">@apply</span> <span class="n">text-</span><span class="p">[</span><span class="m">52px</span><span class="p">]</span> <span class="n">font-bold</span> <span class="n">leading-</span><span class="p">[</span><span class="m">140%</span><span class="p">]</span> <span class="n">tracking-</span><span class="p">[</span><span class="m">-2px</span><span class="p">];</span>
<span class="p">}</span>
<span class="k">@utility</span> <span class="n">txt-preset-1-mobile</span> <span class="p">{</span>
  <span class="k">@apply</span> <span class="n">text-</span><span class="p">[</span><span class="m">46px</span><span class="p">]</span> <span class="n">font-bold</span> <span class="n">leading-</span><span class="p">[</span><span class="m">120%</span><span class="p">]</span> <span class="n">tracking-</span><span class="p">[</span><span class="m">-2px</span><span class="p">];</span>
<span class="p">}</span>

</code></pre>

</div>



