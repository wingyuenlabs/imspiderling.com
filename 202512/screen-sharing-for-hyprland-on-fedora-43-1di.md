---
Title: Screen Sharing for Hyprland on Fedora 43
Description: 
Author: Asher Buk
Date: 2025-12-25T21:02:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've already published a <a href="https://copr.fedorainfracloud.org/coprs/ashbuk/Hyprland-Fedora/" rel="noopener noreferrer">clean Hyprland build for Fedora 43</a> — now I've added <strong>xdg-desktop-portal-hyprland</strong> to the same COPR repository.</p>

<h3>
  
  
  The Solution
</h3>

<p><code>xdg-desktop-portal-hyprland</code> with proper <code>RUNPATH</code> pointing to the vendored libraries.</p>

<h4>
  
  
  Install
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>dnf copr <span class="nb">enable </span>ashbuk/Hyprland-Fedora
<span class="nb">sudo </span>dnf <span class="nb">install </span>hyprland xdg-desktop-portal-hyprland
</code></pre>

</div>



<p>Screen sharing tested in Discord, Google Meet, and other WebRTC apps via Firefox and Chrome.</p>

<h4>
  
  
  Links
</h4>

<ul>
<li>
<strong>COPR repository:</strong> <a href="https://copr.fedorainfracloud.org/coprs/ashbuk/Hyprland-Fedora/" rel="noopener noreferrer">https://copr.fedorainfracloud.org/coprs/ashbuk/Hyprland-Fedora/</a>
</li>
<li>
<strong>Source repository (fork + spec files):</strong> <a href="https://github.com/AshBuk/Hyprland-Fedora" rel="noopener noreferrer">https://github.com/AshBuk/Hyprland-Fedora</a>
</li>
<li>
<strong>Hyprland tech write-up:</strong> <a href="https://ashbuk.hashnode.dev/hyprland-fedora" rel="noopener noreferrer">https://ashbuk.hashnode.dev/hyprland-fedora</a>
</li>
</ul>

<p>70+ users enabled this repository in just a few days — thank you for trusting the build. If you run into any issues, feel free to open an issue on <a href="https://github.com/AshBuk/Hyprland-Fedora" rel="noopener noreferrer">GitHub</a>.</p>

