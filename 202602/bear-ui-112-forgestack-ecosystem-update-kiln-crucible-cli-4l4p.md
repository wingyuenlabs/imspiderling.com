---
Title: Bear UI 1.1.2 + ForgeStack Ecosystem Update (Kiln, Crucible, CLI)
Description: 
Author: John Yaghobieh
Date: 2026-02-27T21:59:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>The ForgeStack ecosystem just got a major quality pass, and <strong>Bear UI 1.1.2</strong> is the center of it.</p>

<p>This update is not only about new components — it also improves docs, theming control, release flow, and the developer experience across tools like <strong>Kiln</strong>, <strong>Crucible</strong>, and the ForgeStack <strong>CLI</strong>.</p>




<h2>
  
  
  What’s New in Bear UI 1.1.2
</h2>

<h3>
  
  
  New components
</h3>

<ul>
<li>
<strong>FormField</strong> (TextField-style behavior)

<ul>
<li>
<code>outlined</code>, <code>filled</code>, and <code>standard</code>
</li>
<li>label starts inside, floats on focus/value</li>
</ul>


</li>

<li>

<strong>AspectRatio</strong>

<ul>
<li>clean media/layout ratio container</li>
</ul>


</li>

<li>

<strong>PasswordInput</strong>

<ul>
<li>visibility toggle</li>
<li>shift/caps indicator option</li>
<li>custom icon props</li>
</ul>


</li>

<li>

<strong>AlertDialog</strong>

<ul>
<li>confirmation flows with a11y defaults</li>
</ul>


</li>

<li>

<strong>InputGroup</strong>

<ul>
<li>grouped field labels, helper/error structure</li>
</ul>


</li>

</ul>

<h3>
  
  
  Existing component upgrades
</h3>

<ul>
<li>
<strong>Input</strong>

<ul>
<li>
<code>clearable</code>, <code>onClear</code>, <code>success</code>, <code>aria-invalid</code>
</li>
<li>character counter support</li>
</ul>


</li>

<li>

<strong>Button</strong>

<ul>
<li>
<code>loadingText</code>, <code>aria-busy</code>
</li>
<li>icon size consistency fixes</li>
</ul>


</li>

<li>

<strong>ResizableTextarea / RichEditor</strong>

<ul>
<li>improved counter/label/error behavior</li>
</ul>


</li>

</ul>




<h2>
  
  
  Theming Improvements (Big One)
</h2>

<p>A key goal was making components truly themeable through <code>BearProvider</code>.</p>

<p>Now you can centrally override styles (like border, color, font size) for more field-level components, including <code>FormField</code> and <code>Input</code>, instead of styling each usage manually.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="p">&lt;</span><span class="nc">BearProvider</span>
  <span class="na">components</span><span class="p">=</span><span class="si">{</span><span class="p">{</span>
    <span class="na">FormField</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">input</span><span class="p">:</span> <span class="p">{</span> <span class="na">borderColor</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#ec4899</span><span class="dl">'</span><span class="p">,</span> <span class="na">fontSize</span><span class="p">:</span> <span class="dl">'</span><span class="s1">15px</span><span class="dl">'</span> <span class="p">},</span>
      <span class="na">label</span><span class="p">:</span> <span class="p">{</span> <span class="na">color</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#ec4899</span><span class="dl">'</span> <span class="p">},</span>
      <span class="na">outlined</span><span class="p">:</span> <span class="p">{</span> <span class="na">borderColor</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#f472b6</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">},</span>
    <span class="na">Input</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">input</span><span class="p">:</span> <span class="p">{</span> <span class="na">borderColor</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#ec4899</span><span class="dl">'</span> <span class="p">},</span>
      <span class="na">label</span><span class="p">:</span> <span class="p">{</span> <span class="na">color</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#ec4899</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">}</span><span class="si">}</span>
<span class="p">&gt;</span>
  <span class="p">&lt;</span><span class="nc">App</span> <span class="p">/&gt;</span>
<span class="p">&lt;/</span><span class="nc">BearProvider</span><span class="p">&gt;</span>
</code></pre>

</div>






<h2>
  
  
  Portal Updates (bearui.com)
</h2>

<ul>
<li>Reworked nav structure</li>
<li>New <strong>Theming &amp; Customization</strong> documentation sections</li>
<li>Better <strong>Store</strong> presentation (MUI-inspired structure, no payment flow)</li>
<li>Improved topbar UX:

<ul>
<li>command-palette style search trigger (<code>⌘K</code> / <code>Ctrl+K</code>)</li>
<li>release/notification updates</li>
<li>settings and cookie preferences</li>
</ul>


</li>

<li>Added Spanish-ready UI strings foundation for key sections</li>

</ul>




<h2>
  
  
  ForgeStack Ecosystem News
</h2>

<h3>
  
  
  Kiln
</h3>

<p>Kiln direction is being refined around publishing workflow and better integration patterns with the broader stack.</p>

<h3>
  
  
  Crucible
</h3>

<p>Crucible continues to evolve as part of the ecosystem roadmap, with focus on practical developer workflows and interoperability.</p>

<h3>
  
  
  ForgeStack CLI
</h3>

<p>Template flow is expanding and becoming more visible in the portal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx create-forge my-app <span class="nt">--template</span> react
</code></pre>

</div>



<p>This is the fastest way to bootstrap with Bear + ForgeStack conventions.</p>




<h2>
  
  
  Why this release matters
</h2>

<p><code>1.1.2</code> is a “developer quality” release:</p>

<ul>
<li>better component behavior parity with familiar UI patterns</li>
<li>cleaner theming system</li>
<li>stronger docs and discoverability</li>
<li>tighter ecosystem story (Bear + CLI + tools)</li>
</ul>

<p>If you already use Bear, this release removes friction.<br><br>
If you are new, it makes the first-run experience much smoother.</p>




<h2>
  
  
  Links
</h2>

<ul>
<li>Bear UI docs: <a href="https://bearui.com" rel="noopener noreferrer">https://bearui.com</a>
</li>
<li>Bear npm: <a href="https://www.npmjs.com/package/@forgedevstack/bear" rel="noopener noreferrer">https://www.npmjs.com/package/@forgedevstack/bear</a>
</li>
</ul>

