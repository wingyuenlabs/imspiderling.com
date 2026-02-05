---
Title: Linux Kernel Deep Dive: Zombie Processes and Modern Process Management
Description: 
Author: kt
Date: 2026-02-05T22:02:08.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Introduction
</h1>

<p>When you type <code>ps aux</code>, have you ever found the eerie letter <strong>"Z"</strong> in the STAT column?<br>
Or have you ever encountered a "PID limit reached" error while investigating a <code>CrashLoopBackOff</code> on a Kubernetes Pod?</p>

<p>These are all the work of <strong>"Zombie Processes."</strong><br>
They consume neither memory nor CPU, but they cling to the precious resource known as a <strong>"Process ID (PID),"</strong> wandering the system unable to rest in peace.</p>

<p>In this article, we will unravel why processes turn into zombies after death, and how modern mechanisms like "Subreaper" and Go's <code>os/exec</code> mourn (reap) them, at the system call level.</p>


<h2>
  
  
  1. The Identity of Zombie Processes: Why do "Corpses" Remain?
</h2>

<p>In Linux, even when a process <code>exit()</code>s and terminates, it does not disappear immediately.<br>
The kernel retains minimal information (PID, exit status, CPU usage time, etc.) in the process table until the parent process comes to check, "For what reason did that guy die?"</p>

<p>This state of <strong>"dead but awaiting reporting to the parent"</strong> is the zombie process.</p>
<h3>
  
  
  Normal Life Cycle
</h3>

<p>If the parent calls <code>wait()</code>, the zombie rests in peace (disappears).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdos1sp4vu7bemr09dncc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdos1sp4vu7bemr09dncc.png" alt="normal" width="658" height="659"></a></p>

<blockquote>
<p><strong>Syscall Memo</strong>:</p>

<ul>
<li>
<code>wait()</code>: Blocks and waits for any one child process to terminate.</li>
<li>
<code>waitpid()</code>: A high-functionality version that can specify a PID or perform non-blocking checks with the <code>WNOHANG</code> option.</li>
</ul>
</blockquote>


<h2>
  
  
  2. Scenarios Where Zombies Multiply: Child Neglect
</h2>

<p>The problem arises when the parent process does not call <code>wait()</code>.<br>
If the parent continues to ignore the <code>SIGCHLD</code> signal due to a bug in the program (e.g., missing signal handler implementation), the child process will remain a zombie forever.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyqrndmn4bkubilnz52pd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyqrndmn4bkubilnz52pd.png" alt="child neglect" width="692" height="530"></a></p>
<h3>
  
  
  PID Exhaustion by cgroup
</h3>

<p>In the past, the system's <code>pid_max</code> (default 32,768) was the upper limit, but in the container era (Kubernetes), <strong>PID limits at the cgroup (Control Groups) level</strong> are mainstream.<br>
When the <code>pids.max</code> set in the Pod definition is reached, the container falls into a state where it "cannot Fork" (Resource Temporarily Unavailable), even if the host itself has capacity. A massive outbreak of zombies easily causes this.</p>

<blockquote>
<p>A Linux kernel feature that limits and isolates resource usage, such as CPU and memory, for a process group. (This is the reality of "resource limits" in Docker and Kubernetes.)</p>
</blockquote>


<h2>
  
  
  3. Orphan Processes and the Mercy of init (and Subreaper)
</h2>

<p>So, what happens if the parent process that spawned the zombie dies itself?</p>
<h3>
  
  
  Collection by Init Process (PID 1)
</h3>

<p>Children who have lost their parents (orphan processes) are forcibly adopted by the <strong>init process (PID 1)</strong>. Init processes like systemd automatically call <code>wait()</code> when an adopted process dies, allowing it to rest in peace.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr6orhtz9tiw0wqv094yl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr6orhtz9tiw0wqv094yl.png" alt="pid 1" width="428" height="403"></a></p>
<h3>
  
  
  Child Subreaper
</h3>

<p>Sometimes, tracing back to PID 1 is too far.<br>
Since Linux 3.4, using the system call <code>prctl(PR_SET_CHILD_SUBREAPER)</code>, an arbitrary process can declare, <strong>"I will take custody of orphans under this hierarchy."</strong></p>

<p>Docker, containerd, and Chrome's process management use this feature to collect orphan processes under their own management without burdening PID 1 (systemd).</p>


<h2>
  
  
  4. Implementation Patterns to Avoid/Clear Zombies
</h2>
<h3>
  
  
  Pattern A: Honestly wait (Basic)
</h3>

<p>Call <code>waitpid(-1, &amp;status, WNOHANG)</code> in a loop within the <code>SIGCHLD</code> signal handler to collect all dead children.</p>
<h3>
  
  
  Pattern B: Double Fork
</h3>

<p><strong>Usage:</strong> When the parent process <strong>intentionally wants to escape</strong> the duty of managing child processes (daemonization, etc.).</p>

<ol>
<li>
<strong>Parent:</strong> Spawns a child and immediately <code>wait</code>s for it.</li>
<li>
<strong>Child:</strong> Spawns a grandchild (actual processing) and immediately <code>exit</code>s.</li>
<li>
<strong>Grandchild:</strong> Becomes an "orphan" because the parent (child process) died, and is immediately adopted by init (or Subreaper).</li>
</ol>

<blockquote>
<p><strong>Note:</strong> Doing this in a service managed by systemd (<code>Type=simple</code>) will cause systemd to lose track of the main process. Use manual daemonization only when necessary.</p>
</blockquote>
<h3>
  
  
  Pattern C: Tini in Containers (Solving PID 1 Problem)
</h3>

<p>When an app runs as PID 1 in a Docker container, default signal handling may not work, and <code>wait</code> might not be performed properly. It is a standard practice to use <code>tini</code> or <code>dumb-init</code> as the entry point.</p>


<h2>
  
  
  5. Language Deep Dive: Go <code>os/exec</code> Concealment and Traps
</h2>

<p>The Go language <code>os/exec</code> package allows for very convenient execution of external commands, but behind that convenience, the <code>wait</code> system call is concealed.</p>
<h3>
  
  
  Behind <code>cmd.Run()</code>
</h3>

<p>The most common <code>cmd.Run()</code> method internally executes the following two steps:</p>

<ol>
<li>
<strong><code>Start()</code></strong>: Issues <code>fork</code> (clone) and <code>exec</code> to start the process.</li>
<li>
<strong><code>Wait()</code></strong>: Blocks until the process terminates, and releases resources after termination.
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Simplified internal logic image of cmd.Run()</span>
<span class="k">func</span> <span class="p">(</span><span class="n">c</span> <span class="o">*</span><span class="n">Cmd</span><span class="p">)</span> <span class="n">Run</span><span class="p">()</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">c</span><span class="o">.</span><span class="n">Start</span><span class="p">();</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span> <span class="c">// Internally clone + execve</span>
        <span class="k">return</span> <span class="n">err</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">c</span><span class="o">.</span><span class="n">Wait</span><span class="p">()</span> <span class="c">// Calls wait4 system call here!</span>
<span class="p">}</span>
</code></pre>

</div>


<p>In this case, <code>Wait()</code> is reliably called, so zombie processes do not occur.</p>
<h3>
  
  
  The Trap: Using only <code>cmd.Start()</code>
</h3>

<p>If you call only <code>cmd.Start()</code> for asynchronous processing and forget <code>cmd.Wait()</code>, <strong>the child process will become a zombie and remain as long as the Go program is running.</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// 💀 Zombie generation code</span>
<span class="k">func</span> <span class="n">spawnAndForget</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">cmd</span> <span class="o">:=</span> <span class="n">exec</span><span class="o">.</span><span class="n">Command</span><span class="p">(</span><span class="s">"sleep"</span><span class="p">,</span> <span class="s">"1"</span><span class="p">)</span>
    <span class="n">cmd</span><span class="o">.</span><span class="n">Start</span><span class="p">()</span> 
    <span class="c">// Not calling Wait()!</span>
    <span class="c">// 1 second later, the sleep command becomes a zombie (Z),</span>
    <span class="c">// and remains until this Go program terminates.</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Go's Garbage Collection (GC) might collect the <code>cmd</code> struct in memory, but <strong>it will not collect the process status (zombie) on the OS.</strong><br>
If using <code>Start()</code>, you have the responsibility to explicitly call <code>Wait()</code>, for example in a separate goroutine.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// ✅ Correct async execution</span>
<span class="k">func</span> <span class="n">spawnAsync</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">cmd</span> <span class="o">:=</span> <span class="n">exec</span><span class="o">.</span><span class="n">Command</span><span class="p">(</span><span class="s">"sleep"</span><span class="p">,</span> <span class="s">"1"</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">cmd</span><span class="o">.</span><span class="n">Start</span><span class="p">();</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span>
    <span class="p">}</span>

    <span class="c">// Wait for collection in a goroutine</span>
    <span class="k">go</span> <span class="k">func</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cmd</span><span class="o">.</span><span class="n">Wait</span><span class="p">()</span> <span class="c">// wait4 is called here, and the zombie rests in peace</span>
    <span class="p">}()</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  What is happening at the low layer?
</h3>

<p>In a Linux environment, <code>cmd.Wait()</code> ultimately calls <code>syscall.Wait4</code>. This receives the exit status from the kernel and stores it in the <code>ProcessState</code> struct.</p>

<p>While the Go runtime handles signals skillfully, for individual external command execution, the design requires developers to explicitly issue the <code>wait</code> system call via the <code>Wait</code> method.</p>




<h2>
  
  
  Conclusion
</h2>

<p>The existence of zombie processes is a specification, not a bug, but leaving them unattended is a "child neglect" bug.</p>

<p>In 2026 troubleshooting, it is required not only to use the <code>ps</code> command but also to monitor "who is skipping <code>wait</code>" or "which container is about to reach the PID limit" using <code>bpftrace</code>. Trace the process tree (<code>pstree -p</code>), identify the parent process failing to fulfill its responsibilities, and fix it.</p>

