---
Title: 💀 I built a full system monitor in Bash — and fought awk along the way
Description: 
Author: DanLin
Date: 2025-11-04T21:16:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>I wanted to see how far I could push pure Bash before it collapses under its own syntax.<br>
So, naturally, I decided to write a system monitoring tool — in Bash.<br>
And thus, system-monitor was born:<br>
👉 <a href="https://github.com/DanLinX2004X/system-monitor" rel="noopener noreferrer">GitHub repo</a>  </p>

<p>Yes, it’s fully functional. Yes, it uses colors. Yes, awk is involved.<br><br>
No, I don’t recommend doing this sober. 😅  </p>


<h2>
  
  
  💡 The idea
</h2>

<p>I wanted a single script that could show:  </p>

<ul>
<li>CPU load and number of cores
</li>
<li>RAM usage and percentage
</li>
<li>Disk space for /
</li>
<li>Network I/O
</li>
<li>Process count
Basically, the “lazy Linux admin toolkit” in one file.
Something like this:
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./system-monitor.sh
</code></pre>

</div>


<p>and boom — everything you’d usually get from top, free, df, and ip combined.</p>



<p>⚙️ The features</p>

<p>What started as a 10-line script turned into a 250+ line CLI tool with:</p>

<ul>
<li>Command-line arguments (--help, --brief, --no-color, --version, -i N)</li>
<li>Colorized output for warnings/critical thresholds</li>
<li>Continuous monitoring mode</li>
<li>Safe shutdown with signal trapping (Ctrl+C)</li>
<li>Brief (machine-readable) mode for piping into other scripts</li>
<li>Dependency checks and graceful error handling</li>
</ul>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./system-monitor.sh <span class="nt">-i</span> 5 <span class="nt">--no-color</span>
</code></pre>

</div>



<blockquote>
<p>Updates every 5 seconds without colors.<br>
Press Ctrl+C to stop — yes, it even says goodbye politely.</p>
</blockquote>




<p>🧩 Some code highlights</p>

<p>Color management:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
NC='\033[0m'
</code></pre>

</div>



<p>Because monitoring your system is serious business — but who doesn’t love a bit of RGB?</p>

<p>Load detection magic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">CPU_LOAD_1MIN</span><span class="o">=</span><span class="si">$(</span><span class="nb">cat</span> /proc/loadavg | <span class="nb">awk</span> <span class="s1">'{print $1}'</span><span class="si">)</span>
<span class="nv">CPU_CORES</span><span class="o">=</span><span class="si">$(</span><span class="nb">nproc</span><span class="si">)</span>
<span class="nv">load_percent</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"scale=0; (</span><span class="nv">$CPU_LOAD_1MIN</span><span class="s2"> * 100) / </span><span class="nv">$CPU_CORES</span><span class="s2">"</span> | bc<span class="si">)</span>
</code></pre>

</div>



<p>That moment when you realize you’re using awk, bc, and /proc in one line and it actually works.</p>

<p>Network metrics:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">interface</span><span class="o">=</span><span class="si">$(</span>ip route get 8.8.8.8 | <span class="nb">awk</span> <span class="s1">'{print $5}'</span> | <span class="nb">head</span> <span class="nt">-1</span><span class="si">)</span>
<span class="nv">rx</span><span class="o">=</span><span class="si">$(</span><span class="nb">cat</span> /sys/class/net/<span class="nv">$interface</span>/statistics/rx_bytes<span class="si">)</span>
<span class="nv">tx</span><span class="o">=</span><span class="si">$(</span><span class="nb">cat</span> /sys/class/net/<span class="nv">$interface</span>/statistics/tx_bytes<span class="si">)</span>
</code></pre>

</div>



<p>I like to imagine Bash crying softly every time it runs this.</p>




<p>🧠 What I learned</p>

<ul>
<li>Bash can do a lot — if you’re patient (and slightly masochistic).</li>
<li>Quoting is a survival skill. Forget one and you’re debugging for hours.</li>
<li>
<code>awk</code> is an ancient curse. It always works, but never for the reason you expect.</li>
<li>Color helps debugging. Visual feedback saves your sanity.</li>
<li>Document your code. Because 3 a.m. you will have no clue what that <code>$7</code> in <code>awk '{print $7}'</code> was.</li>
</ul>




<p>🧰 The brief mode</p>

<p>I added a <code>--brief</code> flag that outputs all metrics in a machine-readable format — perfect for logging or Prometheus-style integration.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./system-monitor.sh <span class="nt">--brief</span>
</code></pre>

</div>



<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>timestamp=1730765432
cpu_load_percent=12
mem_usage_percent=43.5
disk_usage_percent=56
network_rx_mb=12.4
process_count=187
</code></pre>

</div>



<p>Basically, JSON for people who hate themselves.</p>




<p>🧯 Safety first: signal handling</p>

<p>If you hit Ctrl+C, it doesn’t just die — it politely says goodbye:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">trap </span>cleanup INT TERM

cleanup<span class="o">()</span> <span class="o">{</span>
    <span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">Monitoring stopped. Goodbye!"</span>
    <span class="nb">exit </span>0
<span class="o">}</span>
</code></pre>

</div>



<p>Because professionalism.</p>




<p>🚀 AUR packaging</p>

<p>After testing it on Arch, I thought — why not go all in?<br>
So I wrote a PKGBUILD, threw it into the AUR, and now you can literally install it with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>yay <span class="nt">-S</span> system-monitor
</code></pre>

</div>



<p>And that’s the moment I realized:</p>

<blockquote>
<p>“Oh no. I’ve become that guy who writes Bash utilities for other people.”</p>
</blockquote>




<p>🧭 Final thoughts</p>

<p>Was this efficient? Probably not.<br>
Did I learn a lot? Absolutely.</p>

<p>Here’s what I got from it:</p>

<ul>
<li>Deep understanding of Bash syntax and traps</li>
<li>Practical use of /proc and awk</li>
<li>Appreciation for proper CLI design</li>
<li>And a newfound respect for people who don’t do this in Bash</li>
</ul>

<p>If you’re thinking of writing your first utility — do it.<br>
Even if it’s messy. Even if awk haunts your dreams.</p>

<p>Because once you see your tool working, it’s worth every headache.</p>




<p>🔗 Links</p>

<p>🐧 GitHub: <a href="https://github.com/DanLinX2004X/system-monitor" rel="noopener noreferrer">https://github.com/DanLinX2004X/system-monitor</a></p>

<p>📦 AUR: <a href="https://aur.archlinux.org/packages/system-monitor" rel="noopener noreferrer">https://aur.archlinux.org/packages/system-monitor</a></p>

