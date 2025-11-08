---
Title: How to create and publish your own packages on Packagist
Description: 
Author: Antonio Silva
Date: 2025-11-08T20:48:08.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What Is Packagist?
</h2>

<p><a href="https://packagist.org/" rel="noopener noreferrer">Packagist.org</a> is the official package repository for <strong>Composer</strong>, PHP’s dependency manager. It’s where packages are published so anyone can install them easily using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>composer require yourusername/yourpackage
</code></pre>

</div>



<h2>
  
  
  Requirements
</h2>

<p>Before you start, you’ll need:</p>

<ul>
<li>A <a href="https://github.com/" rel="noopener noreferrer">GitHub</a> account</li>
<li>A <a href="https://packagist.org/" rel="noopener noreferrer">Packagist.org</a> account</li>
<li>PHP installed</li>
<li>Composer installed (<code>composer --version</code> to check)</li>
<li>Basic understanding of PHP namespaces and classes</li>
</ul>

<h2>
  
  
  Step 1 — Basic Package Structure
</h2>

<p>Create a folder for your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>MyPackage
<span class="nb">cd </span>MyPackage
</code></pre>

</div>



<p>Inside it, create this structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nt">MyPackage</span><span class="o">/</span>
<span class="err">├──</span> <span class="nt">src</span><span class="o">/</span>
<span class="err">│</span>   <span class="err">└──</span> <span class="nt">Greeting</span><span class="nc">.php</span>
<span class="err">├──</span> <span class="nt">composer</span><span class="nc">.json</span>
<span class="err">└──</span> <span class="nt">README</span><span class="nc">.md</span>
</code></pre>

</div>



<p>Example of <code>src/Greeting.php</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="kn">namespace</span> <span class="nn">MyPackage</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">Greeting</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">static</span> <span class="k">function</span> <span class="n">hello</span><span class="p">(</span><span class="nv">$name</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="s2">"Hello, </span><span class="si">{</span><span class="nv">$name</span><span class="si">}</span><span class="s2">!"</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Step 2 — Create the <code>composer.json</code> File
</h2>

<p>In the root directory, create a file named <code>composer.json</code> with this content:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
    </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"yourusername/my-package"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"A simple greeting package in PHP"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"library"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"license"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MIT"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"autoload"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"psr-4"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"MyPackage\\"</span><span class="p">:</span><span class="w"> </span><span class="s2">"src/"</span><span class="w">
        </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"authors"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
            </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Your Name"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"email"</span><span class="p">:</span><span class="w"> </span><span class="s2">"youremail@example.com"</span><span class="w">
        </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="nl">"require"</span><span class="p">:</span><span class="w"> </span><span class="p">{}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<blockquote>
<p>⚠️ The <code>"name"</code> field must follow the pattern <code>username/package</code>, matching your GitHub username.</p>
</blockquote>

<h2>
  
  
  Step 3 — Test the Autoload Locally
</h2>

<p>Before publishing, test the autoload to make sure everything works.</p>

<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>composer <span class="nb">install</span>
</code></pre>

</div>



<p>Then create a <code>test.php</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="k">require</span> <span class="k">__DIR__</span> <span class="mf">.</span> <span class="s1">'/vendor/autoload.php'</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">MyPackage\Greeting</span><span class="p">;</span>

<span class="k">echo</span> <span class="nc">Greeting</span><span class="o">::</span><span class="nf">hello</span><span class="p">(</span><span class="s2">"MyPackage"</span><span class="p">);</span>
</code></pre>

</div>



<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>php test.php
</code></pre>

</div>



<p>Expected output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Hello, MyPackage!
</code></pre>

</div>



<h2>
  
  
  Step 4 — Push the Project to GitHub
</h2>

<p><strong>1.</strong> Create a public repository on GitHub (e.g., my-package).</p>

<p><strong>2.</strong> Initialize the local repository:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git init
git add <span class="nb">.</span>
git commit <span class="nt">-m</span> <span class="s2">"First version of the package"</span>
git branch <span class="nt">-M</span> main
git remote add origin https://github.com/yourusername/my-package.git
git push <span class="nt">-u</span> origin main
</code></pre>

</div>



<p><strong>3.</strong> Create a version tag (Packagist uses tags to detect releases):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git tag v1.0.0
git push origin v1.0.0
</code></pre>

</div>



<h2>
  
  
  Step 5 — Publish on Packagist
</h2>

<p><strong>1.</strong> Go to <a href="https://packagist.org/packages/submit" rel="noopener noreferrer">https://packagist.org/packages/submit</a></p>

<p><strong>2.</strong> Paste your GitHub repository URL, for example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>https://github.com/yourusername/my-package
</code></pre>

</div>



<p><strong>3.</strong> Click Check, then Submit.</p>

<p>Done 🎉 — your package is now public on Packagist!</p>

<h2>
  
  
  Step 6 — Enable Automatic Updates via GitHub Hook
</h2>

<p>After publishing, you can enable <strong>automatic updates</strong>:</p>

<ul>
<li>Go to your package page on Packagist.</li>
<li>Click “<strong>Maintain</strong>”.</li>
<li>Under “GitHub Hook”, click “<strong>Connect</strong>” to allow Packagist to auto-update your package whenever you push changes to GitHub.</li>
</ul>

<h2>
  
  
  Step 7 — Using Your Package in Another Project
</h2>

<p>In any other PHP project, just run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>composer require yourusername/my-package
</code></pre>

</div>



<p>And use it like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="kn">use</span> <span class="nc">MyPackage\Greeting</span><span class="p">;</span>

<span class="k">echo</span> <span class="nc">Greeting</span><span class="o">::</span><span class="nf">hello</span><span class="p">(</span><span class="s2">"World"</span><span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  Final Tips
</h2>

<ul>
<li>Use <strong>semantic versioning</strong> (<strong>semver</strong>) (<code>v1.0.0</code>, <code>v1.1.0</code>, <code>v2.0.0</code>, etc.)</li>
<li>Keep a clear <strong>README.md</strong> explaining usage.</li>
<li>Add unit tests with <strong>PHPUnit</strong>.</li>
<li>Use <strong>GitHub Actions</strong> for automated testing.</li>
</ul>

