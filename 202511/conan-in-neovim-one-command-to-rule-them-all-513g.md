---
Title: Conan in Neovim: One command to rule them all
Description: 
Author: Igor
Date: 2025-11-30T22:07:43.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What is this about?
</h2>

<p>Hello! I'm Igor — <strong>a Ukrainian developer who created a (maybe more-or-less famous) plugin for Neovim</strong>.<br><br>
It installs libraries for Python (<code>pip</code>), JavaScript (<code>npm</code>), Lua (<code>luarocks</code>), Rust (<code>cargo</code>), and more.<br><br>
Recently, I <strong>added Conan support</strong> and would love some feedback or advice!</p>


<h2>
  
  
  Example command
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight viml"><code><span class="p">:</span>LazyDevInstall fmt glfw glm
</code></pre>

</div>


<p><strong>Resulting directories</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>build_fmt/
├── fmt-config.cmake
├── fmt-release-x86_64-data.cmake
├── other files...
build_glfw/
├── glfw-config.cmake
├── glfw-release-x86_64-data.cmake
├── ...
build_glm/
├── glm-config.cmake
├── glm-release-x86_64-data.cmake
├── ...
</code></pre>

</div>






<h2>
  
  
  Why I created it
</h2>

<p>Well, that`s simple question - all started from Pip3, then i created for Luarocks, then Rust,  and then npm, and etc<br>
All just for convenience in development, because i had insomnia at 2:40 AM, and i literally has though like "what if i do plugin for neovim, just because.. why not" - i wrote this on Notes, and lay down to sleep - on next day - started work on this plugin<br>
And all I can say - thats hard, but interesting</p>




<h2>
  
  
  Why last update (exclude this two on vim.org) was too old
</h2>

<p>If you will watch to vim.org stat - ATTENTION - white theme:<br>
(I also have this plugin in <a href="//github.com/Silletr/LazyDeveloperHelper">Github</a> and thats my source of updates) <br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdrg75dotxiy1iclwfvh2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdrg75dotxiy1iclwfvh2.png" alt=" " width="800" height="272"></a></p>

<p><em>You will see - 2025-11-12 -&gt; 2025-11-28</em><br>
<strong>And i can explain this - i needed time to develop this all, test, and etc. + i completely working solo so that also taking time</strong></p>




<h2>
  
  
  So, what can i say
</h2>

<p>Well, i can say only one thing - try my plugin, maybe you will love it!<br>
<strong>Thanks for attention, have a good day</strong>!</p>

