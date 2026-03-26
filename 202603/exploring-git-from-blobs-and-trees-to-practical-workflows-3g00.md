---
Title: Exploring Git: From Blobs and Trees to Practical Workflows
Description: 
Author: Abhishek
Date: 2026-03-26T22:04:51.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fke5ib0autkutludlvwqq.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fke5ib0autkutludlvwqq.jpeg" alt="Img1" width="800" height="600"></a><br>
How does git store files and keep track of everything while being efficient? What's inside the <code>.git</code> folder?? With these interactive questions the session was kick started by <a href="https://www.linkedin.com/in/alfiyafatima09?utm_source=share_via&amp;utm_content=profile&amp;utm_medium=member_android" rel="noopener noreferrer">Alfiya didi</a> .</p>

<p>One of the most interesting questions she asked was: Can Git store or commit an empty folder? We'll come back to this at the end :)</p>

<p>Git is a distributed <strong>version control tool</strong> that helps in managing code and promotes collaboration. It was started by Linus Torvalds, who built the first version of Git in just 10 days.</p>

<p>Before Git, Linus was using another version control system called BitKeeper for Linux development. But when its license was revoked, he decided to build his own system from scratch which eventually became Git.</p>

<p>But what actually goes on under the hood? And why is Git so fast and efficient?</p>

<h2>
  
  
  1.So how Git actually works?
</h2>

<p>Git doesn't actually track files the way I assumed it did.It tracks the content.</p>

<p>At its core, Git has something called an <strong>object database</strong>, which is where it stores everything as compressed objects identified by hashes. Instead of tracking files directly, Git stores data based on content, meaning every piece of data is uniquely identified by its hash.</p>

<p>This database is what makes Git fast and efficient, because identical content is stored only once and reused wherever needed.Inside this database, Git keeps three kinds of objects: blobs for file content, trees for structure, and commits for snapshots. They all point to each other, which is how Git keeps everything connected.</p>

<p>Git assigns every object a unique ID using <strong>SHA-1</strong>, based on what's inside it. So even the smallest change makes it look completely different to Git</p>

<p>All of this is built using the same three core objects we talked about:</p>

<p>• <strong>Commit</strong> : its not just a list of changes it's a full snapshot of your whole project at a point in time , linked to the previous one . That's how Git builds history. It stores metadata like: → author name<br>
→ author email<br>
→ timestamp<br>
→ parent commit hash(except for the first commit)</p>

<p>• <strong>Tree</strong>: A tree gives the structure . It connects names to blobs and other trees, essentially building your folder structure without actually "storing" folders. Each commit points to a tree, and that tree represents a complete snapshot of your project at that time It also keeps track of: <br>
→folder structure <br>
→Permissions</p>

<p>• <strong>Blob (Binary Large Object)</strong> : a blob is just raw file content, it doesn't store name, file path or anything extra . The most interesting part of this session was how it answered the question: "why is Git efficient?"<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimgs.search.brave.com%2FYLBYhL0uPB2v8ouvkYzbvA-HlBCg6o6A08m30V3P8dw%2Frs%3Afit%3A860%3A0%3A0%3A0%2Fg%3Ace%2FaHR0cHM6Ly9zdWJz%2FdGFja2Nkbi5jb20v%2FaW1hZ2UvZmV0Y2gv%2FJHNfIWpCZnchLHdf%2FMTQ1NixjX2xpbWl0%2FLGZfYXV0byxxX2F1%2FdG86Z29vZCxmbF9w%2Fcm9ncmVzc2l2ZTpz%2FdGVlcC9odHRwczov%2FL3N1YnN0YWNrLXBv%2Fc3QtbWVkaWEuczMu%2FYW1hem9uYXdzLmNv%2FbS9wdWJsaWMvaW1h%2FZ2VzLzczZDJlNTlm%2FLWUxNWQtNGU2NC1i%2FYmRmLWM4ODI5OTNj%2FOTJlNV82MzR4NTUx%2FLnBuZw" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimgs.search.brave.com%2FYLBYhL0uPB2v8ouvkYzbvA-HlBCg6o6A08m30V3P8dw%2Frs%3Afit%3A860%3A0%3A0%3A0%2Fg%3Ace%2FaHR0cHM6Ly9zdWJz%2FdGFja2Nkbi5jb20v%2FaW1hZ2UvZmV0Y2gv%2FJHNfIWpCZnchLHdf%2FMTQ1NixjX2xpbWl0%2FLGZfYXV0byxxX2F1%2FdG86Z29vZCxmbF9w%2Fcm9ncmVzc2l2ZTpz%2FdGVlcC9odHRwczov%2FL3N1YnN0YWNrLXBv%2Fc3QtbWVkaWEuczMu%2FYW1hem9uYXdzLmNv%2FbS9wdWJsaWMvaW1h%2FZ2VzLzczZDJlNTlm%2FLWUxNWQtNGU2NC1i%2FYmRmLWM4ODI5OTNj%2FOTJlNV82MzR4NTUx%2FLnBuZw" alt="img1" width="634" height="551"></a></p>

<p>If two files have the exact same content, Git doesn't store them twice. It just stores one blob and <strong>reuses it with the help of pointers</strong>. That is because Git uses SHA-1, so the same content always produces the same hash, letting Git recognize it as the same blob. That means no unnecessary duplication, even across commits</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Same content → same blob → no duplication.
</code></pre>

</div>

<p>Coming back to the question from the start - can Git store or commit an empty folder?</p>

<p>Git tracks content (blobs) and the state of files. Since empty folder has no content, Git does not generate a hash for it, which makes it effectively invisible to Git. That's why empty folders cannot be committed unless you add a file inside them.</p>

<p>With this <a href="https://www.linkedin.com/in/alfiyafatima09?utm_source=share_via&amp;utm_content=profile&amp;utm_medium=member_android" rel="noopener noreferrer">Alfiya didi</a> wrapped her part of the session and <a href="https://www.linkedin.com/in/chetanr25?utm_source=share_via&amp;utm_content=profile&amp;utm_medium=member_android" rel="noopener noreferrer">Chetan Bhaiya</a> stepped in to show how all of this actually works in practice.</p>

<h3>
  
  
  Conclusion of Part 1
</h3>

<p>• Always try to understand how tools work under the hood, not just use commands.<br>
• Theory + practical usage together give a complete understanding of any tech.<br>
• Git uses SHA-1 hashes to perfectly duplicate content and links everything together through interconnected snapshots to achieve such efficiency, security, and reliability under the hood.<br>
• We have three types of objects in Git: blobs, trees, and commits. Git always generates a key for any of them, a SHA-1 hash, and then persists the content zipped in the repository.<br>
• Commits capture full snapshots, trees organize structure, and blobs store raw data.</p>

<h2>
  
  
  2. Git in practical workflows
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxql3h1k9e2mdahyf7xml.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxql3h1k9e2mdahyf7xml.jpeg" alt="Image2" width="800" height="1198"></a><br>
Most of us use Git and GitHub without really understanding what happens behind the scenes, or what advantages and risks come with it. <a href="https://www.linkedin.com/in/chetanr25?utm_source=share_via&amp;utm_content=profile&amp;utm_medium=member_android" rel="noopener noreferrer">Chetan bhaiya</a> continued the session by explaining the practical side of Git and how the common commands we use every day actually work.</p>

<p>Chetan bhaiya simplified Git into a flow:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  Working directory → Staging area → Local repo → Remote (GitHub)
</code></pre>

</div>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Flh7-us.googleusercontent.com%2FMpGL9X8VgKi1o96riAwbN_4rYwaNyNQboqrd_uBCWW80SfTtstfCybDq7UZ2R4IBZ36BXsXBX8P86ntHdbhBIpU3P5WWfZxTOsAuAv7e2eThReHH2Mj-A7naNL4IZDAtI0C4NC6YL2zht8fMLs2W6aw" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Flh7-us.googleusercontent.com%2FMpGL9X8VgKi1o96riAwbN_4rYwaNyNQboqrd_uBCWW80SfTtstfCybDq7UZ2R4IBZ36BXsXBX8P86ntHdbhBIpU3P5WWfZxTOsAuAv7e2eThReHH2Mj-A7naNL4IZDAtI0C4NC6YL2zht8fMLs2W6aw" alt="from https://kodekloud.com/blog/how-git-works" width="600" height="337"></a></p>

<p>Your code starts in the working directory, where you make changes. When you use <code>git add</code>, you are not saving everything immediately but rather moving the changes you want into the staging area. Then <code>git commit</code> takes that staged version and stores it in your local repository as a proper snapshot. Finally, <code>git push</code> sends it to GitHub, so it is available remotely.</p>

<p>Chetan bhaiya used visuals like nodes to explain us the structure where each node represents a commit, and from there he explained how branching actually works.That part made the whole Git tree feel much easier to imagine. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9sw0fbi2zogqje450zsp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9sw0fbi2zogqje450zsp.png" alt="Image2" width="800" height="800"></a></p>

<p>He also clarified how <code>git fetch</code>, <code>git merge</code>, and <code>git pull</code> are related. <code>git fetch</code> only brings the changes from the remote repository, while <code>git merge</code> applies those changes to your current branch. <code>git pull</code> is essentially a combination of both where it fetches and then merges. <br>
He also spoke about merge conflicts, which happen when Git cannot automatically decide how to combine changes. This usually occurs when the same part of a file is modified in different commits. In such cases, Git pauses and lets you manually resolve what the final version should look like.</p>

<p>During the explanation of pull and push someone jokingly said <strong>"force push"</strong> and Chetan bhaiya immediately said <strong>never use force push</strong> and be very careful while using it and then I remembered the incident in Zenith hackathon conducted by <a href="https://www.linkedin.com/company/pointblank-club/posts/?feedView=all" rel="noopener noreferrer">us</a> where one team force pushed and deleted a very essential file and since force push rewrites history, the file didn't just disappear from the latest version it was it was no longer part of the visible commit history.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2edqsq3al8da6eqymtq5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2edqsq3al8da6eqymtq5.png" alt="Image3" width="600" height="450"></a></p>

<p>Chetan bhaiya also covered <code>git branch</code> and <code>git checkout</code>, which are essential for navigating in Git. Using <code>git checkout</code>, you can switch to a new branch or even move to a specific commit. <br>
For example, when you run <code>git checkout main</code>, you are on the <code>main</code> branch. Any new commit you make moves the <code>main</code> branch forward, and <code>HEAD</code> simply stays attached to that branch.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvdjjwzj09mmlf02bakp.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvdjjwzj09mmlf02bakp.jpg" alt="meme1" width="800" height="302"></a> <br>
Most of the time, <code>HEAD</code> points to a branch. But if you checkout a commit using <code>git checkout</code> <code>HEAD</code> points directly to that commit instead. This is called a <strong>detached HEAD</strong> state.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff6rsiojdtis472txxa4w.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff6rsiojdtis472txxa4w.jpg" alt="meme2" width="800" height="361"></a><br>
You can still make commits here, but they won’t belong to any branch. They exist, but no branch is pointing to them. So if you switch away without creating a branch, those commits can easily get lost.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnq9r9fh7kvz81fgkwz5d.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnq9r9fh7kvz81fgkwz5d.jpg" alt="meme3" width="800" height="389"></a><br>
He suggested a simple fix, if you ever checkout a commit and plan to make changes, create a new branch from there so your work is preserved.And even if you mess this up, Git still gives you a safety net through <code>reflog</code>, which can help you can trace where your <code>HEAD</code> was and recover those commits but for a limited time.</p>

<p>Later, he explained about the <code>reverse</code> and <code>reset</code>. <code>revert</code> is the safer option as it undoes changes by creating a new commit, so history stays intact. But <code>reset</code> actually rewrites history. It can make your commit tree look cleaner, but u can lose commits which made it clear that Git is not just about saving code ,it is also about recovering from mistakes, when used properly. <br>
Even if you lose commits after a reset, Git doesn’t immediately forget them. While git log shows your visible commit history, there’s also <code>reflog</code> like mentioned before, which tracks where your <code>HEAD</code> has been for around 60–90 days. so you can still recover “lost” commits if you know what you’re doing. So even when it feels like something is gone, Git usually still has a memory of it.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimgs.search.brave.com%2F0mb-0aQx3mQrC9xDiCKKvYueYts1cJwPyHKV2OVy1xM%2Frs%3Afit%3A860%3A0%3A0%3A0%2Fg%3Ace%2FaHR0cHM6Ly9pLnBy%2Fb2dyYW1tZXJodW1v%2Fci5pby8yMDI1LzEw%2FLzljY2Y2MTc5N2Zh%2FNjg5NGMxMDdjYWM3%2FM2FiYzdlMmYxMjNi%2FYjY1ZmM0YzA2ZTRm%2FYTcwNzY4OWZjYTk3%2FNzVlYTUucG5n" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimgs.search.brave.com%2F0mb-0aQx3mQrC9xDiCKKvYueYts1cJwPyHKV2OVy1xM%2Frs%3Afit%3A860%3A0%3A0%3A0%2Fg%3Ace%2FaHR0cHM6Ly9pLnBy%2Fb2dyYW1tZXJodW1v%2Fci5pby8yMDI1LzEw%2FLzljY2Y2MTc5N2Zh%2FNjg5NGMxMDdjYWM3%2FM2FiYzdlMmYxMjNi%2FYjY1ZmM0YzA2ZTRm%2FYTcwNzY4OWZjYTk3%2FNzVlYTUucG5n" alt="meme4" width="500" height="500"></a></p>

<p>At the end we got into another really useful concept <code>git cherry-pick</code>. It is basically taking one specific commit and applying it somewhere else without bringing everything along.Its like selectively copying just what you need instead of merging entire branches.</p>

<h3>
  
  
  Conclusion of Part 2
</h3>

<p>• Git is a three stage pipeline: By moving changes from the Working Directory to the Staging Area before the Local Commit, you gain total control over exactly what becomes a permanent part of your project’s history.<br>
• Git is forgiving:Between the reflog and revert commands, the system is designed with the assumption that we will make mistakes and made to help us<br>
• Understand the workflow : Understanding the actual path your code takes, from your laptop to the staging area and finally to the local repo<br>
•<code>git fetch</code> only downloads data, while <code>git merge</code> applies it<br>
 and <code>git pull</code>=<code>git fetch</code> + <code>git merge</code></p>

<p>Thanks to this session by <a href="https://www.linkedin.com/in/alfiyafatima09?utm_source=share_via&amp;utm_content=profile&amp;utm_medium=member_android" rel="noopener noreferrer">Alfiya didi</a> and <a href="https://www.linkedin.com/in/chetanr25?utm_source=share_via&amp;utm_content=profile&amp;utm_medium=member_android" rel="noopener noreferrer">Chetan bhaiya</a>, we learned a lot from Git. From what actually exists inside the <code>.git</code> folder to how Git behaves in real workflows.</p>

<p>Writted by <a href="https://www.linkedin.com/in/abhishek-r-p-87a7b8389" rel="noopener noreferrer">Abhishek R P</a></p>

