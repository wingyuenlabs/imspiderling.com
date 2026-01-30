---
Title: Fix Razorpay Iframe White Background in Dark Mode
Description: 
Author: c3phalik
Date: 2026-01-30T21:34:22.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Problem
</h2>

<p>When embedding <strong>Razorpay Checkout</strong> on a page with dark mode enabled, the iframe renders with an opaque white background instead of the expected semi-transparent backdrop. This creates a jarring visual experience where the modal appears as a white box against a dark-themed page.</p>

<h2>
  
  
  Root Cause
</h2>

<p>This is a <strong>browser behavior</strong>, not a Razorpay bug.</p>

<p>When the embedding page defines a <code>color-scheme</code> (for example, <code>dark</code>), browsers attempt to apply a matching background to embedded iframes. Because Razorpay Checkout does not natively support dark mode, the browser forces an opaque light background on the iframe content.</p>

<h2>
  
  
  Solution
</h2>

<p>Explicitly set <code>color-scheme: light;</code> on the iframe. This tells the browser that the iframe should always be rendered using a light color scheme, overriding the parent page’s dark-mode inference.</p>

<h3>
  
  
  CSS Fix (Recommended)
</h3>

<p>Add the following rule to your stylesheet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="c">/* Prevent the browser from forcing a dark-mode background on iframe content */</span>
<span class="nt">iframe</span> <span class="p">{</span>
  <span class="py">color-scheme</span><span class="p">:</span> <span class="n">light</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Alternative: Inline Style
</h3>

<p>You can also apply this directly to the iframe element:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;iframe</span>
  <span class="na">src=</span><span class="s">"your-razorpay-url"</span>
  <span class="na">style=</span><span class="s">"color-scheme: light;"</span>
<span class="nt">&gt;&lt;/iframe&gt;</span>
</code></pre>

</div>



<h3>
  
  
  Targeted Selector (Optional)
</h3>

<p>If you want to scope the fix only to Razorpay iframes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nt">iframe</span><span class="o">[</span><span class="nt">src</span><span class="o">*=</span><span class="s1">"razorpay"</span><span class="o">]</span> <span class="p">{</span>
  <span class="py">color-scheme</span><span class="p">:</span> <span class="n">light</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Why This Works
</h2>

<p>The <code>color-scheme</code> CSS property controls how browsers render built-in UI and default backgrounds for an element. By setting <code>color-scheme: light;</code> on the iframe:</p>

<ol>
<li>The browser stops inheriting the dark color scheme from the parent page</li>
<li>The iframe renders using its intended light-mode styles</li>
<li>The semi-transparent backdrop displays correctly</li>
</ol>

<h2>
  
  
  Razorpay-Specific Notes
</h2>

<ul>
<li>For the <strong>Razorpay Affordability Widget</strong>, dark mode can be enabled via official configuration options</li>
<li>For standard <strong>Razorpay Checkout</strong>, this CSS rule is the recommended workaround</li>
<li>This fix affects only visual rendering and does <strong>not</strong> impact checkout functionality</li>
</ul>

<h2>
  
  
  References
</h2>

<ul>
<li><a href="https://sergeyski.com/css-color-scheme-and-iframes-lessons-learned-from-disqus-background-bug/" rel="noopener noreferrer">CSS <code>color-scheme</code> and iframes – Sergeyski</a></li>
<li><a href="https://fvsch.com/transparent-iframes" rel="noopener noreferrer">Transparent iframes and dark mode – fvsch</a></li>
<li><a href="https://github.com/tailwindlabs/tailwindcss/discussions/12057" rel="noopener noreferrer">Tailwind CSS iframe dark mode discussion</a></li>
</ul>

