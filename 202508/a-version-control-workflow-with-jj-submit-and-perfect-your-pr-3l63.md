---
Title: A version control tutorial with jj: Submit and Perfect Your PR
Description: 
Author: Altariarite
Date: 2025-08-30T20:43:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>In the first part of the series, we looked at how to set up a jj repo and make our first push to GitHub. We touched on "commit"s and bookmarks in jj. Be sure to check out that one if you don't know what these words mean!</p>

<p>Now we get into the more interesting stuff: how to submit PRs and edit them easily. </p>

<blockquote>
<p>Note: I've since shortened the id for a jj commit to 4 characters in the <a href="https://gist.github.com/Altariarite/0c11fbf8c73d802a4261e1031dd12977" rel="noopener noreferrer">config</a> by setting <code>'format_short_id(id)' = 'id.shortest(4)'</code> under <code>template-aliases</code></p>
</blockquote>




<h3>
  
  
  0: Old friends, new friends, immutable friends
</h3>

<p>Before doing anything, let's run <code>jj log</code> again to check the status after our first push.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3usu0s2gx6fb6c6etlqr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3usu0s2gx6fb6c6etlqr.png" alt=" " width="800" height="520"></a></p>

<p>Hmm, something unusual is going on here. We said in the first tutorial that the line starting with <code>◆</code> is the <strong>root commit</strong>.  Why is the root commit not shown here? Why is the <code>◆</code> on our previous commit? And what is this <code>~</code> thing? </p>

<p>Let's tackle these one by one. First of all,  <code>~</code> means that there are more commits to be shown. We can see all the commits by running <code>jj log -r 'all()'</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdu8zw5vuwsybmthg6blc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdu8zw5vuwsybmthg6blc.png" alt=" " width="800" height="598"></a></p>

<p><code>◆</code> marks the root commit, but it also marks our "Add hello world" commit <code>xvux</code> that we just pushed to main. It means these are immutable commits. Indeed, if we try to edit it, we would get an error.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> jj edit x                                                                                                                                                                         <span class="o">(</span>base<span class="o">)</span>
Error: Commit 47b0d687d09c is immutable
Hint: Could not modify commit: xvux main | Add hello world
Hint: Immutable commits are used to protect shared history.
Hint: For more information, see:
      - https://jj-vcs.github.io/jj/latest/config/#set-of-immutable-commits
      - <span class="sb">`</span>jj <span class="nb">help</span> <span class="nt">-k</span> config<span class="sb">`</span>, <span class="s2">"Set of immutable commits"</span>
Hint: This operation would rewrite 1 immutable commits.
</code></pre>

</div>



<p>Since these commits cannot be modified, they are not very interesting to us. So <code>jj</code> hides them by default and only shows the latest immutable commit. I quite like that.</p>

<blockquote>
<p>Note: technically the commit can be modified if you "force push" to main, but it is a very dangerous operation! We will not cover it here.</p>
</blockquote>

<p>Now to the actual PR.</p>

<h3>
  
  
  1: Create a Pull Request 🚀
</h3>

<p>Let's wrap our hello world in a function. We will make an intentional spelling error to be fixed later.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">hellp</span><span class="p">():</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">hello world!</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>Let's commit like before:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>jj commit <span class="nt">-m</span> <span class="s2">"my first pr"</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzoescuf8mbap6rxxo8ks.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzoescuf8mbap6rxxo8ks.png" alt=" " width="800" height="637"></a></p>

<p>We could create a new bookmark like we did before, but I am lazy and choose to let <code>jj</code> do it automatically this time.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>jj git push <span class="nt">--change</span> t
</code></pre>

</div>



<p><code>jj</code> will generate a bookmark on that commit and push. It will also generate a link for you to open a pull request.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>Creating bookmark push-tsnxozwsqwwt <span class="k">for </span>revision tsnxozwsqwwt
Changes to push to origin:
  Add bookmark &lt;your-bookmark&gt; to &lt;some string&gt;
remote:
remote: Create a pull request <span class="k">for</span> &lt;your-bookmark&gt; on GitHub by visiting:
remote:      https://github.com/&lt;your-repo&gt;/pull/new/&lt;your-bookmark&gt;
remote:
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo321cbvmpq1n5fmizhyd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo321cbvmpq1n5fmizhyd.png" alt=" " width="800" height="319"></a></p>

<h3>
  
  
  2: Edit the PR with Feedback 🔄
</h3>

<p>Code review is a conversation and usually involves multiple back and forth edits. Let’s say a teammate reviews your PR and asks you to fix the typo. </p>

<p>With <code>jj</code>, there are ways to directly <strong>edit the original commit</strong>, which keeps the history clean.</p>

<h4>
  
  
  Fixing the typo
</h4>

<p>After you write the fix, run <code>jj diff</code>. We see that saved changes are reflected.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3t9857bt8yhr7pesmzl8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3t9857bt8yhr7pesmzl8.png" alt=" " width="800" height="360"></a></p>

<p>Since the fix is happening in the "Working copy", it is now one commit ahead of our original commit. We want to move the fix down to our original commit.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsgwnelm1imifqe6qhip5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsgwnelm1imifqe6qhip5.png" alt=" " width="800" height="467"></a></p>

<p><code>jj squash</code> will do exactly that. It takes our new change and move it down to the commit we pushed. Our bookmark will have a <code>*</code>, indicating that something changed in this commit.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> jj squash

Working copy  <span class="o">(</span>@<span class="o">)</span> now at: slzw <span class="o">(</span>empty<span class="o">)</span> <span class="o">(</span>no description <span class="nb">set</span><span class="o">)</span>
Parent commit <span class="o">(</span>@-<span class="o">)</span>      : tsnx push-tsnxozwsqwwt<span class="k">*</span> | my first <span class="nb">pr</span>
</code></pre>

</div>



<p>Running <code>jj git push</code> will push our fix to the branch and to the PR. This way you won't clutter the PR with commits like "Fix typo".<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> jj git push

Changes to push to origin:
Move sideways bookmark push-tsnxozwsqwwt from 4bb496b59686 to 80c14a7f4239
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedvwv0srju34qnnh0bc9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedvwv0srju34qnnh0bc9.png" alt=" " width="800" height="362"></a></p>

<h4>
  
  
  Changing the commit message
</h4>

<p>Another teammate points out that your commit message and PR title shouldn't be named "my first pr" but something like "Refactor to hello()". That's a good point! The commit message is very easy to change in <code>jj</code> . </p>

<p>All we need is  <code>jj describe -r t -m "Refactor to hello()</code></p>

<blockquote>
<p>Note: <code>-r</code> stands for "revision", a synonym to "commit". <code>t</code> is the shortest unique id for our commit</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F539nafwty4kffms0frx0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F539nafwty4kffms0frx0.png" alt=" " width="800" height="458"></a></p>

<p>We can <code>jj git push</code> again.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fothypme7s5c2j1oj45ru.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fothypme7s5c2j1oj45ru.png" alt=" " width="800" height="473"></a></p>

<p>And now the PR is ready to be merged! </p>

<p>This is the place where our local history will start to diverge from the remote repo. The remote repo might also change because other people are working on it at the same time as you. In the next tutorial, we will explore how to deal with the difference and sync from remote to local. Stay tuned!</p>

