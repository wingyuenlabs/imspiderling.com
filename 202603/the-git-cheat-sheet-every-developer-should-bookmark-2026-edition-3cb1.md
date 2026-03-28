---
Title: The Git Cheat Sheet Every Developer Should Bookmark (2026 Edition)
Description: 
Author: Lucas M Dev
Date: 2026-03-28T21:40:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>Git has ~150 commands. You actually need about 20. Here are the ones that matter.</p>

<h2>
  
  
  Setup (do this first)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git config <span class="nt">--global</span> user.name <span class="s2">"Your Name"</span>
git config <span class="nt">--global</span> user.email <span class="s2">"you@example.com"</span>
git config <span class="nt">--global</span> init.defaultBranch main
git config <span class="nt">--global</span> pull.rebase <span class="nb">false</span>

<span class="c"># Better diff and merge tools</span>
git config <span class="nt">--global</span> diff.tool vscode
git config <span class="nt">--global</span> merge.tool vscode
</code></pre>

</div>



<h2>
  
  
  Starting a project
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># New project</span>
git init
git add <span class="nb">.</span>
git commit <span class="nt">-m</span> <span class="s2">"Initial commit"</span>
git remote add origin https://github.com/you/repo.git
git push <span class="nt">-u</span> origin main

<span class="c"># Clone existing</span>
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder  <span class="c"># Custom folder name</span>
</code></pre>

</div>



<h2>
  
  
  Daily workflow
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check what's changed</span>
git status
git diff                    <span class="c"># Unstaged changes</span>
git diff <span class="nt">--staged</span>           <span class="c"># Staged changes (what will be committed)</span>

<span class="c"># Stage files</span>
git add file.js             <span class="c"># Specific file</span>
git add src/                <span class="c"># Entire folder</span>
git add <span class="nt">-p</span>                  <span class="c"># Interactive: choose which hunks to stage</span>

<span class="c"># Commit</span>
git commit <span class="nt">-m</span> <span class="s2">"Fix login bug"</span>
git commit <span class="nt">-am</span> <span class="s2">"Quick fix"</span>  <span class="c"># Add + commit tracked files</span>

<span class="c"># Push / Pull</span>
git push
git pull
git fetch origin            <span class="c"># Download but don't merge</span>
</code></pre>

</div>



<h2>
  
  
  Branches
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create and switch</span>
git checkout <span class="nt">-b</span> feature/user-auth
git switch <span class="nt">-c</span> feature/user-auth  <span class="c"># Modern syntax</span>

<span class="c"># List branches</span>
git branch                  <span class="c"># Local</span>
git branch <span class="nt">-r</span>               <span class="c"># Remote</span>
git branch <span class="nt">-a</span>               <span class="c"># All</span>

<span class="c"># Switch</span>
git checkout main
git switch main

<span class="c"># Merge</span>
git checkout main
git merge feature/user-auth

<span class="c"># Delete</span>
git branch <span class="nt">-d</span> feature/user-auth    <span class="c"># Safe (only merged)</span>
git branch <span class="nt">-D</span> feature/user-auth    <span class="c"># Force</span>
git push origin <span class="nt">-d</span> feature/user-auth  <span class="c"># Delete remote</span>
</code></pre>

</div>



<h2>
  
  
  Undoing things
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Undo last commit (keep changes staged)</span>
git reset <span class="nt">--soft</span> HEAD~1

<span class="c"># Undo last commit (keep changes unstaged)</span>
git reset HEAD~1

<span class="c"># Undo last commit and DISCARD changes (dangerous!)</span>
git reset <span class="nt">--hard</span> HEAD~1

<span class="c"># Undo a specific file</span>
git checkout <span class="nt">--</span> file.js     <span class="c"># Discard unstaged changes</span>
git restore file.js         <span class="c"># Modern syntax</span>

<span class="c"># Revert a commit (creates a new "undo" commit — safe for shared branches)</span>
git revert abc1234

<span class="c"># Fix the last commit message or add forgotten files</span>
git commit <span class="nt">--amend</span>
</code></pre>

</div>



<h2>
  
  
  Stashing
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Save work-in-progress</span>
git stash
git stash push <span class="nt">-m</span> <span class="s2">"WIP: user auth"</span>  <span class="c"># With message</span>

<span class="c"># List stashes</span>
git stash list

<span class="c"># Apply stash</span>
git stash pop               <span class="c"># Apply + delete</span>
git stash apply stash@<span class="o">{</span>0<span class="o">}</span>   <span class="c"># Apply without deleting</span>

<span class="c"># Drop stash</span>
git stash drop stash@<span class="o">{</span>0<span class="o">}</span>
git stash clear             <span class="c"># Delete all</span>
</code></pre>

</div>



<h2>
  
  
  Viewing history
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git log
git log <span class="nt">--oneline</span>           <span class="c"># Compact</span>
git log <span class="nt">--oneline</span> <span class="nt">--graph</span>   <span class="c"># Tree view</span>
git log <span class="nt">--author</span><span class="o">=</span><span class="s2">"Alice"</span>    <span class="c"># Filter by author</span>
git log <span class="nt">--</span> file.js          <span class="c"># History of a file</span>

<span class="c"># Search commits</span>
git log <span class="nt">--grep</span><span class="o">=</span><span class="s2">"fix"</span>        <span class="c"># Search commit messages</span>
git log <span class="nt">-S</span> <span class="s2">"functionName"</span>   <span class="c"># Search added/removed content (pickaxe)</span>

<span class="c"># Show a commit</span>
git show abc1234
git show HEAD               <span class="c"># Last commit</span>
</code></pre>

</div>



<h2>
  
  
  Remote management
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git remote <span class="nt">-v</span>               <span class="c"># List remotes</span>
git remote add upstream https://github.com/original/repo.git

<span class="c"># Sync fork with upstream</span>
git fetch upstream
git merge upstream/main
</code></pre>

</div>



<h2>
  
  
  Tags
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git tag v1.0.0
git tag <span class="nt">-a</span> v1.0.0 <span class="nt">-m</span> <span class="s2">"Release 1.0.0"</span>
git push origin v1.0.0
git push origin <span class="nt">--tags</span>      <span class="c"># Push all tags</span>
</code></pre>

</div>



<h2>
  
  
  Useful one-liners
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># See who changed a line</span>
git blame file.js

<span class="c"># Find which commit introduced a bug (binary search)</span>
git bisect start
git bisect bad              <span class="c"># Current commit is broken</span>
git bisect good v1.0.0      <span class="c"># This version worked</span>
<span class="c"># Git will checkout commits for you to test</span>
git bisect reset            <span class="c"># When done</span>

<span class="c"># Clean untracked files</span>
git clean <span class="nt">-n</span>                <span class="c"># Dry run (show what would be deleted)</span>
git clean <span class="nt">-fd</span>               <span class="c"># Delete untracked files and folders</span>

<span class="c"># Cherry-pick a commit from another branch</span>
git cherry-pick abc1234

<span class="c"># Rebase interactively (squash, reorder, edit commits)</span>
git rebase <span class="nt">-i</span> HEAD~3        <span class="c"># Last 3 commits</span>
</code></pre>

</div>



<h2>
  
  
  .gitignore patterns
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Specific file
.env
secrets.json

# Folder
node_modules/
.venv/
dist/

# Pattern
*.log
*.pyc
.DS_Store

# Negate (include even if parent is ignored)
!important.log
</code></pre>

</div>






<h2>
  
  
  The 10 commands you'll use 90% of the time
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git status
git add <span class="nt">-p</span>
git commit <span class="nt">-m</span> <span class="s2">""</span>
git push
git pull
git checkout <span class="nt">-b</span> feature/
git merge main
git stash
git log <span class="nt">--oneline</span>
git reset <span class="nt">--soft</span> HEAD~1
</code></pre>

</div>






<h2>
  
  
  Git aliases worth setting up
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git config <span class="nt">--global</span> alias.st status
git config <span class="nt">--global</span> alias.co checkout
git config <span class="nt">--global</span> alias.br branch
git config <span class="nt">--global</span> alias.lg <span class="s2">"log --oneline --graph --all"</span>
git config <span class="nt">--global</span> alias.undo <span class="s2">"reset --soft HEAD~1"</span>
</code></pre>

</div>



<p>Now <code>git lg</code> shows a beautiful commit tree. <code>git undo</code> undoes your last commit safely.</p>




<p><em>Found this useful? Bookmark it for your next git emergency.</em></p>

<p><em>Also check out DevToolkit for more developer utilities: <a href="https://lucasmdevdev.github.io/devtoolkit/" rel="noopener noreferrer">https://lucasmdevdev.github.io/devtoolkit/</a></em></p>

