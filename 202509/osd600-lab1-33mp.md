---
Title: OSD600 - Lab1
Description: 
Author: OleksandraKordonets
Date: 2025-09-12T22:11:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hello!<br>
I’m working on a project for the OSD600 course: the Repository Context Packager. It’s a command-line tool that analyzes local Git repositories and generates a text file with repository content optimized for sharing with Large Language Models.</p>

<p>As part of the assignment, we were tasked with reviewing each other’s code. Here’s how my experience went:</p>

<p><strong>How did you go about doing your code reviews? Do you prefer an async or sync approach? Why?</strong><br>
I definitely prefer the async approach. Since all of us are students with very different schedules, it’s much easier to work at our own pace and convenience. That said, I did have a chance to join a call with some classmates to discuss our release, and it turned out to be really valuable as it gave me a strong push forward in my work.</p>

<p><strong>What was it like testing and reviewing someone else's code? Did you run into any problems? Did anything surprise you?</strong><br>
The biggest challenge I faced when reviewing someone else’s code was understanding the design of their application. Since they used a completely different programming language than I did, it took me a little time to get comfortable with their structure and style. Aside from that, it was a really positive experience. I enjoyed seeing how others approached the same project in different ways, and it was fun to “play around” with their applications to see what they had implemented so far.</p>

<p><strong>What was it like having someone test and review your code? Were you surprised by anything?</strong><br>
At first, I felt a bit intimidated because I knew my application wasn’t finished, and I worried about others digging too deeply into it since it’s far from perfect. But that’s really what open source is all about collaborating, sharing, and improving together without worrying about perfection. It is reassuring after you undestand that code reviews are meant to help you grow and contribute, not to judge unfinished work. </p>

<p><strong>What kind of issues came up in your testing and review? Discuss a few of them in detail.</strong><br>
The issues that stood out the most during my review were related to capturing Git information and displaying the Structure. For Git, the tool worked correctly when passing a directory, but it failed when passing a single file. As for the Structure part, it often didn’t display as expected and usually just printed the file or directory name. I highlighted these two issues because I’m currently facing similar challenges in my own project. It was reassuring to see that I’m not alone in struggling with these two parts, and it gave me some insight into how others implemented the same modules.</p>

<p><strong>Provide links to issues you filed, and summarize what you found</strong><br>
<a href="https://github.com/CynthiaFotso/Repository-Context-Packager/issues/1" rel="noopener noreferrer">Issue #1</a><br>
<a href="https://github.com/CynthiaFotso/Repository-Context-Packager/issues/2" rel="noopener noreferrer">Issue #2</a><br>
<a href="https://github.com/CynthiaFotso/Repository-Context-Packager/issues/3" rel="noopener noreferrer">Issue #3</a><br>
<a href="https://github.com/CynthiaFotso/Repository-Context-Packager/issues/8" rel="noopener noreferrer">Issue #4</a></p>

<p>Aside from the Git Info and Structure issues I mentioned earlier, I found a couple of minor bugs. <br>
Include flag is not recognized, it is an optional functionality for --include currently does not recognize the short -i flag. This is a small fix but worth noting for better usability. The other one is that the traverseDir function doesn’t skip certain folders like node_modules, .git, or hidden directories during recursion. While these are eventually filtered out later in packageRepo, the recursion still goes inside them, which can slow down processing on larger repositories. A simple improvement would be to add a check in traverseDir to skip these directories right away, to improve performance as our project grows.</p>

<p><strong>Provide links to issues that were filed on your repo, and what they were about</strong><br>
<a href="https://github.com/OleksandraKordonets/Repository-Context-Packager/issues/1" rel="noopener noreferrer">Issue #1</a><br>
<a href="https://github.com/OleksandraKordonets/Repository-Context-Packager/issues/2" rel="noopener noreferrer">Issue #2</a></p>

<p>These issues arose because I’m adding my code one module at a time, and naturally some parts aren’t fully working yet since other dependent pieces are missing. Looking back, I realize that I don’t need to wait for everything to be perfect before pushing changes I can make improvements as I go. For now, I did take a more challenging approach, which made it a bit harder for both me and anyone testing my project. I also realized I forgot to include an installation guide in my README, which I’ll need to add.</p>

<p><strong>Were you able to fix all your issues? What was that like?</strong><br>
I’m still working on fixing all the issues, but I have to admit that I feel a bit sorry for anyone testing my code in its current state. Some parts are missing, and there’s a bit of inconsistency as I add modules incrementally. </p>

<p><strong>What did you learn through the process of doing the testing and reviewing?</strong><br>
I feel like I learned a lot about software design. Previously, our assignments had clear outlines about what classes and modules we should create, but this time it was just me figuring it out on my own. It was challenging, but also very rewarding. Of course, I sought some advice and guidance along the way, but doing something for the first time rarely results in a perfect outcome, and I’m excited to see how I’ll improve in the future. I also realized how important iterative development is, making small improvements step by step makes it easier for others to contribute. Looking back, some of the stuff I did probably wasn’t the smartest way to go about it, but it’s definitely helping me understand how to make my projects more manageable and collaborative.</p>

