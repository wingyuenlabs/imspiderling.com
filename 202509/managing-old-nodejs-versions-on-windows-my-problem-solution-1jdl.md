---
Title: Managing Old Node.js Versions on Windows: My Problem & Solution
Description: 
Author: arafatruetbd
Date: 2025-09-15T21:39:01.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  Problem
</h3>

<p>I needed to install and switch between older Node.js versions (like <strong>10.12.0</strong> and <strong>12.19.0</strong>) on Windows.</p>

<p>I first tried using <strong>nvm-windows</strong>, but every time I installed an old version, it failed with errors like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>error installing 10.12.0: The system cannot find the file specified.
</code></pre>

</div>



<p>The issue was that nvm-windows tries to download matching npm zip files, and those archives no longer exist for many old Node releases. This meant I couldn’t get the versions I needed without messy manual fixes.</p>




<h3>
  
  
  Solution: Use Volta
</h3>

<p>Instead of fighting with <code>nvm-windows</code>, I switched to <strong><a href="https://volta.sh/" rel="noopener noreferrer">Volta</a></strong>, a modern JavaScript toolchain manager that works perfectly on Windows.</p>

<p>Here’s the complete process I followed:</p>

<h4>
  
  
  1. Install Volta
</h4>

<p>Using <code>winget</code> (recommended):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">winget</span><span class="w"> </span><span class="nx">install</span><span class="w"> </span><span class="nx">Volta.Volta</span><span class="w">
</span></code></pre>

</div>



<p>Or download the latest <code>.msi</code> installer from <a href="https://github.com/volta-cli/volta/releases" rel="noopener noreferrer">Volta releases</a>.</p>

<p>After installation, restart your terminal so Volta is added to your PATH.<br>
Verify it works:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">volta</span><span class="w"> </span><span class="nt">--version</span><span class="w">
</span></code></pre>

</div>






<h4>
  
  
  2. Install Old Node Versions
</h4>

<p>Now I could install older Node.js versions without errors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">volta</span><span class="w"> </span><span class="nx">install</span><span class="w"> </span><span class="nx">node</span><span class="err">@</span><span class="nx">10.12.0</span><span class="w">
</span><span class="n">volta</span><span class="w"> </span><span class="nx">install</span><span class="w"> </span><span class="nx">node</span><span class="err">@</span><span class="nx">12.19.0</span><span class="w">
</span></code></pre>

</div>



<p>Check versions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">node</span><span class="w"> </span><span class="nt">-v</span><span class="w">
</span><span class="n">npm</span><span class="w"> </span><span class="nt">-v</span><span class="w">
</span></code></pre>

</div>






<h4>
  
  
  3. Pin Versions Per Project
</h4>

<p>For projects that depend on specific Node versions, Volta can “pin” the version:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">cd</span><span class="w"> </span><span class="nx">my-legacy-project</span><span class="w">
</span><span class="n">volta</span><span class="w"> </span><span class="nx">pin</span><span class="w"> </span><span class="nx">node</span><span class="err">@</span><span class="nx">10.12.0</span><span class="w">
</span></code></pre>

</div>



<p>This adds the Node version to the project’s <code>package.json</code>. Now whenever I enter this folder, Volta automatically switches to Node 10.12.0.</p>

<p>Another project can have Node 12.19.0 pinned the same way, without conflicts.</p>




<p>✅ <strong>Final takeaway</strong>:<br>
If you’re on Windows and need to run old Node.js versions, skip <code>nvm-windows</code>.<br>
With <strong>Volta</strong>, installation is simple, switching is automatic, and everything just works.</p>




