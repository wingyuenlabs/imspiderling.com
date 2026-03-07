---
Title: I Ditched GUI Tools and My Productivity Doubled — Here Are the 5 Terminal Tools That Did It
Description: 
Author: Alex Shev
Date: 2026-03-07T22:03:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>I used to be a GUI person. Premiere for video. Tower for git. Finder for files.</p>

<p>Then last year, I started building automation pipelines — video production, data processing, server deploys — and I kept hitting the same wall: GUI tools don't scale. You can't loop a drag-and-drop. You can't pipe a button click.</p>

<p>So I went terminal-first. And I'm not going back.</p>

<p>Here are the 5 tools that made the difference.</p>




<h2>
  
  
  1. <strong>ripgrep (rg)</strong> — Search That Actually Respects Your Time
</h2>

<p><code>grep</code> works. <code>ripgrep</code> works <em>fast</em>.</p>

<p>It respects <code>.gitignore</code> by default, searches recursively, and handles Unicode. On a 50GB monorepo, the difference isn't seconds — it's minutes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Search for "TODO" across your entire project, skip node_modules automatically</span>
rg <span class="s2">"TODO"</span> <span class="nt">--type</span> ts

<span class="c"># Case-insensitive search with context lines</span>
rg <span class="nt">-i</span> <span class="s2">"deprecated"</span> <span class="nt">-C</span> 3
</code></pre>

</div>



<p><strong>Why it matters in 2026:</strong> Codebases are bigger than ever. AI-generated code means more files, more boilerplate, more things to search through. You need a search tool that scales.</p>

<p><strong>Install:</strong> <code>brew install ripgrep</code> / <code>cargo install ripgrep</code></p>




<h2>
  
  
  2. <strong>jq</strong> — JSON Surgery from the Command Line
</h2>

<p>Every API returns JSON. Every config file is JSON. Every log pipeline outputs JSON.</p>

<p><code>jq</code> lets you slice, filter, and transform it without opening a browser tab or writing a Python script.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Extract all error messages from an API response</span>
curl <span class="nt">-s</span> api.example.com/logs | jq <span class="s1">'.entries[] | select(.level == "error") | .message'</span>

<span class="c"># Reshape data — pull specific fields into a new structure</span>
<span class="nb">cat </span>data.json | jq <span class="s1">'[.users[] | {name: .name, email: .contact.email}]'</span>
</code></pre>

</div>



<p><strong>The real power:</strong> Piping. Chain <code>curl | jq | xargs</code> and you've built an API automation pipeline in one line. No dependencies. No virtual environments. No "let me spin up a quick script."</p>

<p><strong>Install:</strong> <code>brew install jq</code> / <code>apt install jq</code></p>




<h2>
  
  
  3. <strong>FFmpeg</strong> — The Swiss Army Knife Nobody Teaches You
</h2>

<p>Most developers know FFmpeg exists. Few know it can replace an entire video editing suite.</p>

<p>I recently produced a 49-second educational short — compositing multiple video layers, syncing audio, adding animated subtitles — entirely from the terminal. No Premiere. No DaVinci Resolve.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Trim a video with frame-accurate precision</span>
ffmpeg <span class="nt">-ss</span> 00:01:30 <span class="nt">-to</span> 00:02:45 <span class="nt">-i</span> input.mp4 <span class="nt">-c</span> copy trimmed.mp4

<span class="c"># Overlay a watermark with position control</span>
ffmpeg <span class="nt">-i</span> video.mp4 <span class="nt">-i</span> logo.png <span class="nt">-filter_complex</span> <span class="s2">"overlay=W-w-10:H-h-10"</span> output.mp4

<span class="c"># Generate a thumbnail from a specific timestamp</span>
ffmpeg <span class="nt">-i</span> video.mp4 <span class="nt">-ss</span> 00:00:15 <span class="nt">-frames</span>:v 1 thumbnail.jpg
</code></pre>

</div>



<p><strong>The unlock:</strong> Once you can script video processing, you can automate content pipelines. Batch process 100 videos. Auto-generate thumbnails. Convert formats at scale. I've been collecting FFmpeg patterns on <a href="https://terminalskills.io" rel="noopener noreferrer">terminalskills.io</a> because the documentation alone could fill a textbook.</p>

<p><strong>Install:</strong> <code>brew install ffmpeg</code> / <code>apt install ffmpeg</code></p>




<h2>
  
  
  4. <strong>lazygit</strong> — Git, But You Can Actually See What's Happening
</h2>

<p>I love git. I hate remembering whether it's <code>git rebase -i HEAD~3</code> or <code>git rebase -i HEAD~4</code>.</p>

<p>lazygit gives you a terminal UI for git — staging hunks, interactive rebasing, cherry-picking — without leaving the terminal or touching a mouse.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Just run it in any git repo</span>
lazygit
</code></pre>

</div>



<p><strong>What changed for me:</strong> I stopped being afraid of interactive rebases. The visual diff panel means you SEE what you're about to do before you do it. Conflict resolution went from "let me open VS Code" to "done in 10 seconds."</p>

<p><strong>Pro tip:</strong> Hit <code>?</code> inside lazygit for the full keybinding list. <code>space</code> to stage, <code>c</code> to commit, <code>p</code> to push. That's 90% of your workflow.</p>

<p><strong>Install:</strong> <code>brew install lazygit</code> / <code>go install github.com/jesseduffield/lazygit@latest</code></p>




<h2>
  
  
  5. <strong>zoxide</strong> — cd, But It Remembers Where You've Been
</h2>

<p>This is the smallest tool on the list and the one I'd miss the most.</p>

<p><code>zoxide</code> replaces <code>cd</code> with a smart directory jumper. It learns which directories you visit most and lets you jump to them with partial names.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Instead of: cd ~/projects/company/backend/services/auth</span>
<span class="c"># Just type:</span>
z auth

<span class="c"># It knows you mean that specific auth directory</span>
<span class="c"># because you've been there 50 times this week</span>
</code></pre>

</div>



<p><strong>Why it's not trivial:</strong> I timed it. On an average workday, I <code>cd</code> about 80 times. Each full path takes 3-5 seconds to type. zoxide cuts that to under 1 second. That's 5-6 minutes saved daily. Over a year, that's nearly a full workday of just... not typing paths.</p>

<p><strong>Install:</strong> <code>brew install zoxide</code> / <code>curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh</code></p>




<h2>
  
  
  The Pattern
</h2>

<p>None of these tools are new. But together, they changed how I think about work.</p>

<p>GUI tools are great for one-off tasks. But the moment you need to do something twice — the terminal wins. Every time.</p>

<p>I went from "let me open the app" to "let me write the command" and my throughput doubled. Not because I'm faster at typing. Because scripts don't forget steps, don't get tired, and don't need a mouse.</p>




<p><strong>What's your terminal setup look like?</strong> I'm always looking for tools I've missed — I bet someone here has a workflow that would blow my mind. Drop it in the comments 👇</p>

<p><em>Some parts of this article were refined with AI assistance.</em></p>

