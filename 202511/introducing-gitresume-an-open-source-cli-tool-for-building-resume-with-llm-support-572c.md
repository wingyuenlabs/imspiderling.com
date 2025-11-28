---
Title: Introducing gitresume, an open-source cli tool for building résumé with LLM support
Description: 
Author: Azeez Abiodun Solomon
Date: 2025-11-28T21:51:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>One of the challenges I’ve faced in the past is keeping track of my contributions across different projects, teams, and companies. When it’s time to update my résumé during a job search, it often turns into a struggle to recall what I’ve recently accomplished.</p>

<p>This is why I built Gitresume, a tool that helps me:</p>

<ol>
<li>Keep track of my contributions</li>
<li>Create résumés with AI assistance</li>
<li>Gain insights from my Git commits</li>
<li>Export my résumé in multiple formats</li>
</ol>

<h3>
  
  
  Installation
</h3>

<p>To get started, visit <a href="//gitresume.app">https://gitresume.app</a> and follow the installation instructions.<br>
You’ll need to install the Gitresume CLI, which is available for Windows, macOS, and Linux.</p>

<p><strong>Mac OSX</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code> <span class="nv">$ </span>brew tap iamhabbeboy/homebrew-tap
 <span class="nv">$ </span>brew <span class="nb">install </span>iamhabbeboy/tap/gitresume
</code></pre>

</div>



<p><strong>Linux</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-sL</span> https://raw.githubusercontent.com/iamhabbeboy/gitresume-cli/main/install.sh | bash
</code></pre>

</div>



<p><strong>Windows</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>choco <span class="nb">install </span>gitresume <span class="nt">--version</span><span class="o">=</span>0.1.0

</code></pre>

</div>



<h3>
  
  
  Create your first resume
</h3>

<p>To create a résumé, start the Gitresume server using the command below.<br>
But first, verify that the installation was successful:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>gitresume

Usage:
  gitresume <span class="o">[</span><span class="nb">command</span><span class="o">]</span>

Available Commands:
  ai          Test AI integration
  <span class="nb">help        </span>Help about any <span class="nb">command
  </span>init        Initialize gitresume config
  seed        Seed your commit messages
  serve       Web dashboard <span class="k">for </span>managing your resumes

Flags:
  <span class="nt">-h</span>, <span class="nt">--help</span>   <span class="nb">help </span><span class="k">for </span>gitresume

Use <span class="s2">"gitresume [command] --help"</span> <span class="k">for </span>more information about a <span class="nb">command</span>
</code></pre>

</div>



<p>Then start the server using<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>gitresume serve

🚀 Starting dashboard on http://localhost:4000
✨ Build your resume visually...
</code></pre>

</div>



<p>Next, open your browser and navigate to the dashboard URL. From there, click on Resumes.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fca69wdj0fbstsqnvwvoc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fca69wdj0fbstsqnvwvoc.png" alt="Resume dashboard" width="800" height="478"></a></p>

<p>Click Create Resume to begin<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyzwtrc1vwrr4dstdgkfh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyzwtrc1vwrr4dstdgkfh.png" alt="Create resume" width="800" height="300"></a></p>

<p>You can now start editing the form and customizing your résumé<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe7i201559xeteby4ryws.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe7i201559xeteby4ryws.png" alt="Resume form" width="800" height="438"></a></p>

<p>Check out the full demo on how to create your résumé with Git activity + AI support</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/ewrZfDbZCVE">
</iframe>
</p>

