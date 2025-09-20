---
Title: Raising PR to Another Repository
Description: 
Author: Dharam Ghevariya
Date: 2025-09-20T21:41:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>For lab-02, we were asked to implement a feature for another classmate's repository. I decided to contribute to the <a href="https://github.com/CynthiaFotso/Repository-Context-Packager" rel="noopener noreferrer">Repository-Context-Packager</a>, which is a command-line tool that analyzes repositories and generates a single file containing repository context. This project was written in JavaScript using Node.js.</p>

<h2>
  
  
  Getting Started: Fork, Setup, and Initial Issues
</h2>

<p>To contribute to this repository, I had to fork it to my GitHub account since I wasn't authorized to push directly to the original repo. During the initial setup, something immediately caught my attention: the <code>node_modules</code> directory was being tracked in Git, even though it should have been ignored through the <code>.gitignore</code> file.</p>

<p>This led me to file my first issue - <a href="https://github.com/CynthiaFotso/Repository-Context-Packager/issues/9" rel="noopener noreferrer">Issue #9</a> regarding this caching problem. The author had <code>node_modules</code> listed in <code>.gitignore</code>, but it wasn't working because the files were already being tracked by Git before being added to the ignore file. I created a local branch for this issue, solved it by using <code>git rm --cached</code> to untrack the files, and raised <a href="https://github.com/CynthiaFotso/Repository-Context-Packager/pull/10" rel="noopener noreferrer">PR #10</a>.</p>

<h2>
  
  
  Feature Development: Implementing the Recent Changes Filter
</h2>

<p>After resolving the initial issue, I moved on to implementing the main feature: a <code>--recent</code> flag that filters and packages only files modified within a specified number of days based on Git commit history.</p>

<h3>
  
  
  Implementation Details
</h3>

<p>The feature development involved several technical challenges and decisions:</p>

<p><strong>Git Integration:</strong><br>
Initially, I considered using filesystem timestamps (<code>fs.stat().mtime</code>) to determine file modification dates. However, I quickly realized this approach had significant limitations. Accessing file timestamps are very tricky thing to do as it changes according to the operating systems. Moreover, the tool was specifically developed for the git repositories so it made me opt for a Git-based approach using the <code>simple-git</code> library:</p>

<h2>
  
  
  Code Review Experience
</h2>

<p>I also received a similar PR on <a href="https://github.com/dharamghevariya/repo-contextr/pull/2" rel="noopener noreferrer">my repository</a> - it was the same feature I had implemented for the original author's repository, the difference was just in the language used.</p>

<p>For the review process, I added the contributor's Git repository to my local remotes, then created a local branch pointing to their PR branch. This way I was able to test the feature locally. I found two issues related to code structure and created review threads for them. Once the contributor updated the code with the desired changes, I closed the threads and merged the PR.</p>

<p>During the review, it was important for me to keep the code style consistent as I had been following it, so I created threads to maintain that consistency.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Throughout this whole process, I got to know that when contributing to others' repositories, you have to understand their style so you can follow it to contribute effectively. When you start working on a feature, you may find other bugs to file and solve if possible. In the open source world, this is the best thing to receive and give.</p>

<p>While doing code review, it was very important for me to keep the code style consistent as I had been following it, so I had to create threads for that. This experience taught me that open source development is not just about writing code - it's about understanding project conventions, maintaining quality standards, and building relationships within the community.</p>

<p>The collaborative nature of open source means every contribution, whether fixing bugs or implementing features, helps improve the project for everyone. This cycle of giving and receiving makes the entire ecosystem stronger and more reliable.</p>

