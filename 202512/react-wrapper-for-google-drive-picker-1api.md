---
Title: React Wrapper for Google Drive Picker
Description: 
Author: Justin Poehnelt
Date: 2025-12-18T21:47:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’ve published a new package, <a href="https://www.npmjs.com/package/@googleworkspace/drive-picker-react" rel="noopener noreferrer"><code>@googleworkspace/drive-picker-react</code></a>, to make it easier to use the Google Drive Picker in React applications.</p>

<p>As the creator of the underlying <a href="https://www.npmjs.com/package/@googleworkspace/drive-picker-element" rel="noopener noreferrer"><code>@googleworkspace/drive-picker-element</code></a> web component, I wanted to provide a seamless experience for React developers who might find integrating web components a bit verbose due to the need for manual event listeners and ref handling.</p>

<p>This package wraps the web component, providing a standard React interface with props and event handlers.</p>

<h2>
  
  
  Features
</h2>

<ul>
<li>
<strong>Typed Props</strong>: Full TypeScript support for all Picker options.</li>
<li>
<strong>Event Handling</strong>: Standard <code>onPicked</code> and <code>onCanceled</code> props instead of adding event listeners.</li>
<li>
<strong>SSR Compatible</strong>: Designed to work with Next.js and other SSR frameworks (using client-side directives or dynamic imports).</li>
</ul>

<h2>
  
  
  Usage
</h2>

<h3>
  
  
  Installation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @googleworkspace/drive-picker-react
</code></pre>

</div>



<h3>
  
  
  Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span>
  <span class="nx">DrivePicker</span><span class="p">,</span>
  <span class="nx">DrivePickerDocsView</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@googleworkspace/drive-picker-react</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">App</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">DrivePicker</span>
      <span class="na">clientId</span><span class="p">=</span><span class="s">"YOUR_CLIENT_ID"</span>
      <span class="na">appId</span><span class="p">=</span><span class="s">"YOUR_APP_ID"</span>
      <span class="na">onPicked</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Picked:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">e</span><span class="p">.</span><span class="nx">detail</span><span class="p">)</span><span class="si">}</span>
      <span class="na">onCanceled</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Picker was canceled</span><span class="dl">"</span><span class="p">)</span><span class="si">}</span>
    <span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">DrivePickerDocsView</span> <span class="na">starred</span><span class="p">=</span><span class="si">{</span><span class="kc">true</span><span class="si">}</span> <span class="p">/&gt;</span>
    <span class="p">&lt;/</span><span class="nc">DrivePicker</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Links
</h2>

<ul>
<li><a href="https://www.npmjs.com/package/@googleworkspace/drive-picker-react" rel="noopener noreferrer">NPM Package</a></li>
<li><a href="https://github.com/googleworkspace/drive-picker-element" rel="noopener noreferrer">GitHub Repository</a></li>
</ul>

<p><a href="https://justin.poehnelt.com/posts/react-wrapper-google-drive-picker/" rel="noopener noreferrer">React Wrapper for Google Drive Picker</a> © 2025 by <a href="https://justin.poehnelt.com/" rel="noopener noreferrer">Justin Poehnelt</a> is licensed under CC BY-SA 4.0</p>

