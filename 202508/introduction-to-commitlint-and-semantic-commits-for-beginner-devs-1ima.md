---
Title: Introduction to Commitlint and Semantic Commits for Beginner Devs
Description: 
Author: Werliton Silva
Date: 2025-08-28T21:38:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>Have you ever wondered why some development teams have super organized, easy-to-understand, and standardized commit messages? That’s thanks to <strong>semantic commits</strong> and tools like <strong>commitlint</strong>.</p>

<p>In this article, I’ll show you how to start using these practices in your project with simple language and practical examples you can apply today.</p>




<h2>
  
  
  🧠 What Are Semantic Commits?
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbzz95zawtq70zzjgf9cs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbzz95zawtq70zzjgf9cs.png" alt="scc" width="800" height="227"></a></p>

<p><a href="https://www.conventionalcommits.org/en/v1.0.0/" rel="noopener noreferrer">Semantic commits</a> are commit messages that follow a standardized structure. This helps you quickly understand what was done, enables automation (like changelogs), and improves collaboration between developers.</p>

<h3>
  
  
  📦 Basic Structure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>&lt;<span class="nb">type</span><span class="o">&gt;(</span>&lt;scope&gt;<span class="o">)</span>: &lt;description&gt;
</code></pre>

</div>



<ul>
<li>
<strong>type</strong>: the kind of change (e.g., <code>feat</code>, <code>fix</code>, <code>docs</code>, etc.)</li>
<li>
<strong>scope</strong>: optional, indicates the part of the project affected</li>
<li>
<strong>description</strong>: a brief summary of the change</li>
</ul>

<h2>
  
  
  ✅ Examples
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>feat<span class="o">(</span>auth<span class="o">)</span>: add Google login
fix<span class="o">(</span>MYPROJ-1234<span class="o">)</span>: fix layout bug
docs<span class="o">(</span>readme<span class="o">)</span>: update installation instructions
refactor<span class="o">(</span>api<span class="o">)</span>: improve search performance
</code></pre>

</div>






<h2>
  
  
  🛠️ Installing <a href="https://commitlint.js.org/" rel="noopener noreferrer">Commitlint</a>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fguxs61pfwg09xeeokktt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fguxs61pfwg09xeeokktt.png" alt="cc" width="800" height="161"></a></p>

<p>Let’s set up commitlint in a Node.js project. If you don’t have a project yet, you can create one with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm init <span class="nt">-y</span>
</code></pre>

</div>



<h3>
  
  
  1. Install the dependencies
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> @commitlint/<span class="o">{</span>config-conventional,cli<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Create the config file
</h3>

<p>Create a file named <code>commitlint.config.js</code> in the root of your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>module.exports <span class="o">=</span> <span class="o">{</span>
  extends: <span class="o">[</span><span class="s1">'@commitlint/config-conventional'</span><span class="o">]</span>,
<span class="o">}</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  🔒 Validating Commits with <a href="https://typicode.github.io/husky/" rel="noopener noreferrer">Husky</a>
</h2>

<p>To make sure all commits follow the standard, we’ll use <strong>Husky</strong> to run commitlint automatically.</p>

<h3>
  
  
  1. Install Husky
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> husky
</code></pre>

</div>



<h3>
  
  
  2. Enable hooks
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx husky <span class="nb">install</span>
</code></pre>

</div>



<p>Add this to your <code>package.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="nl">"prepare"</span><span class="p">:</span><span class="w"> </span><span class="s2">"husky install"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  3. Create the commit hook
</h3>

<ul>
<li>
<strong>Create the <code>.husky</code> folder (</strong>if it doesn't already exist*<em>)</em>*:</li>
</ul>

<p>Run the command <code>npx husky init</code>. This will initialize Husky and create the <code>.husky</code> folder in the root of your project.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx husky init
</code></pre>

</div>



<ul>
<li>
<strong>Create the hook file directly</strong>:</li>
</ul>

<p>Inside the <code>.husky</code> folder, create a new file with the name of the hook you need. For Commitlint, the file should be named <code>commit-msg</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Use the 'touch' command on Unix/Linux systems or 'New-Item' in PowerShell (Windows)</span>
<span class="nb">touch</span> .husky/commit-msg
</code></pre>

</div>



<ul>
<li>
<strong>Add the validation script to the file</strong>:</li>
</ul>

<p>Open the <code>.husky/commit-msg</code> file with a text editor and add the command that will run Commitlint. The standard command is <code>npx --no-install commitlint --edit ${1}</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/usr/bin/env sh</span>
npx <span class="nt">--no-install</span> commitlint <span class="nt">--edit</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span>
</code></pre>

</div>



<blockquote>
<p>Note: The <code>npx --no-install</code> command is more efficient because it uses the version of Commitlint that's already installed in your project (via <code>package.json</code>), rather than installing it again on every run.</p>
</blockquote>




<p>Done! Now, if someone tries to make a commit that doesn’t follow the pattern, it’ll be blocked.</p>




<h2>
  
  
  🧪 Testing It Out
</h2>

<p>Try making a commit with the message below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git commit <span class="nt">-m</span> <span class="s2">"fix layout bug"</span>
</code></pre>

</div>



<blockquote>
<p>❌ This commit will be rejected because it doesn’t follow the format.</p>
</blockquote>

<p>Now try:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git commit <span class="nt">-m</span> <span class="s2">"fix(MYPROJ-1234): fix layout bug"</span>
</code></pre>

</div>



<p>✅ This one will be accepted!</p>




<h2>
  
  
  Conclusion
</h2>

<p>Using semantic commits with <code>commitlint</code>is a simple and powerful way to keep your project organized and professional. Even if you’re just starting out, it’s worth adopting this practice right away.</p>

<p>If you want to go further, you can integrate it with tools like Conventional Changelog, semantic-release, and more.</p>




<p>Did you enjoy this content? Let me know how you organize your commits! 💬</p>

