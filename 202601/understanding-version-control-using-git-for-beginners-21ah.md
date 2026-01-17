---
Title: UNDERSTANDING VERSION CONTROL USING GIT : FOR BEGINNERS
Description: 
Author: Otanga Joy Monica 
Date: 2026-01-17T21:39:04.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>1. WHAT IS VERSION CONTROL?</strong><br>
Version control is a way of keeping track of changes made to files over time. Instead of saving many copies of the same file with different names, version control helps you record each change in an organized way. This makes it easy to see what was changed, when it was changed, and to go back to an earlier version if something goes wrong. It is especially useful when working on projects that grow over time or when working with other people on the same code.</p>

<p><strong>2. WHAT IS GIT?</strong><br>
Git is a version control system that helps developers manage and track changes in their projects.It works locally, saves snapshots of one's project and makes it easy to fix mistakes and review their work.<br>
<strong>3. WHAT IS GITHUB?</strong><br>
GitHub is a website where you can store your Git projects online. It keeps your code safe, lets you share it with others, and makes it easy to work on the same project with group members. You can also access your projects from any computer, so you always have the latest version.</p>

<p><strong>4. IMPORTANCE OF VERSION CONTROL</strong><br>
Version control is important as it helps you to:</p>

<ul>
<li><p>Avoid losing your work</p></li>
<li><p>Undo mistakes by going back to previous versions</p></li>
<li><p>Track your project’s progress over time</p></li>
<li><p>Work on the same project from different devices</p></li>
<li><p>Collaborate with others without overwriting their changes</p></li>
</ul>

<p><strong>5.How to Track Changes, Push code and Pull code on Github</strong></p>

<p><strong>STEP 1: INSTALL GIT</strong></p>

<ul>
<li>Go to <a href="https://git-scm.com/install/" rel="noopener noreferrer">Git’s</a> official website and download the version suitable for your operating system.</li>
<li>Run the installer and follow the steps, leave most settings as default</li>
<li>After installation, open Git Bash or the command prompt and type:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git --version
</code></pre>

</div>



<p>This shows the version of git installed</p>

<p><strong>STEP 2: CREATE A GITHUB ACCOUNT</strong><br>
Register or signup for a <a href="//github.com">github</a> account<br>
Once that’s done, open your terminal (Git Bash) and set up your username and email, to allow Git to know who made changes to your files.</p>

<ul>
<li>Use the following commands(replace "YOUR_USERNAME" and "YOUR_EMAIL" with your details.):
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git config --global user.name "YOUR_USERNAME"
git config --global user.email "YOUR_EMAIL"
</code></pre>

</div>



<p><strong>STEP 3: CREATE A FOLDER FOR YOUR PROJECT</strong><br>
First, create a folder on your computer for your project. You can name it anything e.gtestProject.</p>

<ul>
<li>Then, open Git Bash and go into that folder using the cd (change directory) command:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd testProject
</code></pre>

</div>



<p>This makes Git work inside that folder</p>

<p><strong>STEP 4: INITIALIZE GIT</strong></p>

<ul>
<li>Initialize Git in your folder using:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git init
</code></pre>

</div>



<p>This command makes your folder a Git project so Git can start keeping track of your files</p>

<p><strong>STEP 5: ADD A NEW FILE</strong></p>

<ul>
<li>To add a new file to your project, use the touch command:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>touch firstfile.txt
</code></pre>

</div>



<p>This command creates a blank file called firstfile.txt in your folder.</p>

<ul>
<li>After creating the file, check which files Git is tracking using the following command:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git status
</code></pre>

</div>



<p>Git will show the new file in red, which means it exists but isn’t tracked yet.<br>
Green files mean Git is watching them and will save changes when you commit.</p>

<p><strong>STEP 6: TRACK AND SAVE CHANGES</strong></p>

<ul>
<li>To start tracking your file, stage it using:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git add firstfile.txt
</code></pre>

</div>



<ul>
<li>
<strong>Note:</strong>To add all files at once, use:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git add .
</code></pre>

</div>



<ul>
<li>save your changes with a message using:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git commit -m "Commit Message"
</code></pre>

</div>



<p>Use an appropriate commit message explaining clear what exactly has been done.</p>

<p><strong>STEP 7: CONNECT YOUR LOCAL REPOSITORY TO GITHUB</strong><br>
On GitHub, create a new repository for your project.<br>
Copy the repository URL (it looks like <a href="https://github.com/username/repository-name.git" rel="noopener noreferrer">https://github.com/username/repository-name.git</a>).</p>

<ul>
<li>Then run:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git remote add origin &lt;paste-your-repo-link&gt;
</code></pre>

</div>



<p>Remember to replace with your repository link.<br>
Now your local folder knows where to send files online.</p>

<p><strong>STEP 8: PUSH CODE</strong></p>

<ul>
<li>Send your local files online using the following command:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git push origin main
</code></pre>

</div>



<p><strong>STEP 9: PULL CODE</strong></p>

<ul>
<li>If you want to work on the project on another computer or get the latest version, first clone the repository using:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git clone &lt;repo-link&gt;
cd your-project-folder
</code></pre>

</div>



<p>The command above copies the project from Github to your computer</p>

<ul>
<li>To update your local files with the latest changes from GitHub, use the command:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git pull origin main
</code></pre>

</div>



<p><strong>KEY TAKEAWAY</strong><br>
It is much more important to understand how the git commands work than to master them</p>

