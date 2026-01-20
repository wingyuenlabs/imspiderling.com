---
Title: Modern Power App development takes a lot of... SPACE.
Description: 
Author: Riccardo Gregori
Date: 2026-01-20T22:03:37.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🧠 The Paradigm Shift: Why We Don’t Argue UI Anymore
</h2>

<p>There was a time (you remember it) when any client request that drifted outside the "blessed" Power Apps patterns triggered a <strong>long conversation about platform constraints</strong>. We tried to reshape requirements, push standard controls, and hope stakeholders would accept a slightly clunky interaction with the UI in the name of long-term maintainability.</p>

<p><strong>That era is over</strong>.</p>

<p>With <strong>AI-assisted development</strong> (hello my dear friend, GitHub Copilot), it’s now faster and safer to build <em>small</em>, <em>targeted</em> customizations—<em>PCFs or React WebResources</em>—than to litigate UX. </p>

<p>You can ship exactly what the client envisions, document it properly, and make it maintainable long-term. The payoff is huge: <strong>better UX → higher adoption → stronger platform stickiness</strong>. In other words, great UI is not a "nice-to-have"—it’s a retention strategy.</p>

<p>When you replace “discuss why we can’t” with “deliver what delights,” you:</p>

<ul>
<li><strong>Earn trust faster</strong></li>
<li><strong>Accelerate sign-off</strong></li>
<li>
<strong>Anchor the customer to the platform</strong> with experiences they love</li>
</ul>

<p>But there’s a catch…</p>

<h2>
  
  
  🧱 The Problem: A Dozen PCFs/WebResources = Disk Space Meltdown
</h2>

<p>React-based PCFs and WebResources lean on Node’s ecosystem. And Node’s ecosystem leans on… <code>node_modules</code>—the folder that eats hard drives for breakfast.</p>

<p>Every project carries its own dependency tree. Multiply that across 8, 12, 20 PCFs/WebResources… and your laptop screams. It’s common to see hundreds of thousands of files and tens (or hundreds) of GB consumed across projects.</p>

<p><strong>Good news</strong>: there’s a clean, robust solution that many Power Platform devs underuse:</p>

<h2>
  
  
  🧩 The Hidden Gem: NPM Workspaces
</h2>

<p><strong>NPM Workspaces</strong> let you manage multiple packages/projects (PCFs/WebResources) under one repository, sharing a single top-level node_modules whenever possible. This dramatically reduces duplication and makes dependency management more predictable and CI-friendly.</p>

<p><a href="https://docs.npmjs.com/cli/v7/using-npm/workspaces" rel="noopener noreferrer">Official docs: NPM Workspaces</a></p>

<p>What you get:</p>

<ul>
<li>✅ <strong>One central <code>node_modules</code></strong> (plus small per-package shims when needed)</li>
<li>✅ <strong>Shared tooling</strong> (TypeScript config, lint rules, build scripts)</li>
<li>✅ <strong>Shared local packages</strong> (@types/xrm, @fluentui/react-components) used across PCFs/WRs via workspace:</li>
<li>✅ <strong>Unified scripts</strong> (build, test, lint) across all packages</li>
<li>✅ <strong>Faster CI</strong> with a single dependency tree and cache</li>
</ul>

<h2>
  
  
  🤔 How Does It Works?
</h2>

<p>It's quite straightforward. All you need is to define a <code>package.json</code> and a <code>package-lock.json</code> files at the root of your project folder, just above the folders containing PCFs and WebResources. This will make your project folder a <strong>monorepo</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>📂 my-monorepo/
├─ 📄 package.json         &lt;-- workspace root
├─ 📄 package-lock.json
│
├─ 📂 Pcf/
│  ├─ 📂 pcf1/
│  │  ├─ 📄 package.json
│  │  └─ ...
│  └─ 📂 pcf2/
│     ├─ 📄 package.json
│     └─ ...
│
└─ 📂 WebResources/
   ├─ 📂 webresource1/
   │  ├─ 📄 package.json
   │  └─ ...
   └─ 📂 webresource2/
      ├─ 📄 package.json
      └─ ...
</code></pre>

</div>



<p>It is important that the root <code>package.json</code> file contains the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"my-monorepo"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"private"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"workspaces"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"Pcf/*"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"WebResources/*"</span><span class="p">,</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>You can do it manually, or via:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">cd</span><span class="w"> </span><span class="nx">my-monorepo</span><span class="w">
</span><span class="n">npm</span><span class="w"> </span><span class="nx">init</span><span class="w"> </span><span class="nt">-y</span><span class="w">

</span><span class="n">npm</span><span class="w"> </span><span class="nx">pkg</span><span class="w"> </span><span class="nx">set</span><span class="w"> </span><span class="nx">private</span><span class="o">=</span><span class="n">true</span><span class="w">
</span><span class="nx">npm</span><span class="w"> </span><span class="nx">pkg</span><span class="w"> </span><span class="nx">set</span><span class="w"> </span><span class="nx">workspaces</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="o">=</span><span class="s2">"Pcf/*"</span><span class="w">
</span><span class="n">npm</span><span class="w"> </span><span class="nx">pkg</span><span class="w"> </span><span class="nx">set</span><span class="w"> </span><span class="nx">workspaces</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">=</span><span class="s2">"WebResources/*"</span><span class="w">
</span></code></pre>

</div>



<p>Then you can create your PCFs into the PCF and WebResources folder as usual... <strong>Just a tip</strong>: remember that the value of the <code>name</code> node of the <code>package.json</code> files under each of your PCFs/WRs <strong>must</strong> be unique, otherwise you'll get an error while running <code>npm &lt;whatever&gt;</code>.</p>

<p>When you'll run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">npm</span><span class="w"> </span><span class="nx">install</span><span class="w">
</span></code></pre>

</div>



<p>All dependencies will be installed in a single, above all, <code>node_modules</code> folder created just under the <code>my-monorepo</code> folder.</p>

<blockquote>
<p><strong>NOTE</strong>: If two packages need <strong>different versions of the same dep</strong>, npm may place some deps locally. That’s fine; you’ll still save tons of space overall.</p>
</blockquote>

<h2>
  
  
  🗂️ A Practical Folder Structure for Power Platform Projects
</h2>

<p>Here’s a realistic example of the output folder you'll get, on a standard Power App project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>📂 project-folder
├─ 📄 .gitignore
├─ 📂 src
│  ├─ 📄 package.json
│  ├─ 📄 package-lock.json
│  ├─ 📂 node_modules
│  ├─ 📂 Pcf
│  │  ├─ 📂 NumericInputControl
│  │  │  ├─ 📄 package.json
│  │  │  ├─ 📄 package-lock.json
│  │  │  └─ ...
│  │  ├─ 📂 PurchaseOrderGrid
│  │  │  ├─ 📄 package.json
│  │  │  ├─ 📄 package-lock.json
│  │  │  └─ ...
│  │  └─ 📂 CustomTreeView
│  │     ├─ 📄 package.json
│  │     ├─ 📄 package-lock.json
│  │     └─ ...
│  └─ 📂 WebResources
│     ├─ 📂 HomePage
│     │  ├─ 📄 package.json
│     │  ├─ 📄 package-lock.json
│     │  └─ ...
│     └─ 📂 ControlDashboard
│        ├─ 📄 package.json
│        ├─ 📄 package-lock.json
│        └─ ...
└─ 📂 test
   └─ ... # other stuff
</code></pre>

</div>



<h2>
  
  
  📉 How Much Space Do You Actually Save?
</h2>

<p>In Power Platform solutions with 8–20 React packages, we’ve seen:</p>

<ul>
<li>
<strong>60–80% reduction</strong> in duplicated dependency files</li>
<li>
<strong>Build times drop</strong> (less overall I/O and fewer cold installs)</li>
</ul>

<p>Your mileage varies with library choices, but the direction is always the same: less bloat, more flow.</p>

<h2>
  
  
  🪄 Conclusions
</h2>

<p>AI copilot tools make building <strong>custom</strong>, <strong>crisp UX</strong> easy. <strong>NPM Workspaces</strong> make maintaining many customizations sane. Put them together and you’ve got a Power Platform development lifecycle that’s fast, clean, and—dare I say—fun.</p>

<p>Give it a try, and let me know in the comments what you think about it!</p>

