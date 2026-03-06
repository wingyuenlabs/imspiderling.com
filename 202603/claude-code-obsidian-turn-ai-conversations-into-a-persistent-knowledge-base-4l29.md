---
Title: Claude Code + Obsidian: Turn AI Conversations into a Persistent Knowledge Base
Description: 
Author: tommy
Date: 2026-03-06T21:55:28.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  "Which chat was that design decision in?"
</h2>

<p>2 AM. Claude hit its limit.</p>

<p>The authentication design was coming together nicely — JWT middleware, token rotation, the whole thing — and then the session cut off mid-thought. I switched to Gemini. "So this project uses JWT auth, and the middleware is structured like..." Back to square one.</p>

<p>Next day, back to Claude. Yesterday's Gemini conversation? Gone. Explain everything again. Third time.</p>

<p>I tried Notion. Logged design decisions, tech choices, remaining tasks. Lasted three days. Day four, deep in implementation, forgot to update it. A week later, opened Notion — everything was stale.</p>

<p><strong>Closed it.</strong></p>

<p>AI is brilliant. But conversations evaporate. Manual note-taking doesn't stick because humans don't stick with it. What I needed was a system that <strong>accumulates knowledge automatically</strong>.</p>

<p>This article shows how I built that with Claude Code + Obsidian.</p>

<blockquote>
<p>Try <a href="https://puremark.app" rel="noopener noreferrer">PureMark</a> — zero-click developer tools built entirely in this environment.</p>
</blockquote>

<h2>
  
  
  What This Setup Achieves
</h2>

<ul>
<li><strong>Claude Code reads and writes Obsidian files directly</strong></li>
<li><strong>"Read the status file and continue" fully restores context</strong></li>
<li><strong>Dev logs, decisions, and tech notes accumulate in Obsidian automatically</strong></li>
<li><strong>Windows Obsidian and WSL Claude Code work seamlessly together</strong></li>
</ul>

<h2>
  
  
  The First Wall: WSL vs Windows
</h2>

<p>Here's the problem: Claude Code runs on WSL. Obsidian runs on Windows. Same PC, different file systems.</p>

<p>First attempt — symlinks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ln</span> <span class="nt">-s</span> /mnt/c/Users/&lt;username&gt;/Obsidian/my-vault ~/vault
</code></pre>

</div>



<p>Simple enough. Except <strong>Claude Code couldn't reliably write through symlinks</strong>. Reads worked, writes didn't. Useless.</p>

<p>The solution: <strong>bind mount</strong>. Mount the Windows folder directly via WSL's fstab. From Claude Code's perspective, it's just a regular local directory.</p>

<ul>
<li>Reliable reads and writes from Claude Code</li>
<li>Stable paths you can reference in CLAUDE.md</li>
<li>Auto-mounts on first access (zero startup cost)</li>
</ul>

<h2>
  
  
  Setup (~10 minutes)
</h2>

<h3>
  
  
  Step 1: Create mount point
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> ~/projects/vault
</code></pre>

</div>



<h3>
  
  
  Step 2: Configure WSL
</h3>

<p>Edit <code>/etc/wsl.conf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nano /etc/wsl.conf
</code></pre>

</div>



<p>Add:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[boot]</span>
<span class="py">systemd</span><span class="p">=</span><span class="s">true</span>

<span class="nn">[automount]</span>
<span class="py">mountFsTab</span><span class="p">=</span><span class="s">true</span>
<span class="py">options</span><span class="p">=</span><span class="s">"metadata,umask=022,fmask=111"</span>
</code></pre>

</div>



<h3>
  
  
  Step 3: Set up auto-mount
</h3>

<p>Edit <code>/etc/fstab</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nano /etc/fstab
</code></pre>

</div>



<p>Add this line (replace paths):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/mnt/c/Users/&lt;WinUser&gt;/Obsidian/&lt;VaultName&gt;  /home/&lt;WslUser&gt;/projects/vault  none  noauto,x-systemd.automount,bind,x-systemd.requires-mounts-for=/mnt/c  0  0
</code></pre>

</div>



<blockquote>
<p><strong>Path example:</strong><br>
Windows: <code>C:\Users\taro\Obsidian\my-vault</code> -&gt; <code>/mnt/c/Users/taro/Obsidian/my-vault</code><br>
WSL: <code>/home/taro/projects/vault</code></p>
</blockquote>

<h3>
  
  
  Step 4: Restart WSL
</h3>

<p>In PowerShell:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="n">wsl</span><span class="w"> </span><span class="nt">--shutdown</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Step 5: Verify
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check systemd</span>
ps <span class="nt">-p</span> 1 <span class="nt">-o</span> <span class="nb">comm</span><span class="o">=</span>
<span class="c"># -&gt; "systemd" means OK</span>

<span class="c"># Check mount</span>
mountpoint ~/projects/vault <span class="o">&amp;&amp;</span> <span class="nb">echo</span> <span class="s2">"OK: Connected"</span>

<span class="c"># Check files</span>
<span class="nb">ls</span> ~/projects/vault
</code></pre>

</div>



<h3>
  
  
  Step 6: Bidirectional test
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># WSL -&gt; Windows</span>
<span class="nb">echo</span> <span class="s2">"# Test from WSL"</span> <span class="o">&gt;</span> ~/projects/vault/_test.md
<span class="c"># -&gt; Should appear in Obsidian</span>

<span class="c"># Windows -&gt; WSL</span>
<span class="c"># Create _test2.md in Obsidian -&gt; check with ls</span>
</code></pre>

</div>



<p>Both work? You're done. Delete the test files.</p>

<h2>
  
  
  Usage Patterns
</h2>

<h3>
  
  
  Pattern 1: Status File as Single Source of Truth
</h3>

<p><strong>This is the most effective pattern.</strong></p>

<p>One "status file" per project. Everything Claude Code needs to know lives here:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># MyProject Status</span>

<span class="gu">## Phase Progress</span>
| Phase | Status | Done |
|-------|--------|------|
| Auth | Done | 03-01 |
| Dashboard | In Progress | -- |

<span class="gu">## Remaining Tasks</span>
<span class="p">-</span> [x] JWT implementation
<span class="p">-</span> [ ] Dashboard UI &lt;- current
<span class="p">-</span> [ ] E2E tests

<span class="gu">## Decision Log</span>
| Date | Decision | Rationale |
|------|----------|-----------|
| 03-01 | JWT | Stateless API design |
| 03-01 | Tailwind | Component-scoped styling |
</code></pre>

</div>



<p>Start a new Claude Code session with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Read ~/projects/vault/Projects/MyProject/status.md and continue where we left off"
</code></pre>

</div>



<p><strong>Full context restored.</strong> Chat history gone? Knowledge stays.</p>

<blockquote>
<p>I built <a href="https://puremark.app" rel="noopener noreferrer">PureMark</a> — a suite of zero-click developer tools — entirely using this workflow. Status file + decision log + dev diary, all in Obsidian, shared with Claude Code. Consistent development across sessions from Phase 0 through Phase 4.</p>
</blockquote>

<h3>
  
  
  Pattern 2: Reference Obsidian from CLAUDE.md
</h3>

<p>Define Obsidian paths in Claude Code's <code>CLAUDE.md</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># CLAUDE.md</span>

<span class="gu">## Knowledge Base</span>
<span class="p">-</span> Status: ~/projects/vault/Projects/MyProject/status.md
<span class="p">-</span> Tech notes: ~/projects/vault/Notes/
<span class="p">-</span> Decisions: status file "Decision Log" section

<span class="gu">## Rules</span>
<span class="p">-</span> Update status file when completing tasks
<span class="p">-</span> Log important decisions in the decision log
</code></pre>

</div>



<p>Claude Code will <strong>autonomously</strong> update the status file as it works.</p>

<h3>
  
  
  Pattern 3: Automated Dev Logs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nv">DATE</span><span class="o">=</span><span class="si">$(</span><span class="nb">date</span> +%Y-%m-%d<span class="si">)</span>
<span class="nv">LOG</span><span class="o">=</span><span class="s2">"</span><span class="nv">$HOME</span><span class="s2">/projects/vault/Journal/</span><span class="k">${</span><span class="nv">DATE</span><span class="k">}</span><span class="s2">.md"</span>

<span class="nb">echo</span> <span class="s2">"## Dev Log - </span><span class="si">$(</span><span class="nb">date</span> <span class="s1">'+%H:%M'</span><span class="si">)</span><span class="s2">"</span> <span class="o">&gt;&gt;</span> <span class="s2">"</span><span class="nv">$LOG</span><span class="s2">"</span>
git log <span class="nt">--since</span><span class="o">=</span><span class="s2">"today"</span> <span class="nt">--pretty</span><span class="o">=</span>format:<span class="s2">"- %s"</span> <span class="o">&gt;&gt;</span> <span class="s2">"</span><span class="nv">$LOG</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">""</span> <span class="o">&gt;&gt;</span> <span class="s2">"</span><span class="nv">$LOG</span><span class="s2">"</span>
</code></pre>

</div>



<p>Have Claude Code run this daily. Dev logs accumulate with zero effort.</p>

<h2>
  
  
  Troubleshooting
</h2>

<p><strong>"Processing /etc/fstab failed"</strong><br>
Check for typos in paths/options. Run <code>sudo mount -a -v</code> for detailed errors.</p>

<p><strong>Vault folder is empty</strong><br>
Automount triggers on first access. Just run <code>ls ~/projects/vault</code>.</p>

<p><strong>Claude Code can't write files</strong><br>
Confirm it's a bind mount, not a symlink: <code>mountpoint ~/projects/vault</code> should say "is a mountpoint".</p>

<p><strong>Windows edits not reflecting in WSL</strong><br>
Remount: <code>sudo umount ~/projects/vault &amp;&amp; ls ~/projects/vault</code></p>

<h2>
  
  
  Before / After
</h2>

<p><strong>Before:</strong></p>

<ul>
<li>Context resets every time an AI session ends</li>
<li>"I'll document it later" never happens</li>
<li>Switching AIs means re-explaining everything</li>
</ul>

<p><strong>After:</strong></p>

<ul>
<li>"Read the status file" = full context restored</li>
<li>Claude Code accumulates knowledge automatically</li>
<li>Everything searchable in Obsidian</li>
</ul>

<p>10 minutes of setup turns Claude Code conversations from <strong>volatile information</strong> into <strong>compounding assets</strong>.</p>

<p><a href="https://puremark.app" rel="noopener noreferrer">PureMark</a> — zero-click developer tools, born from this workflow.</p>




<h2>
  
  
  References
</h2>

<ul>
<li><a href="https://docs.anthropic.com/en/docs/claude-code/overview" rel="noopener noreferrer">Claude Code Documentation</a></li>
<li><a href="https://learn.microsoft.com/en-us/windows/wsl/" rel="noopener noreferrer">WSL Documentation</a></li>
<li><a href="https://obsidian.md/" rel="noopener noreferrer">Obsidian</a></li>
</ul>

