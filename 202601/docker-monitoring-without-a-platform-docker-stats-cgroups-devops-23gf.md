---
Title: Docker Monitoring Without a Platform: docker stats + cgroups (DevOps)
Description: 
Author: Sajja Sudhakararao
Date: 2026-01-31T21:40:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>When an incident hits a containerized service, you often don’t need a full observability stack to get traction. You need fast answers: Which container is hot? What resource is saturating? Is it an app problem or a limit problem?</p>

<p>This guide shows a practical monitoring stack you can run from any Docker host:</p>

<ol>
<li>Docker-level commands (docker stats, docker inspect, docker logs)</li>
<li>Host Linux tools (ps/top/free/df/iostat/ss/journalctl)</li>
<li>Kernel primitives: cgroups (resource limits/accounting) and namespaces (isolation)</li>
</ol>




<h2>
  
  
  1) Start with docker stats (the fastest signal)
</h2>

<p><code>docker stats</code> streams runtime metrics for containers, including CPU%, memory usage/limit, network I/O, and block I/O.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker stats
</code></pre>

</div>



<p>Common workflows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker stats <span class="nt">--no-stream</span>          <span class="c"># Snapshot (good for scripts)</span>
docker stats tainer_name&gt;     <span class="c"># Focus on one container</span>
</code></pre>

</div>



<p>How to interpret it (in plain language)</p>

<ul>
<li>
<strong>CPU%:</strong> who’s burning compute right now.</li>
<li>
<strong>MEM USAGE / LIMIT:</strong> how close you are to the memory ceiling.</li>
<li>
<strong>NET I/O:</strong> traffic spikes, retries, or unusual egress.
​- <strong>BLOCK I/O:</strong> slow disks, chatty logging, or heavy read/write workloads.</li>
</ul>

<h2>
  
  
  2) Jump from “container name” → “what is it?”
</h2>

<p>Once you identify a hot container, immediately gather identity + configuration.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker ps
docker inspect tainer&gt; | less
</code></pre>

</div>



<p>Useful <code>inspect</code> questions:</p>

<ul>
<li>What image/tag is running?</li>
<li>What env vars/config are set?</li>
<li>What ports and volumes are attached?</li>
<li>Are there memory/CPU limits configured?</li>
</ul>

<p>3) Logs: confirm symptoms fast<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker logs <span class="nt">--tail</span> 200 tainer&gt;
docker logs <span class="nt">-f</span> tainer&gt;
</code></pre>

</div>



<p>This is often enough to spot:</p>

<ul>
<li>crash loops</li>
<li>OOM errors / memory pressure</li>
<li>upstream timeouts</li>
<li>DB connection exhaustion</li>
</ul>

<h2>
  
  
  4) Understand why it’s happening: cgroups + namespaces (the mental model)
</h2>

<p>Docker relies on Linux kernel features:</p>

<ul>
<li>Namespaces isolate views of processes, networking, mounts, etc.</li>
<li>cgroups control and account for resources like CPU, memory, and I/O.</li>
</ul>

<p>Why this matters during incidents:</p>

<ul>
<li>A container can be “slow” because it’s CPU-throttled, not because the app code suddenly got worse.</li>
<li>A container can restart because it hit its memory limit and the kernel’s OOM behavior targeted its processes.</li>
</ul>

<h2>
  
  
  5) Host-level confirmation (tie back to your Linux monitoring toolkit)
</h2>

<p>When docker stats shows a spike, verify on the host to avoid false conclusions.</p>

<h3>
  
  
  CPU hogs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ps aux <span class="nt">--sort</span><span class="o">=</span>-%cpu | <span class="nb">head</span> <span class="nt">-15</span>
</code></pre>

</div>



<h3>
  
  
  Memory pressure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>free <span class="nt">-h</span>
</code></pre>

</div>



<h3>
  
  
  Disk full / log explosions
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">df</span> <span class="nt">-h</span>
<span class="nb">du</span> <span class="nt">-sh</span> /var/lib/docker/<span class="k">*</span> 2&gt;/dev/null | <span class="nb">sort</span> <span class="nt">-h</span> | <span class="nb">tail</span> <span class="nt">-10</span>
</code></pre>

</div>



<h3>
  
  
  Disk I/O saturation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>iostat <span class="nt">-x</span> 1 3
</code></pre>

</div>



<h3>
  
  
  Unexpected listeners / traffic patterns
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ss <span class="nt">-tuln</span>
</code></pre>

</div>



<p>These host checks help you decide whether you’re dealing with a single container or a node-wide saturation problem.</p>

<h2>
  
  
  6) What to do with the data (action mapping)
</h2>

<p>Use the shortest safe path to stability:</p>

<h3>
  
  
  1. CPU high + latency rising
</h3>

<ul>
<li>If CPU is legitimately needed: scale out / add capacity.</li>
<li>If CPU is throttled: revisit limits/requests (or container CPU shares).</li>
</ul>

<h3>
  
  
  2. Memory near limit
</h3>

<ul>
<li>If memory leak suspected: restart as mitigation + open an issue with heap profiling.</li>
<li>If limit too low for normal peaks: adjust limit carefully and monitor.</li>
</ul>

<h3>
  
  
  3. Block I/O high
</h3>

<ul>
<li>Check log volume and disk saturation; reduce noisy logs or move logs off disk.</li>
<li>Consider storage performance constraints and workload patterns.</li>
</ul>

<h3>
  
  
  4. Network I/O abnormal
</h3>

<ul>
<li>Look for retries, timeouts, DDoS/abuse patterns, or upstream issues.</li>
</ul>

<h2>
  
  
  7) Copy/paste triage sequence (5 minutes)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1) Find the hot container</span>
docker stats <span class="nt">--no-stream</span>

<span class="c"># 2) Identify it</span>
docker ps
docker inspect tainer&gt; | less

<span class="c"># 3) Check symptoms</span>
docker logs <span class="nt">--tail</span> 200 tainer&gt;

<span class="c"># 4) Confirm on host (avoid guessing)</span>
ps aux <span class="nt">--sort</span><span class="o">=</span>-%cpu | <span class="nb">head</span> <span class="nt">-10</span>
free <span class="nt">-h</span>
<span class="nb">df</span> <span class="nt">-h</span>
iostat <span class="nt">-x</span> 1 3
ss <span class="nt">-tuln</span>
</code></pre>

</div>



<p>What’s your most common container failure mode: OOM kills, CPU throttling, disk I/O, or network timeouts?</p>

