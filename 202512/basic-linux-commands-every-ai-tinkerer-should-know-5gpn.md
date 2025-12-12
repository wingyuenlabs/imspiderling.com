---
Title: Basic Linux commands every AI tinkerer should know
Description: 
Author: crow
Date: 2025-12-12T21:34:39.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Basic Linux Commands Every AI Tinkerer Should Know
</h1>

<p>If you’re just starting a home‑lab or an AI workstation, the first hurdle is usually the terminal. Even if you’re comfortable with the command line, there are a handful of commands that will make your life a lot easier when you’re juggling data sets, training models, and managing containers. Below is a practical, beginner‑friendly guide that covers the essentials—complete with concrete examples and short snippets you can copy‑paste right away.</p>

<blockquote>
<p><strong>TL;DR</strong>  </p>

<ol>
<li>Navigation (<code>cd</code>, <code>ls</code>, <code>pwd</code>)
</li>
<li>File manipulation (<code>cp</code>, <code>mv</code>, <code>rm</code>, <code>touch</code>, <code>nano</code>)
</li>
<li>System info &amp; monitoring (<code>top</code>, <code>htop</code>, <code>free</code>, <code>df</code>, <code>du</code>, <code>ps</code>)
</li>
<li>Networking (<code>ping</code>, <code>curl</code>, <code>wget</code>, <code>ssh</code>)
</li>
<li>Package management (<code>apt</code>, <code>yum</code>, <code>dnf</code>, <code>pacman</code>, <code>pip</code>)
</li>
<li>Process &amp; job control (<code>nohup</code>, <code>screen</code>, <code>tmux</code>)
</li>
<li>Shell tricks (<code>grep</code>, <code>awk</code>, <code>sed</code>, <code>cut</code>, <code>sort</code>, <code>uniq</code>)
</li>
<li>Disk space &amp; permissions (<code>chmod</code>, <code>chown</code>, <code>umask</code>)
</li>
</ol>
</blockquote>




<h2>
  
  
  1️⃣ Navigation – Getting to the Right Folder
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>pwd</code></td>
<td>Print working directory</td>
<td>
<code>pwd</code> → <code>/home/alex/projects/ai</code>
</td>
</tr>
<tr>
<td><code>ls -la</code></td>
<td>List all files, including hidden ones, with details</td>
<td><code>ls -la /etc</code></td>
</tr>
<tr>
<td><code>cd ..</code></td>
<td>Move up one directory</td>
<td>
<code>cd ..</code> → <code>/home/alex/projects</code>
</td>
</tr>
<tr>
<td><code>cd ~</code></td>
<td>Go to your home folder</td>
<td><code>cd ~</code></td>
</tr>
<tr>
<td><code>cd -</code></td>
<td>Return to the previous directory</td>
<td><code>cd -</code></td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Tip:</strong><br><br>
Use tab‑completion to avoid typos. Type the first few letters of a file or folder and press <code>&lt;Tab&gt;</code>.</p>
</blockquote>




<h2>
  
  
  2️⃣ File Manipulation – Creating, Moving, Deleting
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>touch filename.txt</code></td>
<td>Create an empty file (or update timestamp)</td>
<td><code>touch notes.md</code></td>
</tr>
<tr>
<td><code>cp source dest</code></td>
<td>Copy files or directories</td>
<td><code>cp model.pt /mnt/ssd/models/</code></td>
</tr>
<tr>
<td><code>mv oldname newname</code></td>
<td>Move or rename</td>
<td><code>mv data.csv backup/data-$(date +%F).csv</code></td>
</tr>
<tr>
<td><code>rm file</code></td>
<td>Delete a single file</td>
<td><code>rm temp.log</code></td>
</tr>
<tr>
<td><code>rm -r dir</code></td>
<td>Recursively delete a directory</td>
<td><code>rm -rf build/</code></td>
</tr>
<tr>
<td><code>nano filename</code></td>
<td>Simple text editor in the terminal</td>
<td><code>nano README.md</code></td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Safety:</strong><br><br>
Use <code>-i</code> with <code>rm</code> to get a prompt: <code>rm -ri temp/</code>.</p>
</blockquote>




<h2>
  
  
  3️⃣ System Info &amp; Monitoring – Keep an Eye on Resources
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># CPU + Memory usage (top)</span>
top

<span class="c"># Interactive, easier interface (install first)</span>
<span class="nb">sudo </span>apt <span class="nb">install </span>htop
htop

<span class="c"># RAM usage summary</span>
free <span class="nt">-h</span>

<span class="c"># Disk free space</span>
<span class="nb">df</span> <span class="nt">-h</span>

<span class="c"># Disk usage of a directory</span>
<span class="nb">du</span> <span class="nt">-sh</span> ~/datasets/

<span class="c"># Find all running processes</span>
ps aux | <span class="nb">grep </span>python
</code></pre>

</div>



<blockquote>
<p><strong>Why it matters:</strong><br><br>
Training deep‑learning models can exhaust GPU, CPU or RAM. These commands let you spot bottlenecks before your job crashes.</p>
</blockquote>




<h2>
  
  
  4️⃣ Networking – Test Connectivity &amp; Download Data
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ping host</code></td>
<td>Check if a host is reachable</td>
<td><code>ping google.com -c 4</code></td>
</tr>
<tr>
<td><code>curl url</code></td>
<td>Fetch content (shows response header by default)</td>
<td><code>curl -I https://huggingface.co/bert-base-uncased</code></td>
</tr>
<tr>
<td><code>wget url</code></td>
<td>Download files directly</td>
<td><code>wget https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data</code></td>
</tr>
<tr>
<td><code>ssh user@host</code></td>
<td>Secure shell into another machine</td>
<td><code>ssh alex@192.168.1.10</code></td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Download large datasets</strong><br><br>
Use <code>aria2c -x 16 -s 16 url</code> for multi‑threaded downloads.</p>
</blockquote>




<h2>
  
  
  5️⃣ Package Management – Installing Software
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Distribution</th>
<th>Install command</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ubuntu/Debian</td>
<td><code>sudo apt install package</code></td>
<td><code>sudo apt install python3-pip git</code></td>
</tr>
<tr>
<td>CentOS/Fedora</td>
<td>
<code>sudo yum install package</code> (or <code>dnf</code>)</td>
<td><code>sudo dnf install gcc-c++</code></td>
</tr>
<tr>
<td>Arch Linux</td>
<td><code>sudo pacman -S package</code></td>
<td><code>sudo pacman -S htop</code></td>
</tr>
<tr>
<td>Python packages</td>
<td><code>pip install package</code></td>
<td><code>pip install torch torchvision</code></td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Virtual environments</strong><br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python3 <span class="nt">-m</span> venv ~/ai-env
<span class="nb">source</span> ~/ai-env/bin/activate
</code></pre>

</div>






<h2>
  
  
  6️⃣ Process &amp; Job Control – Keep Things Running
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>nohup command &amp;</code></td>
<td>Run a job that keeps running after logout</td>
<td><code>nohup python train.py &gt; log.txt 2&gt;&amp;1 &amp;</code></td>
</tr>
<tr>
<td><code>screen</code></td>
<td>Terminal multiplexer (install first)</td>
<td>
<code>screen -S ai_session</code> then <code>&lt;Ctrl-A d&gt;</code> to detach</td>
</tr>
<tr>
<td><code>tmux</code></td>
<td>Another terminal multiplexer, more modern</td>
<td><code>tmux new -s training</code></td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Tip:</strong><br><br>
In <code>tmux</code>, press <code>&lt;Ctrl-B c&gt;</code> to create a new window. Press <code>&lt;Ctrl-B w&gt;</code> to switch between windows.</p>
</blockquote>




<h2>
  
  
  7️⃣ Shell Tricks – Filtering &amp; Manipulating Text
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>grep pattern file</code></td>
<td>Search for text patterns</td>
<td><code>grep -i "error" logs/*.log</code></td>
</tr>
<tr>
<td><code>awk '{print $1}' file</code></td>
<td>Print the first column</td>
<td><code>awk '{print $1}' data.csv</code></td>
</tr>
<tr>
<td><code>sed 's/old/new/g' file</code></td>
<td>Replace text in place (use with <code>-i</code>)</td>
<td><code>sed -i 's/localhost/0.0.0.0/g' config.yaml</code></td>
</tr>
<tr>
<td><code>cut -d',' -f1,3 file</code></td>
<td>Cut specific fields by delimiter</td>
<td><code>cut -d',' -f2 data.csv</code></td>
</tr>
<tr>
<td>`sort file</td>
<td>uniq -c`</td>
<td>Count unique lines</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Pipeline example</strong> – Count how many times each model appears in a log:<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">grep</span> <span class="s2">"Model:"</span> train.log | <span class="nb">awk</span> <span class="s1">'{print $3}'</span> | <span class="nb">sort</span> | <span class="nb">uniq</span> <span class="nt">-c</span> | <span class="nb">sort</span> <span class="nt">-nr</span>
</code></pre>

</div>






<h2>
  
  
  8️⃣ Disk Space &amp; Permissions – Protect Your Data
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>chmod 755 file</code></td>
<td>Set read/write/execute permissions</td>
<td><code>chmod 700 ~/.ssh/id_rsa</code></td>
</tr>
<tr>
<td><code>chown user:group file</code></td>
<td>Change ownership</td>
<td><code>sudo chown alex:alex data/</code></td>
</tr>
<tr>
<td><code>umask 022</code></td>
<td>Default permission mask for new files</td>
<td>
<code>umask 0022</code> (makes new files readable by group)</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>Secure your SSH key</strong><br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">chmod </span>600 ~/.ssh/id_rsa
</code></pre>

</div>






<h2>
  
  
  Putting It All Together – A Mini Workflow
</h2>

<p>Let’s walk through a typical AI tinkering routine:</p>

<ol>
<li>
<strong>Clone the repo</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   git clone https://github.com/your-org/ai-project.git
   <span class="nb">cd </span>ai-project
</code></pre>

</div>



<ol>
<li>
<strong>Set up a virtual environment</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   python3 <span class="nt">-m</span> venv .venv
   <span class="nb">source</span> .venv/bin/activate
   pip <span class="nb">install</span> <span class="nt">-r</span> requirements.txt
</code></pre>

</div>



<ol>
<li>
<strong>Download the dataset (parallel, if large)</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   aria2c <span class="nt">-x</span> 16 <span class="nt">-s</span> 16 https://example.com/dataset.zip
   unzip dataset.zip <span class="nt">-d</span> data/
</code></pre>

</div>



<ol>
<li>
<strong>Run training in a detached session</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   screen <span class="nt">-S</span> train_job
   python train.py <span class="nt">--epochs</span> 10 <span class="o">&gt;</span> train.log 2&gt;&amp;1
   <span class="c"># Press Ctrl-A then D to detach</span>
</code></pre>

</div>



<ol>
<li>
<strong>Monitor GPU usage (requires NVIDIA tools)</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   watch <span class="nt">-n</span> 1 nvidia-smi
</code></pre>

</div>



<ol>
<li>
<strong>After training, check the results</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">tail</span> <span class="nt">-n</span> 20 train.log | <span class="nb">grep</span> <span class="s2">"Accuracy"</span>
</code></pre>

</div>



<ol>
<li>
<strong>Clean up old checkpoints</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   find checkpoints/ <span class="nt">-type</span> f <span class="nt">-mtime</span> +30 <span class="nt">-delete</span>
</code></pre>

</div>






<h2>
  
  
  Final Thoughts
</h2>

<p>Mastering these commands will let you:</p>

<ul>
<li>Navigate your filesystem quickly and safely.</li>
<li>Manipulate files and directories without leaving the terminal.</li>
<li>Keep tabs on resource usage, preventing costly crashes.</li>
<li>Install software efficiently across distributions.</li>
<li>Run long‑running AI jobs that survive disconnections.</li>
<li>Filter logs and data with powerful text tools.</li>
</ul>

<p>Don’t worry if you can’t remember every flag right away. The key is to practice—copy a command, run it, tweak the options, and see what happens. Over time, these commands will become second nature, letting you focus on what really matters: building awesome AI models in your home lab. Happy tinkering! 🚀</p>

