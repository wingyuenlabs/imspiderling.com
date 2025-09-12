---
Title: Create a sample resume and Push to GitHub Pages.
Description: 
Author: EMMANUEL
Date: 2025-09-12T21:55:21.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Git</strong> <br>
Git is a distributed version control system, which means that a local clone of the project is a complete version control repository. These fully functional local repositories make it easy to work offline or remotely. Developers commit their work locally, and then sync their copy of the repository with the copy on the server. This paradigm differs from centralized version control where clients must synchronize code with a server before creating new versions of code.</p>

<p>Git's flexibility and popularity make it a great choice for any team. Many developers and college graduates already know how to use Git. Git's user community has created resources to train developers and Git's popularity make it easy to get help when needed. Nearly every development environment has Git support and Git command line tools implemented on every major operating system.</p>

<p><strong>Benefits of Git</strong></p>

<p><strong>Simultaneous development</strong>: Everyone has their own local copy of code and can work simultaneously on their own branches. Git works offline since almost every operation is local.</p>

<p><strong>Faster releases</strong>: Branches allow for flexible and simultaneous development. The main branch contains stable, high-quality code from which you release. Feature branches contain work in progress, which are merged into the main branch upon completion. By separating the release branch from development in progress, it's easier to manage stable code and ship updates more quickly.</p>

<p><strong>Built-in integration</strong>: Due to its popularity, Git integrates into most tools and products. Every major IDE has built-in Git support, and many tools support continuous integration, continuous deployment, automated testing, work item tracking, metrics, and reporting feature integration with Git. This integration simplifies the day-to-day workflow.</p>

<p><strong>Strong community support</strong>: Git is open-source and has become the de facto standard for version control. There is no shortage of tools and resources available for teams to leverage. The volume of community support for Git compared to other version control systems makes it easy to get help when needed.</p>

<p><strong>Git works with any team</strong>: Using Git with a source code management tool increases a team's productivity by encouraging collaboration, enforcing policies, automating processes, and improving visibility and traceability of work. The team can settle on individual tools for version control, work item tracking, and continuous integration and deployment. Or, they can choose a solution like GitHub or Azure DevOps that supports all of these tasks in one place.</p>

<p><strong>Pull requests</strong>: Use pull requests to discuss code changes with the team before merging them into the main branch. The discussions in pull requests are invaluable to ensuring code quality and increase knowledge across your team. Platforms like GitHub and Azure DevOps offer a rich pull request experience where developers can browse file changes, leave comments, inspect commits, view builds, and vote to approve the code.</p>

<p><strong>Branch policies</strong>: Teams can configure GitHub and Azure DevOps to enforce consistent workflows and process across the team. They can set up branch policies to ensure that pull requests meet requirements before completion. Branch policies protect important branches by preventing direct pushes, requiring reviewers, and ensuring clean builds.</p>

<p><strong>Github</strong><br>
GitHub is a cloud-based platform for version control and collaboration, primarily used by developers to store, manage, and share code. It is built on Git, an open-source version control system that tracks changes in code and allows multiple people to collaborate on a project efficiently.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0mh79erds63mivg7uv7y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0mh79erds63mivg7uv7y.png" alt="Git/Github" width="800" height="450"></a></p>

<p><strong>Git Bash</strong><br>
<strong>Git Bash</strong> is an application for Microsoft Windows OS environments that provides Unix based shell utilities and experience for Git command line commands. Git Bash emulates the Git command line experience that Unix environments have, for Windows users. Most Windows users download Git Bash when they first install Git for Windows.</p>

<p>As a version control system, Git was originally delivered within Unix style command line methods. MacOS &amp; Linux Operating Systems have a built-in terminal shell that supports Unix-based command line features whereas Microsoft Windows’ Operating System command line prompt is not a Unix-based terminal. Because the Windows command line does not support Unix-based commands, Git CLI features are mostly delivered with user-friendly GUI applications in the Windows Operating System. These applications provide visual functionalities to the end-user which makes using Git easier.  </p>

<p>As Git experience increases, end-users can prefer using Git command line features for specific commands. Git Bash is one tool that provides command line features in the Windows Operating System to end-users.</p>

<p><strong>Essential Git Bash Commands</strong>:</p>

<p>1.<strong>GIT PUSH</strong>: This command is used to upload local repository content to a remote repository. (In Git, a local repository is a version-controlled directory on your local computer where your project files are stored, along with the metadata Git uses to track changes to these files.)</p>

<p>2.<strong>GIT COMMIT -M</strong>: This takes a snapshot of your repository at one point in time. The -m flag helps u add a message to the commit</p>

<p>3.<strong>GIT ADD</strong> As the name implies it adds a change in the working directory to the staging area. It tells git that u want to include updates to a particular file.</p>

<p>4.<strong>GIT REMOTE ADD ORIGIN</strong>: This creates a new empty Git repository on your remote server.</p>

<p><strong>Steps on how to use Git BASH command to create a sample resume and push it to Github pages</strong></p>

<p>1.To install Git for windows, click on this link <a href="https://git-scm.com/downloads/win" rel="noopener noreferrer">https://git-scm.com/downloads/win</a> </p>

<p>2.Click on windows version to download and install Git for windows.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4pc7afjzrm8ud053skc6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4pc7afjzrm8ud053skc6.png" alt="Download git for windows" width="800" height="393"></a></p>

<p>3.Go to download, choose the downloaded git version for windows, click to install ,choose <strong>Next</strong> throughout as default to complete the installation.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz3g4gh97eoq8ikm1kekb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz3g4gh97eoq8ikm1kekb.png" alt="Downloaded gitbash" width="800" height="388"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo60j0l27wkfbiptfwry2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo60j0l27wkfbiptfwry2.png" alt="Next as default" width="800" height="369"></a></p>

<p>4.when the installation is complete, open your gitbash to Configure the Gitbash with an Identity, to do that create a name and email with the following commands:</p>

<p><strong>git config --global user.name "name"</strong> <br>
<strong>git config --global user.email "email"</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr03lhk0ntuopy3t9suvx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr03lhk0ntuopy3t9suvx.png" alt="Gitbash identity" width="800" height="467"></a></p>

<p>5.Use this command <strong>git config --list</strong> to confirm if the configuration is successful. From our gitbash it shows its successful.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fckjjipdn72j5dtwc8gnr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fckjjipdn72j5dtwc8gnr.png" alt="git config --list" width="800" height="419"></a></p>

<p>6.Prepare your work environment to work with it. To do that, create/make a directory.(Its similiar to what Resource group does in microsoft Azure).To make directory, run this command <strong>mkdir emmanuelobinnadirectory</strong> in your gitbash.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsh4we7yltc44hdtwt4jy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsh4we7yltc44hdtwt4jy.png" alt="mkdir emmanuelobinnadirectory" width="800" height="402"></a></p>

<p>7.To see things that is in your computer or things you have created use <strong>ls</strong> command in your gitbash</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpxg50gyafifqxh2u7v1a.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpxg50gyafifqxh2u7v1a.png" alt="ls command" width="800" height="401"></a></p>

<p>8.Enter/open the emmanuelobinnadirectory we  have created, use <strong>cd emmanuelobinnadirectory</strong> command</p>

<p>9.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr9l9l0nl7oe7gionvcqo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr9l9l0nl7oe7gionvcqo.png" alt="Open directory" width="800" height="416"></a></p>

<p>9.Type this command <strong>clear</strong> your gitbash to clear the screen but its important to note that ,it doesnt mean your previous command have been deleted.</p>

<p>9.In your gitbash type this command <strong>git init</strong> to convert your directory into a local git repository.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7g7kk570weugujyq8jyw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7g7kk570weugujyq8jyw.png" alt="Git init" width="800" height="418"></a></p>

<p>10.To confirm that it has been converted. Type this command <strong>ls -al</strong> in your gitbash to list the hidden files,.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flnlja1la3dygehp65bug.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flnlja1la3dygehp65bug.png" alt="ls -al" width="800" height="416"></a></p>

<p>11.In your gitbash, type this command <strong>touch index.html</strong> to Create an empty text file/html file.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd75o7y9xp70v5qaagch0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd75o7y9xp70v5qaagch0.png" alt="touch index.html" width="800" height="416"></a></p>

<p>12.To verify that we have created the touch index.html file use the command <strong>ls</strong> in your gitbash.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6h1u9w474gqkvbxigkrl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6h1u9w474gqkvbxigkrl.png" alt="list" width="800" height="417"></a></p>

<p>13.Go to chatGPT, upload your resume on your chatGPT, or you can ask chatGPT to create an index.html for cloud engineering resume for Emmanuel obinna Onyemuche.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7o76qmkv6c0qjpyhliz1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7o76qmkv6c0qjpyhliz1.png" alt="ChaTGPT generate code" width="800" height="399"></a></p>

<p>14.Go to gitbash, use the command <strong>vi index.html</strong> to open this index.html to start typing.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa4gkpa6zhf8b2u0ltxad.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa4gkpa6zhf8b2u0ltxad.png" alt="vi index.html" width="800" height="421"></a></p>

<p>15.In other to edit vi file ,you have to make it to be on insert mode by typing <strong>i</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqnrj6dlp3l0khji0f9wc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqnrj6dlp3l0khji0f9wc.png" alt="Insert mode" width="800" height="418"></a></p>

<p>16.Go to chatGPT and copy the <strong>code</strong> generated on the chatGPT and paste it, in  the gitbash.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4pbq0ytpmfkob3jbk18p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4pbq0ytpmfkob3jbk18p.png" alt="Code from chatGpt" width="800" height="417"></a></p>

<p>17.Click <strong>escape</strong> key on the keyboard to save the vim document.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F15t19xdgarzi0lvgw10w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F15t19xdgarzi0lvgw10w.png" alt="Escape key" width="800" height="420"></a></p>

<p>18.Press <strong>shift+colon</strong> on your keyboard at the same time, then type <strong>wq</strong> in front of the colon. Hit enter on the keyboard.<br>
w= is used to save everything.<br>
q= is used to escape the page.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwxrp312dx7jklbxio4a5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwxrp312dx7jklbxio4a5.png" alt="wq" width="800" height="418"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe9chxlvs0z1bej30wioz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe9chxlvs0z1bej30wioz.png" alt="After its saved" width="800" height="418"></a></p>

<p>19.Got to github.<a href="https://github.com/..sign" rel="noopener noreferrer">https://github.com/..sign</a> in to your github or signup if you have not gotten a github account. Go to <strong>+create</strong> to create a new repository.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhppdekoxyb0t08w6j5pe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhppdekoxyb0t08w6j5pe.png" alt="+create new repository" width="800" height="420"></a></p>

<p>20.Give the repository a <strong>name</strong>,choose the visibility to be <strong>public</strong>,make sure Add README is <strong>on</strong> and create the repository.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7getqf57p8vpy9xkvkz7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7getqf57p8vpy9xkvkz7.png" alt="Creating new repository" width="800" height="399"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp8xetzs61bu4yrohqgzf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp8xetzs61bu4yrohqgzf.png" alt="Contd create new repository" width="800" height="392"></a></p>

<p>21.on your Github, after creating the new repository, click on <strong>code</strong> and copy the <strong>https</strong> <a href="https://github.com/EMMANUELOBINNAONYEMUCHE/Cloud-Engineer-Resume.git" rel="noopener noreferrer">https://github.com/EMMANUELOBINNAONYEMUCHE/Cloud-Engineer-Resume.git</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyfgcz8vftbwidgcrtmxl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyfgcz8vftbwidgcrtmxl.png" alt="Https" width="800" height="368"></a></p>

<p>22.Go to back to your gitbash and type this command <strong>git remote add origin "url"</strong> (<a href="https://github.com/EMMANUELOBINNAONYEMUCHE/Cloud-Engineer-Resume.git" rel="noopener noreferrer">https://github.com/EMMANUELOBINNAONYEMUCHE/Cloud-Engineer-Resume.git</a>)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpvsalj3f1ealwa75edzp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpvsalj3f1ealwa75edzp.png" alt="git remote add origin url" width="800" height="415"></a></p>

<p>23.Use <strong>git status</strong> to verify there is a file.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzvvtdxv1bekgwed3rivl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzvvtdxv1bekgwed3rivl.png" alt="git status" width="800" height="418"></a></p>

<p>24.Take the file to the staging area by using a command <strong>git add .</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa3btvy0xx0g4xa7gmjsv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa3btvy0xx0g4xa7gmjsv.png" alt="git add ." width="800" height="418"></a></p>

<p>25.Type <strong>git status</strong> command on your gitbash to verify it has been added to the staging area.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkxjlyxgjnti4exd662o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkxjlyxgjnti4exd662o.png" alt="git status" width="800" height="415"></a></p>

<p>26.Type the command <strong>git commit -m "a commit message"</strong> to take a snapshot of the entire file in the staging area and save it.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72y9rht6w4vr97jp76q3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72y9rht6w4vr97jp76q3.png" alt="git commit -m " width="800" height="416"></a></p>

<p>27.In the gitbash, type the command <strong>git push origin master</strong> to push it to the github. It will prompt a page, asking you to sign in to github. Choose sign in through the browser</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl899glz213xncmmrxh42.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl899glz213xncmmrxh42.png" alt="git push origin master" width="800" height="272"></a></p>

<p>28.Login to your github, while its pushing it to your github, after you have logged in, check your gitbash for confirmation that the push to the github is successful.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz0r3eux3vmq5dphsq1v6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz0r3eux3vmq5dphsq1v6.png" alt="Login" width="800" height="401"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsyrp00ie8okuq4v35nqg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsyrp00ie8okuq4v35nqg.png" alt="Verification that the push to github is completed" width="800" height="417"></a></p>

<p>29.In your github, switch to <strong>master</strong> from main because we pushed it to master in our gitbash.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu1ciu76uiiin45p70l92.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu1ciu76uiiin45p70l92.png" alt="Master" width="800" height="369"></a></p>

<p>30.In your github, click on the <strong>index.html</strong> that you created.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F86xmadp0b5p6nbuxaf79.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F86xmadp0b5p6nbuxaf79.png" alt="Index.html" width="800" height="363"></a></p>

<p>31.When you open it, you will notice that everything that you did in you local environment was brought up to github.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0zyqst3bzhn42sqxcqhy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0zyqst3bzhn42sqxcqhy.png" alt="index.html" width="800" height="367"></a></p>

<p>32.Select <strong>settings</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6jadili2kaie45xeuw2w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6jadili2kaie45xeuw2w.png" alt="Settings" width="800" height="365"></a></p>

<p>33.On the left hand side look for <strong>Pages</strong> and click on it. On the <strong>branch</strong>, select <strong>master</strong> and <strong>save</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkd1a0e83ttz03yfi7fxr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkd1a0e83ttz03yfi7fxr.png" alt="Page" width="800" height="367"></a></p>

<p>34.Refresh the page to see a <strong>link</strong> or <strong>visit site</strong> copy the link and paste on new browser or click on visit site to the Resume</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs2arve22vpzqu2v8d15.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs2arve22vpzqu2v8d15.png" alt="Visit site" width="800" height="391"></a></p>

<p><strong>The Resume</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgp328sapqjsz760j6wgx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgp328sapqjsz760j6wgx.png" alt="The resume" width="800" height="394"></a></p>

