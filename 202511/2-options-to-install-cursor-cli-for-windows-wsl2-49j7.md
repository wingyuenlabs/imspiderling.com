---
Title: 2 options to install Cursor CLI for Windows (WSL2)
Description: 
Author: Henrique Holtz
Date: 2025-11-26T21:31:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hi everyone! In this article I wanna show you 2 options to install and run the <a href="https://cursor.com/cli" rel="noopener noreferrer">cursor CLI</a> on a Windows O.S (both require WSL2 + linux distro like ubuntu) for .NET developers (but you can use it for other stacks as well, just adapt it):</p>

<ol>
<li><p>Everything including git and repositories within the linux distro (better performance but greater overhead to setup)</p></li>
<li><p>Just the Cursor CLI within the linux distro (easier, performance overhead just for the Cursor CLI)</p></li>
</ol>

<p>Here you'll also learn how to setup the terminal within the Visual Studio 2022+ to use the Cursor CLI (or others tools). </p>

<p>We won't focus on details like how to setup WSL2 + linux distro, neither install the git, you can google it. I know you can.</p>

<p>Note: I'll consider you have the WSL2 already set up!</p>

<h2>
  
  
  Initial Setup
</h2>

<p>Once you have the linux distro running through your WSL2 (I'd suggest you to install the distro via Microsoft Store), you can open a terminal within the linux distro by using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Find the distro name
wsl --list

# Run into linux via WSL2
C:\Windows\System32\wsl.exe -d {distro_name}
</code></pre>

</div>



<p>The following steps are within the linux terminal:</p>

<ol>
<li>Install <a href="https://cursor.com/cli" rel="noopener noreferrer">cursor-cli</a> and run <code>cursor-agent</code> (it can require to restart the linux or reopen the terminal). It will prompt you to log in using your browser, so, super easy.</li>
<li>Setup the linux terminal within your Visual Studio (you can use it from Windows Terminal as well):

<ul>
<li>Name: Whatever you want to</li>
<li>Shell location: <code>C:\WINDOWS\system32\wsl.exe</code>
</li>
<li>Arguments: <code>-d {your_distro_name}</code> (you can find out it by running <code>wsl --list</code>)</li>
</ul>
</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6x8noukt3b34lpjpcjlg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6x8noukt3b34lpjpcjlg.png" alt="VS 2022 terminal config for bash on WSl2" width="365" height="403"></a></p>




<p>Here you're ready to use the Cursor CLI, the following steps are if you wanna run everything from WSL2 (better performance for Cursor CLI)</p>

<ol>
<li>Install and setup the git (+ gitlab/github if needed) into the linux</li>
<li>Install .NET SDK <a href="https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet10&amp;pivots=os-linux-ubuntu-2404" rel="noopener noreferrer">steps for .NET 10</a>
</li>
<li>Clone the desired repository and open its solution file with you Visual Studio via WSL2 (click to "open a solution" and find the solution file within the path <code>\\wsl$</code>)</li>
<li>Open the integrated terminal we setup before and run <code>cursor-agent</code>
</li>
</ol>




<p>Done! You can now choose between these 2 options to run the Cursor CLI on your Windows machine + WSL2!</p>

<p>Any questions/suggestions? Let a comment below. Bye!</p>

