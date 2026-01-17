---
Title: GIT BASICS: UNDERSTAND VERSIN CONTROL, TRACK CHANGES, PUSH AND PULL CODES FOR BEGINNERS
Description: 
Author: miranda-1-gatwiri
Date: 2026-01-17T21:30:22.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  GIT BASH
</h1>

<p>Git simplifies version control by letting you save snapshots of your code, collaborate safely, and revert mistakes easily.</p>

<h2>
  
  
  What is version control?
</h2>

<p>Version control is a system that helps you;</p>

<ul>
<li><p>Track changes made to files over time.</p></li>
<li><p>Go back to previous versions if something breaks. </p></li>
<li><p>Collaborate with others without conflicts.</p></li>
</ul>

<h3>
  
  
  What are Git and GitHub?
</h3>

<ul>
<li><p>Git is a tool installed on your computer that tracks changes in your project</p></li>
<li>
<p>GitHub is an online platform where you store Git repositories and collaborate with others.</p>
<h4>
  
  
  Key terms to know:
</h4>
</li>
<li><p>A repository is a folder where Git tracks your project's history.</p></li>
<li><p>Working directory this is where you edit your files </p></li>
<li><p>The staging area is where you prepare file before you permanently save it</p></li>
<li><p>A commit is a snapshot of your project at a specific point in time</p></li>
</ul>

<h3>
  
  
  Setting Up Git
</h3>

<p>first confirm the version of the installed Git Bash.<br>
<em><strong>git --version</strong></em></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5uamx9dq1z4hypptfpkt.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5uamx9dq1z4hypptfpkt.jpeg" alt=" " width="800" height="283"></a></p>

<p>Then configure with your name and email<br>
<em><strong>git config --global user.name "name"<br>
git config --global user.email "email"</strong></em></p>

<h4>
  
  
  Then generate an SSH key
</h4>

<p>An SSH key is a secure acces credential<br>
How To Generate an SSH Key<br>
<em><strong>ssh-keygen -t ed25519 -C "email"</strong></em><br>
ssh-keygen - an instruction of what we need to generate<br>
-t - a type of key<br>
ed225519 is a secure encryption method</p>

<ul>
<li>C - a comment
ssh-keygen -t ed25519 -C "email"
It will generate two keys, which are private and public keys. We will use the public key to link the Git Bash with GitHub.</li>
</ul>

<h4>
  
  
  Create an ssh agent in Git Bash.
</h4>

<p>The agent is the holder of the key<br>
<strong><em>eval "$(ssh-agent -s)"</em></strong><br>
eval —runs those commands in the Git Bash<br>
ssh-agent—a program that mananges ssh keys <br>
-s is a command for the shell</p>

<h4>
  
  
  Add ssh agent to the Git Bash
</h4>

<p><strong><em>ssh-add ~/.ssh/id_ed25519</em></strong><br>
ssh-add - a command of adding a key to the the ssh agent<br>
~/.ssh/id_ed25519 - path to your private ssh key</p>

<h4>
  
  
  Copy your public ssh key
</h4>

<p><strong><em>cat ~/.ssh/id_ed25519.pub</em></strong><br>
cat - displays file content<br>
.pub - public key file</p>

<h4>
  
  
  Add ssh key to GitHub.
</h4>

<ol>
<li>Go to GitHub</li>
<li>Click your profile picture → Settings</li>
<li>Click SSH and GPG keys</li>
<li>Click New SSH key</li>
<li>Title: any name</li>
<li>Paste the copied key</li>
<li>Click Add SSH key</li>
</ol>

<h2>
  
  
  Tracking your changes
</h2>

<p>Git tracks changes in three stages: working directory, staging area, and commit history.</p>

<h3>
  
  
  Working directory
</h3>

<p>It holds all project files as they appear on your computer. <br>
It edits the files</p>

<h4>
  
  
  STEPS
</h4>

<ol>
<li>Create a project folder
<strong><em>mkdir my-first-repo</em></strong> - Create a new folder</li>
<li>Move to the folder
<strong><em>cd my-first-repo</em></strong> - Enters the folder</li>
</ol>

<p>3.Initialise Git<br>
<em><strong>git init</strong></em><br>
Turns this folder into a Git repository<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmm4omsgpyto7ev20e0ae.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmm4omsgpyto7ev20e0ae.jpeg" alt=" " width="800" height="312"></a></p>

<ol>
<li>Add files
<em><strong>touch README.md</strong></em> - creates a file.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1m7g6ak4euid0dnd7vlt.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1m7g6ak4euid0dnd7vlt.jpeg" alt=" " width="800" height="330"></a></p>

<h3>
  
  
  Staging
</h3>

<p>When you make changes to a file, Git is aware of it, although it does not save it. The files you want to include in your next snapshot have to be staged.<br>
<em><strong>git add README.md</strong></em></p>

<h3>
  
  
  Commit
</h3>

<p>After staging your files, you close them in with a message on what you have done.<br>
Saves a snapshot of your project.<br>
 <em><strong>git commit -m " Add README.md file"</strong></em></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fav99v9b701sbu9tjd4bi.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fav99v9b701sbu9tjd4bi.jpeg" alt=" " width="800" height="586"></a><br>
Add the GitHub repository as a remote:<br>
<em><strong>git remote add origin<br>
<a href="mailto:git@github.com">git@github.com</a>:username/repository-name.git</strong></em><br>
Then verify:<br>
<em><strong>git remote -v</strong></em></p>

<h2>
  
  
  How to Push Code to GitHub.
</h2>

<p>Pushing is the act of sending your local commits to the remote server.<br>
<em><strong>git push -u origin main</strong></em><br>
origin - GitHub repository<br>
main - branch name</p>

<h2>
  
  
  How to Pull Code from GitHub
</h2>

<p>Pulling fetches and merges remote changes from Github.<br>
<em><strong>git pull origin main</strong></em><br><br>
Use this when:</p>

<ul>
<li>Working on multiple choices.</li>
<li>Collaborating with a team </li>
<li>Updating your local project</li>
</ul>

<h3>
  
  
  Git workflow
</h3>

<ol>
<li>Edit files</li>
<li>git add → Stage changes</li>
<li>git commit → Save changes</li>
<li>git push → Upload to GitHub</li>
<li>git pull → Download updates</li>
</ol>

<h2>
  
  
  Conlusion.
</h2>

<p>Git may seem to be a daunting tool at its initial introduction, yet it is the greatest thing in the toolkit of a developer. You have already been taught 80 percent of the game by learning add, commit, push and pull, and can now apply it on the daily basis.</p>

