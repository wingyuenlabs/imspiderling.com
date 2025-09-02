---
Title: Learn Bash Scripting With Me 🚀 - Day 1
Description: 
Author: Babs
Date: 2025-09-02T21:59:36.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  LearnBashScriptingWithMe 🚀
</h2>

<h2>
  
  
  Day 1 – Introduction, Shebang &amp; File Permissions
</h2>

<p><strong>Bash scripting is the process of creating a file that contains a sequence of commands and then running that file as a script.</strong></p>

<p>A shell in Kali Linux is a command-line interpreter that provides an interface for the user to interact with the operating system. It allows users to execute commands, scripts, and programs to manage the system, navigate the file system, and perform various administrative tasks.</p>

<p>👉 We start by creating a file using the “sh” extension as this is used to denote that we are creating a shell script, e.g test.sh</p>

<p>👉 After that the next thing we need to do is to define the kind of interpreter we are going to be using and for this, the interpreter is BASH and we denote that using<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>#!/bin/bash
</code></pre>

</div>



<p>👉 Inside the script, we can use the echo command to display output. For example, printing “Hello World!”<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>echo "Hello World!"
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F98ewoghtt5q1lqfx96ud.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F98ewoghtt5q1lqfx96ud.png" alt=" " width="800" height="210"></a></p>

<p>👉 Now we can now save and run the script by using the command “./(filename)” <br>
e.g:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>./test.sh
</code></pre>

</div>



<p>Note that the command <strong>"./"</strong> is telling the shell to use the interpreter defined in the file we want to run which was what was defined first when we wrote <strong>"#!/bin/bash"</strong> in the file</p>

<p>But notice—you might get a “permission denied” error. That’s because the file isn’t yet executable.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0enxetqxxjdrvqwnfc2b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0enxetqxxjdrvqwnfc2b.png" alt=" " width="705" height="178"></a></p>

<p>👉 To fix this, grant execution permission with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>chmod +x test.sh
</code></pre>

</div>



<p>Now, when you run ./test.sh, the script executes successfully 🎉</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdvg1524188ni2s1vyz1j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdvg1524188ni2s1vyz1j.png" alt=" " width="800" height="255"></a></p>

<h1>
  
  
  Bash #scripting
</h1>

