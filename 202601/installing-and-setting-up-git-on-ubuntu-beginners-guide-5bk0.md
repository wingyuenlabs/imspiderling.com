---
Title: Installing and Setting Up Git on Ubuntu (Beginner’s Guide)
Description: 
Author: Nyaenya Ombui
Date: 2026-01-14T21:11:17.000Z
Robots: noindex,nofollow
Template: index
---
<p>When starting your journey in software development, one of the first tools you’ll encounter is <strong>Git</strong>. Git is a distributed version control system that helps developers track changes in their code, collaborate with others, and manage projects efficiently.</p>

<p>Most developers store their projects in repositories (commonly called <em>repos</em>) on platforms such as <strong>GitHub</strong>. Git allows you to interact with these repositories directly from your local machine — pushing changes, pulling updates, and managing versions of your code.</p>

<p>In this guide, we’ll walk through how to <strong>install Git on Ubuntu</strong> and perform the <strong>basic configuration needed to get started</strong>.</p>




<h2>
  
  
  Prerequisites
</h2>

<p>Before you begin, make sure you have:</p>

<ul>
<li>A PC running <strong>Ubuntu</strong> (or another Debian-based Linux distribution)</li>
<li>A <strong>GitHub account</strong>
--&gt; If you don’t have one, sign up at: <a href="https://github.com/" rel="noopener noreferrer">https://github.com/</a>
</li>
</ul>




<h2>
  
  
  Step 1: Update Your System Packages
</h2>

<p>Open your terminal using <strong>Ctrl + Alt + T</strong>, then update your package list:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt update
</code></pre>

</div>



<h2>
  
  
  Step 2: Install Git
</h2>

<p>Install Git by running:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt <span class="nb">install </span>git
</code></pre>

</div>



<p>Press Y when prompted to confirm the installation.</p>

<h2>
  
  
  Step 3: Verify the Installation
</h2>

<p>Once the installation is complete, confirm that Git was installed successfully:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git <span class="nt">--version</span>
</code></pre>

</div>



<p>If Git is installed correctly, you’ll see output similar to:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git version 2.x.x
</code></pre>

</div>



<p>If you see the version number, you’re good to proceed.</p>

<h2>
  
  
  Step 4: Configure Your Git Username
</h2>

<p>Git uses a username to identify who made each commit. This should match the username you use on GitHub.</p>

<p>Run the command below and replace your-github-username with your actual GitHub username:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git config <span class="nt">--global</span> user.name <span class="s2">"your-github-username"</span>
</code></pre>

</div>



<h2>
  
  
  Step 5: Configure Your Git Email
</h2>

<p>Next, set the email address associated with your GitHub account. This email will be attached to your commits.</p>

<p>Replace <a href="mailto:your-email@example.com">your-email@example.com</a> with the email you used when signing up on GitHub:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git config <span class="nt">--global</span> user.email <span class="s2">"your-email@example.com"</span>
</code></pre>

</div>



<h2>
  
  
  Step 6: Confirm Your Git Configuration
</h2>

<p>To verify that your username and email were set correctly, run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git config <span class="nt">--list</span>
</code></pre>

</div>



<p>You should see output that includes both your user.name and user.email.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  user.email<span class="o">=</span>your-email@example.com
  user.name<span class="o">=</span>your-github-username
</code></pre>

</div>



<h1>
  
  
  What’s Next?
</h1>

<p>At this point, Git is successfully installed and configured on your machine.<br>
However, to fully interact with GitHub (especially pushing and pulling code), it’s recommended to set up SSH authentication.</p>

<p>In the next article, we’ll cover:</p>

<ul>
<li><p>Generating an SSH key</p></li>
<li><p>Adding it to GitHub</p></li>
<li>
<p>Connecting securely without typing your password every time</p>
<h1>
  
  
  Conclusion
</h1>
</li>
</ul>

<p>Installing and configuring Git is an essential first step for any developer. With Git set up on your Ubuntu system, you’re now ready to start managing projects, collaborating with others, and contributing to open-source software.</p>

<p>Happy coding!!!</p>

