---
Title: Why Angular ARIA in v21 is pretty neat
Description: 
Author: Alexander Thalhammer
Date: 2025-12-01T21:48:45.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Angular ARIA</em> is a collection of <strong>headless, accessible directives</strong> that implement common <strong>WAI-ARIA patterns</strong>. The directives handle <strong>keyboard interactions</strong>, <a href="https://www.angulararchitects.io/blog/aria-roles-attributes/" rel="noopener noreferrer"><strong>ARIA attributes</strong></a>, <strong>focus management</strong>, and <strong>screen reader support</strong>. All you have to do is provide the <strong>HTML</strong> structure, <strong>CSS</strong> styling, and <strong>business logic</strong>!</p>

<p>You feel like you've already read this before? Perfectly possible, because I've just copied and pasted the paragraph from the official <em><a href="https://angular.dev/guide/aria/overview#what-is-angular-aria" rel="noopener noreferrer">Angular ARIA docs</a></em>. Why should I reinvent the wheel, right? It perfectly summarizes the concept.</p>

<p>So the <em>Angular team</em> just (together with <em>v21</em> on <em>Nov 19th, 2025</em>) released a brand-new collection of components – I mean directives – that implement common web patterns while letting you choose your own HTML, styling (CSS, SCSS, or even Tailwind), and business logic (I'd suggest TypeScript).</p>

<p>This is pretty much the opposite approach of <a href="https://material.angular.dev/" rel="noopener noreferrer"><em>Angular Material</em></a>, which is an opinionated Plug &amp; Play Design System. Both are built on top of the <a href="https://www.angulararchitects.io/blog/angular-cdk-accessibility/" rel="noopener noreferrer"><em>Angular CDK</em> A11y</a> package. So what do we get here?</p>

<h2>
  
  
  Components
</h2>

<p>In the first release with <em>v21</em>, the following <strong>12 components / directives / UI patterns</strong> are available:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>search and select</th>
<th>navigation / menu</th>
<th>content organization</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="https://angular.dev/guide/aria/autocomplete" rel="noopener noreferrer">Autocomplete</a></td>
<td><a href="https://angular.dev/guide/aria/menu" rel="noopener noreferrer">Menu</a></td>
<td><a href="https://angular.dev/guide/aria/accordion" rel="noopener noreferrer">Accordion</a></td>
</tr>
<tr>
<td><a href="https://angular.dev/guide/aria/listbox" rel="noopener noreferrer">Listbox</a></td>
<td><a href="https://angular.dev/guide/aria/menubar" rel="noopener noreferrer">Menubar</a></td>
<td><a href="https://angular.dev/guide/aria/tabs" rel="noopener noreferrer">Tabs</a></td>
</tr>
<tr>
<td>
<a href="https://angular.dev/guide/aria/select" rel="noopener noreferrer">Select</a> &amp; <a href="https://angular.dev/guide/aria/multiselect" rel="noopener noreferrer">Multiselect</a>
</td>
<td><a href="https://angular.dev/guide/aria/toolbar" rel="noopener noreferrer">Toolbar</a></td>
<td><a href="https://angular.dev/guide/aria/tree" rel="noopener noreferrer">Tree</a></td>
</tr>
<tr>
<td><a href="https://angular.dev/guide/aria/combobox" rel="noopener noreferrer">Combobox</a></td>
<td></td>
<td><a href="https://angular.dev/guide/aria/grid" rel="noopener noreferrer">Grid</a></td>
</tr>
</tbody>
</table></div>

<p> <br>
There are – of course – plans to extend this in the future 😏</p>

<h2>
  
  
  Why is this neat?
</h2>

<p>Using the <em>Angular ARIA</em> directives is a great way to build <em>Angular apps</em> by <strong>offloading some of the heavy lifting</strong> to the <em>Angular team</em> while still allowing <strong>full customization</strong> and <strong>user-interface branding</strong>. BTW: Check out <a href="https://www.youtube.com/watch?v=ihOlHa3I7eI" rel="noopener noreferrer">Jessica's talk at ViteConf on YT</a> for other things in <em>NG v21</em> that are neat.</p>

<h3>
  
  
  When to use Angular ARIA?
</h3>

<ul>
<li>You <strong>know</strong> how to style things (at least Senior in CSS)</li>
<li>You build a custom <strong>Design System</strong> or an <strong>enterprise component library</strong>
</li>
<li>You don't use any Design System and instead handcraft everything yourself</li>
</ul>

<h3>
  
  
  When to avoid Angular ARIA?
</h3>

<ul>
<li>You <strong>don't know</strong> how to style (Junior in CSS – join my <a href="https://www.angulararchitects.io/en/training/angular-styling-workshop/" rel="noopener noreferrer"><strong>NG Styling Workshop</strong></a> 🎨)</li>
<li>You use a <strong>Design System</strong> or an <strong>enterprise component library</strong> (should be covered there)</li>
<li>You don't care about accessibility (come on – don't be a jerk!)</li>
</ul>

<h2>
  
  
  Angular Styling Best Choices Matrix
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Level</strong></th>
<th>
<strong>Angular Material</strong> 🔌</th>
<th>
<strong>3rd-party DS</strong> 🎨</th>
<th>
<strong>Angular ARIA</strong> ♿</th>
<th>
<strong>Custom UI</strong> 🛠️</th>
<th>
<strong>Custom DS</strong> 🏢</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Design</strong></td>
<td>🎨 Very opinionated</td>
<td>⚡ Plug &amp; Play</td>
<td>🤷 Depends on you</td>
<td>🤷 Depends on you</td>
<td>🤷 Depends on you</td>
</tr>
<tr>
<td><strong>Accessibility</strong></td>
<td>⚡ Plug &amp; Play</td>
<td>🙂 Usually okay</td>
<td>⭐ Best Choice</td>
<td>🤷 Depends on you</td>
<td>🤷 Depends on you</td>
</tr>
<tr>
<td><strong>Effort</strong></td>
<td>⭐ Very low</td>
<td>🙂 Medium</td>
<td>😬 High</td>
<td>😅 Only if 1 app</td>
<td>💀 Boss will kill ya</td>
</tr>
<tr>
<td><strong>CSS Skills</strong></td>
<td>🟢 None</td>
<td>🟡 Junior</td>
<td>🔴 Senior</td>
<td>🔴 Senior</td>
<td>🟣 Master</td>
</tr>
<tr>
<td><strong>Customization</strong></td>
<td>🔒 Hard</td>
<td>🤷 Depends</td>
<td>✔ Included</td>
<td>♾️ No limits</td>
<td>⭐ Best Choice</td>
</tr>
<tr>
<td><strong>NG Updates</strong></td>
<td>🚫 Don’t customize</td>
<td>😬 Sometimes painful</td>
<td>🙂 Smooth</td>
<td>😎 Very smooth</td>
<td>😵 A lot of work</td>
</tr>
<tr>
<td><strong>Use it for</strong></td>
<td>🚀 Prototypes &amp; demos</td>
<td>💸 Low-budget/legacy</td>
<td>🌱 Greenfield</td>
<td>🎨 Hobbies</td>
<td>🏢 Enterprise apps</td>
</tr>
</tbody>
</table></div>

<p> <br>
Stay tuned for my next blog post comparing popular <strong>3rd party Design Systems</strong> for Angular vs. <em>Angular Material</em>.</p>

<h2>
  
  
  Workshops
</h2>

<p>If you want to deep-dive into <em>Angular</em>, we offer a variety of workshops – in English and German.</p>

<ul>
<li>
<a href="https://www.angulararchitects.io/en/training/angular-best-practices/" rel="noopener noreferrer"><strong>Best Practices Workshop</strong></a> 📈 (including Design Systems &amp; ARIA)</li>
<li>
<a href="https://www.angulararchitects.io/en/training/angular-styling-workshop/" rel="noopener noreferrer"><strong>NG Styling Workshop</strong></a> 🎨 (including Design Systems)</li>
<li>
<a href="https://www.angulararchitects.io/en/training/angular-accessibility-workshop/" rel="noopener noreferrer"><strong>Accessibility Workshop</strong></a> ♿ (including WAI ARIA)</li>
<li>
<a href="https://www.angulararchitects.io/en/training/angular-performance-optimization-workshop/" rel="noopener noreferrer"><strong>Performance Workshop</strong></a> 🚀 (because it's neat)</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p><em>Angular ARIA</em> delivers headless, accessible building blocks for fully customizable <em>Angular components</em>. It shifts complexity – keyboard handling, ARIA attributes, focus management – to the <em>Angular team</em>, making it ideal for custom Design Systems and tailored UIs without relying on heavy, opinionated frameworks.</p>

<p>This blog post was written by <a href="https://alex.thalhammer.name" rel="noopener noreferrer">Alexander Thalhammer</a>. Comment it <a href="https://www.linkedin.com/posts/thalhammer_angular-aria-initial-version-released-angulararchitects-activity-7401368955434545152-_bms" rel="noopener noreferrer">LinkedIn</a>, <a href="https://bsky.app/profile/lxt.bsky.social/post/3m6xfgslozs25" rel="noopener noreferrer">bsky</a>, <a href="https://x.com/LX_T/status/1995603392560033812" rel="noopener noreferrer">X</a> or <a href="https://www.reddit.com/r/angular/comments/1odc1ni/comment/nrrzc6k/" rel="noopener noreferrer">Reddit</a>.</p>

