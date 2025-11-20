---
Title: From 20 to 24 LTS: Safe Way to Upgrade Ubuntu on DigitalOcean
Description: 
Author: Ivan Mykhavko
Date: 2025-11-20T21:36:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Upgrading an old production server is not a popular topic today. Most run Docker, Kubernetes, or at least keep OS fresh. But sometimes you inherit a legacy VM, full of configs, and you just need to push it to a modern LTS without breaking anything.</p>

<p>I had exactly this case on DigitalOcean. Here's a quick, real-world rundown of what I actually did.</p>

<h2>
  
  
  The Problem
</h2>

<p>The project was running on Ubuntu 20.04 with an outdated Nginx. Direct upgrade to 24.04 LTS is not possible. I needed a way to bring everything up-to-date quickly and safely and keep downtime to minimum.</p>

<h2>
  
  
  Why It Matters
</h2>

<p>Those kinetic and lunar are eol versions. They no longer receive security updates. Running outdated versions exposes your server to vulnerabilities and leaves you without support. The jump to 24.04 LTS is significant - you get five years of support, better performance, and peace of mind.</p>

<h2>
  
  
  How I Did It
</h2>

<p>To avoid downtime, I leaned on DigitalOcean snapshots + temp droplet. Then I upgraded the system in steps:</p>

<p>20.04 → 22.10 → 23.04 → 23.10 → 24.04 lts + Nginx upgrade</p>

<p>It's a bit long, but it gets the job done if you're stuck on an old upgrade path.</p>

<h2>
  
  
  1. Prep the Server (No Downtime Yet)
</h2>

<p>First things first:</p>

<ul>
<li>Take a snapshot of your existing droplet.</li>
<li>Use that snapshot to create a temporary droplet.</li>
<li>Set up Floating IP so you can point your domain to this temp droplet when it's ready.</li>
</ul>

<p>This gives you a safe rollback if something goes wrong.</p>

<h2>
  
  
  2. Do the Upgrades
</h2>

<h3>
  
  
  Upgrade to Ubuntu 22.10
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s|http://mirrors.digitalocean.com/ubuntu/|http://old-releases.ubuntu.com/ubuntu/|g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s|http://security.ubuntu.com/ubuntu|http://old-releases.ubuntu.com/ubuntu|g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt dist-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt autoremove <span class="nt">--purge</span> <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt <span class="nb">install</span> <span class="nt">-y</span> ubuntu-release-upgrader-core <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s/kinetic/lunar/g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt dist-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt full-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt autoremove <span class="nt">--purge</span> <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">Upgrade complete! Rebooting now...</span><span class="se">\n</span><span class="s2">"</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>reboot
</code></pre>

</div>



<h3>
  
  
  Upgrade 23.04 → 23.10
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s|http://old-releases.ubuntu.com/ubuntu|http://old-releases.ubuntu.com/ubuntu|g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s/lunar/mantic/g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt dist-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt full-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt autoremove <span class="nt">--purge</span> <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">Upgrade complete! Rebooting now...</span><span class="se">\n</span><span class="s2">"</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>reboot
</code></pre>

</div>



<h3>
  
  
  Upgrade 23.10 → 24.04 LTS
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s|http://old-releases.ubuntu.com/ubuntu|http://archive.ubuntu.com/ubuntu|g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo sed</span> <span class="nt">-i</span> <span class="s1">'s/mantic/noble/g'</span> /etc/apt/sources.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt dist-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt full-upgrade <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt autoremove <span class="nt">--purge</span> <span class="nt">-y</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">Upgrade complete! Rebooting now...</span><span class="se">\n</span><span class="s2">"</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>reboot
</code></pre>

</div>



<p>Now you're on <strong>Ubuntu 24.04 LTS</strong>. Double-check with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lsb_release <span class="nt">-a</span>
</code></pre>

</div>



<h2>
  
  
  3. Upgrade to the Latest Nginx
</h2>

<p>Don't skip this backup step:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cp</span> <span class="nt">-r</span> /etc/nginx /root/nginx-backup
</code></pre>

</div>



<p>Then install the latest Nginx package:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://nginx.org/keys/nginx_signing.key | <span class="nb">sudo </span>gpg <span class="nt">--dearmor</span> <span class="nt">-o</span> /usr/share/keyrings/nginx-archive-keyring.gpg
<span class="nb">echo</span> <span class="s2">"deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/ubuntu noble nginx"</span> | <span class="nb">sudo tee</span> /etc/apt/sources.list.d/nginx.list
<span class="nb">sudo </span>apt update
<span class="nb">sudo </span>apt <span class="nb">install </span>nginx
nginx <span class="nt">-v</span>
</code></pre>

</div>



<p>You should see version ^1.28.0. Reload your configuration to make sure it still parses:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nginx <span class="nt">-t</span>
<span class="nb">sudo </span>systemctl restart nginx
</code></pre>

</div>



<h2>
  
  
  4. Point the Floating IP Back
</h2>

<p>Wait a few minutes. Check your app. Run your health checks, monitoring, whatever you've got. If you have automated tests, don't forget to run them.<br>
Look at logs and metrics make sure it's all good. Everything working? Cool. Move to the next.</p>

<p>Once you're sure everything's stable, switch the Floating IP back from the temp droplet to your freshly upgraded one.</p>

<p>Hang on to the temp droplet and snapshot for few days. If nothing catches fire, you can remove them.</p>

<h2>
  
  
  Timeline
</h2>

<p>Total time: about one hour. Most of that is just waiting for reboots and packages to download. You can grab coffee and check on progress every few minutes.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>This isn't a workflow I want to repeat often. If you can move to Docker or rebuild your server from scratch — do it.</p>

<p>But sometimes legacy systems don't give you that choice.<br>
And when you must upgrade an old Ubuntu machine step by step, this method works. Fast, safe, and almost zero downtime.</p>

<h2>
  
  
  Author's Note
</h2>

<p>Thanks for sticking around!<br>
Find me on <a href="https://dev.to/tegos">dev.to</a>, <a href="https://www.linkedin.com/in/ivan-mykhavko/" rel="noopener noreferrer">linkedin</a>, or you can check out my work on <a href="https://github.com/tegos" rel="noopener noreferrer">github</a>.</p>

<p><strong>Notes from real-world Laravel.</strong></p>

