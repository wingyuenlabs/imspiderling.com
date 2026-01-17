---
Title: Git & GitHub: A Beginner’s Guide to Version Control for Data Professionals
Description: 
Author: ondieki frank
Date: 2026-01-17T21:39:57.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Whether you're a data engineer building pipelines or a data scientist working on models, keeping track of your code changes is crucial. That's where Git and GitHub come in. In this guide, I'll walk you through setting up Git and mastering the basics of version control.
</h2>

<h2>
  
  
  1. Installing Git Bash
</h2>

<p><strong>Windows Users</strong><br>
1.<a href="//git-scm.com">Visit</a></p>

<p>2.Download the Windows installer</p>

<p>3.Run the installer with these recommended settings:</p>

<ul>
<li>Select "Use Git from Git Bash only"</li>
<li>Choose "Checkout Windows-style, commit Unix-style line endings"</li>
<li>Use MinTTY as the terminal emulator</li>
<li>Enable file system caching</li>
</ul>

<p><strong>Mac Users:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>brew install git
</code></pre>

</div>



<p><strong>Linux Users:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo apt-get install git  # Debian/Ubuntu
sudo yum install git      # CentOS/Fedora
</code></pre>

</div>



<p>Verify installation by opening Git Bash/Terminal and typing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git --version
</code></pre>

</div>



<h2>
  
  
  2. Connecting Git to Your GitHub Account
</h2>

<h3>
  
  
  Step 1: Configure Your Identity
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
</code></pre>

</div>



<h3>
  
  
  Step 2: Generate SSH Key (Secure Connection)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
</code></pre>

</div>



<p><em>Press Enter to accept default file location, then create a passphrase.</em></p>

<h3>
  
  
  Step 3: Add SSH Key to GitHub
</h3>

<ol>
<li>View your public key:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cat ~/.ssh/id_rsa.pub
</code></pre>

</div>



<ol>
<li><p>Copy the entire output</p></li>
<li><p>Go to GitHub → Settings → SSH and GPG keys → New SSH key</p></li>
<li><p>Paste your key and save</p></li>
</ol>

<h2>
  
  
  Step 4: Test Connection
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ssh -T git@github.com
</code></pre>

</div>



<p><em>You should see: "Hi username! You've successfully authenticated...</em></p>

<p>## 5. Understanding Version Control: The What &amp; Why<br>
<strong>What is Version Control?</strong></p>

<ul>
<li><p>Think of it as a time machine for your code. Every change is saved, so you can:</p></li>
<li><p>Track who made what changes</p></li>
<li><p>Revert to previous versions</p></li>
<li><p>Work on features without breaking your main code</p></li>
<li><p>Collaborate without overwriting others' work</p></li>
</ul>

<h2>
  
  
  6. Why Data Professionals Need Git:
</h2>

<ul>
<li><p>Reproducibility: Track exactly which version of code produced which results</p></li>
<li><p>Collaboration: Multiple team members can work on same project</p></li>
<li><p>Experimentation: Try new approaches without fear of breaking working code</p></li>
<li><p>Documentation: Commit messages explain why changes were made</p></li>
</ul>

