---
Title: Samba on Linux - File Sharing for Mixed Environments
Description: 
Author: Richard Chamberlain
Date: 2025-10-26T21:54:07.000Z
Robots: noindex,nofollow
Template: index
---


<p>title: Samba on Linux - Secure File Sharing for Mixed Environments<br>
subtitle: Enable Controlled File Access Between Windows and Linux with This Step-by-Step Samba Guide<br>
date: 2025-10-25 10:00 +0000<br>
categories: [Linux, Infrastructure]<br>
tags: [Samba, OracleLinux, FileSharing, LinuxAdmin, Security, SMB]<br>
image:<br>
  path: /assets/img/Samba_Init.png</p>
<h2>
  
  
    alt: Secure Samba file sharing setup on Oracle Linux 9 for mixed environments
</h2>

<blockquote>
<p><em>Enable secure, controlled file sharing across Windows and Linux clients using Samba. This guide walks you through a basic setup, which forms the foundation for building a secure, maintainable file server.</em></p>
</blockquote>


<h2>
  
  
  🧰 Introduction to Samba
</h2>

<p>I might be showing my gray hairs here, but <em>Samba</em> was one of the first open source projects I ever heard about—long before I even knew what Linux was.</p>

<p>Back then, I was fresh out of university, working at a small company where the system administrator had set up a Samba server for shared drives across the office. It was one of those “Linux can do that?” moments that stuck with me.</p>

<p>Fast forward to today: even in the age of cloud storage and SaaS platforms, <strong>on-premises file shares are still very much alive</strong>. Samba remains relevant, shipping by default with many Linux distributions or available via core package repositories. Whether you're deploying a small home server or maintaining enterprise file services, Samba is still a reliable and versatile solution.</p>

<p>But what exactly <em>is</em> Samba?</p>


<h2>
  
  
  📚 Table of Contents
</h2>

<ul>
<li>🧰 Introduction to Samba</li>
<li>📦 What Is Samba?</li>
<li>🛠️ Samba Setup: Layers to Watch</li>
<li>🖥️ Setup Environment</li>
<li>🛠️ Install Samba and Start Services</li>
<li>🗂️ Plan and Create Share Structure</li>
<li>👥 Set Up Users and Groups</li>
<li>⚙️ Configure <code>smb.conf</code> for Secure Share Access</li>
<li>🛡️ SELinux Integration</li>
<li>🧪 Test Share Access</li>
<li>✅ Conclusion</li>
<li>🧭 Need a Quick Reference?</li>
</ul>


<h2>
  
  
  📦 What Is Samba?
</h2>

<p>Originally, <strong>Samba</strong> was created to centralize file storage when local disk space on workstations was limited and expensive. With a few spare hard drives, a Linux distro, and some patience, admins could spin up a Samba server to share files over a network using Windows-compatible protocols like SMB/CIFS.</p>

<p>Today, Samba is embedded everywhere—even if you don’t realize it:</p>

<ul>
<li>It powers <strong>home media servers</strong>, streaming media to smart TVs, phones, and tablets.</li>
<li>It serves as <strong>centralized file storage</strong> for teams, labs, and small businesses.</li>
<li>It's the software layer inside many off-the-shelf <strong>NAS appliances</strong>, quietly enabling Windows-style file sharing.</li>
</ul>

<p>If you've ever accessed <code>\\server\share</code>, there's a good chance Samba—or its commercial cousin—is behind the scenes.</p>


<h2>
  
  
  🛠️ Samba Setup: Layers to Watch
</h2>

<p>Samba interacts with <strong>multiple layers of system security</strong>, which means getting a basic share working can be surprisingly complex. Even with a flawless <code>smb.conf</code>, things can still fail due to unrelated system-level issues.</p>

<p>I’ve personally lost hours troubleshooting what I thought was a misconfiguration, only to discover it was a firewall rule, a Linux file permission, or SELinux quietly denying access.</p>

<p>Understanding these layers is key to both troubleshooting and securing Samba:</p>

<ul>
<li>🔥 <strong>Firewall:</strong> Ensure required ports (<code>137</code>, <code>138</code>, <code>139</code>, <code>445</code>) are open.</li>
<li>👤 <strong>Linux User Account:</strong> Samba users must also exist as system users.</li>
<li>🔐 <strong>Samba User Account:</strong> Users must be added via <code>smbpasswd</code>.</li>
<li>🧩 <strong>PAM Integration:</strong> Some distros use PAM for authentication—be aware of this layer.</li>
<li>🛡️ <strong>SELinux / AppArmor:</strong> Security modules can block Samba from accessing files.</li>
<li>📁 <strong>Filesystem Permissions:</strong> Correct Linux ownership and mode bits are essential.</li>
</ul>

<p>Planning for these up front can save hours of troubleshooting down the line.</p>


<h2>
  
  
  🖥️ Setup Environment
</h2>

<p>This guide is based on:</p>

<ul>
<li>
<strong>OS:</strong> Oracle Linux 9 (RHEL-compatible)</li>
<li>
<strong>Services:</strong> <code>smb</code> (Samba server daemon)</li>
</ul>


<h2>
  
  
  🛠️ Install Samba and Start Services
</h2>

<p>Install Samba and enable the service:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>dnf <span class="nb">install </span>samba samba-client samba-common <span class="nt">-y</span>
<span class="nb">sudo </span>systemctl <span class="nb">enable</span> <span class="nt">--now</span> smb
</code></pre>

</div>



<p>Configure the firewall to allow Samba traffic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>firewall-cmd <span class="nt">--permanent</span> <span class="nt">--add-service</span><span class="o">=</span>samba
<span class="nb">sudo </span>firewall-cmd <span class="nt">--reload</span>
</code></pre>

</div>






<h2>
  
  
  🗂️ Plan and Create Share Structure
</h2>

<p>We’ll store shared folders in <code>/srv/samba</code>, following a clean, FHS-friendly layout:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/srv/samba/
└── share/        # Restricted share for authenticated users
</code></pre>

</div>



<p>Create the directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo mkdir</span> <span class="nt">-p</span> /srv/samba/share
</code></pre>

</div>






<h2>
  
  
  👥 Set Up Users and Groups
</h2>

<p>First, create a Linux user account to access the Samba share. We’ll disable shell login for this account:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>useradd <span class="nt">-M</span> <span class="nt">-s</span> /sbin/nologin smbuser
<span class="nb">sudo </span>smbpasswd <span class="nt">-a</span> smbuser
</code></pre>

</div>



<p>If you want to allow group-based access (e.g. a finance team):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create group and add users</span>
<span class="nb">sudo </span>groupadd finance
<span class="nb">sudo </span>usermod <span class="nt">-aG</span> finance smbuser

<span class="c"># Set group ownership and permissions</span>
<span class="nb">sudo chown</span> :finance /srv/samba/share
<span class="nb">sudo chmod </span>770 /srv/samba/share
</code></pre>

</div>






<h2>
  
  
  ⚙️ Configure <code>smb.conf</code> for Secure Share Access
</h2>

<p>Samba configuration is managed in <code>/etc/samba/smb.conf</code>.</p>

<p>A basic configuration includes:</p>

<ul>
<li>A <code>[global]</code> section for server-wide settings</li>
<li>One or more share sections (e.g. <code>[SecureShare]</code>)
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[global]</span>
   <span class="py">workgroup</span> <span class="p">=</span> <span class="s">WORKGROUP</span>
   <span class="py">security</span> <span class="p">=</span> <span class="s">user</span>
   <span class="err">map</span> <span class="err">to</span> <span class="py">guest</span> <span class="p">=</span> <span class="s">Bad User</span>

<span class="nn">[SecureShare]</span>
   <span class="py">path</span> <span class="p">=</span> <span class="s">/srv/samba/share</span>
   <span class="err">valid</span> <span class="py">users</span> <span class="p">=</span> <span class="s">@finance</span>
   <span class="err">guest</span> <span class="py">ok</span> <span class="p">=</span> <span class="s">no</span>
   <span class="py">writable</span> <span class="p">=</span> <span class="s">yes</span>
</code></pre>

</div>



<p>Apply changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>systemctl restart smb
</code></pre>

</div>






<h2>
  
  
  🛡️ SELinux Integration
</h2>

<p>By default, SELinux does <strong>not</strong> allow Samba to access arbitrary directories. To make <code>/srv/samba</code> accessible:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>semanage fcontext <span class="nt">-a</span> <span class="nt">-t</span> samba_share_t <span class="s2">"/srv/samba(/.*)?"</span>
<span class="nb">sudo </span>restorecon <span class="nt">-Rv</span> /srv/samba
</code></pre>

</div>



<blockquote>
<p>⚠️ <strong>Note:</strong> Avoid enabling <code>samba_export_all_rw</code> unless absolutely necessary. Using specific contexts (<code>samba_share_t</code>) is safer and more maintainable.</p>
</blockquote>




<h2>
  
  
  🧪 Test Share Access
</h2>

<h3>
  
  
  From Linux:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>smbclient <span class="nt">-L</span> //&lt;server-ip&gt; <span class="nt">-U</span> smbuser
smbclient //192.168.35.42/SecureShare <span class="nt">-U</span> smbuser
</code></pre>

</div>



<h3>
  
  
  From Windows:
</h3>

<p>In the File Explorer address bar:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>\\&lt;server-ip&gt;\SecureShare
</code></pre>

</div>



<p>Login using the Samba credentials.</p>




<h2>
  
  
  ✅ Conclusion
</h2>

<p>Samba remains a powerful tool for bridging Windows and Linux in mixed environments. In this guide, we covered:</p>

<ul>
<li>What Samba is and why it’s still relevant</li>
<li>How to install and configure Samba on Oracle Linux 9</li>
<li>How to set up both secure and group-based file shares</li>
<li>How to navigate common security and system integration layers (like SELinux)</li>
</ul>

<p>This foundational setup is preproduction-ready for small teams or homelabs. In the <strong>next articles</strong>, we’ll dive deeper into adding additional security.</p>

<h2>
  
  
  🧭 Need a Quick Reference?
</h2>

<p>If you're looking for commands, error fixes, or configuration tips while working with Samba:</p>

<p>➡️ Check out the companion article:<br>
**<a href="https://richard-sebos.github.io/sebostechnology/posts/Samba-Cheatsheet/" rel="noopener noreferrer">🧰 Samba Admin Cheatsheet for Oracle Linux 9</a></p>

<p>It includes:</p>

<ul>
<li>Common error resolutions</li>
<li>Security best practices</li>
<li>Test and validation commands</li>
<li>Handy <code>systemctl</code> and <code>smbclient</code> usage</li>
</ul>

<p>Perfect for troubleshooting and day-to-day Samba admin work!</p>




<p><strong>Need Linux expertise?</strong> I help businesses streamline servers, secure infrastructure, and automate workflows. Whether you're troubleshooting, optimizing, or building from scratch—I've got you covered.<br><br>
📬 Drop a comment or <a href="//mailto:info@sebostechnology.com">email me</a> to collaborate. For more tutorials, tools, and insights, visit <a href="https://sebostechnology.com" rel="noopener noreferrer">sebostechnology.com</a>.</p>

<p>☕ Did you find this article helpful?<br>
Consider supporting more content like this by buying me a coffee:<br>
Buy Me A Coffee<br>
Your support helps me write more Linux tips, tutorials, and deep dives.</p>

