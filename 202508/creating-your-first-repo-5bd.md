---
Title: Creating Your First Repo
Description: 
Author: Sarah Bartley-Dye
Date: 2025-08-25T20:15:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>Today, you will create your first repository in Git. In this lesson from Skillcrush 105, students learn what a repo is, why developers like to use them, and how to create one. Skillcrush students create their first repo and use some new Git commands to look at the history inside a repo.</p>

<p>As you continue working in Git, you’ll often hear developers talk about creating a repo. Git is a desirable skill because it is great for tracking file changes. Git enables easier access to past versions and maintains the file's history. </p>

<h3>
  
  
  What is a repo?
</h3>

<p>Repo is short for a repository. When developers create a repository, it means that the directory and files are being put into version control or are getting ready to track changes. </p>

<p>Creating a repo is similar to how a car goes through a car wash. When a car is at a car wash, it moves through different steps. This is similar to how repositories are created because you will need to do specific steps to create one and track the changes you make. </p>

<h4>
  
  
  1. Check your directory
</h4>

<p>Let’s start at the beginning and move through the process. Before you can create a repo, always double-check what directory you are in. You don’t want to continue this process in the wrong directory. So save your future self a headache and double-check where you are.</p>

<p>Once you are in the right directory, you can move on to the next step. However, if you are in the wrong directory, you will need to type cd followed by the directory name into the command line. This is short for change directory, and it will move you from directory to directory.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd coding
</code></pre>

</div>



<p>If you need to go back to the previous directory, you will still need the cd command. Instead of putting a directory name, you will need to put two dots. When you press Enter, it will take you back to the previous directory.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd ..
</code></pre>

</div>



<p>Not sure what file path to use? Skillcrush suggests a way to navigate to your directory. PC users can find a directory using File Explorer. command. PC Users need to right-click on the directory and pick “Git Bash Here”.</p>

<p>Got a Mac? Mac users will go to their terminal app and type the cd command and a space. They will need to drag the directory icon from their Finder to the terminal window before they release it. When they press Enter, it will move into the directory. Mac users can also right-click or control-click on a directory, then pick “New Terminal at Folder”.</p>

<h4>
  
  
  2. Create a Repo
</h4>

<p>You can now create a repo. If you aren’t sure if you made this a repo already, you can use this step to double-check. To create a new repo, you will type the command git init into the command line.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git init
</code></pre>

</div>



<p>When you press Enter, the computer will initialize your repo or set it up. If this repo has already been made, the computer will display a message letting you know it is already a repository. </p>

<p>Developers like to double-check any kind of setup when they code, and you can do this in Git. To check that the repo has been set up, type the command git status into the command line. If anything was incorrectly set up, you will see it here.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git status
</code></pre>

</div>



<p>Everything will be set up when you see a list of commits that need to be made for the repo. A commit is any changes that haven’t been saved to the repo. Git is helping you out by letting you know that there are no versions or there are untracked changes in a repo. You will know there are untracked changes because the file names will be in red.</p>

<h4>
  
  
  3. Tracking Files in a Repo 
</h4>

<p>When you create your first repo, you will have no commits. So let’s make one. The next step in this process is to start tracking the file.</p>

<p>To track a file, you will need to use the command git add. To use this command, you will need to type git add [file name] into the command line. When you press Enter, the computer will start tracking that file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git add index.html
</code></pre>

</div>



<p>Not sure if a file is being tracked? Type git status into the command line. Any files that are being tracked will be in green now. </p>

<p>That means it is put into the staging area and is ready for the next step. Do you need to track all the files in that repo? If there are a lot of files you need to track, you can use git add with the -A instead of the file name. The computer will then track all the files in that repo.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git add -A
</code></pre>

</div>



<h4>
  
  
  4. Commit your changes
</h4>

<p>You have tracked all your changes. But now there’s one more step to go. You need to commit your changes. </p>

<p>This will make sure the changes are recorded in the repo. Commits are unique from other git commands because you will need to include a message when you commit your files to the staging area. Skillcrush wants students to think of these messages as little community notes that describe the changes that were made to a repo. </p>

<p>Commit messages are important to have for you and any team members working on that repo. These messages let everyone see what changes have been made in each version of a project. To commit changes in Git, you will use the git commit command.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git commit -m “initial commit” 
</code></pre>

</div>



<p>After you type git commit, you need to type -m, then a message in quotes. If you don’t include a commit message, Git will give you an error message and won’t commit your files. When you press Enter, your changes will be committed to the repo. </p>

<p>You can use the git status command to check if everything went through. If it wasn’t successful, read the error message and make changes before you try again. Otherwise, you will get a message from the computer saying “nothing to commit” in the command line.</p>

<h3>
  
  
  How to write a good commit message
</h3>

<p>Writing a commit message is something I continually work on improving. Skillcrush helps students by sharing some tips to help them write a good commit message.</p>

<ul>
<li>Describe changes and additions to the code</li>
<li>Use clear, concise language</li>
<li>Capitalize the first word of a message</li>
<li>Do not end the commit message with a period</li>
<li>Keep your message under 50 characters.</li>
</ul>

<h3>
  
  
  Troubleshooting FAQs
</h3>

<h4>
  
  
  I got a “fatal error” in the command line. What should I do?
</h4>

<p>The “fatal error” message means that your request wasn’t completed. Git includes extra information about the error. Whenever you get an error in code, you always want to read the entire message because it will tell you what type of error you have. </p>

<p>For example, an error message might be written as “not a git repository”. That is the computer’s way of telling you that this directory isn’t a git repo yet. You will need to use the git init command or the cd command to change to the right directory.</p>

<h4>
  
  
  How can I access my commit history?
</h4>

<p>To see a history of your commits, use the command git log on the command line. When you press Enter, you will see all the commits that have been made and specific details about each one, such as:</p>

<ul>
<li>author</li>
<li>when they were made</li>
<li>the message</li>
</ul>

<p>Your first repo might not have many commits, but as more commits are made, git log will come in handy to see what changes have been made in case you need to revisit a previous version.</p>

<h4>
  
  
  I didn’t write a commit message, and now I don’t know where I am in the command line. Help!
</h4>

<p>When you don’t write a commit message or forget the quotation marks, you will be taken to VIM. VIM is a text editor in the command line. If you find yourself there, you can exit by typing in a specific command depending on what type of computer you are using. </p>

<p>Make sure you use the right command for the right computer, so double-check if you are using the right 3 characters. PC users will type the command :ql into VIM.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>:q!
</code></pre>

</div>



<p>Mac users will need to use the :wq command.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>:wq
</code></pre>

</div>



<h4>
  
  
  A Quick Note on OneDrive (PC Users Note)
</h4>

<p>If you are using Windows, you might see a OneDrive icon on your computer. OneDrive is Microsoft’s cloud storage. OneDrive is very similar to Dropbox or Google Drive.</p>

<p>Windows will upload files and folders to OneDrive automatically. That means the local version and the OneDrive version are connected or synced. If these files aren’t synced, your code will be different depending on which one you are working in. </p>

<p>You can work in either the local or OneDrive version. You need to remember that they are separate directories, so they are completely different. So you will want to make sure you are in the right directory before you start working and, most importantly, committing any files.</p>

<h3>
  
  
  Conclusion
</h3>

<p>Congratulations! You have made your first repository. Now that you know how to create repos and have more git commands, you know how to use. </p>

<p>You are now ready for GitHub. The next lesson in Skillcrush 105 is when Skillcrush introduces students to GitHub. This is where students learn how GitHub works, how it works with Git, and how to create a GitHub account.</p>

