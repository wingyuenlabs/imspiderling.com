---
Title: Wasma indepent-full WM,compositor,UI-fronted
Description: 
Author: Zaman Hüseyinli
Date: 2026-01-17T21:27:46.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbuuve9moyoywz2d347am.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbuuve9moyoywz2d347am.png" alt=" " width="800" height="800"></a></p>

<h1>
  
  
  WASMA
</h1>

<p>WASMA, or Windows Assignment System Monitoring Architecture, is a WM Compositor UI-fronted layer system currently based on ICED. It is completely independent, written in Rust, and offers its own environment. It is currently under development, but its features are as follows:</p>

<ul>
<li>It supports manually managing resource usage in windows.</li>
<li>You can share windows over any network; no additional software like a remote viewer is needed. You can start windows via GRPC, HTTP, HTTPS, or Tor. You can use the multi-instances option to start them all at once; if you want to start them individually, you need to use singularity instances. You can open, start, and manage multiple windows over a single protocol.</li>
<li>Optional application permission management support; you can directly define and restrict the permissions of any application, but this is optional. Instead of .desktop, it supports the more modern and innovative .manifest format, allowing you to define and manage the window management features I mentioned through the manifest and application. However, using the source* method for permissions is recommended.</li>
<li>And finally, with wasma-ubin, you can now directly add GTK/QT/ICED platformer features from Windows/macos/Linux via the binary, adding missing GTK/QT/ICED features.</li>
<li>Wasma-ubin offers a unified binary interface and directly adds, modifies, and enforces all features via the binary through a single ABI, thus providing support for translating features from other platforms to other platforms.</li>
<li>Full translation integration support is offered for x11 Wayland applications.</li>
<li>Legacy platform or Windows-only support is offered separately, but for full support, it is recommended to use the WGClient WASMA ground client.</li>
<li>It has a CPU-powered rendering system; GPU is optional. You can directly adjust resource management.
For more information, visit <a href="https://github.com/Azencorporation/Wasma" rel="noopener noreferrer">Wasma</a>
</li>
</ul>

