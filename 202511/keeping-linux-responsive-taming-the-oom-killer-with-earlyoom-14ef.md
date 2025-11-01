---
Title: Keeping Linux Responsive - Taming the OOM Killer with EarlyOOM
Description: 
Author: prasadkjose
Date: 2025-11-01T22:00:39.000Z
Robots: noindex,nofollow
Template: index
---
<center>Ready, Aim, Kill. The Linux hitman that decides what to kill. But is it a dud? </center>




<p>Here is another, more accurate analogy. Linux kernel's OOM killer is like a firefighter who shows up after the house is already ashes. I am pretty sure it kills something, but your screen is already grey, system is frozen, and those Ctrl+ALT+Del panic smashing is fruitless. </p>




<h3>
  
  
  Findings
</h3>

<p>How does the linux kernel choose it's target to kill? <br>
 Let me quote the original kernel method doc for <code>out_of_memory()</code></p>

<blockquote>
<p>[!cite] <a href="https://github.com/torvalds/linux/blob/ec0b62ccc986c06552c57f54116171cfd186ef92/mm/oom_kill.c#L1118" rel="noopener noreferrer">Kernal Doc</a></p>

<blockquote>
<p>If we run out of memory, we have the choice between either killing a random task (bad), letting the system crash (worse) OR try to be smart about which process to kill. Note that we don't have to be perfect here, we just have to be good.</p>
</blockquote>


</blockquote>

<p>It chooses a method based on its <em>badness</em> — a kind of naughty list. <br>
A process is “naughty” if it uses a lot of memory but hasn’t been running long.  It’s important to note that it’s not easy to determine which process to kill.<br><br>
The OOM killer is partial toward root processes, as they’re assumed to be well-behaved.  If processes have direct access to any hardware, they’re pushed further down the naughty list.</p>

<p><em>So why does Linux hang indefinitely when the system runs out of memory?</em><br>
In theory, the OOM killer should free up space and fix this, but why doesn’t it? I became curious about this and set out to find why Linux’s default OOM killer doesn’t work as expected.</p>

<p><strong>The big why:</strong><br><br>
It turns out that the OOM killer is called by the kernel as the absolute <em>last resort.</em> It provides no guarantees about the unpredictable state of the system. In reality, the system isn’t truly locked up. It will eventually process the pending tasks, or the OOM killer will finally get enough time to kill a few processes.<br>
So here’s the question I always have when my machine just freezes: Why didn’t the mighty hitman take out a process and unfreeze my system?<br>
It turns out the native OOM killer tries its best to stay away from any processes in <em>userspace.</em><br>
Do I have 20 Chrome tabs open, devouring my RAM? Yes.  Will the OOM killer do anything about it? No.</p>




<p>So here is where the question gets more complicated. On my machine, I expect the OOM killer to terminate a couple of older Chrome tabs. But what about a server? Do you really want to kill your DB and hope that those beautiful schemas remain unaffected? <br>
What about the combustion control system on the International Space Station? What process do we kill there? The last one might have been an hyperbole, but the complexity of the issue remain. </p>

<p>I know some of you are thinking: <em>what about memory swaps</em>? It doesn't really help to allocate half of your SSD as swap memory and wait for the processes to crawl along painfully.</p>

<p>What I needed specifically for my use case was an OOM killer that targets user processes, like those memory hungry Chrome tabs.  If you have gotten this far, I'll assume you have also been in this situation. So, without further ado, let me introduce you to <a href="https://github.com/rfjakob/earlyoom" rel="noopener noreferrer">Earlyoom</a>, a simple and solid tool that's written in pure C with absolutely no dependencies. You can read about it in detail on their <a href="https://github.com/rfjakob/earlyoom" rel="noopener noreferrer">Github</a>. </p>

<p>This is how it works. Earlyoom monitors available memory(both primary and swap) and when user defined thresholds(default is 10%) are reached, it kills the process consuming the most memory. </p>

<p>Earlyoom came out as a winner to me among alternatives like Meta's <a href="https://github.com/facebookincubator/oomd" rel="noopener noreferrer">OOMD</a> for it's simplicity, easy usage and excellent documentation. </p>

<p>Did I forget to mention that it checks the memory 10 times per second? It leaves no chances for a frozen system. Guess how much memory it uses for all this? 2MiB total and almost 90% of it is native c libraries that's shared with other processes. </p>




<h3>
  
  
  Summary
</h3>

<ol>
<li>Linux's kernel is very robust and plans to act when there is a serious issue. sometimes this might take some time or won't yield the results you expect. </li>
<li>The native OOM killer is built to save your machine or server from many general issues like memory leaks and total collapse.
</li>
<li>This is where Earlyoom is effective and gets to work before the kernel has to. It's fast, small and efficient and get the job done. </li>
<li>You can run it as a <code>systemd</code> service on machine startup and access all it's logs with <code>journalctl</code>.</li>
<li>You can set up desktop notifications in GUI to know which process was terminated.</li>
</ol>




<h2>
  
  
  TL;DR
</h2>

<ol>
<li>Don't worry if your personal computer has only 4 or 8 GB ram. Use Earlyoom to prevent your system being slow or frozen. </li>
<li>The best solution is the one that acts before you even notice the issue. </li>
</ol>




<h2>
  
  
  Commands to Remember
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Installation - </span>
<span class="nb">sudo </span>apt <span class="nb">install </span>earlyoom

<span class="c"># Make sure it's running as a systemd service</span>
<span class="nb">sudo </span>systemctl status earlyoom

<span class="c"># If it's not, start it as a systemd service</span>
<span class="nb">sudo </span>systemctl <span class="nb">enable</span> <span class="nt">--now</span> earlyoom

<span class="c"># Check logs</span>
<span class="nb">sudo </span>journalctl <span class="nt">-u</span> earlyoom | <span class="nb">grep</span> &lt;process name/pid or SIGTERM to see terminated process&gt;

<span class="c"># Change threshold from default 10% to 20%, </span>
<span class="c"># edit /etc/default/earlyoom config file and set line to</span>
<span class="nv">EARLYOOM_ARGS</span><span class="o">=</span><span class="s2">"-m 20"</span>

<span class="c"># Set a preferred process to kill over others. </span>
<span class="c">#EARLYOOM_ARGS="-m 20 --prefer '(^|/)(java|chromium)$'"</span>

<span class="c"># Avoid any critical process like db to be killed by earlyoom</span>
<span class="nv">EARLYOOM_ARGS</span><span class="o">=</span><span class="s2">"-m 20--avoid '(^|/)(sql)</span><span class="nv">$'</span><span class="s2"> --prefer '(^|/)(java|chromium)</span><span class="nv">$'</span><span class="s2">"</span>
</code></pre>

</div>






<h2>
  
  
  Nerd Score
</h2>

<ol>
<li>
<strong>Coolness:</strong> 🔥🔥🔥🔥</li>
<li>
<strong>Effort:</strong> 🔨</li>
</ol>




<ol>
<li>Do you prefer the native linux kernel or a different OOM Killer? </li>
<li>Do you have a different strategy to prevent your system from freezing? </li>
</ol>




<h2>
  
  
  Sources:
</h2>

<ol>
<li><a href="https://github.com/rfjakob/earlyoom" rel="noopener noreferrer">https://github.com/rfjakob/earlyoom</a></li>
<li><a href="https://www.kernel.org/doc/gorman/html/understand/understand016.html" rel="noopener noreferrer">https://www.kernel.org/doc/gorman/html/understand/understand016.html</a></li>
<li><a href="https://chrisdown.name/2018/01/02/in-defence-of-swap.html" rel="noopener noreferrer">https://chrisdown.name/2018/01/02/in-defence-of-swap.html</a></li>
</ol>

